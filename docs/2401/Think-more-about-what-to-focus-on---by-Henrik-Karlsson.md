<!--yml

category: 未分类

date: 2024-05-27 14:30:40

-->

# 更多地思考要专注于什么 - Henrik Karlsson

> 来源：[https://www.henrikkarlsson.xyz/p/multi-armed-bandit](https://www.henrikkarlsson.xyz/p/multi-armed-bandit)
> 
> *我遇到的几乎每个人都会受益于花更多时间思考要专注于什么。* ——山姆·阿特曼

在2020年5月，我们把两辆搬家卡车停在了港口，把我们所有的东西从一辆车搬到了另一辆车上。约翰娜、莫德和我离开了瑞典，而且由于Covid限制，一旦我们登上渡轮就禁止返回。因此第二辆卡车，我们让一个陌生人把它从岛上运到我们这里：瑞典的卡车必须留在瑞典。

离开的动机是我们想要让三岁的莫德（Maud）自学，而这在瑞典是违法的，所以大多数瑞典自学生最终会在波罗的海的各个小岛上度过。在我们的岛上，我们一个人都不认识。我们没有等待的工作。我们在离开某些东西，而不只是去某个地方。我们在30年的时间里逐渐建立起来的生活在一夜之间消失了。我们必须想办法用什么来取代它。我应该开办另一家软件咨询公司来维持我们的生计吗？我可以写作吗？我们将如何找到有意义的社交背景？

我们租的发霉的公寓可以看到大海。每天，深入冬天，我会走到海边，从悬崖上跳水。在岩石之间的水道中游泳，我意识到我可以使用概率论中的一个概念来建模我们的情况。

这是一个[多臂老虎机问题](https://en.wikipedia.org/wiki/Multi-armed_bandit)。这个问题最早在1933年由生物学家[威廉·R·汤普森](https://en.wikipedia.org/wiki/William_R._Thompson)研究过，尽管是用不同的名称。这个问题围绕着一个相当超现实的思维实验。一个赌徒面对一个老虎机（“一个手臂的老虎机”），但这台老虎机不止有一个手臂——根据一些扭曲的梦境逻辑，它有*k*根手臂，手臂向各个方向伸展。其中一些手臂有很高的中奖概率，而其他手臂则不太好。但赌徒不知道哪个是哪个。

这个问题在于按照一种顺序拉动手臂，使得预期总收益最大化。（“收益”可以是任何东西。早期，这个问题被用来设计药物试验。在那里，中奖大奖被定义为找到一个成功的治疗方法。如果你正在寻找伴侣，和人们交谈就是你拉动多臂老虎机的方式，而共鸣（或缺乏共鸣）就是回报。）

赌徒需要学习关于机器的新知识，*同时*利用他们已经学到的知识来优化他们的决策。在文献中，这两种活动被称为*探索*和*利用*。你不能同时做两件事。当你探索时，你会拉动老虎机的新把手，试图弄清它们的预期赔付。当你利用时，你会拉动你找到的最好的把手。你需要找到平衡点。如果你花太少时间探索，你就会陷入玩预期回报低的机器的困境。但如果你花太多时间探索，你将比玩最好的把手赚得更少。这就是探索/利用的权衡。

人们倾向于在探索/利用谱上不同的一侧。如果你像我一样对开放性很感兴趣，探索就很容易。但是要做出承诺并利用自己对自己和世界的了解就更难。其他人更专注，但冒着过于常规选择的风险。他们错过了更好的努力途径。大多数情况下，他们倾向于做*两者都不要*的最优选择——不探索，不利用；而是盲目习惯做事，半心半意。

首先，我会在现实生活中谈一些探索和利用的问题。然后我会回到如何协调它们之间的权衡问题。

有两种人。一种是不理解世界有多复杂的人，另一种是知道自己不理解世界有多复杂的人。

为了应对生活，我们创造了关于外面世界的心理模型，然后我们混淆了模型和现实。当你和一个对自己的模型不够准确的人交流时，你是否注意到了这一点：就好像他们戴着 VR 头盔，与你知道不存在的怪物搏斗？例如，上世纪九十年代在新德里的瑞典大使馆工作的一对夫妇。他们有一个女佣。女佣生日那天，他们为她做了一个蛋糕，并邀请她来他们的餐桌吃。她拒绝了。她坚持说她必须在地板上吃，否则她会作为低等动物重生。很诱人地说，“你可以摘下头盔，那里什么都没有。吃点蛋糕吧。”但我们都戴着头盔。没有办法与现实直接接触。

诀窍是尽可能经常地    将你的心理模型与外部世界碰撞。这就是探索所做的事情。你认为你知道老虎机的支付分布，但你尝试了新的东西。你发现你错了。你更新了你的模型。

我从别人那里学到的许多心理模型。仔细检查后，结果是他们也是从别人那里学到的，然后继续向后追溯到一个生活在上世纪五十年代的人。这不是五十年代。吃点蛋糕吧。

例如，正如我在“[博客文章是一个非常长且复杂的搜索查询，以找到有趣的人并让他们将有趣的事物路由到您的收件箱中](https://www.henrikkarlsson.xyz/p/search-query)”中所写的那样，直到我摆脱了我从大众传媒中学到的沟通模式，我的博客才能“起作用”。博客不是大众传媒。它们比那更强大。我被困在了50年代。

有些心理模型比其他模型具有更多的杠杆作用。我试图专注于这些模型。意识到我不理解[过程](https://www.henrikkarlsson.xyz/p/being-patient-with-problems)，或优先事项，或[关系](https://www.henrikkarlsson.xyz/p/looking-for-alice)，是非常有益的。更好地理解这些领域每天都会产生回报。

如果你能打破不准确的心理模型，生活就会变得更容易。但你该如何做呢？我知道两种方法。

1.  找到比你更了解事物的人，并阅读他们的观点。带着回答你的问题的意图去阅读。如果你找不到答案，就给他们发电子邮件。

1.  进行实验。我不是说做一些随意的事情。我的意思是，*陈述你的假设*，*找到测试它们是否错误的方法*。大多数时候，实验的老虎机不会有所产出。但没关系。其中几个会重新排列你周围的世界。

但是，正如我之前所说的，这是一个权衡。花时间探索以收集新信息意味着花在行动上的时间减少。此外，开发往往比看起来更有价值，因为将你的注意力集中在你知道有前途的“老虎机”上可能会带来非线性回报。

作为一个经验法则，你只能做好一两件事。有些人异常出众：他们可以做3件事。我不是例外。

我像许多人一样，是在我有了第一个孩子时学会了这一点。我对成为一个父亲感到有些紧张。由于未能达到我预期的目标，我以为把孩子绑在胸前意味着让自己永远失败。但并不是这样。当莫德占去我大约一半的时间时，我不得不强迫自己确定优先事项：我会照顾她，我会写作，*而且我会拒绝其他一切*。

像这样缩小我的生活，至少使我能够达到的目标翻了一番。当我有更多时间时，我将自己分散得太薄，无法完成任务。

这是我现在每当我读到做出杰出工作的人的传记时都会注意到的事情：他们过着狭窄的生活。他们允许自己比其他人关心的更少。随意挑两个报价，这里是设计了iPhone的乔尼·艾维斯：

> [乔布斯]总是对我说，因为他担心我无法专注——他会说，“你拒绝了多少事情？”我告诉他我拒绝了这个。还有那个。但他知道我对做那些事情不感兴趣。拒绝这些事情并不意味着*牺牲*。专注意味着拒绝某个你全身心认为是绝妙主意，一想到它就兴奋不已，但你却拒绝了它去专注于别的事情。

这位德国电影制作人韦纳·赫尔佐格说：

> 虽然多年来我生活贫困——有时甚至是半穷——但自从我开始拍电影以来，我一直像富人一样生活。在我的整个生活中，我都能做我真正热爱的事情，这比任何金钱对我来说都更有价值。在朋友们通过获得大学学位、进入商界、建立事业和购买房屋来确立自己的时候，我正在拍电影，把一切都投入到我的工作中。

用多臂老虎机的术语来说，他们发现了一个好的“臂”。然后他们对它进行了开发，而其他的一概不顾。

为什么有些人实现了他们想要的那么多的事情，而其他人没有呢？人们在一生中能实现的事情数量是固定的吗？看起来似乎并不是。相反，我们的成就预算似乎是与我们拥有的优先事项数量有关的函数。有趣的是，它似乎是一个*非线性*的函数。这意味着，如果你将优先事项从4个减少到3个，你可以多做大约10%的事情；但如果你将其从4个减少到1个，你可以多做整整400%的事情。（我显然是随便编这些数字的。）我看一看伊隆·马斯克，我甚至很难理解他和我一样每天都有相同的小时数。但他当然是有的。我们每个人都有。只是他的决定由于他的专注而复利。

为什么专注会增加？其中的一部分是时间。如果你关心的东西更少，你就可以花更多时间做自己最关心的事。此外，你一直在潜意识中处理关注的事情。删减优先事项就意味着，即使看起来你在“不工作”，你也在工作。这些天，我会整个下午都和孩子们玩耍，洗碗，修房子——以一种清醒头脑的方式忙碌。然后，第二天早上当我坐下来写作时，我可以毫不费力地打出700个单词。这些想法一直在我脑中翻腾，就在意识思维的表面下，已经完全形成。

当我年轻的时候，我从未这么幸运过。这部分是因为我技能不够。但也有一部分是因为我以前会中断无意识的处理。不知不觉地，我会告诉我的大脑去关注其他事情——比如我正在看的电视剧中的冲突。我会在睡前看一集，而悬念会在我脑海中开启一个循环。这个循环会在我睡觉时在我脑海中不断翻滚；我醒来时是一片空白。我没有时间再这样做了。我确保始终有一个关于我的写作的未完成循环。我关闭每一个其他的循环——尽快结束它，或者把它写在清单上，或者，最好的情况是根本不打开这个循环。

但仅仅分配更多时间和心理加工能力并不能解释非线性，更多时间只是线性增加。我猜想非线性来自于：

+   **专注加速技能和准确世界模型的积累。** 在开放性的领域，如写作、人际关系或商业中，技能增长的空间几乎是无限的。当你花更多时间时，你会对情况有更好的模型，这使你能够更好地分配时间，从而加速学习速度，等等，以非线性的方式。

+   **专注吸引“资源”。** 在商业中这一点显而易见：如果你有强烈的专注力，投资者会开始跟着你，乞求你接受他们的钱。然后你可以用这笔钱更快地拉动强盗的手臂。写作也是如此：我写得越多，我的博客就会吸引越多有趣的人。他们开始给我反馈和建议，这有助于我写得更好，这在一个飞轮上吸引了更多有趣的人（以及一些钱）。如果你对周围的人好奇而友善，你会吸引到强大的支持网络。网络具有非线性的特性。

但对我来说，作为一个不喜欢狭窄专注的人，最值得注意的是它给我的丰富感觉。我这些日子的生活很平淡无奇。我每天都骑自行车穿过同样的田野，我注意到风力涡轮机如何转向迎风，我很少旅行，我把闲暇时间都花在盯着一张文档上。安妮·迪拉德称作家的生活是色彩单调到感官剥夺的地步。那说得对。但正如她也知道的那样，还有另一种色彩，只有在写作洞穴中三年后才能发现。它是一种微妙的、夜间的颜色；你的眼睛需要时间适应黑暗才能看到它们。如果我告诉你它们有多美，你是不会相信的。

所以，正如我所说的，我在悬崖边游泳。每天，沿着海岸走来走去，我发现了新的潜水地点。我对高空跳水产生了兴趣——这是我小时候因为过于胆小而不敢做的事情。我对水产生了童心的兴奋。海滩上的孩子们会看着我，低声交谈，然后笑。那是年轻的快乐之一。做爸爸的快乐之一是知道我比他们玩得更开心。

但除了做一个父亲之外，我不知道该怎么处理我的生活。在游泳并思考我在这篇文章中所说的事情时，我做出了决定。我要以算法的方式来处理我的情况。我将应用规则来决定何时探索，何时利用，以抵消我自然的倾向（它会让我两者都做得太少）。

对于多臂赌博问题，存在几种算法解决方案，从上世纪30年代的汤普森抽样法，一直到机器学习中使用的当代算法，如EXP3和置信上界。它们的共同点在于某种形式的：优先在早期进行探索，并在情况变得更清晰时增加利用。如果你是一个城市的新手，尽可能地认识更多人是有意义的。如果你早早就找到了一个喜欢与之相处的人，你将会拥有多年的幸福。但如果你即将离开，与你最好的朋友们呆在一起更有意义。即使你找到了更喜欢的人，你也没有时间再呆在一起。最优探索量取决于问题的复杂性和时间范围。

我决定要做的是这样的。我将接下来的30个月划分为三个部分。在接下来的十个月里，我将允许自己自由探索。之后，我会切换到用2/3时间来探索，用1/3时间加倍专注于我发现的最有趣的机会，然后我会用1/3时间探索，2/3时间利用，依此类推。

我们可以将我在探索性开放式搜索上花费的时间视为我的“温度”。当你加热原子时，它们会跳动得更快；当你加热生活时，它会变得更具探索性。当你冷却时，你会缩小范围并花更多时间利用你所知道的。当你搜索最高峰时，逐渐降低“温度”的效果如下：

请注意，搜索是从右边的区域开始的，该区域并不包含最高峰。如果早期进行了专注的搜索，它就会卡在局部最优点上。（顺便说一句，这也是为什么对许多人来说教育是一场噩梦的原因。学校迫使你在尝试之前就决定你将追求什么。它让你的未来更有知识的自己成为年幼无知的自己的仆人。）

像这样给自己制定严格规则让我感到安慰，特别是在搜索的早期阶段。我担心自己找不到正确的事情来花费时间并支持我的家庭。这让我想过早进入利用模式。知道以后会有时间专注让我更愿意尝试任何事情并失败。我学习了钢琴并尝试编程。我写了几章小说。我参与了一个幼儿园/羊场并研究了岛屿的历史。我在艺术画廊工作过。

经过 10 个月，我第一次减少了随机性。 我决定花费三分之一的业余时间来编程。 但我继续探索我的三分之二的日子。

下一次我降低温度时，我开始了这个博客。 因为写作比编程更有趣，所以我停止了编码，并将我的三分之二的业余时间转移到了博客上。 回顾起来，这似乎是一个显而易见的选择。 写作一直是我的主要迷恋。 但当时并不明显。 多年来，我一直为没有人对我写的东西感兴趣而感到沮丧。 在我二十多岁时发表我的杂志对我感兴趣的东西与我现在探索的东西无关。 写作似乎是一条死路。

我没有告诉任何人我在写博客。 让我的朋友们阅读它会让我更难以尝试并做出冒险或失败的事情。 我想把自己放在一个社会环境中，那里我因为[探索我无法辨认的潜力](https://www.henrikkarlsson.xyz/p/writing-as-communion)而受到奖励。

它奏效了。 2023 年 1 月，我将温度调至 0。 我已经达到了一个我无法想象的高峰：通过给陌生人发送电子邮件的想法来支持我的家人。 更好的是，这是一个小家庭写作业务，因为约翰娜和我现在一起工作（我们预计到 2024 年底，这些文章将成为我们的主要收入来源）。

如果我在探索之前，当我游过悬崖时就决定了一条道路，这种可能性就不会发生在我身上。 周到的电子邮件似乎不像是能够支持一个家庭的事情。 如果我知道它是的，我会认为我缺乏成功所需的东西。 这是要记住的重要事情：在尝试之前，你不知道。

诚挚地，

亨利克