<!--yml
category: 未分类
date: 2024-05-27 14:39:21
-->

# An Approach to Sound Synthesis with L-Systems | Nathan Ho

> 来源：[https://nathan.ho.name/posts/sound-synthesis-with-l-systems/](https://nathan.ho.name/posts/sound-synthesis-with-l-systems/)

An early obsession of mine when I was first learning about music tech was a paper by Stelios Manousakis titled “Non-Standard Sound Synthesis with L-Systems,” published 2009 in Leonardo Music Journal. At the time I was a snob about making strictly “academic” electronic music (think Stockhausen or Xenakis) and viewed things like FM and subtractive as too plebian, so synthesis methods that branded themselves as “non-standard” were very alluring. If I could give my younger self some advice, I’d tell him that you have to build foundations as a sound designer first before you can get to the crazy experimental stuff. How are you going to make a cool alien soundscape from another dimension if you can’t make a nice snare from scratch? Every good post-tonal composer understands tonality, if only to know how to avoid it. You have to know what rules you’re breaking.

With more artistic maturity under my belt, I revisited Manousakis’ paper recently and still found it interesting, so I decided to do some riffing on it. In this post, I’ll quickly explain what L-systems are (there are lots of better explanations online) and walk through a complete, “minimum viable” system for generating sound with them. Sound examples are embedded.

## L-systems and turtle graphics

Briefly, L-systems (short for “Lindenmayer systems”) generate a sequence of words (strings) using rewriting rules applied repeatedly to an initial word called the axiom. A classic L-system is given by the rules `a → ab` and `b → a` with the axiom `a`. At each iteration, each symbol in the word is replaced by the right-hand side of its corresponding rule. The first few iterations of this L-system are:

```
a
ab
aba
abaab
abaababa
abaababaabaab
abaababaabaababaababa
```

Most L-systems that are objects of study undergo approximately exponential growth in this way. L-systems can produce simple patterns like `abababab` as well as aperiodic results like the above, generating apparent randomness while being completely deterministic.

A popular use of L-systems is to apply them to turtle graphics. To do so, each symbol in the L-system corresponds to an instruction to a turtle in 2D space, such as “move forward one unit” or “turn left 30 degrees.” Following the instructions sequentially from one string and plotting the turtle’s trajectory in space produces a drawing. Organic-looking fractals may result.

In “Non-Standard Sound Synthesis with L-Systems” as well as his more detailed 2006 master’s thesis *Musical L-Systems*, Manousakis proposes a lot of different possibilites to explore, but the basic idea is that he’s sonifying turtle graphics produced by L-systems and mapping the turtle’s state over time (position, orientation, etc.) to synthesis parameters. These L-systems operate at different time scales, from microsound to the form of an entire piece.

## L-systems and microsound

At first impression, this all may sound rather boring, as one can take any synthesis algorithm and use turtle graphics to modulate its parameters over time. As there is a separation of concerns between the L-system and the synthesis method, calling this “nonstandard synthesis” may be an overstatement of its novelty. You could argue that the L-system with turtle graphics are effectively a random LFO generator.

The key to averting this criticism is to use an L-system deeply integrated into the synthesis algorithm using microsound techniques. Take for example an L-system with symbols interpreted as follows:

*   `p`: output one cycle of a sine wave

*   `.`: output one cycle of silence

*   `F`: increase sine wave frequency

*   `f`: decrease sine wave frequency

*   `+`: increase amplitude

*   `-`: decrease amplitude

Depending on the L-system’s rules and axiom, a variety of sounds may result. A long uninterrupted sequence of `p`’s will generate a static tone, patterns like `pFpFpF` and `pfpfpf` will produce sweeps and chirps, and patterns like `p.p.p.` can even output distinct timbres by periodically alternating between pulses of audio and silence.

There are a few issues with this toy example that make it unmusical. First, the synthesis method is too simple, and it mostly produces wandering sine wave tones. Second, the symbols that decrease and increase frequency and amplitude can cause those parameters to rocket off to positive or negative infinity.

## A functional L-system synthesizer

To upgrade the synthesis technique, let’s use windowed oscillator hard sync. In this synthesis method, there are two frequencies: the *sync frequency* and *intrinsic frequency*. A sine wave runs at the intrinsic frequency, and every 1 / sync frequency seconds, it has its phase reset to 0\. The resulting waveform is multiplied by a signal that starts at 1, decreases to 0, and resets to 1 every 1 / sync frequency seconds. There are now three synthesis parameters to modulate: the two frequencies, and the amplitude. The new set of symbols is as follows:

*   `p`: output one cycle of a sine wave

*   `.`: output one cycle of silence

*   `S`: increase sync frequency

*   `s`: decrease sync frequency

*   `I`: increase intrinsic frequency

*   `i`: decrease intrinsic frequency

*   `+`: increase amplitude

*   `-`: decrease amplitude

To solve the out-of-range problem, one approach is to clamp the frequencies and amplitude to minimum and maximum values. However, this causes another problem: words that contain a lot of `S`’s and not many `s`’s will immediately max out the sync frequency, and this strikes me as unmusical. The idea I had is to fold over the frequencies and amplitudes in a sine wave shape, so if the word keeps outputting `S`’s repeatedly, the sync frequency will eventually start decreasing instead. In other words, the six symbols `SsIi+-` have the effect of stepping the phases of three random LFO’s forward or backward. LFO shapes other than sine waves are possible too.

We now have a solid system for interpreting the output of the L-system. The remaining work to make this a complete sound-generation system is to design the L-system itself. This is pretty easy, as it turns out that randomly generating rules works surprisingly well. To do this, we assign a word of 2-3 random symbols as the right-hand side of each rule, and voila, we have an L-system. As a special provision, the rule corresponding to the symbol `.` must generate at least one `p` to reduce the chance of long stretches of silence.

As a loose end, there are a few ways to approach generating the word iteratively. We can start with an axiom, iterate the L-system until the word is long enough, and synthesize the final word. Alternatively, we can iterate the L-system and synthesize the result of every generation, concatenating all the synthesized waveforms. The latter is somewhat more attractive to me as there’s a chance that the listener might hear the iterative process, although I’m not sure how much it actually matters. I also cut off each word to a maximum of 1500 symbols; it will become clear why I did this later.

We have completely described an approach to sound generation with L-systems and windowed oscillator sync. It has the following parameters:

*   minimum and maximum sync frequency

*   amount sync frequency is increased or decreased with the `S` and `s` symbols

*   minimum and maximum intrinsic frequency

*   amount intrinsic frequency is increased or decreased with the `I` and `i` symbols

*   amount amplitude is increased or decreased with the `+` and `-` symbols

So, how’s it sound? Here’s the result of randomizing the above parameters and generating some uncurated sounds:

The windowed oscillator sync algorithm and the sine wave LFOs very much imprint on the resulting sounds, possibly more so than the L-system. Still, we get a mix of obvious patterns and aperiodic noisy motions that clearly sonify the systematic way the symbols are produced. To me, the sounds are pretty interesting and many seem like they’d be fairly difficult to produce without L-systems, so I’d declare this experiment a modest success.

## Mutating the rules

One problem is that the results get pretty repetitive and boring after around 10 seconds. For sounds that display long-term evolution, we can try changing the L-system’s rules over time. Every generation, we take each rule and randomly remove and add symbols. I’ve found that the evolution of the sound is highly sensitive to rule mutations (which is a good thing – it means the thing we’re sonifying is really impacting the sound), so it suffices to mutate each rule with only a 10% chance and add and delete only one symbol.

As the L-system process exponentially amplifies the number of symbols, it’s necessary to cut the words from each generation short to ensure that mutations happen in a timely manner. Otherwise the mutations will become sparser and sparser. The maximum word size becomes a parameter that determines how quickly the L-system evolves. I’ve set this to 1500.

Here are some results, again uncurated:

## Assessment and further directions

I went with the simplest possible synthesis method because I wanted to get as close to “hearing the L-system” as possible while still being sort of musical. As such, the sounds I got are raw, digital, and shamelessly granular. I wouldn’t use them as a final piece, but they could serve as raw materials for sampling and adding post-effects.

To make this more musical, the first thing I’d try is a more sophisticated synthesis algorithm. Pulsar synthesis would be my top pick, as it is very general. I’m envisioning using a buffer of recorded sound as the source for the pulsarets, modulating the buffer position with turtle graphics, and blurring the line between sampling recognizable chunks (0.1 seconds) to extracting timbre as a wavetable (< 0.02 seconds). If a pure synthesis approach is preferred, having multiple symbols corresponding to multiple types of synthesized pulsarets could help with the diversity of output.

The sine wave LFOs are very distinctive and wear themselves out rather quickly. To reduce fatigue, it may be wise to expose the minimum and maximum values of those LFOs as user parameters (or targets for random modulation). Maybe there are better approaches to using symbols to modulate parameters. It is common in graphical L-systems to have symbols that execute small bits of conditional logic, which could be exploited to produce a wider variety of modulations that avoid the “rocketing” problem. However, as much as possible I prefer complexity to come from the L-system rather than its interpretation.

Rule mutation is a big deal. It upgrades this system from a static texture synthesizer to an algorithmic music generator that allows for a balance between coherence and change. If you wait long enough (or set mutation to happen fast enough), the system can arrive at a completely different place to where it was before. I imagine running this in real time and allowing the user to save and recall the L-system to create repetition and formal structure rather than just letting the thing do a random walk.

As-is, rules are generated and mutated in a mostly nondiscerning way. The rule generation system could be biased to produce more musical results. Alternatively, instead of using random generation, the rules could be exposed to the user for direct editing as a mode of expression.

There are many variations to L-systems, including stochastic L-systems and context-sensitive L-systems. One significant one is “branched” L-systems where, for instance, the symbol `[` stores and pushes the current state of the turtle to a stack and the symbol `]` pops and restores the state. These are common in graphical L-systems when creating fractals. Manousakis proposes an interpretation of branched L-systems as producing polyphony, with appropriate limits on the number of simultaneous voices.

I’m not yet clear on how to use L-systems to generate macro-level structure in a way that’s musically compelling and transparent with respect to the L-system’s behavior. To throw out an idea, the mutation process could itself be commandeered by an L-system.

## Conclusions

Admittedly, the sound examples in this post demonstrate more potential than musicality. I’m fine with that. I don’t know if I’ll end up incorporating L-systems into a more serious artistic pursuits, but it’s good to know that they show promise.