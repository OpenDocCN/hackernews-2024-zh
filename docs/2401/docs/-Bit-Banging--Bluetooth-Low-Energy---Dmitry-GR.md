<!--yml
category: 未分类
date: 2024-05-27 14:38:05
-->

# "Bit-Banging" Bluetooth Low Energy - Dmitry.GR

> 来源：[https://dmitry.gr/?r=05.Projects&proj=11.%20Bluetooth%20LE%20fakery](https://dmitry.gr/?r=05.Projects&proj=11.%20Bluetooth%20LE%20fakery)

# Faking Bluetooth LE

**Bluetooth LE** is a new technology, introduced in the Bluetooth 4.0 spec. It has *absolutely nothing* to do with bluetooth besides the name. Now that we have that out of the way, why is it cool? Well, it was made for low power, and the design shows. Unlike real bluetooth that does frequency hoppping on a precise schedule, regardless of anything, LE hops after some number of packets are sent, and thus one does not need to be awake to keep a running clock to know where to hop to next. In fact, LE allows a device to completely shut down its radio for large periods of time while maintaining a connection. This makes it awesome for keyboards and mice and all kinds of other such things. Another cool feature in LE is that devices can send unsolicited broadcasts of small chunks of data. Unlike real bluetooth, scanning for devices in the LE world can be passive - you just listen for advertisement packets on the right channels and you hear all the advertisements.

**The simple channel hopping behaviour** of LE means that we can probably pretend to be an LE device without the complex radio that normal bluetoth requires. The frequency is 2.4GHz, chanels are 1MHz apart, modulation is GFSK, datarate is 1MBps, preamble is 10101010 or 01010101 based on first data byte, and addressing is done using a 32-bit address. Gee, don't we know a device that can do all that? Of course we do! The ever-popular [Nordic nRF24L01+](http://www.semiconductorstore.com/cart/pc/viewPrd.asp?idproduct=41939). So let's look at the differences between what LE needs and what we have:

*   **CRC:** BTLE uses a 24-bit CRC that is not something nRF24 can do. Luckily for us, nRF24 allows us to disable the CRC, and then we can send our own and check received ones manually. Unpleasant? Sure, but doable.
*   **Frequency Hopping:** BTLE needs ut to be able to hop quickly - 150us or faster. Sadly here nRF24 fails us. After every transmission it shuts down its PLL, and then restarts it on every transmit or receive command. This takes 130us. We are left with too little time to do anything here, sadly. Luckily this is not the major issue.
*   **Packet Length:** BTLE packets are of various sizes, up to 39 bytes of payload in fact. But the one packet we would really want to support is the CONNECT_REQ packet, which is in fact 34 bytes of payload + 3 of CRC, meaning we'd need to receive 37 bytes if we want to check CRCs, and even if we were willing to throw caution to the wind and ignore them, we need to receive 34\. nRF24 will not handle packets over 32 bytes. Sadly the last two bytes are kind of important (hop size and a part of the channel map). This is where we lose.

**So... we cannot accept connections,** but not all is lost... BTLE allows unsolicited advertisements, so we can still do some cool things by making them broadcast data to anyone who'll listen. Let's work out just how much data we can send... Out of our 32-byte budget: 3 go into CRC, 2 go into the ADV_NONCONN_IND packet header, and 6 go into the MAC address, leaving us with 21 bytes of payload. This payload, however, must have structure. Assuming minimum required headers, we can get away with sending 19 bytes of data if we do not want our device to broadcast a name. If we do want a name, we have 17 bytes to split between the name (in UTF-8) and our data. And if we want to comply with the spec better and broadcast device attributes, we'll have 14 bytes to split between name and data or 16 bytes of pure data - not too bad.

**Let's sort out the details** then... first of all, BTLE and nRF24 send data bits in the air in opposite order, so we'll have to reverse all our bits. Second, BTLE uses data whitening, and nRF24 does not, so we'll need to do that by hand too. Lastly, there is the previously-mentioned 24-bit CRC. All LE broadcasts get sent to the same "address": 0x8E89BED6, also known as "bed six." Of course, for us it'll be bit-reversed. BTLE applies CRC to the whole payload but not the address. Whitening is applied to the payload and the CRC. Knowing this, thus, gives us the ordering of events needed to assemble a complete working packet. Advertisement packets are sent on 3 channels: 37, 38, and 39, which are 2.402HGz, 2.426GHz, and 2.480GHz respectively. We'll alternate between them, spewing our broadcast everywhere methodically.

**BTLE CRC** is not too hard to implement in C, uses the initial value of 0x555555, and looks something like this:

```
void btLeCrc(const uint8_t* data, uint8_t len, uint8_t* dst){

	uint8_t v, t, d;

	while(len--){
		d = *data++;
		for(v = 0; v < 8; v++, d >>= 1){

			t = dst[0] >> 7;
			dst[0] <<= 1;
			if(dst[1] & 0x80) dst[0] |= 1;
			dst[1] <<= 1;
			if(dst[2] & 0x80) dst[1] |= 1;
			dst[2] <<= 1;

			if(t != (d & 1)){

				dst[2] ^= 0x5B;
				dst[1] ^= 0x06;
			}
		}	
	}
}
```

**The data whitening function** is also not too complicated. It is a 7-bit linear-shift feedback style and is initialized by the value that is equal to (channelNum << 1) + 1\. The code goes something like this:

```
void btLeWhiten(uint8_t* data, uint8_t len, uint8_t whitenCoeff){
	uint8_t  m;

	while(len--){

		for(m = 1; m; m <<= 1){

			if(whitenCoeff & 0x80){

				whitenCoeff ^= 0x11;
				(*data) ^= m;
			}
			whitenCoeff <<= 1;
		}
		data++;
	}
}
```

**The advertisement packet payload** looks like this:

```
struct adv_hdr{
	uint8_t header; //we use 0x40 to say it is a non-connectable undirected
			//advertisement and address we're sending is random (not assigned)
	uint8_t dataLen; //length of following data (including MAC addr)
	uint8_t MAC[6]; //the mac address
}
```

**So if we lump all this together,** we'll end up with a packet with the above header. We then CRC it using the above CRC function. We then whiten it using the above whitening function. After this we send it. Let's see... yup, it works. An iPad3 running BTLExplorer shows that our device is visible and is a BTLE device. Cool! One caveat: if you do not broadcast a device name, BTLExplorer will crash - this is their bug, so do not worry.

**What is the payload data format?** you may ask. Well, data is made of chunks, each of which has a 2-byte header: length and type (in that order, length includes the length of the type byte). Types you care about are:

*   **2:** Flags - length will be 2 and the lone byte's value you want is 5 - this means a single-mode (not BTLE/BT combo) device in limited discovery mode.
*   **8 or 9:** Name - length is the length of the name in UTF-8 without NULL-termination. 8 is for "shortened name" and 9 is for "complete name."
*   **255:** Custom data. This is where you can shove your custom data. On iPhone you'll have complete access to this.

**Why all this?** Well, if you know a simpler way for an embedded project to communicate data to an iPhone, let me know. WiFi is power hungry and messy. Real bluetooth is locked down in iPhone, as are serial ports. This method works. If and when Android gets a BTLE API, I am sure broadcast data will be available to you too - it just makes sense.

**Future work:** I just got some non-nordic 2.4GHz parts that support packets up to 64 bytes and can keep their PPLs on, meaning that a full BTLE stack may be possible on them. I am working on this as you read this.

All the code as well as the research that went into this and is published here is under this license: you may use it in any way you please if and only if it is for non-commercial purposes, you must provide a link to this page as well. Any commercial use must be discussed with me.

**Full sample code**, that will run on the nordic-fob from Sparkfun looks **approximately** like this:

```
#include <stdio.h>
#include <avr/io.h>
#include <avr/interrupt.h>
#include <avr/sleep.h>
#define F_CPU	8000000
#include <avr/delay.h>

#define PIN_CE	1 //Output
#define PIN_nCS	2 //Output

#define MY_MAC_0	0xEF
#define MY_MAC_1	0xFF
#define MY_MAC_2	0xC0
#define MY_MAC_3	0xAA
#define MY_MAC_4	0x18
#define MY_MAC_5	0x00

ISR(PCINT0_vect)
{
	//useless
}

void btLeCrc(const uint8_t* data, uint8_t len, uint8_t* dst){

	uint8_t v, t, d;

	while(len--){

		d = *data++;
		for(v = 0; v < 8; v++, d >>= 1){

			t = dst[0] >> 7;

			dst[0] <<= 1;
			if(dst[1] & 0x80) dst[0] |= 1;
			dst[1] <<= 1;
			if(dst[2] & 0x80) dst[1] |= 1;
			dst[2] <<= 1;

			if(t != (d & 1)){

				dst[2] ^= 0x5B;
				dst[1] ^= 0x06;
			}
		}	
	}
}

uint8_t  swapbits(uint8_t a){

	uint8 v = 0;

	if(a & 0x80) v |= 0x01;
	if(a & 0x40) v |= 0x02;
	if(a & 0x20) v |= 0x04;
	if(a & 0x10) v |= 0x08;
	if(a & 0x08) v |= 0x10;
	if(a & 0x04) v |= 0x20;
	if(a & 0x02) v |= 0x40;
	if(a & 0x01) v |= 0x80;

	return v;
}

void btLeWhiten(uint8_t* data, uint8_t len, uint8_t whitenCoeff){

	uint8_t  m;

	while(len--){

		for(m = 1; m; m <<= 1){

			if(whitenCoeff & 0x80){

				whitenCoeff ^= 0x11;
				(*data) ^= m;
			}
			whitenCoeff <<= 1;
		}
		data++;
	}
}

static inline uint8_t btLeWhitenStart(uint8_t chan){
	//the value we actually use is what BT'd use left shifted one...makes our life easier

	return swapbits(chan) | 2;	
}

void btLePacketEncode(uint8_t* packet, uint8_t len, uint8_t chan){
	//length is of packet, including crc. pre-populate crc in packet with initial crc value!

	uint8_t i, dataLen = len - 3;

	btLeCrc(packet, dataLen, packet + dataLen);
	for(i = 0; i < 3; i++, dataLen++) packet[dataLen] = swapbits(packet[dataLen]);
	btLeWhiten(packet, len, btLeWhitenStart(chan));
	for(i = 0; i < len; i++) packet[i] = swapbits(packet[i]);

}

uint8_t spi_byte(uint8_t byte){

	uint8_t i = 0;

	do{
		PORTB &=~ (uint8_t)(1 << 6);
		if(byte & 0x80) PORTB |= (uint8_t)(1 << 6);
		CLK |= (uint8_t)(1 << 4);
		byte <<= 1;
		if(PINA & (uint8_t)32) byte++;
		CLK &=~ (uint8_t)(1 << 4);

	}while(--i);

	return byte;
}

void nrf_cmd(uint8_t cmd, uint8_t data)
{
	cbi(PORTB, PIN_nCS);
	spi_byte(cmd);
	spi_byte(data);
	sbi(PORTB, PIN_nCS); //Deselect chip
}

void nrf_simplebyte(uint8_t cmd)
{
	cbi(PORTB, PIN_nCS);
	spi_byte(cmd);
	sbi(PORTB, PIN_nCS);
}

void nrf_manybytes(uint8_t* data, uint8_t len){

	cbi(PORTB, PIN_nCS);
	do{

		spi_byte(*data++);

	}while(--len);
	sbi(PORTB, PIN_nCS);
}

void fob_init (void)
{
	DDRA = (uint8_t)~(1<<5);
	DDRB = 0b00000110;
	PORTA = 0b10001111;
	cbi(PORTB, PIN_CE);
	TCCR0B = (1<<CS00);
	MCUCR = (1<<SM1)|(1<<SE);
	sei();
}

int main (void)
{
	static const uint8_t chRf[] = {2, 26,80};
	static const uint8_t chLe[] = {37,38,39};
	uint8_t i, L, ch = 0;
	uint8_t buf[32];

	fob_init();

	DDRA |= 4;
	PORTA |= 4;

	nrf_cmd(0x20, 0x12);	//on, no crc, int on RX/TX done
	nrf_cmd(0x21, 0x00);	//no auto-acknowledge
	nrf_cmd(0x22, 0x00);	//no RX
	nrf_cmd(0x23, 0x02);	//5-byte address
	nrf_cmd(0x24, 0x00);	//no auto-retransmit
	nrf_cmd(0x26, 0x06);	//1MBps at 0dBm
	nrf_cmd(0x27, 0x3E);	//clear various flags
	nrf_cmd(0x3C, 0x00);	//no dynamic payloads
	nrf_cmd(0x3D, 0x00);	//no features
	nrf_cmd(0x31, 32);	//always RX 32 bytes
	nrf_cmd(0x22, 0x01);	//RX on pipe 0

	buf[0] = 0x30;			//set addresses
	buf[1] = swapbits(0x8E);
	buf[2] = swapbits(0x89);
	buf[3] = swapbits(0xBE);
	buf[4] = swapbits(0xD6);
	nrf_manybytes(buf, 5);
	buf[0] = 0x2A;
	nrf_manybytes(buf, 5);

	while(1){

		L = 0;

		buf[L++] = 0x40;	//PDU type, given address is random
Xbuf[L++] = 11;X//17 bytes of payload

		buf[L++] = MY_MAC_0;
		buf[L++] = MY_MAC_1;
		buf[L++] = MY_MAC_2;
		buf[L++] = MY_MAC_3;
		buf[L++] = MY_MAC_4;
		buf[L++] = MY_MAC_5;

		buf[L++] = 2;		//flags (LE-only, limited discovery mode)
		buf[L++] = 0x01;
		buf[L++] = 0x05;

		buf[L++] = 7;
		buf[L++] = 0x08;
		buf[L++] = 'n';
		buf[L++] = 'R';
		buf[L++] = 'F';
		buf[L++] = ' ';
		buf[L++] = 'L';
		buf[L++] = 'E';

		buf[L++] = 0x55;	//CRC start value: 0x555555
		buf[L++] = 0x55;
		buf[L++] = 0x55;

		if(++ch == sizeof(chRf)) ch = 0;

		nrf_cmd(0x25, chRf[ch]);
		nrf_cmd(0x27, 0x6E);	//clear flags

		btLePacketEncode(buf, L, chLe[ch]);

		nrf_simplebyte(0xE2); //Clear RX Fifo
		nrf_simplebyte(0xE1); //Clear TX Fifo

		cbi(PORTB, PIN_nCS);
		spi_byte(0xA0);
		for(i = 0 ; i < L ; i++) spi_byte(buf[i]);
		sbi(PORTB, PIN_nCS);

		nrf_cmd(0x20, 0x12);	//tx on
		sbi(PORTB, PIN_CE);	 //do tx
		delay_ms(10);
		cbi(PORTB, PIN_CE);	(in preparation of switching to RX quickly)
	}

	return 0;
}

```