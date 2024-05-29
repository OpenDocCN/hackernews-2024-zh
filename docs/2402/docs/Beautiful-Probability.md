<!--yml
category: 未分类
date: 2024-05-29 13:19:03
-->

# Beautiful Probability

> 来源：[https://www.readthesequences.com/Beautiful-Probability](https://www.readthesequences.com/Beautiful-Probability)

# Beautiful Probability

❦

Should we expect rationality to be, *on some level*, simple? Should we search and hope for *underlying* beauty in the arts of belief and choice?

Let me introduce this issue by borrowing a complaint of the late great Bayesian Master, E. T. Jaynes:[1](#footnote1)

> Two medical researchers use the same treatment independently, in different hospitals. Neither would stoop to falsifying the data, but one had decided beforehand that because of finite resources he would stop after treating *n* = 100 patients, however many cures were observed by then. The other had staked his reputation on the efficacy of the treatment, and decided he would not stop until he had data indicating a rate of cures definitely greater than 60%, however many patients that might require. But in fact, both stopped with exactly the same data: *n* = 100 [patients], *r* = 70 [cures]. Should we then draw different conclusions from their experiments?” [Presumably the two control groups also had equal results.]

[Cyan](https://www.greaterwrong.com/lw/mt/beautiful_probability/hnz) directs us to [chapter 37](https://web.archive.org/web/20081116034128/http://www.inference.phy.cam.ac.uk/mackay/itprnn/ps/457.466.pdf) of MacKay’s [excellent statistics book](https://web.archive.org/web/20080823053541/http://www.inference.phy.cam.ac.uk/mackay/itila/book.html), free online, for a more thorough explanation of this problem.[2](#footnote2)

According to old-fashioned statistical procedure—which I believe is still being taught today—the two researchers have performed different experiments with different stopping conditions. The two experiments *could* have terminated with different data, and therefore represent different tests of the hypothesis, requiring different statistical analyses. It’s quite possible that the first experiment will be “statistically significant,” the second not.

Whether or not you are disturbed by this says a good deal about your attitude toward probability theory, and indeed, rationality itself.

Non-Bayesian statisticians might shrug, saying, “Well, not all statistical tools have the same strengths and weaknesses, y’know—a hammer isn’t like a screwdriver—and if you apply different statistical tools you may get different results, just like using the same data to compute a linear regression or train a regularized neural network. You’ve got to use the right tool for the occasion. Life is messy—”

And then there’s the Bayesian reply: “Excuse *you*? The evidential impact of a fixed experimental method, producing the same data, depends on the researcher’s private thoughts? And you have the nerve to accuse *us* of being ‘too subjective’?”

If Nature is one way, the likelihood of the data coming out the way we have seen will be one thing. If Nature is another way, the likelihood of the data coming out that way will be something else. But the likelihood of a given state of Nature producing the data we have seen, has nothing to do with the researcher’s private intentions. So whatever our hypotheses about Nature, the likelihood ratio is the same, and the evidential impact is the same, and the posterior belief should be the same, between the two experiments. At least one of the two Old Style methods must discard relevant information—or simply do the wrong calculation—for the two methods to arrive at different answers.

The ancient war between the Bayesians and the accursèd frequentists stretches back through decades, and I’m not going to try to recount that elder history in this essay.

But one of the central conflicts is that Bayesians expect probability theory to be… what’s the word I’m looking for? “Neat?” “Clean?” “Self-consistent?”

As Jaynes says, the theorems of Bayesian probability are just that, *theorems* in a coherent proof system. No matter what derivations you use, in what order, the results of Bayesian probability theory should always be consistent—every theorem compatible with every other theorem.

If you want to know the sum 10+10, you can redefine it as (2 × 5) + (7 + 3) or as (2 × (4 + 6)) or use whatever other *legal* tricks you like, but the result always has to come out to be the same, in this case, 20\. If it comes out as 20 one way and 19 the other way, then you may conclude you did something illegal on at least one of the two occasions. (In arithmetic, the illegal operation is usually division by zero; in probability theory, it is usually an infinity that was not taken as the limit of a finite process.)

If you get the result 19 = 20, look hard for that error you just made, because it’s unlikely that you’ve sent arithmetic itself up in smoke. If anyone should ever succeed in deriving a *real* contradiction from Bayesian probability theory—like, say, two different evidential impacts from the same experimental method yielding the same results—then the whole edifice goes up in smoke. Along with set theory, ’cause I’m pretty sure ZF provides a model for probability theory.

Math! That’s the word I was looking for. Bayesians expect probability theory to be *math*. That’s why we’re interested in Cox’s Theorem and its many extensions, showing that any representation of uncertainty which obeys certain constraints has to map onto probability theory. Coherent math is great, but unique math is even better.

And yet… *should* rationality be math? It is by no means a foregone conclusion that probability should be pretty. The real world is messy—so shouldn’t you need messy reasoning to handle it? Maybe the non-Bayesian statisticians, with their vast collection of ad-hoc methods and ad-hoc justifications, are strictly more competent because they have a strictly larger toolbox. It’s nice when problems are clean, but they usually aren’t, and you have to live with that.

After all, it’s a well-known fact that you can’t use Bayesian methods on many problems because the Bayesian calculation is [computationally intractable](https://www.readthesequences.com/IsRealityUgly). So why not let many flowers bloom? Why not have more than one tool in your toolbox?

*That’s* the fundamental difference in mindset. Old School statisticians thought in terms of *tools*, tricks to throw at particular problems. Bayesians—at least this Bayesian, though I don’t think I’m speaking only for myself—we think in terms of *laws*.

Looking for laws isn’t the same as looking for especially neat and pretty tools. The Second Law of Thermodynamics isn’t an especially neat and pretty refrigerator.

The Carnot cycle is an ideal engine—in fact, *the* ideal engine. No engine powered by two heat reservoirs can be more efficient than a Carnot engine. As a corollary, all thermodynamically reversible engines operating between the same heat reservoirs are equally efficient.

But, of course, you can’t use a Carnot engine to power a real car. A real car’s engine bears the same resemblance to a Carnot engine that the car’s tires bear to perfect rolling cylinders.

Clearly, then, a Carnot engine is a useless *tool* for building a real-world car. The Second Law of Thermodynamics, obviously, is not applicable here. It’s too hard to make an engine that obeys it, in the real world. Just ignore thermodynamics—use whatever works.

This is the sort of confusion that I think reigns over they who still cling to the Old Ways.

No, you can’t always do the exact Bayesian calculation for a problem. Sometimes you must seek an approximation; often, indeed. This doesn’t mean that probability theory has ceased to apply, any more than your inability to calculate the aerodynamics of a 747 on an atom-by-atom basis implies that the 747 is not made out of atoms. Whatever approximation you use, it works to the extent that it approximates the ideal Bayesian calculation—and fails to the extent that it departs.

Bayesianism’s coherence and uniqueness proofs cut both ways. Just as any calculation that obeys Cox’s coherency axioms (or any of the many reformulations and generalizations) must map onto probabilities, so too, anything that is not Bayesian must fail one of the coherency tests. This, in turn, opens you to punishments like Dutch-booking (accepting combinations of bets that are sure losses, or rejecting combinations of bets that are sure gains).

You may not be able to compute the optimal answer. But whatever approximation you use, both its failures and successes will be *explainable* in terms of Bayesian probability theory. You may not know the explanation; that does not mean no explanation exists.

So you want to use a linear regression, instead of doing Bayesian updates? But look to the underlying structure of the linear regression, and you see that it corresponds to picking the best point estimate given a Gaussian likelihood function and a uniform prior over the parameters.

You want to use a regularized linear regression, because that works better in practice? Well, that corresponds (says the Bayesian) to having a Gaussian prior over the weights.

Sometimes you can’t use Bayesian methods *literally*; often, indeed. But when you *can* use the exact Bayesian calculation that uses every scrap of available knowledge, you are done. You will never find a statistical method that yields a *better* answer. You may find a cheap approximation that works excellently nearly all the time, and it will be cheaper, but it will not be more accurate. Not unless the other method uses knowledge, perhaps in the form of disguised prior information, that you are not allowing into the Bayesian calculation; and then when you feed the prior information into the Bayesian calculation, the Bayesian calculation will again be equal or superior.

When you use an Old Style ad-hoc statistical tool with an ad-hoc (but often quite interesting) justification, you never know if someone else will come up with an even more clever tool tomorrow. But when you *can* directly use a calculation that mirrors the Bayesian law, you’re *done*—like managing to put a Carnot heat engine into your car. It is, as the saying goes, “Bayes-optimal.”

It seems to me that the toolboxers are looking at the sequence of cubes { 1, 8, 27, 64, 125, ... } and pointing to the first differences { 7, 19, 37, 61, ... } and saying “Look, life isn’t always so neat—you’ve got to adapt to circumstances.” And the Bayesians are pointing to the third differences, the underlying stable level { 6, 6, 6, 6, 6, … }. And the critics are saying, “What the heck are you talking about? It’s 7, 19, 37 not 6, 6, 6\. You are oversimplifying this messy problem; you are too attached to simplicity.”

It’s not necessarily simple on a *surface* level. You have to dive deeper than that to find stability.

Think laws, not tools. Needing to calculate approximations to a law doesn’t change the law. Planes are still atoms, they aren’t governed by special exceptions in Nature for aerodynamic calculations. The approximation exists in the map, not in the territory. You can know the Second Law of Thermodynamics, and yet apply yourself as an engineer to build an imperfect car engine. The Second Law does not cease to be applicable; your knowledge of that law, and of Carnot cycles, helps you get as close to the ideal efficiency as you can.

We aren’t enchanted by Bayesian methods merely because they’re beautiful. The beauty is a side effect. Bayesian *theorems* are elegant, coherent, optimal, and provably unique because they are *laws*.