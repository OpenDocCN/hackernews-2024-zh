<!--yml
category: 未分类
date: 2024-05-29 12:49:09
-->

# hardware - How Do Computers Work? - Software Engineering Stack Exchange

> 来源：[https://softwareengineering.stackexchange.com/questions/81624/how-do-computers-work](https://softwareengineering.stackexchange.com/questions/81624/how-do-computers-work)

I will start from the lowest level that might be relevant (I can start from even lower level, but they are probably way too irrelevant), starting from Atom, to Electricity, to Transistors, to Logic Gates, to Integrated Circuits (Chip/CPU), and finishes at Assembly (I'd assume you are familiar with the higher levels).

# In the Beginning

## Atom

[Atom](http://en.wikipedia.org/wiki/Atom) is a structure composed of electrons, protons, and neutron (which themselves are composed of [elementary particles](http://en.wikipedia.org/wiki/Elementary_particle)). The most interesting part of the atom for computers and electronics are the [electrons](http://en.wikipedia.org/wiki/Electron) because electron are mobile (i.e. it can move around relatively easily, unlike protons and neutrons which are more difficult to move) and they can free-float by itself without being held inside an atom.

Usually, each atoms has equal number of protons and electrons, we call this "neutral" state. As it happens, it is possible for an atom to lose or gain extra electrons. Atoms in this unbalanced state are said to be "positively charged" atom (more proton than electrons) and "negatively charged" atom (more electron than proton) respectively.

Electrons are unconstructible and indestructible (not so in quantum mechanics, but that's irrelevant for our purpose); so if an atom loses an electron, some other atom nearby had to receive the extra electrons or the electron had to released into a free floating electron, conversely since electron is unconstructible, to gain extra electron, an atom had to sap it off nearby atoms or from a free floating electron. The mechanics of electrons is such that if there is a negatively-charged atom near a positively-charged atom, then some electrons will migrate until both atoms have the same charge.

## Electricity

[Electricity](http://en.wikipedia.org/wiki/Electricity) is just a flow of electrons from an area with very high number of negatively-charged atoms to an area with very high number of positively-charged atoms. Certain chemical reactions can create a situation where we have one nodes with lots of negatively-charged atoms (called "anode"), and another node with lots of positively-charged atoms (called "cathode"). If we connect two oppositely charged nodes with a wire, masses of electrons will flow from the anode to cathode, and this flow is what we call "electric current".

Not all wires can transmit electrons equally easily, electrons flows much easily in "conducting" materials than in "resistant" materials. A "conducting" material have low electrical resistance (e.g. copper wires in cables) and a "resistant" material have high electrical resistance (e.g. rubber cable insulation). Some interesting materials are called semi-conductors (e.g. silicons), because they can alter their resistance easily, under certain conditions a semiconductor might act as a conductor and at other conditions it might turn into a resistor.

Electricity always prefers to flow through the material with least resistance, so if a cathode and anode are connected with two wires, one having very high resistance and the other with very low resistance, the majority of electrons will flow through the low resistance cable and nearly none flows through the high resistance material.

# The Middle Age

## Switches and Transistors

Switches/Flip-Flops are like your regular light switches, a switch can be placed between two pieces of wire to cut off and/or restore electricity flow. Transistors works exactly the same as a light switch, except that instead of physically connecting and disconnecting wires, a transistor connects/disconnects electricity flow by altering its resistance depending on whether there is electricity in the base node, and, as you might have already guessed/know, transistors are made from semiconductors because we can alter semiconductor to become either a resistor or a conductor to connect or disconnect electric currents.

One common type of transistor, the [NPN Bipolar Junction Transistor](http://en.wikipedia.org/wiki/Bipolar_junction_transistor) (BJT), has three nodes: "base", "collector", and "emitter". In an NPN BJT, electricity can flow from the "emitter" node to the "collector" node only when the "base" node is charged. When the base node is not charged, practically no electron can flow through and when the base node is charged, then electrons can flow between the emitter and the collector.

## The behavior of a transistor

(I highly suggest you read through [this](http://www.spsu.edu/cs/faculty/bbrown/web_lectures/transistors/) before continuing, as it can explain better than me with interactive graphics)

Let's say we have a transistor connected to an electric source at its base and collector, and then we wire up an Output cable near its collector (see Figure 3 in [http://www.spsu.edu/cs/faculty/bbrown/web_lectures/transistors/](http://www.spsu.edu/cs/faculty/bbrown/web_lectures/transistors/)).

When we apply electricity to neither base nor collector, then no electricity can flow at all since there is no electricity to talk about:

```
B   C  |  E   O
0   0  |  0   0 
```

When we apply electricity to the collector but not the base, electricity cannot flow to the emitter since the base becomes a high resistance material, so the electricity escapes to the Output wire:

```
B   C  |  E   O
0   1  |  0   1 
```

When we apply electricity to the base but not the collector, also no electricity can flow since there is no charge difference between the collector and the emitter:

```
B   C  |  E   O
1   0  |  0   0 
```

When we apply electricity to both base and collector, we get electricity flowing through the transistor, but since the transistor now has lower resistance than the Output wire, nearly no electricity flows through the Output wire:

```
B   C  |  E   O
1   1  |  1   O 
```

## Logic Gates

When we connect the emitter of one transistor (E1) to the collector of another transistor (C2) and then we connect an output near the base of the first transistor (O) (see Figure 4 in [http://www.spsu.edu/cs/faculty/bbrown/web_lectures/transistors/](http://www.spsu.edu/cs/faculty/bbrown/web_lectures/transistors/)), then something interesting happens. Let's also say we always apply electricity to the collector of the first transistor (C1) and so we only play around with the the base nodes of the transistors (B1,B2):

```
B1   B2   C1   E1/C2  |  E2   O
----------------------+----------
0    0    1    0      |  0    1
0    1    1    0      |  0    1
1    0    1    0      |  0    1
1    1    1    1      |  1    0 
```

Let's summarize the table so we only see B1, B2, and O:

```
B1   B2  |  O
---------+-----
0    0   |  1
0    1   |  1
1    0   |  1
1    1   |  0 
```

*Lo and behold*, if you're familiar with Boolean Logic and/or Logic Gates, you should notice that this is precisely the NAND gate. And if you're familiar with Boolean Logic and/or Logic Gates you might also know that a NAND (as well as NOR) is [functionally complete](http://en.wikipedia.org/wiki/Functional_completeness), i.e. using NAND only, you can construct all the other logic gates and the rest of the truth tables. In other word, you can design a whole computer chip using NAND gates alone.

In fact, most CPUs are (or is it used to be?) designed using NAND only since it is cheaper to manufacture than using a combination of NAND, NOR, AND, OR, etc.

## Deriving the other boolean operators from NAND

I would not describe how to make all boolean operators, only the NOT and the AND gate, you can find the rest somewhere else.

Given a NAND operator, then we can construct a NOT gate:

```
Given one input B
O = NAND(B, B)
Output O 
```

Given a NAND and NOT operator, then we can construct an AND gate:

```
Given two inputs B1, B2
C = NAND(B1, B2)
O = NOT(C) // or NAND(C,C)
Output O 
```

We can construct other logic gates in a similar way. Since NAND gate is *functionally complete*, it is also possible to construct logic gates with more than 2 inputs and more than 1 output, I'm not going to discuss how to construct such logic gates here.

# Enlightenment Age

## Building a Turing Machine from Boolean Gates

A CPU are just a more complicated version of a Turing Machine. The CPU registers are the Turing Machine's internal state, and the RAM is a Turing Machine's tape.

A Turing Machine (CPU) can do three things:

*   read a 0 or 1 from the tape (read a cell of memory from RAM)
*   change its internal state (change its registers)
*   move left or right (read multiple position from the RAM)
*   write a 0 or 1 to the tape (write to a cell of memory to RAM)

For our purpose, we're building Wolfram's [2-state 3-symbol Turing Machine](http://en.wikipedia.org/wiki/Wolfram%27s_2-state_3-symbol_Turing_machine) using combinatorial logic (modern CPUs would use microcode, but they're more complex than is necessary for our purpose).

The state table of the Wolfram's (2,3) Turing Machine is as follow:

```
 A       B
0   P1,R,B  P2,L,A
1   P2,L,A  P2,R,B
2   P1,L,A  P0,R,A 
```

We want to reencode the state table above as a truth table:

```
Let I1,I2 be the input from the tape reader (0 = (0,0), 1 = (0,1), 2 = (1,0))
Let O1,O2 be the tape writer (symbol encoding same as I1,I2)
Let M be connected to the machine's motor (0 = move left, 1 = move right)
Let R be the machine's internal state (A = 0, B = 1)
(R(t) is the machine's internal state at timestep t, R(t+1) at timestep t+1)
(Note that we used two input and two outputs since this is a 3-symbol Turing machine.)

      R  0          1
I1,I2
(0,0)    (0,1),1,1  (1,0),0,0
(0,1)    (1,0),0,0  (1,0),1,1
(1,0)    (0,1),0,0  (0,0),1,0

The truth table for the state table above:

I1  I2  R(t) | O1  O2  M   R(t+1)
-------------+--------------------
0   0   0    | 0   1   1   1
0   0   1    | 1   0   0   0
0   1   0    | 1   0   0   0
0   1   1    | 1   0   1   1
1   0   0    | 0   1   0   0
1   0   1    | 0   0   1   0 
```

I'm not really going to construct such a logic gate (I'm not sure how to draw it in SE and it's probably going to be quite huge), but since we know that NAND gate is *functionally complete*, then we have a way to find a series of NAND gates that will implement this truth table.

An important property of Turing Machine is that it is possible to emulate a [stored-program computer](http://en.wikipedia.org/wiki/Stored-program_computer) using a Turing machine that only have a fixed states table. Therefore, any Universal Turing Machine can read its program from the Tape (RAM) instead of having to have its instruction hardcoded into the internal state table. In other word, our (2,3) Turing Machine can read its instructions from I1,I2 pins (as software) instead of being hardcoded in the logic gate implementation (as hardware).

## Microcodes

Due to the increasing complexity of modern CPUs, it becomes prohibitively difficult to use combinatorial logic alone to design a whole CPU. Modern CPU is usually designed as an interpreter of microcodes instruction; a microcode is a small program embedded in the CPU that is used by the CPU to interpret the actual machine code. This microcode interpreter itself are generally designed using combinatorial logic.

## Register, Cache, and RAM

We have forgotten something above. How do we remember something? How do we implement the tape and RAM? The answer is in an electronic component called Capacitor. A capacitor is like a rechargeable battery, if a capacitor is charged it will retain extra electrons and it can also return electrons to the circuitry.

To write to a capacitor, we fill the capacitor with electron (write 1) or drain all the electrons in the capacitor until it's empty (write 0). To read the value of a capacitor, we try to discharge it. If, when we try to discharge, there is no electricity flowing, then the capacitor is empty (read 0), but if we detect electricity, then the capacitor must be charged (read 1). You might notice that reading a capacitor drains its electron store, modern RAMs have the circuitry to periodically recharge capacitor so they can retain their memory as long as there is electricity.

There are multiple types of capacitors used in a CPU, the CPU registers and the higher level CPU caches are made using very high-speed "capacitors" that is actually built from transistors (therefore there is almost no "lag" to read/write from them), these are called static RAM (SRAM); while the main memory RAM is made using lower power, but slower and much cheaper capacitors, these are called Dynamic RAM (DRAM).

## Clock

A very important component of a CPU is the clock. A clock is a component that "ticks" regularly to synchronize processing. A clock typically contains a quartz or other materials with well-known and relatively constant oscillation period, and the clock circuitry maintain and measures this oscillation to maintain its sense of time.

CPU operations are done *between* clock ticks and read/writes are done *in* the ticks to ensure that all components move synchronously and not trample into each other while in intermediate states. In our (2,3) Turing Machine, *between* clock ticks electricity passes through the logic gates to calculate the output from the input (I1, I2, R(t)); and *in* the clock ticks, the tape writer will write O1,O2 to the tape, the motor will move depending on the value of M, and the internal register is written from the value of R(t+1), then the tape reader will read the current tape and put charge into I1,I2 and the internal register is reread back to R(t).

## Talking with Peripherals

Note how the (2,3) Turing Machine interfaces with its motor. That is a very simplified view of how a CPU may interface with an arbitrary hardware. Arbitrary hardware can listen or write to a specific wire for inputs/outputs. In the case for the (2,3) Turing Machine, its interface with the motor is just a single wire that instructs the motor to turn clockwise or counterclockwise.

What is left unsaid in this machine is that the Motor had to have another "clock" that runs in synchrony with the Machine's internal "clock" to know when to start and stop running, so this is an example of a [synchronous data transmission](http://en.wikipedia.org/wiki/Data_transmission#Asynchronous_and_synchronous_data_transmission). The other commonly used alternative, asynchronous transmission uses another wire, called the interrupt line, to communicate synchronization points between the CPU and the asynchronous device.

# Digital Age

## Machine code and Assembly

Assembly language is a human readable mnemonic for machine codes. In the simplest case, there is a one-to-one mapping between assembly to machine code; although in modern assembly languages some instructions may map to multiple opcodes.

## Programming Language

We all are familiar with this aren't we?

* * *

Phew, finally finished, I typed all this in just 4 hours, so I'm sure there is a mistake somewhere (I'm primarily a programmer, not electric engineer nor physicists, so there might be several things that is blatantly wrong). Please if you found a mistake, don't hesitate to give a @yell or fix it yourself if you have the rep or create a complementary answer.