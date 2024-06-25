<!--yml

类别：未分类

日期：2024-05-27 14:38:05

-->

# "按位传输"蓝牙低功耗 - Dmitry.GR

> 来源：[`dmitry.gr/?r=05.Projects&proj=11.%20Bluetooth%20LE%20fakery`](https://dmitry.gr/?r=05.Projects&proj=11.%20Bluetooth%20LE%20fakery)

# 模拟蓝牙低功耗

**蓝牙低功耗**是一种新技术，是在蓝牙 4.0 规范中引入的。 它与蓝牙除了名字之外*绝对没有任何关系*。 现在我们已经解释清楚了，为什么它很酷？ 嗯，它是为低功耗而设计的，这一点从设计上可以看出。 与真正的蓝牙不同，它会根据严格的时间表进行频率跳跃，而 LE 则是在发送一定数量的数据包后进行跳跃，因此不需要保持清醒来保持一个运行时钟以知道下一步跳到哪里。 事实上，LE 允许设备在保持连接的同时完全关闭其无线电长达一段时间。 这使得它非常适用于键盘、鼠标和各种其他设备。 在 LE 中的另一个很酷的功能是，设备可以发送小块数据的未请求广播。 与真正的蓝牙不同，在 LE 世界中扫描设备可以是被动的 - 你只需在正确的频道上监听广告数据包，你就能听到所有的广告。

**LE 的简单信道跳频行为**意味着我们可能可以假装成 LE 设备，而无需正常蓝牙所需的复杂无线电。 频率为 2.4GHz，通道间隔为 1MHz，调制为 GFSK，数据速率为 1MBps，前导码为 10101010 或 01010101，取决于第一个数据字节，寻址使用 32 位地址。 哎呀，我们难道不知道一个可以做到这一切的设备吗？ 当然知道！ 那就是广受欢迎的[Nordic nRF24L01+](http://www.semiconductorstore.com/cart/pc/viewPrd.asp?idproduct=41939)。 那么让我们看看 LE 需要什么以及我们所拥有的有什么区别：

+   **CRC：**BTLE 使用的是 24 位 CRC，这是 nRF24 无法做到的。 幸运的是，对于我们来说，nRF24 允许我们禁用 CRC，然后我们可以自己发送并手动检查接收到的 CRC。 不愉快？ 当然，但可行。

+   **频率跳跃：**BTLE 需要我们能够快速跳跃 - 150 微秒或更快。 遗憾的是，nRF24 在这里让我们失望。 每次传输后，它都会关闭其 PLL，然后在每个传输或接收命令上重新启动它。 这需要 130 微秒。 遗憾的是，我们在这里几乎没有时间做任何事情。 幸运的是，这并不是主要问题。

+   **数据包长度：**BTLE 数据包的大小各不相同，实际上可达 39 字节的有效负载。 但我们真正想要支持的一个数据包是 CONNECT_REQ 数据包，实际上它有 34 字节的有效负载+3 字节的 CRC，这意味着我们需要接收 37 字节，如果要检查 CRC，即使我们愿意不顾一切地忽略它们，我们也需要接收 34 字节。 nRF24 将无法处理超过 32 字节的数据包。 遗憾的是，最后两个字节相当重要（跳频大小和频道映射的一部分）。 这就是我们失败的地方。

**所以...我们不能接受连接**，但并非一切都完了... BTLE 允许未经请求的广告，所以我们仍然可以通过让它们向愿意倾听的任何人广播数据来做一些酷事情。让我们计算一下我们可以发送多少数据... 在我们的 32 字节预算中：3 用于 CRC，2 用于 ADV_NONCONN_IND 数据包标头，6 用于 MAC 地址，剩下 21 字节的负载。然而，这个负载必须具有结构。假设最小的必需标头，如果我们不想让设备广播名称，我们可以发送 19 字节的数据。如果我们想要一个名称，我们有 17 个字节可以在名称（以 UTF-8 编码）和我们的数据之间分配。如果我们想更好地符合规范并广播设备属性，我们将有 14 个字节可以在名称和数据之间分配，或者 16 个字节的纯数据 - 不太糟糕。

**然后让我们梳理一下细节**...首先，BTLE 和 nRF24 在空中发送数据位的顺序相反，因此我们将不得不反转所有位。其次，BTLE 使用数据白化，而 nRF24 不使用，所以我们也需要手动处理。最后，还有前面提到的 24 位 CRC。所有 LE 广播都被发送到同一个"地址"：0x8E89BED6，也被称为"bed six"。当然，对于我们来说，它将被位反转。BTLE 将 CRC 应用于整个负载，但不应用于地址。白化应用于负载和 CRC。了解这一点，因此，我们知道了组装完整工作数据包所需的事件顺序。广告数据包被发送到 3 个通道：37、38 和 39，它们分别是 2.402HGz、2.426GHz 和 2.480GHz。我们将在它们之间交替，有条不紊地将我们的广播涌入到各个地方。

**BTLE CRC** 在 C 中实现起来并不太困难，使用初始值为 0x555555，并且大致如下所示：

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

**数据白化函数**也不是太复杂。它是一个 7 位线性移位反馈样式，并通过等于 (channelNum << 1) + 1 的值进行初始化。代码大致如下：

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

**广告数据包负载**看起来是这样的：

```
struct adv_hdr{
	uint8_t header; //we use 0x40 to say it is a non-connectable undirected
			//advertisement and address we're sending is random (not assigned)
	uint8_t dataLen; //length of following data (including MAC addr)
	uint8_t MAC[6]; //the mac address
}
```

**所以如果我们把这些放在一起**，我们将得到一个具有上述标头的数据包。然后我们使用上述 CRC 函数对其进行 CRC 计算。然后我们使用上述白化函数对其进行白化。之后我们发送它。让我们看看... 是的，它起作用了。运行 BTLExplorer 的 iPad3 显示我们的设备可见且是 BTLE 设备。酷！一个注意事项：如果你不广播设备名称，BTLExplorer 将崩溃 - 这是他们的 bug，所以不用担心。

**负载数据格式是什么？**你可能会问。好吧，数据由块组成，每个块都有一个 2 字节的标头：长度和类型（按顺序，长度包括类型字节的长度）。你关心的类型是：

+   **2：** 标志 - 长度为 2，唯一字节的值你想要的是 5 - 这意味着单模式（非 BTLE/BT 组合）设备处于有限发现模式。

+   **8 或 9：** 名称 - 长度是名称的 UTF-8 编码长度，不包括 NULL 结尾。8 用于"缩写名称"，9 用于"完整名称"。

+   **255：** 自定义数据。这是您可以放入自定义数据的地方。在 iPhone 上，您将完全访问此数据。

**为什么这样做？** 嗯，如果您知道一种更简单的方式让嵌入式项目向 iPhone 通信，请告诉我。WiFi 耗电量大且凌乱。真正的蓝牙在 iPhone 上被封锁，串口也是如此。这种方法行得通。如果 Android 有了 BTLE API，我相信广播数据也将对您可用-这只是有道理的。

**未来工作：** 我刚收到了一些支持最大传输单元为 64 字节且可以保持其 PPL（可能传输延迟）的非北欧 2.4GHz 部件，这意味着可能可以在它们上实现完整的 BTLE 堆栈。我正在阅读此内容时致力于此。

所有在此处发布的代码以及研究都受到此许可证的约束：您可以按照任何您喜欢的方式使用它，但仅限于非商业目的，您必须同时提供指向此页面的链接。任何商业用途必须与我讨论。

**完整样例代码**，将在 Sparkfun 的 nordic-fob 上运行，看起来**大致**像这样：

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
