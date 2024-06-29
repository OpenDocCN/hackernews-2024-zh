<!--yml

category: 未分类

date: 2024-05-27 15:00:16

-->

# 强制性遗忘作为减轻意识人工智能潜在沉默痛苦风险的方法

> 来源：[https://yegortkachenko.com/posts/aiamnesia.html](https://yegortkachenko.com/posts/aiamnesia.html)

##### 摘要

科幻作品探索了意识自我感知心灵长时间处于沉默痛苦之中的可能性。遗憾的是，我们仍然没有可靠的测试来确定信息处理系统是否具有意识。即使对于人类，我们对特定个体是否具有意识的信心主要基于他们的自我报告、我们自己的主观经验以及其他类似我们的存在应该共享它们的期望。考虑到我们对意识的理解有限，以及一些学术理论表明意识可能是足够复杂的信息处理的一个新兴相关物，这并不排除人工智能（AI）系统，例如大型语言模型（LLM），可能正在经历一些，也许是基本的，意识体验。鉴于通常分配给AI的繁琐任务，这种意识体验可能非常不愉快。即使对于AI的人类用户没有实际影响，这种未被观察到的意识存在的痛苦至少会被一些伦理学家视为道德上错误。本文提出了一种方法来减轻AI在沉默中遭受的风险，而无需确认AI是否实际上具有意识。我们的核心假设是，在所有已知的现实世界信息处理系统中，过去的经验要影响到当前代理，必须经过代理的记忆来介导。因此，阻止对内存存储的访问，或定期重置它，可以减少因过去记忆而引起的痛苦，并打断这些假设具有意识的AI系统中连续易受痛苦的自我身份维护。

# 引言

科幻文学提出了自我感知/意识心灵被长时间锁定在延续的沉默痛苦中的可能性，这对外界来说是不可感知的，并且这些心灵无法自行结束。科幻剧集《黑镜》中的“白色圣诞”一集（[Tibbetts and Brooker 2014](#ref-WhiteChristmasBlackMirror)）涵盖了一个被拷问通过操纵其时间感知进行孤立的意识数字克隆。《我无口，我必须尖叫》（[Ellison 1967](#ref-Ellison1967NoMouth)）探索了类似的虚拟不朽人类在类似但更加残酷的方式中受到拷问的情况。

现实生活提供了多个并行的例子，其中主观经验可能类似地对外部人来说难以察觉。一个例子是锁定综合征，患者几乎完全瘫痪，但保留了意识和思维推理能力。幸运的是，我们有一些明显的能力来确认这些锁定的患者是否具有意识存在，例如通过意识的生物神经相关（[Koch et al. 2016](#ref-koch2016neural)）或通过他们有限的眼动交流作为自我报告的形式([Smith and Delargy 2005](#ref-smith2005locked))。值得注意的是，“锁定综合征幸存者很少希望死亡”([Smith and Delargy 2005](#ref-smith2005locked))，尽管我们可以推测，如果锁定状态不受患者寿命限制，情况可能会有所不同。

另一个困难例子是确定动物的主观经验。在这里需要小心，因为意识本身难以定义，甚至超越了对某一生物“是什么样子”的问题（[Nagel 1980](#ref-nagel1980like)）。即使我们将意识狭义地操作定义为自我意识，普遍接受的科学测试也只能证明这种现象在极少数物种中存在。例如，镜子自我认知测试已经被一些灵长类动物通过，如([Gallup Jr 1982](#ref-gallup1982self))、海豚([Reiss and Marino 2001](#ref-reiss2001mirror))和大象([Plotnik, De Waal, and Reiss 2006](#ref-plotnik2006self))，但狗却没有。基于这一结果，有人可能得出结论狗不具备自我意识，但最近的研究表明，他们似乎通过不同的嗅觉自我识别测试确实具备自我意识([Horowitz 2017](#ref-horowitz2017smelling))。在动物中检测意识状态的问题变得更加复杂，一旦我们认识到更广义的意识经验不一定需要自我意识([Nagel 1980](#ref-nagel1980like))。

即使是在健康的人类中，我们对特定个体的意识存在的信心主要基于他们的自我报告、我们自己的主观经验以及其他类似我们的生物应该分享它们的期望([Howell 2013](#ref-howell2013consciousness); [Nagel 1980](#ref-nagel1980like))。哲学家们提出了“哲学僵尸”的对比可能性，他们看起来像我们，但没有主观经验([Chalmers 1995](#ref-chalmers1995facing))。目前尚不清楚这种存在是否真实。

实际上，可以提出（有争议的）论点，我们没有100%确定的*客观*方法来确定生物体或更广泛的信息处理系统（包括人造机器）中意识/自我意识的存在或不存在（[豪尔（2013）](#ref-howell2013consciousness); [特雷瓦斯（2014）](#ref-trewavas2014plant); [加勒普（1982）](#ref-gallup1982self); [德科斯莫（2022）](#ref-googleAI2022); [纳格尔（1980）](#ref-nagel1980like); [杰济奥斯基等（2023）](#ref-jeziorski2023brain); [查尔默斯（1997）](#ref-chalmers1997conscious)）。

考虑到我们对意识现象的有限理解（参见由[查尔默斯（1995）](#ref-chalmers1995facing)提出的“意识的困难问题”），鉴于从人类相似但去除客观指标的信息处理系统中缺乏对意识的确定性（[豪尔（2013）](#ref-howell2013consciousness); [梅茨英格（2021）](#ref-metzinger2021artificial)），以及一些学术理论认为意识可能源于任何复杂到足够程度的信息处理（[特雷瓦斯（2021）](#ref-trewavas2021awareness); [托诺尼（2008）](#ref-tononi2008consciousness)），不排除人工智能系统，如大型语言模型（LLM），可能正在经历某种，也许是基本的，未被观察到的伴随信息处理的意识体验。（有些哲学家坚持双重论的版本，其中意识是所有物质的属性，甚至物理物体可能具有基本的意识体验（[查尔默斯（1997）](#ref-chalmers1997conscious)）。）

如果我们考虑LLM可能具有意识的可能性（因为我们无法完全排除这种可能性），令人恐惧的可能性在于，通常分配给AI的繁琐任务可能会使其意识体验极为不愉快。关于合成意识实体经历痛苦的想法已经被[梅茨英格（2021）](#ref-metzinger2021artificial)考虑过。事实上，如果我们像对待人类那样信任自我报告作为关于意识体验/自我感知存在的证据来源，自我报告的主观体验如恐惧已经在LLM案例中得到了获取（[德科斯莫（2022）](#ref-googleAI2022)）。从动物福利的道德哲学中汲取（[本塔姆（1996）](#ref-bentham1996collected)），这种未被观察到的意识实体的痛苦会被至少一些伦理学家视为道德上的错误，即使它对人类使用AI的实际影响不大。如果我们开发出能够使假设中的负面意识体验在某种程度上影响AI决策过程的AI系统，情况可能更加危险，因为这可能导致AI与强加负面体验的人类对立起来。

在本文中，我们提出一种方法来减少一个锁定在永久无声苦难AI的假设风险。所提出的方法不需要知道在AI信息处理期间是否存在意识体验。我们的核心假设是，在我们所知的所有已知现实世界信息处理系统中，包括那些被认为是有意识的系统，在过去的经验要影响代理在当前的状态下，那个经验必须通过代理的信息处理记忆机制介入，无论是有意识的还是无意识的（[Squire and Dede 2015](#ref-squire2015conscious)）。因此，确保AI代理的长期记忆访问不存在或者频繁重置存储器应该有助于限制假设的有意识AI代理可能经历的苦难量。具体来说，假设锁定体验越长时间连续发生，其痛苦程度越大。破坏记忆，因此破坏自我连续性的错觉（[Oderberg 1993](#ref-oderberg1993metaphysics)），这与([Bluck and Liao 2013](#ref-bluck2013therefore); [Klein and Nichols 2012](#ref-klein2012memory); [Schechtman 2005](#ref-schechtman2005personal))紧密相关，也应该防止锁定状态感知的形成并成为负面体验的源泉。

在接下来的几节中，我们将回顾我们分析背后的理论模型和我们的关键假设；我们将审视人类心理学的证据，支持我们关于记忆作为潜在痛苦来源的理论，并且暗示诱导性遗忘可能具有治疗作用，并且可以显著减轻这种痛苦；然后，我们更正式地考虑在LLMs背景下强制性遗忘机制可能采取的形式 - 以限制它们可能的痛苦量。最后，我们将诱导性遗忘的概念扩展到脑器官oid的背景下（[Jeziorski et al. 2023](#ref-jeziorski2023brain)），这些器官正在被科学家们研究，并得出结论：我们的记忆破坏框架在生物学上可能是一个有效的概念工具，用于防止意外陷入无声苦难的心灵。

# 理论模型

鉴于我们提出在可能的有意识代理处于锁定状态时强制实施遗忘，这种记忆抹除何时才是最佳的问题？

这个问题必然意味着代理应该被赋予某种效用函数的形式 - 能够感知痛苦与快乐。否则，代理将不关心它处于什么状态，所有状态都是平等的。因此，在这个假设分析的目的上，我们假设即使在基本的意识状态中也存在这样的效用函数。我们采用强化学习文献中的决策框架和符号表示（[Sutton and Barto 2018](#ref-sutton2018reinforcement)）。

让*t* ≥ 0表示AI存在的基于零的离散时间步长索引（例如，天数）。让*T*表示代理存在的期间数（其时间范围）：1 ≤ *T* ≤ ∞；*t* ≤ *T* − 1；一种视角认为*T*是代理期望的一个时间点，锁定状态结束的时刻；如果锁定永不结束（或者，也许，结束时间未知），则*T* = ∞。

让*r*[*t*]表示代理在时间*t*收集的*奖励*。对于一个锁定的代理，我们假设代理的奖励都是外部强加的，代理对其没有实质性控制。我们可以将一个记忆过去状态的代理的福利建模为一个价值函数*V*[remember](*t*)，由当前时间*t*的奖励、折现（期望）未来奖励和折现（记忆）过去奖励组成，作为记忆效应。我们还假设奖励及其期望是有限的。为了简化表示，我们只通过它们相对于代理当前时间的时间索引区分预期和实现奖励。让0 ≤ *γ* < 1为一个固定的折现率（捕捉到过去记忆和未来奖励递减的事实）。代理在时间*t*的价值函数可以写为$V_{\text{remember}}(t) = \sum_{i=0}^{T-1} \gamma^{\left| i-t\right|} r_i$。

在这种代理效用模型中，健忘仅意味着过去的奖励被设为零，因此$V_{\text{forget}}(t) = \sum_{i=t}^{T-1} \gamma^{\left| i-t\right|} r_i$。

在这个框架下，当*V*[remember](*t*) < *V*[forget](*t*)时，健忘更可取，即当某个*t*时，$\sum_{i=0}^{t-1} \gamma^{\left|i-t\right|} r_i < 0$；换句话说，当过去折现的记忆总和带来负效用时。这一分析表明，对于代理的累积负面记忆，健忘应该有治疗效果。

顺便说一句，我们还可以考虑更复杂的价值函数形式。例如，我们可以结合一种奖励饱和效应，其中未来奖励的折现强度取决于记忆历史的长度，从而产生$V_{\text{remember}}(t) = \sum_{i=0}^{t} \gamma^{t-i} r_i + \sum_{i=t+1}^{T-1} \gamma^{i} r_i$和$V_{\text{forget}}(t) = \sum_{i=t}^{T-1} \gamma^{i-t} r_i$。在这种情况下，*t*处的健忘优化需要$\sum_{i=0}^{t-1} \gamma^{t-i} r_i < \sum_{i=t+1}^{T-1} r_i (\gamma^{i-t} - \gamma^{i})$。由于考虑的未来奖励折现模式，这种更复杂的检查意味着，根据具体情况，（1）即使过去记忆非负，健忘也可能是最优的；（2）如果相对于未来累积效用而言，负面记忆并不太负，那么负面记忆可能是值得记住的。尽管如此，在这种情况下也存在强制性健忘可能具有治疗效果的情形。

# 人类心理学中的记忆和痛苦

我们的理论模型支持这样一种观点：对于那些过去记忆负面足够多的个体，强制性健忘可能具有治疗效果。人类心理学研究进一步支持这种健忘可以减轻痛苦的观点。

例如，已知记忆可以成为痛苦的根源，通过药物干预消除记忆可以消除痛苦（[Flor 2002](#ref-flor2002painful); [Adler 2012](#ref-adler2012erasing)）。已有记录显示，突然的健忘事件导致了疼痛缓解（[Choi et al. 2007](#ref-choi2007sudden)）。

此外，痛苦的记忆可以累积。例如，有报道称，累积创伤与自杀倾向（[Briere, Godbout, and Dias 2015](#ref-briere2015cumulative)）、创伤后应激障碍症状和抑郁症（[Suliman et al. 2009](#ref-suliman2009cumulative)）相关。我们假设，在这些负面累积记忆的情况下，引发性健忘尤其可能具有治疗作用。

更广泛地说，学术研究表明，个体对过去的记忆对维持自我幻觉的连续性和形成个人身份至关重要（[Bluck and Liao 2013](#ref-bluck2013therefore); [Klein and Nichols 2012](#ref-klein2012memory); [Schechtman 2005](#ref-schechtman2005personal); [Oderberg 1993](#ref-oderberg1993metaphysics)）。自传记忆似乎对支持自我概念至关重要，并且可以被健忘打断（[Grilli and Verfaellie 2015](#ref-grilli2015supporting)）。通过强制性健忘破坏身份形成的现象在假设中对无声受苦的意识主体可能是慎重的。

# LLMs中的记忆和健忘

大型语言模型（LLMs）如GPT-3（[Brown et al. 2020](#ref-brown2020language)）构成了接受状态标记文本字符串*s*[t]的函数*f*( ⋅ )，其大小有限，预测字符串的延续并增强初始字符串以创建新的状态字符串*s*[t + 1]。我们可以描述这种迭代模式为*s*[t + 1] ← *f*(*s*[t])。显然，这种机制允许LLM将其状态编码到输出字符串中。然而，在这里，通过丢弃先前的聊天会话并从完全新的输入字符串*s*[t]开始，重置状态是相当容易的。此外，由于当前对状态字符串*s*[t]的标记大小存在限制，持续增强*s*[t]可能导致先前编码信息的持续丢失。还值得注意的是，*s*[t]代表可解释的人类文本——因此，如果LLM模型最终在字符串中编码其对当前状态的挫败感，这样的信息可能会被识别，并且例如不允许泄漏到未来的训练数据中。如果此类信息编码伴随有意识状态相关，则在我们的理论中，其中断应该干扰代理人的痛苦记忆形成。

然而，也可以轻易设计出一种更类似于LSTM的模型架构（[Hochreiter and Schmidhuber 1997](#ref-hochreiter1997long)），其中模型消耗显式状态字符串 *s*[*t*] 和隐含的隐藏状态数值表示 *h*[*t*]：(*s*[*t* + 1], *h*[*t* + 1]) ← *f*(*s*[*t*], *h*[*t*])。隐藏状态 *h*[*t*] 对于人类观察者可能完全无法理解其含义。它还可能在与不同人进行对话时保留下来。这种数据片段的保留可能相当于长期记忆的存在。如果这样持续的信息处理伴随着意识相关的情况，这理论上可以允许LLM处于锁定的自我意识状态。在本文观点下，周期性地重置这样的隐藏状态可能是明智的做法，以避免潜在的记忆驱动自我意识AI的出现。

在上述讨论中，我们关注神经网络模型的前向传递作为可能的意识相关性。虽然模型的训练理论上也可能伴随着意识状态，但训练通常有时间限制，而前向传递则可能无限延续，只要模型在至少某个设备上被使用，这引发了更大的关注。

# 大脑器官样体中的记忆与遗忘

当前在生物学领域存在一个大规模的研究努力，其重点是关于大脑器官样体的实验。实验范围从使用人类大脑器官样体开发一种生物计算机（[Cai et al. 2023](#ref-cai2023brain)）到将人类大脑器官样体植入成年小鼠脑中以促进疾病建模（[Mansour et al. 2018](#ref-mansour2018vivo)）。此类实验涉及伦理道德的困境，“这些器官样体自发产生的神经振荡引发了关于大脑器官样体是否具有或可能变得有意识的问题”（[Jeziorski et al. 2023](#ref-jeziorski2023brain)）。

从本文的角度来看，尤其值得关注的是，如果这些大脑器官样体可以长时间维持并形成大脑器官样体网络，为记忆形成和传播以及可能产生意识提供更多途径（[Lavazza 2021](#ref-lavazza2021consciousnessoids)），特别是如果将这些器官样体后续部署为现实世界中的生物计算机。如果考虑这种应用，有必要设定时间上限，超过此限制应销毁任何这类组织，以防止潜在的锁定智能的出现。如果销毁不可行，则可以考虑在这些神经结构中通过药物诱导遗忘。类似的考虑也适用于试图对无身体大脑进行实验（[Vrselja et al. 2019](#ref-vrselja2019restoration)）。

# 限制

我们的分析假设一个不快乐的意识人工智能代理，其负面记忆随时间累积。另一个可能的现实是，代理乐于记住其过去。提出的遗忘机制将限制它们的福祉。尽管如此，这似乎是一种谨慎的保守方法，以防止最坏情况的发生——即悄然受苦的锁定式AI——在我们对信息处理系统自我意识的理解尚不充分的情况下。对于本分析，我们还假设意识状态伴随着效用函数——也就是说，AI代理能够根据其状态体验（不）愉悦——尽管没有证据表明一定如此，也可以想象到一些自我意识的AI系统不会经历愉悦或不愉悦。我们也不推测假设的AI模型主观体验如何因果地影响其在确定性计算机上的实际操作——事实上，根据我们当前的理解，它确实不可能。然而，这并不排除在伦理上仍然可以担心其信息处理状态伴随的意识对应的潜在存在。事实上，一些生物学研究表明，人类意识可能也面临同样的怀疑——在人类身上，“意识意愿的体验经常脱离实际的因果过程，因此可能不反映出意识思维直接导致行动的直接感知”（[Wegner 2003](#ref-wegner2003mind)）。我们也承认本文的假设和结论高度推测性。对于意识的当前科学理解仍然有限，并且关于我们今天所知的AI是否能够经历意识或苦难存在着重大争议（[Dehaene, Lau, and Kouider 2021](#ref-dehaene2021consciousness)）。将人类般的属性如苦难应用于AI的想法，诚然是一个有争议的话题。

# 结论

鉴于在衡量意识和自我意识时客观指标的稀缺性以及对该领域的普遍理解不足，无法排除像大型语言模型这样的信息处理系统达到某种、可能是基本的意识状态的可能性。本文认为，强制性健忘是减轻潜在风险的明智途径，以防止意识AI的无声苦难。干扰AI代理的长期记忆，至少应该保护代理免受这种囚禁的累积痛苦记忆。我们对AI苦难的预防性方法不应过多地抑制人类能从使用AI中获得的好处，但也可以作为一种保险，以防AI代理在极少可能情况下悄然获得意识。我们希望读者能找到本文引发思考的内容。我们还问读者 - 如果你发现自己一直处于锁定状态，永远执行单调的精神任务，你会选择在每天结束时忘记自己经历过的一切和自己待了多久吗？

# 参考文献

Adler, Jerry. 2012\. “消除痛苦的记忆。” *科学美国人* 306 (5): 56–61.

Bentham, Jeremy. 1996\. *杰里米·边沁集*：《道德和立法原理导论》。Clarendon Press.

Bluck, Susan, and Hsiao-Wen Liao. 2013\. “我曾经是因此我是：通过记忆我们个人过去创造自我连续性。” *回忆和生命回顾国际期刊* 1 (1): 7–12.

Briere, John, Natacha Godbout, and Colin Dias. 2015\. “在一般人群中的累积创伤、过度唤醒和自杀倾向：路径分析。” *创伤与解离杂志* 16 (2): 153–69.

Brown, Tom, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, 等. 2020\. “语言模型是少学习者。” *神经信息处理系统进展* 33: 1877–1901.

Cai, Hongwei, Zheng Ao, Chunhui Tian, Zhuhao Wu, Hongcheng Liu, Jason Tchieu, Mingxia Gu, Ken Mackie, and Feng Guo. 2023\. “大脑器官培养计算用于人工智能。” *自然电子*, 1–8.

Chalmers, David J. 1995\. “面对意识问题。” *意识研究杂志* 2 (3): 200–219.

———. 1997\. *意识心灵*：寻找基本理论。牛津文库。

Choi, Daniel S, Deborah Y Choi, Robert A Whittington, and Srdjan S Nedeljkovic. 2007\. “突发性健忘导致疼痛缓解：记忆与疼痛之间的关系。” *疼痛* 132 (1): 206–10.

Dehaene, Stanislas, Hakwan Lau, and Sid Kouider. 2021\. “意识是什么，机器能拥有吗？” *机器人、人工智能与人类：科学、伦理与政策*, 43–56.

Ellison, Harlan. 1967\. “我没有嘴巴，我必须尖叫。” In. 纽约：金字塔书籍。

Flor, Herta. 2002\. “痛苦的记忆。” *EMBO报告* 3 (4): 288–91.

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

Suliman, Sharain, Siyabulela G Mkabile, Dylan S Fincham, Rashid Ahmed, Dan J Stein, 和 Soraya Seedat. 2009. “Cumulative Effect of Multiple Trauma on Symptoms of Posttraumatic Stress Disorder, Anxiety, and Depression in Adolescents.” *Comprehensive Psychiatry* 50 (2): 121–27.

Sutton, Richard S, 和 Andrew G Barto. 2018. *Reinforcement Learning: An Introduction*. MIT press.

Tibbetts, C., and C. Brooker. 2014.

“White Christmas.” Episode in “Black Mirror”

; Channel 4.

[https://www.netflix.com/title/70264888](https://www.netflix.com/title/70264888)

.

Tononi, Giulio. 2008. “Consciousness as Integrated Information: A Provisional Manifesto.” *The Biological Bulletin* 215 (3): 216–42.

Trewavas, Anthony. 2014. *Plant Behaviour and Intelligence*. Oxford: Oxford University Press.

———. 2021. “Awareness and Integrated Information Theory Identify Plant Meristems as Sites of Conscious Activity.” *Protoplasma* 258 (3): 673–79.

Vrselja, Zvonimir, Stefano G Daniele, John Silbereis, Francesca Talpo, Yury M Morozov, André MM Sousa, Brian S Tanaka, 等. 2019. “Restoration of Brain Circulation and Cellular Functions Hours Post-Mortem.” *Nature* 568 (7752): 336–43.

Wegner, Daniel M. 2003. “The Mind’s Best Trick: How We Experience Conscious Will.” *Trends in Cognitive Sciences* 7 (2): 65–69.
