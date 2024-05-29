<!--yml
category: 未分类
date: 2024-05-27 14:45:56
-->

# When Random Isn't | orlp.net

> 来源：[https://orlp.net/blog/when-random-isnt/](https://orlp.net/blog/when-random-isnt/)

2024-01-10

This post is an anecdote from over a decade ago, of which I lost the actual code. So please forgive me if I do not accurately remember all the details. Some details are also simplified so that anyone that likes computer security can enjoy this article, not just those who have played World of Warcraft (although the [Venn diagram](https://en.wikipedia.org/wiki/Venn_diagram) of those two groups likely has a solid overlap).

When I was around 14 years old I discovered [World of Warcraft](https://en.wikipedia.org/wiki/World_of_Warcraft) developed by Blizzard Games and was immediately hooked. Not long after I discovered add-ons which allow you to modify how your game’s user interface looks and works. However, not all add-ons I downloaded did exactly what I wanted to do. I wanted more. So I went to find out how they were made.

In a weird twist of fate, I blame World of Warcraft for me seriously picking up programming. It turned out that they were made in the [Lua](https://www.lua.org/) programming language. Add-ons were nothing more than a couple `.lua` source files in a folder directly loaded into the game. The barrier of entry was incredibly low: just edit a file, press save and reload the interface. The fact that the game loaded *your* source code and you could see it running was magical!

I enjoyed it immensely and in no time I was only writing add-ons and was barely playing the game itself anymore. I published [quite a few add-ons](https://www.wowinterface.com/downloads/author-207710.html) in the next two years, which mostly involved copying other people’s code with some refactoring / recombining / tweaking to my wishes.

A thought you might have is that it’s a really bad idea to let users have fully programmable add-ons in your game, lest you get bots. However, the system Blizzard made to prevent arbitrary programmable actions was quite clever. Naturally, it did nothing to prevent actual botting, but at least regular rule-abiding players were fundamentally restricted to the automation Blizzard allowed.

Most UI elements that you could create were strictly decorative or informational. These were completely unrestricted, as were most APIs that strictly gather information. For example you can make a health bar display using two frames, a background and a foreground, sizing the foreground frame using an API call to get the health of your character.

Not all API calls were available to you however. Some were protected so they could only be called from official Blizzard code. These typically involved the API calls that would move your character, cast spells, use items, etc. Generally speaking anything that actually makes you perform an in-game action was protected.

However, some UI elements needed to actually interact with the game itself, e.g. if I want to make a button that casts a certain spell. For this you could construct a special kind of button that executes code in a secure environment when clicked. You were only allowed to create/destroy/move such buttons when not in combat, so you couldn’t simply conditionally place such buttons underneath your cursor to automate actions during combat.

The catch was that this [secure environment](https://wowwiki-archive.fandom.com/wiki/RestrictedEnvironment) *did* allow you to programmatically set which spell to cast, but doesn’t let you gather the information you would need to do arbitrary automation. All access to state from outside the secure environment was blocked. There were some information gathering API calls available to match the more accessible in-game macro system, but nothing as fancy as getting skill cooldowns or unit health which would enable automatic optimal spellcasting.

So there were two environments: an insecure one where you can get all information but can’t act on it, and a secure one where you can act but can’t get the information needed for automation.

Fast forward a couple years and I had mostly stopped playing. My interests had mainly moved on to more “serious” programming, and I was only occasionally playing, mostly messing around with add-on ideas. But this secure environment kept on nagging in my brain; I wanted to break it.

Of course there was third-party software that completely disables the security restrictions from Blizzard, but what’s the fun in that? I wanted to do it “legitimately”, using the technically allowed tools, as a challenge.

So I scanned the secure environment allowed function list to see if I could smuggle any information from the outside into the secure environment. It all seemed pretty hopeless until I saw one tiny, innocent little function: `random`.

An evil idea came in my head: random number generators (RNGs) used in computers are almost always [pseudorandom number generators](https://en.wikipedia.org/wiki/Pseudorandom_number_generator) with (hidden) internal state. If I can manipulate this state, perhaps I can use that to pass information into the secure environment.

It turned out that `random` was just a small shim around C’s [`rand`](https://en.cppreference.com/w/c/numeric/random/rand). I was excited! This meant that there was a single global random state that was shared in the process. It also helps that `rand` implementations tended to be on the weak side. Since World of Warcraft was compiled with MSVC, the actual implementation of `rand` was as follows:

```
uint32_t state;   int rand() {
 state = state * 214013 + 2531011;
  return (state >> 16) & 0x7fff; } 
```

This RNG is, for the lack of a better word, shite. It is a naked [linear congruential generator](https://en.wikipedia.org/wiki/Linear_congruential_generator), and a weak one at that. Which in my case, was a good thing.

So let’s get to breaking this thing. Since the state is so laughably small and you can see 15 bits of the state directly you can keep a full list of all possible states consistent with a single output of the RNG and use further calls to the RNG to eliminate possibilities until a single one remains. But we can be significantly more clever.

First we note that the top bit of `state` never affects anything in this RNG. `(state >> 16) & 0x7fff` masks out 15 bits, after shifting away the bottom 16 bits, and thus effectively works mod $2^{31}$. Since on any update the new state is a linear function of the previous state, we can propagate this modular form all the way down to the initial state as $$f(x) \equiv f(x \bmod m) \mod m$$ for any linear $f$.

Let $a = 214013$ and $b = 2531011$. We observe the 15-bit output $r_0, r_1$ of two RNG calls. We’ll call the 16-bit portion of the RNG state that is hidden by the shift $h_0, h_1$ respectively, for the states after the first and second call. This means the state of the RNG after the first call is $2^{16} r_0 + h_0$ and similarly for $2^{16} r_1 + h_1$ after the second call. Then we have the following identity:

$$a\cdot (2^{16}r_0 + h_0) + b \equiv 2^{16}r_1 + h_1 \mod 2^{31},$$

$$ah_0 \equiv h_1 + 2^{16}(r_1 - ar_0) - b \mod 2^{31}.$$

Now let $c \geq 0$ be the known constant $(2^{16}(r_1 - ar_0) - b) \bmod 2^{31}$, then for some integer $k$ we have

$$ah_0 = h_1 + c + 2^{31} k.$$

Note that the left hand side ranges from $0$ to $a (2^{16} - 1) \approx 2^{33.71}$. Thus we must have $-1 \leq k \leq 2^{2.71} < 7$. Reordering we get the following expression for $h_0$: $$h_0 = \frac{c + 2^{31} k}{a} + h_1/a.$$ Since $a > 2^{16}$ while $0 \leq h_1 < 2^{16}$ we note that the term $0 \leq h_1/a < 1$. Thus, assuming a solution exists, we must have $$h_0 = \left\lceil\frac{c + 2^{31} k}{a}\right\rceil.$$

So for $-1 \leq k < 7$ we compute the above guess for the hidden portion of the RNG state after the first call. This gives us 8 guesses, after which we can reject bad guesses using follow-up calls to the RNG until a single unique answer remains.

An example implementation of this process in Python:

```
import random   A = 214013 B = 2531011   class MsvcRng:
  def __init__(self, state):
 self.state = state   def __call__(self):
 self.state = (self.state * A + B) % 2**32
  return (self.state >> 16) & 0x7fff   # Create a random RNG state we'll reverse engineer. hidden_rng = MsvcRng(random.randint(0, 2**32))   # Compute guesses for hidden state from 2 observations. r0 = hidden_rng() r1 = hidden_rng() c = (2**16 * (r1 - A * r0) - B) % 2**31 ceil_div = lambda a, b: (a + b - 1) // b h_guesses = [ceil_div(c + 2**31 * k, A) for k in range(-1, 7)]   # Validate guesses until a single guess remains. guess_rngs = [MsvcRng(2**16 * r0 + h0) for h0 in h_guesses] guess_rngs = [g for g in guess_rngs if g() == r1] while len(guess_rngs) > 1:
 r = hidden_rng() guess_rngs = [g for g in guess_rngs if g() == r]
  # The top bit can not be recovered as it never affects the output, # but we should have recovered the effective hidden state. assert guess_rngs[0].state % 2**31 == hidden_rng.state % 2**31 
```

While I did write the above process with a `while` loop, it appears to only ever need a third output at most to narrow it down to a single guess.

Once we could reverse-engineer the internal state of the random number generator we could make arbitrary automated decisions in the supposedly secure environment. How it worked was as follows:

1.  An insecure hook was registered that would execute right before the secure environment code would run.

2.  In this hook we have full access to information, and make a decision as to which action should be taken (e.g. casting a particular spell). This action is looked up in a hardcoded list to get an index.

3.  The current state of the RNG is reverse-engineered using the above process.

4.  We predict the outcome of the next RNG call. If this (modulo the length of our action list) does not give our desired outcome, we advance the RNG and try again. This repeats until the next random number would correspond to our desired action.

5.  The hook returns, and the secure environment starts. It generates a “random” number, indexes our hardcoded list of actions, and performs the “random” action.

That’s all! By being able to simulate the RNG and looking one step ahead we could use it as our information channel by choosing exactly the right moment to call `random` in the secure environment. Now if you wanted to support a list of $n$ actions it would on average take $n$ steps of the RNG before the correct number came up to pass along, but that wasn’t a problem in practice.

I don’t know when Blizzard fixed the issue where the RNG state is so weak and shared, or whether they were aware of it being an issue at all. A few years after I had written the code I tried it again out of curiosity, and it had stopped working. Maybe they switched to a different algorithm, or had a properly separated RNG state for the secure environment.

All-in-all it was a lot of effort for a niche exploit in a video game that I didn’t even want to use. But there certainly was a magic to manipulating something supposedly random into doing exactly what you want, like a magician pulling four aces from a shuffled deck.