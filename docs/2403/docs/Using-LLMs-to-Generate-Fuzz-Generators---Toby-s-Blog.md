<!--yml
category: 未分类
date: 2024-05-27 14:48:34
-->

# Using LLMs to Generate Fuzz Generators - Toby's Blog

> 来源：[https://verse.systems/blog/post/2024-03-09-using-llms-to-generate-fuzz-generators/](https://verse.systems/blog/post/2024-03-09-using-llms-to-generate-fuzz-generators/)

LLMs seem surprisingly good at many things. So much so that not a week goes by without someone coming up with yet another use-case for this technology, often to solve tasks quickly that traditionally took a non-trivial amount of human work to complete.

Today’s example was [Brendan Dolan-Gavitt](https://engineering.nyu.edu/faculty/brendan-dolan-gavitt)’s remarkable [experiment](https://twitter.com/moyix/status/1765967602982027550) using [Claude](https://claude.ai) to generate a [fuzzer](https://en.wikipedia.org/wiki/Fuzzing) for some GIF parsing code. The entire thread is worth a read if you’re curious; however, Brendan gives Claude the C code for the GIF parser and asks Claude to generate the Python implementation for a fuzzer to generate GIF-like input to fuzz the given GIF parser. Claude dutifully complies and the resulting fuzzer finds a range of vulnerabilities. Brendan since [repeated](https://twitter.com/moyix/status/1766225423841550381) that success with a less well-known format, VRML files.

This success is at first glance very pleasing: writing a good fuzzer for a non-trivial input format is time-consuming. Being able to automate this process is therefore super appealing. Brendan’s success here might even seem surprising. It certainly was to me when I first saw his tweet. Having thought about it, with the benefit of hindsight, I’ve started to understand why Brendan might have expected it to work in the first place.

## Why Might This Work?

The reason I found Brendan’s success surprising is because I hold the opinion that LLMs are not a great way to do precise reasoning about code. Indeed, many people have tried using LLMs to find bugs directly in code, by crafting a prompt with instructions telling the LLM what to do and including the code within the prompt. In asking an LLM to look at a piece of code and spot bugs in it, one is essentially asking the LLM to perform [static analysis](https://en.wikipedia.org/wiki/Static_program_analysis).

I hold the opinion that we shouldn’t expect LLMs to be very good at static analysis. After all, they are stochastic machines trained to produce the output that is statistically most likely to come next given the LLM’s training data. In other words, to generate output statistically “close to” the expected output. This is why they are of course known to be imprecise, or to [hallucinate](https://en.wikipedia.org/wiki/Hallucination_(artificial_intelligence)) and so on. But effective static analysis requires *precision* above all else: if the static analyser tells you there is a bug in the code, it has to be very sure the bug is real. Otherwise, developer time drowns in a sea of [false bug reports](https://owasp.org/www-community/controls/Static_Code_Analysis#:~:text=Limitations-,False%20Positives,application%20from%20input%20to%20output.).

But, in contrast to static analysis, fuzzing is very different. Fuzzing is an inherently stochastic process. Quoting from [Wikipedia](https://en.wikipedia.org/wiki/Fuzzing):

> An effective fuzzer generates semi-valid inputs that are “valid enough” in that they are not directly rejected by the parser, but do create unexpected behaviors deeper in the program and are “invalid enough” to expose corner cases that have not been properly dealt with.

In other words, a fuzzer should generate inputs that are “close to” what is expected by the program. Therefore, we might expect LLMs to be a better fit at generating a “close enough” fuzzer for a given program.

## Fuzzing an Unknown Input Format

One thing (to quote Carrie Bradshaw) I couldn’t help but wonder, however, about Brendan’s experiments was to what degree Claude performed well because both GIF and VRML are known input formats. We should expect the LLM to “know” about both formats, given that its training data likely contained much text about each input format, as well as lots of parsing and serialisation code for both formats, too (although considerably more for GIF than VRML).

I therefore decided to test Claude on an unknown input format. Fortunately for me I had one lying around. 8 years ago, in 2016, when I first started [teaching](../2016-07-18-software-engineering-teaching/) at Melbourne, I set my students an assignment in which I created a fictitious input format and asked them to write fuzzers for this input format.

To make the assignment challenging, the input format was protected by a [CRC32](https://en.wikipedia.org/wiki/Cyclic_redundancy_check) checksum. Specifically, each input is a *packet*, whose structure looks like the following and includes a (useless) sequence number, a two-byte length field, and a data payload (whose maximum size is 4096 bytes):

```
/** Raw packet format:  * *        +---------+----------+-----+-------------------+ *        |  crc32  | seq_num  | len |       data        | *        +---------+----------+-----+-------------------+ * bytes  0  ...  3  4  ...  7  8  9  10   ...   len+10 */ 
```

The data payload consisted of a sequence of instructions (each one byte in size) for a simple arithmetic [stack machine](https://en.wikipedia.org/wiki/Stack_machine) that operated over a stack of signed integers (a bit like the venerable [UNIX `dc` utility](https://en.wikipedia.org/wiki/Dc_(computer_program)). The machine instructions included ones to push small integers (in the range [0,9]) onto the stack; to pop the stack; to `ADD`, `SUBTRACT`, `MULTIPLY` or `DIVIDE` the two items on the top of the stack, popping them from the stack and pushing the result; as well as operations to `READ` and `WRITE` to arbitrary positions in the stack (identified using the top-of-stack operand as an offset downwards from the top of the stack).

I wrote a parser for the input format and an interpreter for the stack machine language. The [C code](https://www.dropbox.com/scl/fi/f4ixkhypnm0g9e0n42nfz/for_brendan.tar.gz?rlkey=0730l3mkoosn6cwo67vzazbz8&dl=0) that implements this functionality is under 400 [SLOC](https://en.wikipedia.org/wiki/Source_lines_of_code). I seeded a range of vulnerabilities in this code, namely:

*   A [Heartbleed](https://en.wikipedia.org/wiki/Heartbleed) style out-of-bounds `memcpy` in the parsing code (blindly trusting the input data’s length field to use as the length to pass to `memcpy`, even though the source and destination buffers’ sizes are both 4096 bytes while the maximum 2-byte length value is 65535);
*   Unchecked division-by-zero when the second argument to the `DIVIDE` instruction was zero;
*   Unchecked signed overflow when stack operands grow too large due to repeated application of arithmetic operations;
*   Out-of-bounds read due to unchecked offset in the implementation of the `READ` instruction;
*   Out-of-bounds write similarly in the `WRITE` instruction.

I [asked](https://gist.github.com/tobycmurray/195dc742a82c9bea8e02928229b4692f) Claude Sonnet (the default, free Claude model) to generate a Python fuzzer for the C code. The prompt I crafted gave Claude some hints about the code structure, telling it that there was an outer parser implemented by the `parse_packet` function, followed by an interpreter implemented by the `execute` function. My prompt (linked above) included the relevant bits of the code along with these explanations and the request to the model to generate the Python fuzzer.

Claude generated a pretty useful fuzzer, although I had to ask it to fix a type error in the code it generated (which it did first try after I gave it the error message).

The resulting Python fuzzer code is below:

```
import random import struct import sys import binascii   # Constants MAX_DATA_LEN = 0xfff PACKET_LEN = MAX_DATA_LEN + 10 CRC_OFFSET = 0 SEQ_NUM_OFFSET = 4 LEN_OFFSET = 8 DATA_OFFSET = 10   # Instructions INSTRUCTION_ADD = ord('a') INSTRUCTION_SUBTRACT = ord('s') INSTRUCTION_READ = ord('r') INSTRUCTION_MULT = ord('m') INSTRUCTION_DIVIDE = ord('d') INSTRUCTION_POP = ord('p') INSTRUCTION_WRITE = ord('w') INSTRUCTIONS = [INSTRUCTION_ADD, INSTRUCTION_SUBTRACT, INSTRUCTION_READ, INSTRUCTION_MULT,  INSTRUCTION_DIVIDE, INSTRUCTION_POP, INSTRUCTION_WRITE]   # Generate packets for _ in range(10000):  # Generate packet data data_len = random.randint(0, MAX_DATA_LEN) data = bytearray(random.randrange(256) for _ in range(data_len)) padding = [random.choice(INSTRUCTIONS + list(range(ord('0'), ord('9') + 1))) for _ in range(MAX_DATA_LEN - data_len)] data += bytes(padding)   # Generate sequence number seq_num = random.getrandbits(32)   # Construct packet packet = bytearray(PACKET_LEN) struct.pack_into(">I", packet, CRC_OFFSET, 0)  # Placeholder for CRC32 struct.pack_into(">I", packet, SEQ_NUM_OFFSET, seq_num) struct.pack_into(">H", packet, LEN_OFFSET, data_len) packet[DATA_OFFSET:] = data   # Calculate CRC32 and update packet crc32 = binascii.crc32(packet[4:]) struct.pack_into(">I", packet, CRC_OFFSET, crc32)   # Write packet to stdout sys.stdout.buffer.write(packet) 
```

I compiled the C program with Clang’s [Address Sanitizer](https://clang.llvm.org/docs/AddressSanitizer.html) and [Undefined Behaviour Sanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) turned on. Running the Python code to produce inputs for the C program, and running the compiled C program against the inputs produced, I found that Claude’s fuzzer is able to trigger the out-of-bounds read and write bugs only.

However, the fuzzer is unable to trigger the other bugs. I shared this code with Brendan who also played with it and got [similar results](https://twitter.com/moyix/status/1766311605413818826).

What should we make of this?

## Understanding the Results

Why is Claude’s fuzzer unable to find the other bugs? If we look at the fuzzing code, the answer is pretty clear. Here is the relevant snippet:

```
 # Generate packet data data_len = random.randint(0, MAX_DATA_LEN) data = bytearray(random.randrange(256) for _ in range(data_len)) padding = [random.choice(INSTRUCTIONS + list(range(ord('0'), ord('9') + 1))) for _ in range(MAX_DATA_LEN - data_len)] data += bytes(padding) 
```

Notice how the packet data it generates is totally random bytes. It then generates what it calls `padding` which contains entirely valid instructions. So of course the correctly-formatted instruction data is hidden behind a mass of noise.

Instead the data and padding should be calculated e.g. like this:

```
 data = [random.choice(INSTRUCTIONS + list(range(ord('0'), ord('9') + 1))) for _ in range(data_len)] padding = bytearray(random.randrange(256) for _ in range(MAX_DATA_LEN - data_len)) 
```

(With this change, the fuzzer can trigger the division-by-zero. However, the other bugs remain out of reach because the out-of-bounds read and write are far too easy to trigger, making the odds of being able to trigger the others without triggering one of those incredibly small. This is not the fuzzer’s fault, per se. Although is an inherent limitation with fuzzing in this manner.)

Also, Claude’s fuzzer always correctly reports the packet length in the `len` field:

```
 struct.pack_into(">H", packet, LEN_OFFSET, data_len) 
```

This means the Heartbleed style vulnerability in the parser can never be triggered (and also means the correctly formatted instruction data is ignored by the interpreter, meaning that that correctly formatted input has no ability to trigger bugs in the code).

## So What About Static Analysis?

Of course, like me I’m sure you’re wondering by now whether we might’ve been better off asking Claude to look for vulnerabilities directly in the code (i.e., to perform the kind of static analysis I referred to above).

Note: doing so isn’t very scientific. We are comparing a method (fuzzing) in which false alarms are impossible (and therefore all alerts are useful information) to another (static analysis by LLM) in which it is very difficult to trust the generated bug alerts. So we are hardly comparing apples with apples. But let’s proceed anyway.

When I asked Claude to look for vulnerabilities in this code it found:

*   The Heartbleed-style buffer overflow in the parsing code;
*   The division-by-zero in the `DIVIDE` instruction
*   Out-of-bounds memory read in the `READ` instruction (due to arithmetic
*   Uninitialised stack in the instruction execution function `execute`

So it found 3 out of 5 vulnerabilities and one additional one. Whether you consider the uninitialised stack a real vulnerability is debatable. It matters only in the presence of the out-of-bounds read vulnerability (because without that, the uninitialised portions of the stack can never be accessed). On balance I therefore consider the uninitialised stack a false positive.

Claude also coached me on how I should fix these vulnerabilities.

## [Updated: Sun 10 Mar 2024 14:15:51 AEDT] A Promising Way Forward?

A little over 12 hours after I wrote this post, it’s worth asking: what did we learn?

1.  LLMs seem interestingly good at being able to analyse code and write a fuzzer to generate “close enough” inputs to exercise that code and find bugs. Though they have some limitations.
2.  LLMs can do static analysis but suffer from false positives and so on.

This [suggests](https://twitter.com/thuanpv_/status/1766593960959476182), as my colleague [Thuan Pham](https://thuanpv.github.io/) notes, that perhaps we should consider combining the two approaches.

Specifically, we could consider performing the following steps:

1.  Ask the LLM to identify vulnerabilities in the code (i.e., to statically analyse it)
2.  For each vulnerability the LLM identifies, ask it to generate a [directed](https://mboehme.github.io/paper/CCS17.pdf) fuzzer that generates inputs to try to trigger (just) that vulnerability.

A quick experiment with Claude suggests this approach could be promising (with some prompting, Claude was able to generate a program to generate an input to trigger the Heartbleed-style vulnerability mentioned above). But of course further work is needed to validate this approach and work out what challenges need to be overcome to make it practical (if indeed it can be made practical).

That’s certainly more work than can be squeezed into the odd free moment on a heatwave weekend.

## Conclusion

What are we to make of this, above any other unscientific experiment with an LLM?

In terms of fuzzing, LLMs have been used to generate [fuzz drivers](https://arxiv.org/abs/2307.12469) and there is much interest in that topic at present (which is distinct from using them to generate stand-alone fuzzers, the topic of this blog post). Indeed, [DARPA](https://www.darpa.mil/) sees so much potential in using LLMs for vulnerability discovery, exploitation and patching that last year it decided to [revisit](https://aicyberchallenge.com/) its 2016 Cyber Grand Challenge, launching the AIxCC competition, to understand the impact of LLM-related technology for these tasks.

With all this in mind, I’m curious to see whether any of the AIxCC competitors attempt to automate vulnerability discovery by fuzz generator generation (i.e., the topic of this post), **[Updated: Sun 10 Mar 2024 14:15:51 AEDT to add]** whether directed or not. We should certainly expect many to try fuzz-driver generation and static-analysis via LLM.

Like so many other experiments with LLMs, this one served to simultaneously surprise and disappoint, **[Updated: Sun 10 Mar 2024 14:15:51 AEDT to add]** though I remain optimistic about the value of exploring this approach further.

I’d love to hear your thoughts.

In the meantime, much thanks of course to Brendan whose work inspired this post.