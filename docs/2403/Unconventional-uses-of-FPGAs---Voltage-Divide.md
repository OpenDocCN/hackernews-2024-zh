<!--yml
category: 未分类
date: 2024-05-29 12:29:22
-->

# Unconventional uses of FPGAs | Voltage|Divide

> 来源：[https://voltagedivide.com/2024/03/18/unconventional-uses-of-fpgas/](https://voltagedivide.com/2024/03/18/unconventional-uses-of-fpgas/)

My favorite phenomenon in digital circuitry is when digital threatens to become analog again. For example, a lot of people are interested in the idea of overclocking an FPGA for more performance and are usually encouraged not to do so. However, thinking about the reasons why overclocking goes wrong and learning to understand the issues with overclocking can allow you to achieve a stable overclock or at least help you understand your chip better. This is only one example of exploring odd behavior of FPGAs when pushing beyond suggested limits.

Every sensor is a temperature sensor, nearly everything is a resistor or a conductor if you try hard enough and anything is an antenna. Datasheets are just a suggestion, and finally, often we pretend things are ideal, when they often are not.

 # Why?

Being able to understand the reasoning behind common advice, occasionally pushing the limits in extreme situations and generally knowing what is possible and maybe trying some of these tricks for yourself will help you understand the failure conditions and the real limits of your parts and tooling which will, overall, make you a better developer.

I personally believe that common advice should not simply be followed, but understood and exceptions applied.

Note that some of the techniques below may result in damage to an FPGA chip. You have been warned.

# Background

At the end of the day, the digital circuitry in your FPGA chip are analog circuits. When they are operating within their bounds, they properly behave like ideal digital circuits. Usually you want your digital parts to operate digitally but some of the analog characteristics might be useful to you as we will see later.

## Process, Voltage and Temperature Variation

Every silicon chip exhibits process, voltage and temperature variation.

Process variation are effects related to the manufacturing of the chip, it is why FPGAs are often sold with speed grades or speed bins with faster dies being sold for more. CPUs used for overclocking may be called lucky samples or golden chips to denote their exceptional performance when the chip is pushed to the limit. Within a speed grade or product category, chips still exhibit variation. The bins are simply not granular enough to capture all possible outcomes of a manufacturing process. A good sample may exhibit lower leakage, resulting in lower static power, or lower propagation delay.

Voltage variation in operation also usually variation in the propagation delay of a digital circuit when voltage is increased, propagation delay decreases. Silicon chips also draw power relative to the square of the voltage, therefore, small increases in voltage can result in significant increases in power consumption and as a result, heat generation. The reverse is true when voltage is decreased. Electron migration increases with higher voltage and dielectric breakdown could occur as well which can result in destruction of the chip.

Temperature variation also often causes an increase in propagation delay due to increased temperature. Leakage often goes up resulting in higher power consumption at elevated temperature as well.

## Propagation Delay

Within any digital circuit, there is signal propagation delay due to parasitic inductance, capacitance and resistance. It is what generally limits the maximum clock speed of a circuit.

Propagation delay is modeled and used in static timing analysis within the FPGA synthesis, placement and routing tools to predict behavior and ensure reliable functionality in the field.

Propagation delay in a circuit is one of the analog effects exploited in digital circuitry to do fancy things like making delay lines or ring oscillators as we will see.

### Combinational Logic Glitching

Propagation delay within the lookup tables (LUTs) and routing can be used to produce very short pulses, known as glitches in a circuit. These can be tens to hundreds of picoseconds in duration. Often, the glitches are undesired and result in increased power consumption in an FPGA design. However, the high speed pulses could be used in wave union time to digital converters as we will see.

## Metastability

Metastability of edge sensitive flip flops in an FPGA is a problem when setup and hold timing of a flip flop is not respected. When metastability occurs, a flip flop may randomly settle to a 1 or 0 output, irrespective of its input value. It is usually a problem that needs to be avoided when crossing between clock domains. It is generally undesired behavior that must be avoided for reliable operation of digital circuitry and can also arise when overclocking or sampling the data in various unconventional FPGA circuitry. In some cases, it could be used as an entropy source in random number generators.

## Overclocking

Overclocking is when chips are operated with clock frequencies beyond the manufacturers recommendation to gain performance. It is often done with increased voltage to reduce internal propagation delays to avoid metastability and increased cooling to prevent damage due to chip.

There are ways to overclock with high certainty that the design will be stable, some ways involve measuring propagation delay using ring oscillators as we will see below.

Sometimes there are applications where you don’t need 100% reliability or it is not possible to hit the throughput or latency thresholds required for your application and a custom ASIC is not an option. For a while, when crypto mining was more popular on FPGA, overclocking the FPGA was pretty common.

Examples

1.  [https://tspace.library.utoronto.ca/bitstream/1807/101178/3/Ahmed_Ibrahim_%20_202006_PhD_thesis.pdf](https://tspace.library.utoronto.ca/bitstream/1807/101178/3/Ahmed_Ibrahim_%20_202006_PhD_thesis.pdf)

## Device Primitives

Device primitives are the individual building blocks within an FPGA that are used to construct the logic you described. Among others, these include lookup tables, flip-flops (registers), multipliers, adders, carry chains, variable delay lines, clock generators, high speed serial transceivers, clock buffers, IO drivers and receivers and more. Understanding these individual blocks and learning about their inherent behavior is what makes it possible to construct unconventional circuitry on an FPGA. For everyday work, understanding these blocks well is one important cornerstone of your knowledge as an FPGA developer.

# Ring Oscillators

Ring oscillators are constructed using a combinational loop of lookup tables or other delays elements. A chain of digital inverters will oscillate, creating a clock that varies with propagation delay through the individual inverters. The lower the delay, the higher the clock frequency. The frequency will also vary with PVT and other effects.

Note that a ring oscillator operating at a high frequency can result in high power consumption and localized heating which can damage the FPGA.

## Dynamic Propagation Delay Measurement

One use of a ring oscillator is to measure the propagation delay of the inverter chain by comparing the frequency of the ring oscillator against a stable oscillator. This measurement of propagation delay could be used in a dynamic voltage and frequency scaling scheme to allow for stable overclock across PVT.

Examples

1.  [https://www.eecg.utoronto.ca/~vaughn/papers/apec2016_dvs.pdf](https://www.eecg.utoronto.ca/~vaughn/papers/apec2016_dvs.pdf)

## Strain Gauge

A ring oscillator in an FPGA can be used as a strain gauge. If you implement a ring oscillator on an FPGA and then bend the FPGA, the frequency of the ring oscillator will shift. I don’t know the reason for this. People have suggested the [piezoresistive effect](https://en.wikipedia.org/wiki/Piezoresistive_effect) of silicon.

Examples

1.  [https://hackaday.com/2015/09/27/mystery-fpga-circuit-feels-the-pressure/](https://hackaday.com/2015/09/27/mystery-fpga-circuit-feels-the-pressure/)

# Time to Digital Converters

Time to digital converters (TDCs) are circuits which time stamp events. The simplest of which is just a regular counter with logic that logs the count when rising or falling edges of input signals are observed. The problem with this approach is that the timing precision is limited to the clock frequency of the circuit. A 500 MHz clock can only time events to within about 2 nanoseconds. TDCs capable of timing events to within picoseconds are important instruments used in laser ranging, ultrasonic ranging, particle physics experiments and slope ADCs.

Several TDC designs exist but tapped delay lines are one of the most interesting in my opinion because they exploit the analog properties of an FPGA.

## Tapped Delay Lines

One method to produce reliable, timing of events to about 10 picoseconds resolution is known as time interpolation. The time interpolation method utilizes the propagation delay of device primitives in an FPGA to produce a tapped delay line (TDL). The tapped delay line is sampled to measure more precisely, how long before a clock edge an event actually occurred.

The TDL method of time interpolation can be improved through the generation and measurement of several events generated from one event. This is known as the Wave Union TDC (WU-TDC). This method requires a fast pulse generator which we will go into below.

# Short Pulse Generator

Very short pulses can be useful in tapped delay lines. Pulses with lengths of hundreds of picoseconds can be produced by exploiting combinational glitching. This is useful in the creation of Wave Union TDCs as mentioned above.

It may also be possible to use these super short pulses for ultra-wideband communications (UWB).

Examples

1.  UWB pulse generator, [https://www.edn.com/build-a-uwb-pulse-generator-on-an-fpga/](https://www.edn.com/build-a-uwb-pulse-generator-on-an-fpga/)
2.  Wave Union TDC, [https://www.mdpi.com/2079-9292/11/1/30](https://www.mdpi.com/2079-9292/11/1/30)

# True Random Number Generation

True random numbers are an important part of encryption. There are a few attempts at making a true random number generator (TRNG) in an FPGA. Many use ring oscillators or metastability as a source of randomness.

Examples

1.  Ring oscillator based TRNG, [https://github.com/stnolting/neoTRNG](https://github.com/stnolting/neoTRNG)
2.  Metastability based TRNG, [https://people.csail.mit.edu/devadas/pubs/ches-fpga-random.pdf](https://people.csail.mit.edu/devadas/pubs/ches-fpga-random.pdf)
3.  Commercially available core, [https://www.bertendsp.com/products/trng-p200/](https://www.bertendsp.com/products/trng-p200/)
4.  Commercially available core, [https://www.latticesemi.com/products/designsoftwareandip/intellectualproperty/ipcore/xipheracores/xip8001b](https://www.latticesemi.com/products/designsoftwareandip/intellectualproperty/ipcore/xipheracores/xip8001b)

# Physical Unclonable Functions

A physical unclonable function (PUF) is like a fingerprint, it is a design that gives a unique ID for a single bitstream AND FPGA die pair. Two FPGAs should never have the same ID and if you change the bitstream, the IDs should all change.

This is a particularly tricky one. The circuit design must be resistant to temperature and voltage variation, but not resistant to process variation for the PUF to be reliably consistent on a single FPGA die but reliably inconsistent across FPGA dies.

If they can be made reliably, a PUF can be used as a cryptographic key source that does not require a battery backup. In this case, it is probably advisable to use a PUF as part of a challenge/response system, rather than using the ID as a key directly.

Examples

1.  [https://github.com/stnolting/fpga_puf](https://github.com/stnolting/fpga_puf)

# Digital to Analog Conversion

There are many ways to produce a digital to analog converter (DAC) using logic on an FPGA and the regular IO pins.

1.  PWM and an external low pass filter
    1.  Not the most effective but the easiest to implement, probably ok for a basic audio output
2.  Low pass delta sigma modulator and external low pass filter
    1.  Quite a bit better then a PWM DAC, can likely get to about 10-12 bit resolution with very high oversampling
    2.  [https://github.com/hamsternz/second_order_sigma_delta_DAC](https://github.com/hamsternz/second_order_sigma_delta_DAC)
3.  Band pass delta sigma modulator with external band pass filter
    1.  Taking the low pass delta sigma modulator and replacing the integrators with resonators, you can make a band pass delta sigma modulator that can reach low RF frequencies with regular IO pins, it is pretty easy to transmit an FM signal that lands in the broadcast band with just a single IO pin at 600 MHz using an output serializer
4.  Radio frequency DACs using multi-gigabit transceivers
    1.  Applying similar techniques as above, much higher carrier frequencies and much higher performance can be reached with the use of multi-gigabit transceivers
    2.  [https://ieeexplore.ieee.org/document/6927248](https://ieeexplore.ieee.org/document/6927248)
    3.  [https://ieeexplore.ieee.org/document/8421247](https://ieeexplore.ieee.org/document/8421247)

# Analog to Digital Conversion

Analog to digital conversion (ADC) using an FPGA is quite a bit trickier then digital to analog conversion. There have however been many successes in this field.

ADCs in FPGAs are popular in cryogenic applications like quantum computing due to their low power. If you need an ADC near the cooling head where you are trying to maintain near absolute zero, any heat generated makes your design more difficult.

The analog properties of LVDS receiver pins are often exploited as an analog comparator. Slope ADCs with external capacitors or even using just the parasitic capacitance of a pin can be used to produce high speed or high precision ADCs.

Examples

1.  600 MSPS ADC with 7 effective number of bits, suitable for RF application, [https://github.com/LukiLeu/FPGA_ADC](https://github.com/LukiLeu/FPGA_ADC)
2.  Over 1 GSPS using several LVDS pins with time interleaving, [https://ieeexplore.ieee.org/document/7593301](https://ieeexplore.ieee.org/document/7593301)
3.  LVDS pin AM receiver, [https://github.com/dawsonjon/FPGA-radio](https://github.com/dawsonjon/FPGA-radio)

# Frequency Synthesis

While there are PLLs for clock generation on an FPGA, it is possible to create a PLL on an FPGA usable for radio frequency frontends with some external components. In theory, this could replace an external frequency synthesizer in some RF frontends.

Examples

1.  38-76 MHz Fractional-N Synthesizer, [http://www.aholme.co.uk/Frac3/Main.htm](http://www.aholme.co.uk/Frac3/Main.htm)

# Antenna

The routing on an FPGA is basically just copper. Using custom routing either through manual routing or a custom tool that generates a suitable layout, an FPGA can be used as an antenna.

In theory, this could be used for some sort of data exfiltration to transmit data. Receive would be quite a bit more difficult since you normally need an amplifier after the antenna. It could be possible to use a flip-flop as a single bit ADC with gain if you could somehow bias it in the middle of its input operating range so it is as metastable as possible.

Examples

1.  Exploration of FPGA interconnect for the design of unconventional antennas, [https://dl.acm.org/doi/10.1145/1950413.1950455](https://dl.acm.org/doi/10.1145/1950413.1950455)