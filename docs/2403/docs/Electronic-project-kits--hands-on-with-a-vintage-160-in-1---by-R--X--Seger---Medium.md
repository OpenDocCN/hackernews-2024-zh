<!--yml
category: 未分类
date: 2024-05-29 12:39:10
-->

# Electronic project kits: hands on with a vintage 160-in-1 | by R. X. Seger | Medium

> 来源：[https://medium.com/@rxseger/electronic-project-kits-hands-on-with-a-vintage-160-in-1-eea39e6193f4](https://medium.com/@rxseger/electronic-project-kits-hands-on-with-a-vintage-160-in-1-eea39e6193f4)

This particular IC is not too interesting in modern times, it is fairly obscure, and many other amplifiers are available instead. Can the IC in these circuits replaced with opamps in a [closed loop](https://en.wikipedia.org/wiki/Operational_amplifier#Closed_loop) configuration?

With a non-inverting opamp: Vout = Vin*(1 + Rf/Rg).

Opamp chips: [741](https://en.wikipedia.org/wiki/Operational_amplifier#Internal_circuitry_of_741-type_op-amp) (popular but there is newer), LM324N quad operational amplifier (which I had [salvaged from an UPS](/@rxseger/building-an-h-bridge-from-a-salvaged-uninterruptible-power-supply-ab1576ec313f#.7tcvsnkwy)).

TODO: test LM324N replacement in these circuits

## Knobs: Control and Tuning

“Control” is a potentiometer conveniently located near the speaker, useful for volume or frequency adjustments. Specified as a 50 kΩ variable resistor in the parts list, I measured 43.38 kΩ across the two resistor terminals. Rough measurements adjusting the wiper to each digit:

*   0: 45 Ω
*   1: 160 Ω
*   2: 5.4 kΩ
*   3: 10 kΩ
*   4: 17 kΩ
*   5: 24 kΩ
*   6: 31 kΩ
*   7: 36 kΩ
*   8: 40.04 kΩ
*   9: 43.55 kΩ
*   10: 43.39 kΩ

Curious how the resistance barely varies from 9 to 10, maybe by design or an artifact of old age.

“Tuning” is an unlabeled variable capacitor in radio circuits:

specified as 265 pF in the parts list, I measured… 0–1 nF, and ∞ Ω. Broken?

This may be a possible replacement: [Fudan brand single joint air medium variable capacitor 12–365PF](https://www.aliexpress.com/item/Fudan-brand-single-joint-air-medium-variable-capacitor-12-365PF/32406520331.html?ws_ab_test=searchweb0_0%2Csearchweb201602_1_116_10065_117_10068_114_115_10069_113_10084_10083_10017_10080_10082_10081_10060_10061_10062_10056_10055_10054_10059_10078_10079_10073_10070_421_420_10052_10053_10050_10051%2Csearchweb201603_3&btsid=6c95a6db-565e-4c59-8b5b-97eea3f717ea). TODO: test a circuit using the variable cap

## Speakers

The speaker is one of the most important components of this kit. It provides an essential output: auditory feedback to the listener who built the circuit. I still think of low-capacity ceramic capacitors as “high-pitch”, due to the tone heard from this speaker when used in an oscillator.

It is always connected to the output transformer (except in “103\. Relay and Speaker Buzzer” where the relay is used to generate an AC waveform instead), although the output transformer is sometimes used by itself.

A good opportunity to test some old speakers I had around, using the same machine gun oscillator circuit:

The speaker is only specified as “57 mm S-4565” in the parts list, but I measured 8.2 Ω, which looks about right. The three extra speakers shown above were all labeled 8 Ω, and worked just fine in this circuit.

There’s also a set of earphones, listed as “Earphone, high Z crystal type (no DC path) E-0007”, could be useful sadly no longer present in this kit I own.

TODO: can one use cellphone/computer headphones as an alternative?

## Transformers

Two small transformers:

*   Input transformer (yellow), TD-0097 / 11106069
*   Output transformer (red), TD-0136 / 11106077

These part numbers appear to be kit-specific, couldn’t find much information. However, [Wikipedia: Transformer types](https://en.wikipedia.org/wiki/Transformer_types#Audio_transformer) has some info:

> Audio transformers are specifically designed for use in audio circuits to carry audio signal. They can be used to block radio frequency interference or the DC component of an audio signal, to split or combine audio signals, or to provide impedance matching between high and low impedance circuits, such as between a high impedance tube (valve) amplifier output and a low impedance loudspeaker.

Taking some measurements on the red (output) audio transformer:

*   1.3 Ω secondary coil (output, connects to speaker)
*   56 Ω primary coil (input, connects to circuit)
*   28 Ω from center tap on primary coil to either side

Comes to an estimated 56/1.3 = 43:1 ratio.

And on the yellow (input) transformer:

*   156.3 Ω on secondary coil
*   190.9 Ω on primary coil
*   ~96.1 Ω from center tap

The input transformer comes to an estimated 1.2:1 turns ratio, much lower.

Measuring the AC voltage on the output transformer secondary coil, when producing sound using the machine gun pulse oscillator project, it reads in the hundreds of millivolts, and AC current in the tens of milliamps. Input voltage on the other hand reads from several volts to about ten volts. The purpose of this output transformer is now clear: transform high voltage / low current into low voltage / high current to drive the speaker.

How could we find a suitable replacement transformer? Here’s what I have:

The biggest transformer is [from a microwave oven](/@rxseger/emerson-mw8675w-microwave-oven-teardown-salvaging-the-led-display-a27cd86580b3#.jrq0ahif8), next biggest [from an UPS](/@rxseger/building-an-h-bridge-from-a-salvaged-uninterruptible-power-supply-ab1576ec313f#.18iyulrq8), the rest from various sources long lost to time. But what’s important here is the turns ratio. We want ~40x more turns on the primary coil (with a center tap) than on the secondary, or at least enough to drive the speaker.

Most of the transformers I had either possessed a low ratio (including 1.0, for [isolation transformers](https://en.wikipedia.org/wiki/Isolation_transformer)) or lacked a center tap. The lower ratio transformers may be useful for replacing 160-in-1’s “input transformer”. For the output transformer, I found one suitable:

*   primary 61.5 Ω, secondary 0.8 Ω, ratio = 76:1

This is the small yellow transformer in the bottom with five leads coming out of it (the leads are small stranded wires, giving it away this is for audio). Wired it up into the circuit, and it works fine:

## Switches

On the main board, there is a sliding [double-pole double-throw (DPDT)](https://en.wikipedia.org/wiki/Switch#Contact_terminology) switch. The kit also comes with a morse key, simply a momentary switch with [morse code](https://en.wikipedia.org/wiki/Morse_code) printed on the base, but my second-hand kit was missing this. Here’s a picture from the manual instead:

## Lights

The lamp is listed as “Lamp, red 2.5 volt, 300 mA L-0541”, but it shines well on 3 volts (2xAA), shown here (the manual warns it will burn out if powered with 9V). Brighter than I expected, and it hasn’t burnt out after all these years, or maybe it has and was replaced by previous owners of this kit. Measured resistance across the lamp of 2.5 Ω.

There is another set of lights on this kit, the LED Digital Display:

Thanks to [Jeff Keyzer for this photo](https://www.flickr.com/photos/mightyohm/2729475946/in/photostream/) (from Flickr, used with attribution, cropped), who either has a better camera or cleaner kit than mine. Close up with each segment illuminated, in low light:

This segmented display is common cathode (negative terminal), here powered by the 3 volt battery power. Listed as:

*Light Emitting Diode Display (1.6 V min, per segment, 25 mA max. DC current per segment, 3V max. reverse voltage) L-0541*

The hardwired resistors are 360 Ω, assembled on “Printed Circuit Board for LED Display X-7159”. See more about my experiences with segmented LED displays in [*Emerson MW8675W microwave oven teardown: salvaging the LED display*](/@rxseger/emerson-mw8675w-microwave-oven-teardown-salvaging-the-led-display-a27cd86580b3#.9qg0aqsvc) if you are interested. Nowadays LEDs are commonplace, and the newer kits like the 300-in-1 or even newer like various Raspberry Pi kits (as I used in [*Interrupt-driven I/O on Raspberry Pi 3 with LEDs and pushbuttons: rising/falling edge-detection using RPi.GPIO*](/@rxseger/interrupt-driven-i-o-on-raspberry-pi-3-with-leds-and-pushbuttons-rising-falling-edge-detection-36c14e640fef#.r5lh29dw2)) include a multitude of LEDs, but this kit only had an incandescent lamp and this modest 7-segment LED display.

TODO: can the *left* decimal point be powered?

## Relay

The electromagnetic relay runs on 9 volts and has a 225 Ω coil (R-8158). It is of the common [single-pole double-throw](https://en.wikipedia.org/wiki/Relay#Pole_and_throw) (SPDT) variety, with two leads for the coil, and three contacts: normally-closed, common, and normally-open. This is a good relay, the relays I salvaged in Building an *H*[*-Bridge from a salvaged Uninterruptible Power Supply*](/@rxseger/building-an-h-bridge-from-a-salvaged-uninterruptible-power-supply-ab1576ec313f#.7ed8rm454) were all SPST, so I had to use four instead of two. Of course, this puny relay is not rated for 220V on the contacts, unlike those in the UPS.

There are more modern components for switching (= the [transistor](https://en.wikipedia.org/wiki/Transistor)) but the relay remains an easy to understand instructive simple component for understanding switching circuit operation and has some other benefits. 37 of the 160 projects use the relay, from the very first (1\. Electronic Candle) to logic circuits (54\. Logic “NOR” Circuit, and others) and even high voltage generators (43\. High Voltage Generator, and 44\. High Voltage Generator II). I tested pressing a 9V battery up against its coil, and it still works, producing a satisfying click.

## Antenna coil

Speaking of coils, there is another coil in the “radio circuits” section:

it is only listed as “antenna coil (with 5 leads) CA-0619”, couldn’t find much information about it except it has a solid ferrite core. It may be damaged and needs testing in a radio circuit. TODO: measure inductance, [LCR meter](https://en.wikipedia.org/wiki/LCR_meter)

TODO: feasible to wind my own antenna coil using the wire salvaged from [*Opening a Winsson 62–1225 Power Transformer*](/@rxseger/opening-a-winsson-62-1225-power-transformer-a66c4afd25f0#.bahska9fr)?

## Meter

0–250 µA, 650 Ω movement (“Blue scale is proportional to meter current. Black is logging reference.”) Analog meters have largely since fallen out of favor with the advent of digital displays (7-segment or otherwise). This one included in the kit is for microamps, I happen to have another one for amps:

but have yet to find a practical use for it.

## CdS Cell and Solar Cell

[image credit: Jeff Keyzer](https://www.flickr.com/photos/mightyohm/2729475080/in/photostream/) via Flickr

The [CdS Cadmium Sulfide Cell photoresistor](https://en.wikipedia.org/wiki/Photoresistor) (KC-4S, CS-0100) is a light sensor. It comes with a thumble-shaped “light shield” (missing):

Similar components are available such as from [Adafruit: Photo cell CdS photoresistor](https://www.adafruit.com/product/161). Photoresistors change their resistance based on light levels, but there are alternatives to sensing light such as the [photodiode](https://en.wikipedia.org/wiki/Photodiode) which converts light to current, as I used in [*SPI interfacing experiments: EEPROMs, Bus Pirate, ADC/OPT101 with Raspberry Pi*](/@rxseger/spi-interfacing-experiments-eeproms-bus-pirate-adc-opt101-with-raspberry-pi-9c819511efea#.9yky4ya96).

“Solar Cell, open voltage 2.5V typical, short circuit current 29 µA typical at 300 lux” is how the parts list describes the solar cell. A handful of circuits in this kit can be powered entirely off of solar. For those that can’t:

## Batteries

A standard 9V battery clip and 2xAA battery holder for 3V. Not that battery-powered circuits are unheard, but modern hobbyists often use AC mains for power these days, often 5 volts through a USB adapter. But all the circuits in this kit can be powered by either of these batteries (some by both), except for “20\. Coin Battery” and related projects which itself are about creating a battery from galvanized metal and a cent: