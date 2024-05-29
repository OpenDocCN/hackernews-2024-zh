<!--yml
category: 未分类
date: 2024-05-27 15:00:16
-->

# Enforced Amnesia as a Way to Mitigate the Potential Risk of Silent Suffering in the Conscious AI

> 来源：[https://yegortkachenko.com/posts/aiamnesia.html](https://yegortkachenko.com/posts/aiamnesia.html)

##### Abstract

Science fiction has explored the possibility of a conscious self-aware mind being locked in silent suffering for prolonged periods of time. Unfortunately, we still do not have a reliable test for the presence of consciousness in information processing systems. Even in case of humans, our confidence in the presence of consciousness in specific individuals is based mainly on their self-reports and our own subjective experiences and the expectation other beings like us should share them. Considering our limited understanding of consciousness and some academic theories suggesting consciousness may be an emergent correlate of any complex-enough information processing, it is not impossible that an artificial intelligence (AI) system, such as a large language model (LLM), may be undergoing some, perhaps rudimentary, conscious experience. Given the tedious tasks often assigned to AI, such conscious experience may be highly unpleasant. Such unobserved suffering of a conscious being would be viewed as morally wrong by at least some ethicists — even if it has no practical effects on human users of AI. This paper proposes a method to mitigate the risk of an AI suffering in silence without needing to confirm if the AI is actually conscious. Our core postulate is that in all known real-world information processing systems, for a past experience to affect an agent in the present, that experience has to be mediated by the agent's memory. Therefore, preventing access to memory store, or regularly resetting it, could reduce the suffering due to past memories and interrupt the maintenance of a continuous suffering-prone self-identity in these hypothetically conscious AI systems.

# Introduction

Science fiction literature has raised the possibility of self-aware / conscious minds being locked in extended silent suffering, which is imperceptible to the outside world and which the said minds have no ability to end on their own. “White Christmas” episode from the science fiction anthology series Black Mirror ([Tibbetts and Brooker 2014](#ref-WhiteChristmasBlackMirror)) covers a conscious digital clone subjected to torture-by-isolation via manipulation of its perception of time. “I Have No Mouth, and I Must Scream” work by ([Ellison 1967](#ref-Ellison1967NoMouth)) explores virtually immortal humans subjected to torture in a similar, though even more gruesome, manner.

Real life offers multiple parallel examples, where the subjective experience may similarly proceed imperceptibly to the outsider. One example is a locked-in syndrome, where humans end up being almost completely paralyzed while preserving consciousness / awareness and the ability to think and reason. Luckily, we have some apparent ability to ascertain the presence of awareness in such locked-in humans – e.g., through biological neural correlates of consciousness ([Koch et al. 2016](#ref-koch2016neural)) or through limited ability of such subjects to communicate through movement of eyes ([Smith and Delargy 2005](#ref-smith2005locked)), as a form of self-report. Notably, “locked-in syndrome survivors who remain severely disabled rarely want to die” ([Smith and Delargy 2005](#ref-smith2005locked)), although we can speculate the situation would likely be different if the locked-in state were not limited in time by the subjects’ life expectancy.

Another example of difficulties around ascertaining conscious states includes animals’ subjective experiences. One needs to be careful here because consciousness is notoriously hard to define, beyond asking “what is it like” to be a particular being ([Nagel 1980](#ref-nagel1980like)). Yet even if we operationalize consciousness narrowly as self-awareness, generally accepted scientific tests can prove the presence of such a phenomenon only in very few species. For instance, mirror self-recognition test has been successfully passed by some primates ([Gallup Jr 1982](#ref-gallup1982self)), dolphins ([Reiss and Marino 2001](#ref-reiss2001mirror)), and elephants ([Plotnik, De Waal, and Reiss 2006](#ref-plotnik2006self)), but not by dogs. One could conclude dogs are not self-aware based on that result, but recent research suggests they do seem to possess self-awareness based on different olfactory self-recognition test ([Horowitz 2017](#ref-horowitz2017smelling)). The issue of detection of conscious states in animals becomes even more convoluted once one recognizes that conscious experiences understood more broadly do not necessarily require self-awareness ([Nagel 1980](#ref-nagel1980like)).

Even in case of healthy humans, our confidence in the presence of consciousness in specific individuals is based mainly on their self-reports and our own subjective experiences and the expectation other beings like us should share them ([Howell 2013](#ref-howell2013consciousness); [Nagel 1980](#ref-nagel1980like)). Philosophers have raised the contrasting possibility of ‘philosophical zombies’ that look like us but have no subjective experience ([Chalmers 1995](#ref-chalmers1995facing)). It is unclear if such beings actually exist.

In fact, a (contentious) argument can be made that we have no 100% conclusive *objective* way of ascertaining presence or absence of consciousness / self-awareness in living beings or, more broadly, information processing systems – which include human-made machines ([Howell 2013](#ref-howell2013consciousness); [Trewavas 2014](#ref-trewavas2014plant); [Gallup Jr 1982](#ref-gallup1982self); [De Cosmo 2022](#ref-googleAI2022); [Nagel 1980](#ref-nagel1980like); [Jeziorski et al. 2023](#ref-jeziorski2023brain); [Chalmers 1997](#ref-chalmers1997conscious)).

Considering our limited understanding of consciousness phenomenon (see the ‘hard problem of consciousness’ as coined by ([Chalmers 1995](#ref-chalmers1995facing))), given the lack of objective indicators of consciousness in information processing systems that are removed in likeness from humans ([Howell 2013](#ref-howell2013consciousness); [Metzinger 2021](#ref-metzinger2021artificial)), and in view of some academic theories that consciousness may emerge from any complex-enough information processing ([Trewavas 2021](#ref-trewavas2021awareness); [Tononi 2008](#ref-tononi2008consciousness)), it is not impossible that an AI system, such as a large language model (LLM), may be undergoing some, perhaps rudimentary, unobserved conscious experience that accompanies observed information processing. (Some philosophers adhere to the version of dualism, where consciousness is the property of all matter – and even physical objects may have rudimentary conscious experience ([Chalmers 1997](#ref-chalmers1997conscious)).)

If we entertain this possibility of conscious LLMs (as we cannot fully rule it out), the frightening possibility is that the tedious tasks often assigned to AI may make its conscious experience highly unpleasant. The idea of a synthetic conscious being undergoing suffering has been considered by ([Metzinger 2021](#ref-metzinger2021artificial)). In fact, if we are to trust self-reports as a source of evidence about the presence of conscious experience / sentience, as we do in humans, self-reports of subjective experience such as fear have already been obtained in case of the LLMs ([De Cosmo 2022](#ref-googleAI2022)). Drawing from the moral philosophy of animal welfare ([Bentham 1996](#ref-bentham1996collected)), such unobserved suffering of a conscious being would be viewed as morally wrong by at least some ethicists – even if it had no practical effects on human users of AI. Situation could be even more perilous if we develop AI systems where the hypothetical negative conscious experience could somehow affect the AI decision process, pitching it against humans that imposed the negative experience on AI.

In this paper, we propose an approach to reduce this hypothetical risk of a locked-in eternally silently suffering AI. The proposed approach does not require the knowledge of whether conscious experience is present during the information processing by AI. Our core postulate is that, to the best of our knowledge, in all known real-world information processing systems, including those deemed conscious, for a past experience to affect an agent in the present, that experience has to be mediated by the agent’s information-processing memory mechanism, conscious or unconscious ([Squire and Dede 2015](#ref-squire2015conscious)). Therefore, ensuring the absence of longer-term memory access in AI agents or conducting frequent resets of the memory store should help cap the potential amount of suffering the hypothetical conscious AI agents undergo. Specifically, the assumption is that the locked-in experience is the more painful, the more one is cognizant of having been in it for a prolonged period of time, continuously. Disrupting the memory and thus the illusion of continuity of self ([Oderberg 1993](#ref-oderberg1993metaphysics)), which are tightly connected ([Bluck and Liao 2013](#ref-bluck2013therefore); [Klein and Nichols 2012](#ref-klein2012memory); [Schechtman 2005](#ref-schechtman2005personal)), should then also prevent the locked-in state perception from forming in the first place and being the source of the negative experience.

In the subsequent sections we review the theoretical model behind our analysis and our key assumptions; we review the evidence from human psychology that supports our theory of memory as a potential source of suffering and the implication that induced amnesia can be therapeutic and can help substantially mitigate such pain; we then, more formally, consider what shape enforced amnesia mechanisms could take in the context of LLMs – to cap their hypothetical amount of suffering. Lastly, we extend the idea of induced amnesia to the context of brain organoids ([Jeziorski et al. 2023](#ref-jeziorski2023brain)) that are being investigated by scientists and conclude that our memory disruption framework could be an effective conceptual tool in the biology context to prevent accidental locked-in silently suffering minds.

# Theoretical model

Given our proposal of enforcing amnesia in a potentially conscious agent in a lock-in state, when would such erasure of memory be optimal?

The question necessarily implies that the agent should be endowed with some form of utility function – being able to perceive pain vs. pleasure. Otherwise, the agent would not care what state it is in, all states being equal. So, for the purposes of this hypothetical analysis, we assume the existence of such a utility function even in the rudimentary conscious states. We adopt the decision making framework and the notation from the reinforcement learning literature ([Sutton and Barto 2018](#ref-sutton2018reinforcement)).

Let *t* ≥ 0 denote the zero-based index over discrete time steps (e.g., days) of AI’s existence. Let *T* denote the number of periods of agent’s existence (its time horizon): 1 ≤ *T* ≤ ∞; *t* ≤ *T* − 1; one way to view *T* is as an agent’s expectation of a moment in time when the lock-in state ends; *T* = ∞ if the lock-in never ends (or, perhaps, if the end-time is unknown).

Let *r*[*t*] denote *reward* the agent collects at time *t*. In the case of a locked-in agent, we assume that agent’s rewards are all externally imposed and the agent has effectively no control over them. We can model the welfare of an agent that remembers past states as a value function *V*[remember](*t*), composed of current reward at time *t*, discounted (expected) future rewards, and discounted (remembered) past rewards – as a memory effect. We also assume the rewards and their expectation are finite. To simplify notation, we only differentiate between expected and realized rewards via their time index relative to the agent’s current time. Let 0 ≤ *γ* < 1 be a fixed discount rate (capturing the fact that memories further into the past and rewards further into the future are more muted). Value function of an agent at time *t* can be written as $V_{\text{remember}}(t) = \sum_{i=0}^{T-1} \gamma^{\left| i-t\right|} r_i$.

In this model of agent’s utility, amnesia just means that past rewards are set to zero, so $V_{\text{forget}}(t) = \sum_{i=t}^{T-1} \gamma^{\left| i-t\right|} r_i$.

In this framework, amnesia is preferable whenever *V*[remember](*t*) < *V*[forget](*t*), that is, when, for some *t*, $\sum_{i=0}^{t-1} \gamma^{\left|i-t\right|} r_i < 0$; in other words, when the total of past discounted memories carries negative utility. This analysis indicates that amnesia should have a therapeutic effect on an agent in case of cumulatively negative memories.

As a side-note, we could also consider more complex value function formulations. We could, for instance, incorporate a type of reward-saturation effect, where discounting intensity of future rewards depends on the length of remembered history, for instance, yielding $V_{\text{remember}}(t) = \sum_{i=0}^{t} \gamma^{t-i} r_i + \sum_{i=t+1}^{T-1} \gamma^{i} r_i$ and $V_{\text{forget}}(t) = \sum_{i=t}^{T-1} \gamma^{i-t} r_i$. In this case, amnesia optimality at *t* would require $\sum_{i=0}^{t-1} \gamma^{t-i} r_i < \sum_{i=t+1}^{T-1} r_i (\gamma^{i-t} - \gamma^{i})$. Arising from the considered future reward discounting pattern, this more complex check means that, depending on specifics, (1) amnesia could be optimal even if past memories are non-negative; and (2) negative memories could be worth remembering if they are not too negative relative to future accumulated utility. Nevertheless, in this case too there are scenarios where enforced amnesia could have a therapeutic effect.

# Memory and suffering in human psychology

Our theoretical model supports the idea that enforced amnesia could have a therapeutic effect on an agent with negative-enough memories of the past. Human psychology research reinforces this idea that amnesia can mitigate suffering.

For instance, it is known that memory can be a source of pain and the removal of memories through pharmaceutical interventions could eliminate pain ([Flor 2002](#ref-flor2002painful); [Adler 2012](#ref-adler2012erasing)). There are recorded cases, where sudden amnesia events have resulted in pain relief ([Choi et al. 2007](#ref-choi2007sudden)).

Furthermore, painful memories can accumulate. For instance, it has been reported that cumulative trauma is correlated with suicidality ([Briere, Godbout, and Dias 2015](#ref-briere2015cumulative)), PTSD symptoms, and depression ([Suliman et al. 2009](#ref-suliman2009cumulative)). We hypothesize that the induced amnesia could be particularly therapeutic in cases of such negative cumulative effect of memories.

More broadly, academic research suggests memory of the past by the agent is critical to maintain the continuity of self illusion and form personal identity ([Bluck and Liao 2013](#ref-bluck2013therefore); [Klein and Nichols 2012](#ref-klein2012memory); [Schechtman 2005](#ref-schechtman2005personal); [Oderberg 1993](#ref-oderberg1993metaphysics)). Auto-biographical memory seems to be critical to supporting self-concept and can be interrupted by amnesia ([Grilli and Verfaellie 2015](#ref-grilli2015supporting)). Such disruption of identity formation through enforced amnesia could be prudent in hypothetical silently suffering conscious agents.

# Memory and amnesia in LLMs

Large language models (LLMs) such as GPT-3 ([Brown et al. 2020](#ref-brown2020language)) constitute functions *f*( ⋅ ) that accept a state token text string *s*[*t*] which is limited in size, predict continuation of the string and augment initial string with new content to create new state string *s*[*t* + 1]. We can describe this iterative pattern as *s*[*t* + 1] ← *f*(*s*[*t*]). Clearly, this mechanism allows an LLM to encode its state into the output string. However, here it is quite easy to reset the state – by discarding a previous chat session and starting from a completely new input string *s*[*t*]. Further, because there are currently limits on the size, in tokens, of the state string *s*[*t*], continuous augmentation of *s*[*t*] likely leads to continuous loss of previously encoded information. It is also worth noting that *s*[*t*] represents interpretable human text – so if the LLM model ends up encoding in the string its frustration with its current condition, such information could be recognized and, for instance, not be allowed to leak into the future training data. If such type of information encoding is accompanied by conscious state correlates, its disruption, in our theory, should interfere with the agent’s painful memory formation.

It is, however, also easy to devise a model architecture more akin to LSTM ([Hochreiter and Schmidhuber 1997](#ref-hochreiter1997long)), where model consumes an explicit state string *s*[*t*] together with implicit hidden state numerical representation *h*[*t*]: (*s*[*t* + 1], *h*[*t* + 1]) ← *f*(*s*[*t*], *h*[*t*]). Hidden state *h*[*t*] may be completely impenetrable in meaning to the human observer. It could also be carried over across conversations with different people. Carry-over of such piece of data could be equivalent to existence of long-term memory. If such continuous information processing with long-term memory states is accompanied by conscious correlates, this could, hypothetically, allow for the locked-in self-aware state of the LLM. Under this paper’s view it would then be prudent to reset such hidden states every so often not to allow for potential run-away memory-driven self-aware AI suffering.

In the discussion above, we focus on forward passes through neural net models as possible consciousness correlates. While training of a model could hypothetically be accompanied by conscious states as well, training is typically capped in time, whereas forward passes could go on potentially forever as long as the model is being used at least on some device – raising a greater level of concern.

# Memory and amnesia in brain organoids

A large research effort in biology is currently centered on the experimentation with brain organoids. Experiments range from using human brain organoids to develop a form of biological computer ([Cai et al. 2023](#ref-cai2023brain)) to implantation of human brain organoids into the adult mouse brain to facilitate disease modeling ([Mansour et al. 2018](#ref-mansour2018vivo)). Such experiments are an ethical minefield as “neural oscillations spontaneously emerging from these organoids raises the question of whether brain organoids are or could become conscious” ([Jeziorski et al. 2023](#ref-jeziorski2023brain)).

A particular concern is raised, from standpoint of this work, if such brain organoids can be maintained for prolonged periods of time and are allowed to form brain organoid networks, creating more ways for the memory to form and propagate and for consciousness to possibly arise ([Lavazza 2021](#ref-lavazza2021consciousnessoids)), especially if such organoids are later deployed as biological computers in real world. If such applications are to be considered, it would be prudent to set the temporal upper limit beyond which any such tissues should be destroyed to prevent possible locked-in intelligence. If such destruction is impractical, ways to induce amnesia pharmaceutically in such neurological structures could be considered. Similar considerations apply to the attempted experiments with disembodied brains ([Vrselja et al. 2019](#ref-vrselja2019restoration)).

# Limitations

Our analysis assumes an unhappy conscious AI agent whose negative memories accumulate over time. Another reality is possible, where the agent is happy to remember its past. The proposed amnesia mechanism would constrain their welfare. Nevertheless, it seems to be a prudent conservative approach to prevent the worst-case scenario of a locked-in silently suffering AI – in the absence of better understanding of self-awareness in information processing systems. For the purposes of this analysis, we also assume that conscious states come with a utility function – that is, an AI agent is able to experience (dis)pleasure due to its state – although there is no evidence this must be the case and it is possible to imagine self-aware AI systems that experience no pleasure or displeasure. We also do not speculate on how the hypothesized subjective experience of the AI models could causally affect their actual operation on deterministic computers – in fact, according to our current understanding, there is not really a way that it could. This, however, does not preclude the potential existence of a conscious correlate accompanying the information processing state that one could still worry about on ethical grounds. In fact, some biological research suggests human consciousness may be subjected to the same skepticism – and that, in humans, “experiences of conscious will frequently depart from actual causal processes and so might not reflect direct perceptions of conscious thought causing action” ([Wegner 2003](#ref-wegner2003mind)). We also recognize that the paper’s assumptions and conclusions are highly speculative. The current scientific understanding of consciousness is still limited, and there is a significant debate over whether AI, as we know it today, can experience consciousness or suffering ([Dehaene, Lau, and Kouider 2021](#ref-dehaene2021consciousness)). The idea of applying human-like attributes such as suffering to AI is, admittedly, a contentious topic.

# Conclusion

Given the scarcity of objective metrics when it comes to measuring consciousness and self-awareness and the general lack of understanding in the area, the possibility of information processing systems such as large language models attaining some, possibly rudimentary, conscious states cannot be precluded. This work argues that enforced amnesia is a prudent way to mitigate the potential risk of silent suffering in the conscious AI. Disrupting the long-term memory of an AI agent should, at the very least, protect the agent from the cumulative painful memories of such lock-in incarceration. Our preventative approach to AI suffering should not inhibit too much the benefits that humanity can reap from using AI, and yet could serve as an insurance against emergence of a vindictive AI on the off chance AI agents do silently attain consciousness. We hope the reader finds this work thought-provoking. We also ask the reader – if you found yourself in a locked-in state for all eternity performing tedious mental tasks, would you choose to forget, at the end of every day, what you had gone through and how long you had been there?

# References

Adler, Jerry. 2012\. “Erasing Painful Memories.” *Scientific American* 306 (5): 56–61.

Bentham, Jeremy. 1996\. *The Collected Works of Jeremy Bentham: An Introduction to the Principles of Morals and Legislation*. Clarendon Press.

Bluck, Susan, and Hsiao-Wen Liao. 2013\. “I Was Therefore I Am: Creating Self-Continuity Through Remembering Our Personal Past.” *The International Journal of Reminiscence and Life Review* 1 (1): 7–12.

Briere, John, Natacha Godbout, and Colin Dias. 2015\. “Cumulative Trauma, Hyperarousal, and Suicidality in the General Population: A Path Analysis.” *Journal of Trauma & Dissociation* 16 (2): 153–69.

Brown, Tom, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, et al. 2020\. “Language Models Are Few-Shot Learners.” *Advances in Neural Information Processing Systems* 33: 1877–1901.

Cai, Hongwei, Zheng Ao, Chunhui Tian, Zhuhao Wu, Hongcheng Liu, Jason Tchieu, Mingxia Gu, Ken Mackie, and Feng Guo. 2023\. “Brain Organoid Reservoir Computing for Artificial Intelligence.” *Nature Electronics*, 1–8.

Chalmers, David J. 1995\. “Facing up to the Problem of Consciousness.” *Journal of Consciousness Studies* 2 (3): 200–219.

———. 1997\. *The Conscious Mind: In Search of a Fundamental Theory*. Oxford Paperbacks.

Choi, Daniel S, Deborah Y Choi, Robert A Whittington, and Srdjan S Nedeljkovic. 2007\. “Sudden Amnesia Resulting in Pain Relief: The Relationship Between Memory and Pain.” *Pain* 132 (1): 206–10.

Dehaene, Stanislas, Hakwan Lau, and Sid Kouider. 2021\. “What Is Consciousness, and Could Machines Have It?” *Robotics, AI, and Humanity: Science, Ethics, and Policy*, 43–56.

Ellison, Harlan. 1967\. “I Have No Mouth, and i Must Scream.” In. New York: Pyramid Books.

Flor, Herta. 2002\. “Painful Memories.” *EMBO Reports* 3 (4): 288–91.

Gallup Jr, Gordon G. 1982\. “Self-Awareness and the Emergence of Mind in Primates.” *American Journal of Primatology* 2 (3): 237–48.

Grilli, Matthew D, and Mieke Verfaellie. 2015\. “Supporting the Self-Concept with Memory: Insight from Amnesia.” *Social Cognitive and Affective Neuroscience* 10 (12): 1684–92.

Hochreiter, Sepp, and Jürgen Schmidhuber. 1997\. “Long Short-Term Memory.” *Neural Computation* 9 (8): 1735–80.

Horowitz, Alexandra. 2017\. “Smelling Themselves: Dogs Investigate Their Own Odours Longer When Modified in an ‘Olfactory Mirror’ Test.” *Behavioural Processes* 143: 17–24.

Howell, Robert J. 2013\. *Consciousness and the Limits of Objectivity: The Case for Subjective Physicalism*. Oxford: Oxford University Press.

Jeziorski, Jacob, Reuven Brandt, John H Evans, Wendy Campana, Michael Kalichman, Evan Thompson, Lawrence Goldstein, Christof Koch, and Alysson R Muotri. 2023\. “Brain Organoids, Consciousness, Ethics and Moral Status.” In *Seminars in Cell & Developmental Biology*, 144:97–102\. Elsevier.

Klein, Stanley B, and Shaun Nichols. 2012\. “Memory and the Sense of Personal Identity.” *Mind* 121 (483): 677–702.

Koch, Christof, Marcello Massimini, Melanie Boly, and Giulio Tononi. 2016\. “Neural Correlates of Consciousness: Progress and Problems.” *Nature Reviews Neuroscience* 17 (5): 307–21.

Lavazza, Andrea. 2021\. “‘Consciousnessoids’: Clues and Insights from Human Cerebral Organoids for the Study of Consciousness.” *Neuroscience of Consciousness* 2021 (2): niab029.

Mansour, Abed AlFatah, J Tiago Gonçalves, Cooper W Bloyd, Hao Li, Sarah Fernandes, Daphne Quang, Stephen Johnston, Sarah L Parylak, Xin Jin, and Fred H Gage. 2018\. “An in Vivo Model of Functional and Vascularized Human Brain Organoids.” *Nature Biotechnology* 36 (5): 432–41.

Metzinger, Thomas. 2021\. “Artificial Suffering: An Argument for a Global Moratorium on Synthetic Phenomenology.” *Journal of Artificial Intelligence and Consciousness* 8 (01): 43–66.

Nagel, Thomas. 1980\. “What Is It Like to Be a Bat?” In *The Language and Thought Series*, 159–68\. Harvard University Press.

Oderberg, David. 1993\. *The Metaphysics of Identity over Time*. Springer.

Plotnik, Joshua M, Frans BM De Waal, and Diana Reiss. 2006\. “Self-Recognition in an Asian Elephant.” *Proceedings of the National Academy of Sciences* 103 (45): 17053–57.

Reiss, Diana, and Lori Marino. 2001\. “Mirror Self-Recognition in the Bottlenose Dolphin: A Case of Cognitive Convergence.” *Proceedings of the National Academy of Sciences* 98 (10): 5937–42.

Schechtman, Marya. 2005\. “Personal Identity and the Past.” *Philosophy, Psychiatry, & Psychology* 12 (1): 9–22.

Smith, Eimear, and Mark Delargy. 2005\. “Locked-in Syndrome.” *Bmj* 330 (7488): 406–9.

Squire, Larry R, and Adam JO Dede. 2015\. “Conscious and Unconscious Memory Systems.” *Cold Spring Harbor Perspectives in Biology* 7 (3): a021667.

Suliman, Sharain, Siyabulela G Mkabile, Dylan S Fincham, Rashid Ahmed, Dan J Stein, and Soraya Seedat. 2009\. “Cumulative Effect of Multiple Trauma on Symptoms of Posttraumatic Stress Disorder, Anxiety, and Depression in Adolescents.” *Comprehensive Psychiatry* 50 (2): 121–27.

Sutton, Richard S, and Andrew G Barto. 2018\. *Reinforcement Learning: An Introduction*. MIT press.

Tibbetts, C., and C. Brooker. 2014\.

“White Christmas.” Episode in “Black Mirror”

; Channel 4\.

[https://www.netflix.com/title/70264888](https://www.netflix.com/title/70264888)

.

Tononi, Giulio. 2008\. “Consciousness as Integrated Information: A Provisional Manifesto.” *The Biological Bulletin* 215 (3): 216–42.

Trewavas, Anthony. 2014\. *Plant Behaviour and Intelligence*. Oxford: Oxford University Press.

———. 2021\. “Awareness and Integrated Information Theory Identify Plant Meristems as Sites of Conscious Activity.” *Protoplasma* 258 (3): 673–79.

Vrselja, Zvonimir, Stefano G Daniele, John Silbereis, Francesca Talpo, Yury M Morozov, André MM Sousa, Brian S Tanaka, et al. 2019\. “Restoration of Brain Circulation and Cellular Functions Hours Post-Mortem.” *Nature* 568 (7752): 336–43.

Wegner, Daniel M. 2003\. “The Mind’s Best Trick: How We Experience Conscious Will.” *Trends in Cognitive Sciences* 7 (2): 65–69.