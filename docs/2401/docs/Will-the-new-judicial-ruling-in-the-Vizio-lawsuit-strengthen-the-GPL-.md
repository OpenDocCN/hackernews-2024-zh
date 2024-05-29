<!--yml
category: 未分类
date: 2024-05-27 14:58:58
-->

# Will the new judicial ruling in the Vizio lawsuit strengthen the GPL?

> 来源：[https://blog.tidelift.com/will-the-new-judicial-ruling-in-the-vizio-lawsuit-strengthen-the-gpl](https://blog.tidelift.com/will-the-new-judicial-ruling-in-the-vizio-lawsuit-strengthen-the-gpl)

 Last week an [important judicial ruling](https://sfconservancy.org/docs/Order_Denying_Vizio_Motion_for_Summary_Judgement_12-29-23.pdf) came down on a very intriguing case about [open source license compliance](https://tidelift.com/subscription/video/open-source-licenses). In this post, I'll talk about what makes it so interesting and potentially impactful across our industry.

# Legal background

Traditionally, open source licenses have been enforced through the law of copyright. In other words, the key question has been did copying occur, and if so, were the rights of the author violated? This has a subtle, but very important effect: only the author can initiate the lawsuit. In addition in the United States, such lawsuits must be filed in federal court, rather than in state courts, and the remedies available are primarily financial.

However, arguably, open source licenses could also be enforced through the law of contracts. Contracts can be about copyright, but in general they are a different beast. In the United States, remedies for contracts can include what is called "specific performance”—in other words, a judge can order someone who has broken a contract to do a specific act. Contracts also are typically enforced through state courts, not federal courts.

Finally, and most importantly for our discussion today, contracts can—under certain conditions—be enforced by third parties. These parties are known as third-party beneficiaries. Because they benefit from the contract, they can sometimes enforce the contract. For example, if I signed a contract with a baker to deliver a cake to my mother, my mother would be able to sue the baker if the cake did not arrive. 

## The Vizio case

In October of 2021 the Software Freedom Conservancy (SFC) decided to launch what is believed to be the first significant open source lawsuit based in contract rather than in copyright. Critically, the SFC’s case argued that anyone who benefits from the General Public License (GPL), not just the authors of the software, should be able to bring a lawsuit to enforce the terms of the GPL.

This case was brought in Orange County, California against Vizio, a large TV manufacturer. Like most TVs these days, Vizio TVs include Linux and a lot of other open source software that is under the GPL. The GPL says that buyers of those TVs should be able to get copies of that source code, so SFC walked into a store in Orange County, bought a TV, and requested copies of the source code. Vizio did not comply with the request, and so SFC brought suit.

To win their case, SFC knew that they would have to jump through several hoops. The most obvious one was the imbalance in resources—Vizio’s law firm makes about as much in a day as SFC makes from donations in a year. (You can [buy a t-shirt](https://sfconservancy.org/sustainer/) if you want to help.) But there are a lot of legal barriers too.

Under U.S. law, the first hoop was that they had to prove that the case was not a copyright case. This is because, under a doctrine known as preemption, state courts generally cannot rule on questions of federal law, like copyright. They won that in [May of 2022](https://sfconservancy.org/news/2022/may/16/vizio-remand-win/), and then the case turned to the next question.

(I realize it sounds odd to say that a case about software licenses is not a copyright case. The legal issue there is shockingly complex, so I won’t go into it much here. The short version is that, by asking for specific performance (a contract remedy) rather than financial penalties (a copyright remedy), and by claiming violations of rights granted by the contract (the license) rather than rights granted by copyright, the federal court found that this was a contract case and not a copyright case.)

## The question in last week’s ruling

Last week’s ruling was primarily about the next important question in the case, what’s known as “standing.” Courts, for fairly obvious reasons, don't like to have random people in the courtroom. So they have developed the question of standing: do you have the right to even launch the lawsuit at all? In contracts, the question of standing is usually easy: are you one of the parties named in the contract or not?

Third-party beneficiaries add a small wrinkle to this: if a third-party is named in the contract, and the parties who signed the contract intended for that third-party to benefit from the contract, then the third party might have standing.

Here, Vizio attempted to argue that the state court should end the case because SFC was not a third-party beneficiary, and so had no standing. This question is, for open source license enforcement, a truly revolutionary one. Depending on how the court answered, the tradition that only software authors could enforce open source licenses would be maintained, or the dam would break open—and anyone who buys hardware with the Linux kernel in it would have the right to sue.

It is important to note that at this stage, Vizio was (in essence) arguing that SFC’s case had “no merit” and so no reasonable person could find SFC to be a third-party beneficiary. If the court agreed, the case would then end. But if the court found reasonable people could disagree then the case would proceed.

## The court’s ruling

For such a potentially revolutionary holding, the court’s argument is quite short and to the point. First, the court rehashed the question of whether this is a contract claim or a copyright claim. Ultimately, it agreed with the federal court last May that despite being about a copyrighted work, the claims made fall into the category of contract rather than the rights and remedies of the Copyright Act.

That left the question of standing—since the California court is going to treat this like a contract, is SFC a third-party beneficiary who can bring a claim? Quoting an earlier California case, the court said that the third party must show three things:

1.  the third party would in fact benefit from the contract;
2.  a motivating purpose of the contracting parties was to provide a benefit to the third party; and
3.  permitting the third party to enforce the contract is consistent with the objectives of the contract and the reasonable expectations of the contracting parties.

Vizio “did not dispute” the first two questions, focusing instead on the “expectations” of the contracting parties. Relying on the Free Software Foundation’s (FSF) GPL FAQs, it argued that the FSF never intended for third parties to enforce the contract, and therefore the parties to the contract could not have intended it. The court was harsh here:

“much of Defendant Vizio’s argument is based on inadmissible evidence, and there is no competent evidence that suggests that the intent of FSF was to preclude recipients of the source code as beneficiaries to the GPLs.”

To divine the objective of the contract, the court used the sophisticated technique of “read it.” And so we end up with the judge quoting the GPL directly, including

“you must give the recipients all the rights that you have”

and that distributors must provide

“a written offer … to give any third party … a complete machine-readable copy of the corresponding source code”

Not surprisingly, in an argument about third party benefits, the court found that last point particularly persuasive, since it spells out the benefits that must be provided to “any third party.”

The court, then, did not mince words:

“Allowing third parties such as SFC to enforce their rights to receive source code is not only consistent with the GPLs’ objectives; it is both essential and necessary to achieve these objectives. Recipients of GPL-licensed software will be assured of their right to receive source code only if they have standing to enforce that right.” 

At this stage of a case, judges are generally inclined to insert phrases like “giving the benefit of the doubt.” Because they are talking about what reasonable people might believe, they also often say “may” and “might.” In that context, the statement that third party enforcement is “essential and necessary” jumps off the page.

The judge follows this up with an economic analysis based on California case law. Again, they find this supports SFC, saying that if only copyright holders could enforce, they “would have no economic incentive to enforce ... because they would bear all the enforcement costs with no real benefit themselves.” This is again quite strikingly strong language—no “arguably” or “maybe” to be found.

The court ultimately concludes that the question of whether SFC is a third-party beneficiary is a “triable issue of material fact,” which is lawyer-speak for “reasonable people might disagree about what happened, so we should go to trial to figure it out.”

## What is next?

The judge’s simple and straightforward ruling suggests that changing the judge’s mind on this point will be an uphill battle for Vizio. In this light, the judge’s note about how much of the evidence was “inadmissible” and not “competent” is particularly harsh. The evidence in this brief was presumably the best written evidence that extremely good lawyers could find, so Vizio will have to try other routes to show the judge what the “reasonable expectations” of the “parties” were. Since Vizio presumably relied on the best public documents they could find, it wouldn’t surprise me if they ask Linus to talk about his expectations when he put the kernel under the GPL.

Before we get there, though, SFC has asked the court to rule on whether the contract (the GPL) creates a “legal duty” for Vizio to “share source code with SFC as provided by the GPLs.” In essence, SFC is saying that (given the text of the GPL) the court already has all it needs to know about the nature of the duty, and so if there is a trial, it should be merely about the scope of the duty, not its existence. Here, the benefit of the doubt will go the other way, with Vizio only needing to prove that there are questions about that duty best settled in a full trial. I suspect the court will find that there is a duty to provide source code, but allow the trial to settle the important question of how much source code.

## Possible impacts

In some sense, not much has changed: if you were obligated to comply with the GPL two weeks ago, you have the same obligations today. If you didn’t have obligations then, you don’t have them now.

What has changed is who can enforce those obligations. Two weeks ago, we mostly believed that enforcement could only come from the authors of the code. Those folks rarely had time, money, or interest for litigation, and they might also face a lot of pressure from their peers and employers to avoid litigation.

If this ruling holds up at the end of the case, the number of potential enforcers just went way up. The limitations on financial claims will (probably) not make this a lucrative line of mass litigation, but a threat letter from the SFC or similar groups will carry much more weight. One could easily imagine other activist groups using similar arguments to free source code to systems they oppose, like surveillance systems or [locked up trains](https://arstechnica.com/tech-policy/2023/12/manufacturer-deliberately-bricked-trains-repaired-by-competitors-hackers-find/). I’ll continue to watch this case closely, but for now we should see this recent ruling as welcome news for those who believe in the importance of staying true to the original principles that led to the creation of the GPL in the first place.