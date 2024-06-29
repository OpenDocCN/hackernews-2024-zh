<!--yml

category: 未分类

date: 2024-05-27 13:03:25

-->

# Persistent interaction patterns across social media platforms and over time | Nature

> 来源：[https://www.nature.com/articles/s41586-024-07229-y](https://www.nature.com/articles/s41586-024-07229-y)

### Data collection

In our study, data collection from various social media platforms was strategically designed to encompass various topics, ensuring maximal heterogeneity in the discussion themes. For each platform, where feasible, we focus on gathering posts related to diverse areas such as politics, news, environment and vaccinations. This approach aims to capture a broad spectrum of discourse, providing a comprehensive view of conversation dynamics across different content categories.

#### Facebook

我们使用了先前研究中关于疫苗讨论^([50](/articles/s41586-024-07229-y#ref-CR50 "Schmidt, A. L., Zollo, F., Scala, A., Betsch, C. & Quattrociocchi, W. Polarization of the vaccination debate on Facebook. Vaccine 36, 3606–3612 (2018).")), 新闻^([51](/articles/s41586-024-07229-y#ref-CR51 "Schmidt, A. L. et al. Anatomy of news consumption on Facebook. Proc. Natl Acad. Sci. USA 114, 3035–3039 (2017).")) 和脱欧^([52](/articles/s41586-024-07229-y#ref-CR52 "Del Vicario, M., Zollo, F., Caldarelli, G., Scala, A. & Quattrociocchi, W. Mapping social dynamics on Facebook: the brexit debate. Soc. Netw. 50, 6–16 (2017).")) 的讨论数据集。对于疫苗话题，我们的数据集包含大约200万条评论，这些评论来自公共群组和页面，时间跨度从2010年1月2日到2017年7月17日。对于新闻话题，我们从欧洲媒体监控器选取了一系列用英语报道新闻的页面。因此，我们获取的数据集涵盖了2009年9月9日至2016年8月18日之间约3.62亿条评论。此外，我们收集了大约45亿次用户对这些页面相关帖子和评论的点赞。最后，对于脱欧话题，数据集包含了大约46万条评论，时间跨度从2015年12月31日至2016年7月29日。

#### Gab

我们从Pushshift.io存档中收集了来自Gab平台的数据([https://files.pushshift.io/gab/](https://files.pushshift.io/gab/))，涵盖了该平台从2016年8月10日启动到2018年10月29日因匹兹堡枪击事件暂时关闭的讨论。因此，我们总共收集了大约1400万条评论。

#### Reddit

数据来自Pushshift.io档案（[https://pushshift.io/](https://pushshift.io/)），时间跨度为2018年1月1日至2022年12月31日。对于每个主题，我们尽可能地手动识别和选择最能代表目标主题的子社区。由此操作，我们从r/conspiracy子社区获得了约800,000条有关阴谋主题的评论。对于疫苗主题，我们从r/VaccineDebate子社区收集了约70,000条关于COVID-19疫苗辩论的评论。我们从r/News子社区收集了约400,000条有关新闻主题的评论。我们从r/environment子社区收集了约70,000条关于气候变化主题的评论。最后，我们从r/science子社区收集了约550,000条有关科学主题的评论。

#### Telegram

我们创建了一个包含14个频道的列表，每个频道与研究中考虑的一个主题相关联。对于每个频道，我们手动收集了消息及其相关评论。因此，从与新闻主题相关的四个频道（news notiziae, news ultimora, news edizionestraordinaria, news covidultimora）中，我们在2018年4月9日至2022年12月20日期间的帖子中获得了约724,000条评论。对于政治主题，相应的两个频道（politics besttimeline, politics polmemes）在2017年8月4日至2022年12月19日期间总共产生了约490,000条评论。最后，分配给阴谋主题的八个频道（conspiracy bennyjhonson, conspiracy tommyrobinsonnews, conspiracy britainsfirst, conspiracy loomeredofficial, conspiracy thetrumpistgroup, conspiracy trumpjr, conspiracy pauljwatson, conspiracy iononmivaccino）在2019年8月30日至2022年12月20日期间总共产生了约1.4百万条评论。

#### Twitter

我们使用了前期研究中包括关于疫苗^([54](/articles/s41586-024-07229-y#ref-CR54 "Valensise, C. M. et al. Lack of evidence for correlation between COVID-19 infodemic and vaccine acceptance. Preprint at                    arxiv.org/abs/2107.07946                                     (2021)."))、气候变化^([49](/articles/s41586-024-07229-y#ref-CR49 "Falkenberg, M. et al. Growing polarization around climate change on social media. Nat. Clim. Change 12, 1114–1121 (2022).")) 和新闻^([55](/articles/s41586-024-07229-y#ref-CR55 "Quattrociocchi, A., Etta, G., Avalle, M., Cinelli, M. & Quattrociocchi, W. in Social Informatics (eds Hopfgartner, F. et al.) 245–256 (Springer, 2022).")) 主题的讨论数据集列表。对于疫苗主题，我们从2010年1月23日至2023年1月25日收集了约5000万条评论。对于新闻主题，我们扩展了之前使用的数据集^([55](/articles/s41586-024-07229-y#ref-CR55 "Quattrociocchi, A., Etta, G., Avalle, M., Cinelli, M. & Quattrociocchi, W. in Social Informatics (eds Hopfgartner, F. et al.) 245–256 (Springer, 2022)."))，通过收集所有由少于20条评论组成的主题，获得了约950万条评论，时间跨度为2020年1月1日至2022年11月29日。最后，对于气候变化主题，我们从2020年1月1日至2023年1月10日收集了约970万条评论。

#### Usenet

我们通过查询Usenet档案（[https://archive.org/details/usenet?tab=about](https://archive.org/details/usenet?tab=about)）收集了Usenet讨论系统的数据。我们选择了一系列被认为足以包含大量广泛和异质的讨论的主题，涉及活跃和人口众多的新闻组。作为此选择的结果，我们选择了阴谋、政治、新闻和讨论作为我们分析的主题候选项。对于阴谋主题，我们从alt.conspiracy新闻组收集了大约280,000条评论，时间跨度为1994年9月1日至2005年12月30日。对于政治主题，我们从alt.politics新闻组收集了大约2.6百万条评论，时间跨度为1992年6月29日至2005年12月31日。对于新闻主题，我们从alt.news新闻组收集了大约620,000条评论，时间跨度为1992年12月5日至2005年12月31日。最后，对于讨论主题，我们从同名新闻组收集了从1989年2月13日至2005年12月31日的所有对话，内容约为2.1百万条。

#### Voat

我们使用了之前提出的数据集^([56](/articles/s41586-024-07229-y#ref-CR56 "Mekacher, A. & Papasavva, A. “I can’t keep it up” a dataset from the defunct voat.co news aggregator. In Proc. International AAAI Conference on Web and Social Media Vol. 16, 1302–1311 (AAAI, 2022)."))，该数据集覆盖了平台的整个生命周期，从2018年1月9日到2020年12月25日，包括大约16.2百万条帖子和评论，由约113,000名用户在大约7,100个子版块（Voat的等同Reddit的子版块）中分享。与之前的平台类似，我们将话题与特定的子版块关联起来。因此，在阴谋话题中，我们在2018年1月9日至2020年12月25日之间收集了约1百万条来自greatawakening子版块的评论。在政治话题中，我们收集了从2014年6月16日到2020年12月25日之间的约1百万条来自政治子版块的评论。最后，在新闻话题中，我们在2013年11月21日到2020年12月25日之间收集了约1.4百万条来自新闻子版块的评论。

#### [YouTube](https://developers.google.com/youtube/v3)

我们使用了在先前的研究中提出的数据集，该数据集收集了关于气候变化话题的对话^([49](/articles/s41586-024-07229-y#ref-CR49 "Falkenberg, M. et al. Growing polarization around climate change on social media. Nat. Clim. Change 12, 1114–1121 (2022)."))，与之前的平台一致，还包括关于疫苗和新闻话题的对话。YouTube的数据收集过程是通过YouTube数据API进行的（[https://developers.google.com/youtube/v3](https://developers.google.com/youtube/v3)）。对于气候变化话题，我们在2014年3月16日到2022年2月28日之间收集了约84万条评论。对于疫苗话题，我们收集了2020年1月31日至2021年10月24日期间包含关于COVID-19疫苗关键词的对话，包括Sinopharm、CanSino、Janssen、Johnson&Johnson、Novavax、CureVac、Pfizer、BioNTech、AstraZeneca和Moderna。由于此操作，我们总共收集了约260万条评论和视频。最后，在新闻话题中，我们在2006年2月13日到2022年2月8日之间收集了约2千万条评论，包括来自英国的新闻机构提供的视频和评论列表，由Newsguard提供（参见“极化和用户倾向归因”部分）。

### 内容审查政策

内容审查政策是在线平台用来监控用户在其网站上发布内容的指导方针。平台有不同的目标和受众群体，因此其审查政策可能差异很大，有些平台更加强调自由表达，而其他则优先考虑安全和社区准则。

Facebook 和 YouTube 有严格的内容审核政策，禁止仇恨言论、暴力和骚扰^([57](/articles/s41586-024-07229-y#ref-CR57 "Facebook社区标准，transparency.fb.com/policies/community-standards/hate-speech/ (Facebook, 2023)."))。为了解决有害内容，Facebook 实行“删除、减少、告知”的策略，并结合人工审核和人工智能来执行其政策^([58](/articles/s41586-024-07229-y#ref-CR58 "Rosen, G. & Lyons, T. Remove, reduce, inform: new steps to manage problematic content. Meta, about.fb.com/news/2019/04/remove-reduce-inform-new-steps/ (2019年4月10日)."))。同样，YouTube 也有类似的社区准则，涵盖了仇恨言论政策，包括诸如粗俗语言^([59](/articles/s41586-024-07229-y#ref-CR59 "粗俗语言政策，support.google.com/youtube/answer/10072685? (YouTube, 2023)."))、骚扰^([60](/articles/s41586-024-07229-y#ref-CR60 "骚扰和网络欺凌政策，support.google.com/youtube/answer/2802268 (YouTube, 2023).")) 以及一般情况下不允许存在基于各种属性的仇恨言论和针对个人或群体的暴力行为^([61](/articles/s41586-024-07229-y#ref-CR61 "仇恨言论政策，support.google.com/youtube/answer/2801939 (YouTube, 2023)."))。为了确保这些准则得到遵守，该平台采用人工智能算法和人工审核员的组合^([62](/articles/s41586-024-07229-y#ref-CR62 "YouTube如何执行其社区准则？www.youtube.com/intl/enus/howyoutubeworks/policies/community-guidelines/enforcing-community-guidelines (YouTube, 2023)."))。

Twitter同样有全面的内容审查政策，并且针对仇恨行为设有具体规定^([63](/articles/s41586-024-07229-y#ref-CR63 "Twitter规则, help.twitter.com/en/rules-and-policies/twitter-rules (Twitter, 2023)."), [64](/articles/s41586-024-07229-y#ref-CR64 "仇恨行为, help.twitter.com/en/rules-and-policies/hateful-conduct-policy (Twitter, 2023)."))。他们在内容审查过程中采用自动化技术^([65](/articles/s41586-024-07229-y#ref-CR65 "Gorwa, R., Binns, R. & Katzenbach, C. 算法内容审查：平台治理自动化的技术和政治挑战. Big Data Soc. 7, 2053951719897945 (2020)."))和人工审核^([66](/articles/s41586-024-07229-y#ref-CR66 "我们的执法选项范围, help.twitter.com/en/rules-and-policies/enforcement-options (Twitter, 2023)."))。在提交日期时，Twitter的内容政策自埃隆·马斯克接管以来未曾改变，除了2022年11月23日停止执行关于COVID-19误导信息的政策。他们的政策执行因不一致性而受到批评^([67](/articles/s41586-024-07229-y#ref-CR67 "Elliott, V. & Stokel-Walker, C. Twitter的审查系统一团糟. WIRED (2022年11月17日)."))。

Reddit在内容审查政策的严格程度上处于中间位置。Reddit的内容政策有八条规定，包括禁止暴力、骚扰和基于身份或弱势群体的仇恨传播^([68](/articles/s41586-024-07229-y#ref-CR68 "Reddit内容政策, www.redditinc.com/policies/content-policy (Reddit, 2023)."), [69](/articles/s41586-024-07229-y#ref-CR69 "基于身份或弱势群体的仇恨传播, www.reddithelp.com/hc/en-us/articles/360045715951 (Reddit, 2023)."))。Reddit依赖用户举报和志愿者版主，因此在执行规则方面可能比Facebook、YouTube和Twitter宽松。2022年10月，Reddit宣布他们计划更新内容审查实践，引入自动化技术^([70](/articles/s41586-024-07229-y#ref-CR70 "Malik, A. Reddit从ML内容审查初创公司Oterlu团队中引进人员。TechCrunch, tcrn.ch/3yeS2Kd (2022年10月4日)."))。

相比之下，Telegram、Gab和Voat采取了更为放任的态度，对内容的限制较少。Telegram在其指南中存在模糊之处，这源于广泛或主观的术语，可能导致不同的解释^([71](/articles/s41586-024-07229-y#ref-CR71 "Terms of Service, telegram.org/tos (Telegram, 2023).")). 尽管他们提到可能会使用自动算法分析消息，Telegram主要依靠用户报告各种内容，例如暴力、儿童虐待、垃圾邮件、非法毒品、个人详细信息和色情内容^([72](/articles/s41586-024-07229-y#ref-CR72 "Durov, P. The rules of @telegram prohibit calls for violence and hate speech. We rely on our users to report public content that violates this rule. Twitter, twitter.com/durov/status/917076707055751168?lang=en (8 October 2017).")). 根据Telegram的隐私政策，报告的内容可能会由版主审核，并且如果确认违反他们的条款，可能会对账户施加临时或永久限制^([73](/articles/s41586-024-07229-y#ref-CR73 "Telegram Privacy Policy, telegram.org/privacy (Telegram, 2023).")). Gab的服务条款允许在美国宪法第一修正案保护下的所有言论，并删除非法内容。他们表示在内容发布前不会审核材料，并且无法保证违法内容发布后能够及时删除^([74](/articles/s41586-024-07229-y#ref-CR74 "Terms of Service, gab.com/about/tos (Gab, 2023).")). Voat曾被称为Reddit的“言论自由”替代品，允许发布即使可能被视为冒犯或有争议的内容^([56](/articles/s41586-024-07229-y#ref-CR56 "Mekacher, A. & Papasavva, A. “I can’t keep it up” a dataset from the defunct voat.co news aggregator. In Proc. International AAAI Conference on Web and Social Media Vol. 16, 1302–1311 (AAAI, 2022).")).

Usenet是一个于1979年创建的分布式在线讨论系统。由于其分布式特性，Usenet一直难以有效地进行管理，因此它被认为是一个可以发布有争议甚至非法内容而无后果的地方。Usenet上的每个群组都可以有自己的版主，负责监督和执行群组的规则，而没有适用于整个平台的统一规则^([75](/articles/s41586-024-07229-y#ref-CR75 "Salzenberg, C. & Spafford, G. What is Usenet?, www0.mi.infn.it/∼calcolo/Wis usenet.html (1995).")).

### 对数分箱和对话规模

由于对话长度的重尾分布（扩展数据图 [1](/articles/s41586-024-07229-y#Fig5)），为了绘制图表和执行分析，我们采用了对数分箱。因此，根据其长度，每个数据集的每个话题被分配到21个分箱中的一个。为了确保每个分箱中有最少数量的数据点，我们迭代地调整最后一个分箱的左边界，以至少包含 *N* = 50 个元素（在Facebook新闻的情况下，由于其更大的规模，我们设置 *N* = 100）。具体来说，考虑到按长度递增排序的话题，最大话题的大小被更改为倒数第二大的话题的大小，并相应地重新计算分箱，直到最后一个分箱至少包含 *N* 个数据点。

为了可视化目的，我们提供了对对数分箱结果的归一化，将离散点映射到 *x* 轴的坐标，使分箱对应于 {0, 0.05, 0.1, ..., 0.95, 1}。

为了执行分析的部分，我们选择属于归一化对数分箱线段长度 [0.7, 1] 的对话。这个区间确保对话足够长且我们有大量的话题。参与度和毒性趋势通过将这些对话应用于21个元素的线性分箱来获得，这些对话是按时间顺序排列的评论线。结果数据集的详细信息见补充表 [2](/articles/s41586-024-07229-y#MOESM1)。

最后，为了评估有毒和非有毒话题中参与价值增长率的平等性（参见“对话演变和毒性”部分），我们实现了以下线性回归模型：

$${\rm{p}}{\rm{a}}{\rm{r}}{\rm{t}}{\rm{i}}{\rm{c}}{\rm{i}}{\rm{p}}{\rm{a}}{\rm{t}}{\rm{i}}{\rm{o}}{\rm{n}}={\beta }_{0}+{\beta }_{1}\cdot {\rm{b}}{\rm{i}}{\rm{n}}+{\beta }_{2}\cdot \,({\rm{b}}{\rm{i}}{\rm{n}}\cdot {\rm{i}}{\rm{s}}{\rm{T}}{\rm{o}}{\rm{x}}{\rm{i}}{\rm{c}}),$$

其中术语 *β*[2] 解释了毒性对参与增长的影响。我们的结果显示，在大多数原始和验证数据集中，*β*[2] 与0没有显著差异（补充表 [8](/articles/s41586-024-07229-y#MOESM1) 和 [11](/articles/s41586-024-07229-y#MOESM1)）。

### 毒性检测和使用的模型验证

检测毒性的问题备受争议，甚至目前对于毒性言论的定义没有达成一致^([64](/articles/s41586-024-07229-y#ref-CR64 "Hateful Conduct, help.twitter.com/en/rules-and-policies/hateful-conduct-policy (Twitter, 2023)."),[76](/articles/s41586-024-07229-y#ref-CR76 "Castelle, M. 

The linguistic ideologies of deep abusive language classification.

In Proc. 

2nd Workshop on Abusive Language Online (ALW2)（eds Fišer, D. 

等。）160-170，aclanthology.org/W18-5120（计算语言学协会，2018年））。

有毒评论可以被视为包含粗俗或贬损性语言^（[32](/articles/s41586-024-07229-y#ref-CR32 "Davidson，T.，Warmsley，D.，Macy，M.

& Weber，I.

自动仇恨言论检测及其攻击性语言问题。

在Proc中。

国际AAAI网页和社交媒体大会11（人工智能促进协会，2017年））），使用严厉、辱骂性语言和人身攻击^（[33](/articles/s41586-024-07229-y#ref-CR33 "Kolhatkar，V.

等。

SFU意见与评论语料库：

用于在线新闻评论分析的语料库。

Corpus Pragmat。

4，155-190（2020年））。）或包含极端主义、暴力和骚扰^（[11](/articles/s41586-024-07229-y#ref-CR11 "Sheth，A.，Shalin，V.

L.

& Kursuncu，U.

在社交媒体上定义和检测毒性：

上下文和知识至关重要。

Neurocomputing 490, 312-318（2022年））。举几个例子。

尽管理论上应将有毒言论与仇恨言论区分开来，后者通常与针对个人或群体的攻击有关，攻击基于种族、宗教、性别、性取向等属性^（[77](/articles/s41586-024-07229-y#ref-CR77 "Tontodimamma，A.，Nissi，E.

& Sarra，A.

E.

A.

三十年的仇恨言论研究：

兴趣主题及其演变。

Scientometrics 126, 157-179（2021年））。它有时也可能被用作一个总称^（[78](/articles/s41586-024-07229-y#ref-CR78 "Sap, M.

等。

态度不同的注释者：

注解者的信念和身份如何偏向有毒语言检测。

在Proc中。

2022北美计算语言学协会年会：

人类语言技术（eds。

Carpuat，M.

等。）5884-5906（计算语言学协会，2022年）），[79](/articles/s41586-024-07229-y#ref-CR79 "Pavlopoulos，J.，Sorensen，J.，Dixon，L.，Thain，N.

& Androutsopoulos，I.

毒性检测：

上下文真的重要吗？

在Proc中。

美国人工智能协会第58届年会（eds Jurafsky，D.

等。）4296-4305（计算语言学协会，2020年）。

这种缺乏一致性直接反映了毒性概念的挑战性和固有的主观性质。

该主题的复杂性使得尽管自然语言处理模型在自动检测毒性方面取得了令人印象深刻的进展，但评估其可靠性特别困难。

现代自然语言处理模型，如Perspective API，是利用词嵌入技术构建单词的表示，这些表示在高维空间中作为向量，其中度量距离应反映词语之间的概念距离，从而提供语言上下文。

关于毒性检测模型的主要关切是它们有限的能力来情境化对话^([11](/articles/s41586-024-07229-y#ref-CR11 "Sheth, A., Shalin, V. 

L. 

和Kursuncu，U。

在社交媒体上定义和检测毒性内容：

上下文和知识至关重要。

Neurocomputing 490, 312–318 (2022.)，[80](/articles/s41586-024-07229-y#ref-CR80 "Yin, W. 

和Zubiaga，A。

显而易见背后隐藏着：

在社交媒体上误导性关键词和隐晦的辱骂性语言。

线上Soc.

网络

媒体30，100210（2022.）")。

这些模型通常难以将超越文本本身的因素纳入考虑，例如参与者的个人特征、动机、关系、群体成员身份以及讨论的整体语调^([11](/articles/s41586-024-07229-y#ref-CR11 "Sheth, A., Shalin, V. 

L.

和Kursuncu，U。

在社交媒体上定义和检测毒性内容：

上下文和知识至关重要。

Neurocomputing 490, 312–318 (2022).")。

因此，在不同群体（如族裔或年龄组）中，被认为是有毒内容的定义可能会有很大不同^([81](/articles/s41586-024-07229-y#ref-CR81 "Sap, M., Card, D., Gabriel, S., Choi, Y. 

和Smith, N. 

A。

种族偏见在仇恨言论检测中的风险。

在Proc。

计算语言学年会第57届年会（eds Kohonen, A. 

等。）1668–1678 (Association for Computational Linguistics, 2019)."))，从而导致潜在偏见。

这些偏见可能源自注释者的背景和用于训练的数据集，这些数据集可能不充分代表文化异质性。

此外，像间接暗示、表情和针对特定群体的内部笑话这样的微妙形式的有毒内容，检测起来尤为具有挑战性。

词嵌入赋予当前分类器丰富的语言环境，增强其识别有毒表达特征模式的能力。

但是，理解对话更广泛背景的要求，如个人特征、动机和群体动态，仍超出自动检测模型的范围。

我们承认我们方法中的这些固有限制。

尽管如此，依赖于自动检测模型对于像这项研究中进行的大规模在线毒性分析是至关重要的。

我们特别依赖Perspective API完成这项任务，因为它代表着最先进的自动毒性检测技术，提供了在语言细微差别和可扩展分析能力之间的平衡。

为了定义适当的分类阈值，我们参考现有文献^([64](/articles/s41586-024-07229-y#ref-CR64 "Hateful Conduct, help.twitter.com/en/rules-and-policies/hateful-conduct-policy (Twitter, 2023)."))，其使用0.6作为被视为有毒评论的阈值。

这个阈值也可以被认为是合理的，因为根据Perspective提供的开发者指南，这表明样本读者中的大多数人，即10个中的6个，会认为该评论具有毒性。

由于上述限制（有关Perspective API的批评，请参阅参考文献。

^([82](/articles/s41586-024-07229-y#ref-CR82 "Rosenblatt, L.，Piedras, L.等。

和Wilkins, J. 

批判性视角：

一项显示PerspectiveAPI中缺陷的基准。

在Proc。

第二届自然语言处理对积极影响的研讨会（NLP4PI）（主编Biester, L.等。

等。) 15–24 (计算语言学协会，2022)。"))，我们通过使用另外两个毒性检测器进行了比较分析来验证我们的结果：

Detoxify ([https://github.com/unitaryai/detoxify](https://github.com/unitaryai/detoxify))，这与Perspective类似，以及IMSYPP，一个为欧洲仇恨言论项目开发的分类器^([16](/articles/s41586-024-07229-y#ref-CR16 "Cinelli, M.等。

等。

线上仇恨和虚假信息的动态。

Sci.

重现。

11, 22083 (2021)。")) ([https://huggingface.co/IMSyPP](https://huggingface.co/IMSyPP))。

在[第14张补充表](/articles/s41586-024-07229-y#MOESM1)，我们报道了从每个数据集中随机抽取的10万条评论在三个模型分类中的一致性百分比。

对于Detoxify，我们使用了与Perspective相同的二进制毒性阈值（0.6）。

尽管IMSYPP在之前的毒性定义上有所不同^([16](/articles/s41586-024-07229-y#ref-CR16 "Cinelli, M.等。

等。

线上仇恨和虚假信息的动态。

Sci。

重现。

11, 22083 (2021)。")), 我们的比较分析显示了结果的普遍一致性。

尽管存在基本定义和方法论差异，但这种一致性强调了我们在各种毒性检测框架中发现的结果的稳健性。

此外，我们使用进一步的大规模和异构数据集对本研究的核心分析进行了所有分类器的性能评估。

如补充图表。

[1](/articles/s41586-024-07229-y#MOESM1)和[2](/articles/s41586-024-07229-y#MOESM1)，关于毒性随对话规模和用户参与增加的结果，毒性量化上非常相似。

此外，我们验证了在不同毒性阈值下我们发现结果的稳定性。

尽管本文的主要分析使用Perspective API建议的阈值值为0.6，以最小化假阳性，但我们的结果在应用更不保守的阈值值为0.5时仍然保持一致。

这在扩展数据图表。

[5](/articles/s41586-024-07229-y#Fig9)，证实了我们在不同毒性水平下的观察结果的稳健性。

为了本研究，我们使用了支持欧洲和美洲流行语言的API，包括英语、西班牙语、法语、葡萄牙语、德语、意大利语、荷兰语、波兰语、瑞典语和俄语。

Detoxify also offers multilingual support. 

However, IMSYPP is limited to English and Italian text, a factor considered in our comparative analysis.

### Polarization and user leaning attribution

Our approach to measuring controversy in a conversation is based on estimating the degree of political partisanship among the participants. 

This measure is closely related to the political science concept of political polarization. 

Political polarization is the process by which political attitudes diverge from moderate positions and gravitate towards ideological extremes, as described previously^([83](https://doi.org/10.1038/s41586-024-07229-y#ref-CR83 "DiMaggio, P., Evans, J. 

& Bryson, B. 

Have American’s social attitudes become more polarized? 

Am. 

J. 

Sociol. 

102, 690–755 (1996).")). 

By quantifying the level of partisanship within discussions, we aim to provide insights into the extent and nature of polarization in online debates. 

In this context, it is important to distinguish between ‘ideological polarization’ and ‘affective polarization’. 

Ideological polarization refers to divisions based on political viewpoints. 

By contrast, affective polarization is characterized by positive emotions towards members of one’s group and hostility towards those of opposing groups^([84](https://doi.org/10.1038/s41586-024-07229-y#ref-CR84 "Fiorina, M. 

P. 

& Abrams, S. 

J. 

Political polarization in the American public. 

Annu. 

Rev. 

Polit. 

Sci. 

11, 563–588 (2008)."),[85](https://doi.org/10.1038/s41586-024-07229-y#ref-CR85 "Iyengar, S., Gaurav, S. 

& Lelkes, Y. 

Affect, not ideology: 

a social identity perspective on polarization. 

Publ. 

Opin. 

Q. 

76, 405–431 (2012).")). 

Here we focus specifically on ideological polarization. 

The subsequent description of our procedure for attributing user political leanings will further clarify this focus. 

On online social media, the individual leaning of a user toward a topic can be inferred through the content produced or the endorsement shown toward specific content. 

In this study, we consider the endorsement of users to news outlets of which the political leaning has been evaluated by trustworthy external sources. 

Although not without limitations—which we address below—this is a standard approach that has been used in several studies, and has become a common and established practice in the field of social media analysis due to its practicality and effectiveness in providing a broad understanding of political dynamics on these online platforms^([1](https://doi.org/10.1038/s41586-024-07229-y#ref-CR1 "Cinelli, M., Morales, G. 

D. 

F., Galeazzi, A., Quattrociocchi, W. 

& Starnini, M. 

The echo chamber effect on social media. 

Proc.

Natl Acad. 

Sci. 

USA 118, e2023301118 (2021)."),[43](https://doi.org/10.1038/s41586-024-07229-y#ref-CR43 "Garimella, K., Morales, G. 

D. 

F., Gionis, A. 

& Mathioudakis, M. 

Quantifying controversy on social media. 

ACM Trans. 

Soc. 

Comput. 

1, 3 (2018)."),[86](#ref-CR86 "Cota, W., Ferreira, S. 

& Pastor-Satorras, R. 

E. 

A. 

在政治传播网络中量化信息传播中的回音室效应。

EPJ数据科学。

8, 38 (2019)。"),[87](#ref-CR87 "贝西，A。

等人。

在Facebook和Youtube上的用户极化。

PLoS ONE 11, e0159641 (2016)。"),[88](/articles/s41586-024-07229-y#ref-CR88 "贝西，A。

等人。

科学与阴谋：

在信息误导时代的集体叙事。

PLoS ONE 10, e0118093 (2015)。")。

我们根据媒体偏见/事实检查（MBFC）（[https://mediabiasfactcheck.com](https://mediabiasfactcheck.com)）提供的信息，结合Newsguard（[https://www.newsguardtech.com/](https://www.newsguardtech.com/)）的等效信息，标记新闻媒体。

MBFC是一个独立的事实检查组织，根据它们产生和分享的内容的可靠性和政治偏见对新闻媒体进行评级。

同样，Newsguard是一个由国际记者团队创建的工具，提供新闻媒体的信任和政治偏见评分。

遵循文献中使用的标准方法^([1](/articles/s41586-024-07229-y#ref-CR1 "Cinelli, M., Morales, G.

D。

F.，Galeazzi，A.，Quattrociocchi，W.

& Starnini，M。

社交媒体上的回音室效应。

会议。

Natl Acad。

科学。

美国118，e2023301118 (2021)。"),[43](/articles/s41586-024-07229-y#ref-CR43 "Garimella，K.，Morales，G。

D.

F.，Gionis，A。

& Mathioudakis，M。

在社交媒体上量化争议。

ACM Trans.

Soc。

计算。

1, 3 (2018)。"），我们计算了用户*l*的个体倾向度 ∈ [-1, 1]，作为其所产生/分享的每个内容的倾向性评分*l*[c] ∈ [-1, 1]的平均值，其中*l*[c]由MBFC和Newsguard提供的新闻机构政治评分映射而来：

[左翼，中左翼，中间，中右翼，右翼] 到 [-1, -0.5, 0, 0.5, 1]，和 [极左，左翼，右翼，极右] 到 [-1, -0.5, 0.5, 1]）。

我们的数据集结构不同，因此我们必须以不同的方式评估用户的倾向性。

对于Facebook新闻，我们给那些在至少三次点赞和在至少三次评论下发布政治评分的新闻媒体页面的用户分配了一个倾向性评分。

对于推特新闻，对于在得分新闻媒体页面下发布至少15条评论的用户，分配了一种倾向性。

对于推特疫苗和Gab，我们考虑那些至少三次分享由得分新闻媒体页面制作的内容的用户。

我们方法的一个限制是与政治立场一致的内容互动并不总是意味着同意；

用户可能与对立观点进行关键讨论。

然而，研究表明，用户主要分享与其自身观点一致的内容，特别是在政治上具有争议的背景下^([87](/articles/s41586-024-07229-y#ref-CR87 "贝西，A。

等人。

在Facebook和Youtube上的用户极化。

PLoS ONE 11, e0159641 (2016)."),[89](/articles/s41586-024-07229-y#ref-CR89 "Himelboim, I., McCreery, S.

& Smith, M.

鸟类志同道合地推文：

整合网络和内容分析以检查Twitter上跨意识形态暴露。

J.

Comput.

Med.

Commun.

18, 40–60 (2013)."),[90](/articles/s41586-024-07229-y#ref-CR90 "An, J., Quercia, D.

& Crowcroft, J.

党派分享：

Facebook证据与社会后果。

在Proc.

第二届ACM在线社交网络会议，COSN′14 13–24 (Association for Computing Machinery, 2014).")).

此外，我们的方法捕捉到积极表达其政治倾向的用户，而省略了‘被动’用户。

这是由于缺乏有关未明确陈述其意见的用户数据。

尽管如此，分析活跃用户为我们提供了对社交媒体平台上最积极、最有影响力的人群言论的宝贵见解。

### 爆发分析

我们使用了Kleinberg突发检测算法^([46](/articles/s41586-024-07229-y#ref-CR46 "Kleinberg, J. Bursty and hierarchical structure in streams. Data Min. Knowl. Discov. 7, 373–397 (2003)."))（见“争议和毒性”部分）来处理数据集中至少有50条评论的所有对话。在我们的分析中，我们随机抽样最多5,000个对话，每个对话包含特定数量的评论。为确保数据可靠性，我们排除了具有过多双时间戳的对话，定义为在前24小时内连续超过10次或超过100次。此标准有助于减少可能扭曲人类活动模式的机器人的影响。此外，我们关注每个主题的前24小时以分析其评论流的高峰活动期。因此，Usenet被排除在我们的研究之外。Usenet的独特使用特性使得这种时间限制的分析不适用，因为其活动模式与其他考虑中的平台不符合。通过重建评论流的密度剖面，该算法根据其强度级别将整个流的时间段分成子时间段。标记为离散正值，较高水平的突发性代表较高的活动段。为了避免考虑平坦密度阶段，最大突发水平等于2的主题被排除在本分析之外。为了评估评论强度是否导致评论毒性的增加，我们在三个强度阶段的评论毒性分数*t*[*i*]的分布之间执行了Mann–Whitney *U*检验^([91](/articles/s41586-024-07229-y#ref-CR91 "Mann, H. B. & Whitney, D. R. On a test of whether one of two random variables is stochastically larger than the other. Ann. Math. Stat. 18, 50–60 (1947)."))，并对多重测试进行了Bonferroni校正。扩展数据表格 [4](/articles/s41586-024-07229-y#Tab5) 显示了每个测试的校正*P*值，在0.99置信水平下，H1在列标题中表示。在考虑的对话的三个阶段（高峰前、高峰期间和高峰后）中，毒性评论频率分布的示例报告在图 [4c](/articles/s41586-024-07229-y#Fig4) 中。

### 在Usenet上的毒性检测

如上文讨论的有关毒性检测和Perspective API的部分，自动检测器从它们训练的标注数据集中获取毒性理解。Perspective API主要训练于最近的文本，其人工标注者遵循当代文化规范。因此，尽管我们的数据集最早不晚于1990年代，我们提供了有关将Perspective API应用于Usenet及验证分析的可行性讨论。与几十年前相比，当代社会，尤其是在西方背景下，对毒性问题，包括性别、种族和性取向，更为敏感。这意味着今天被识别为有毒的一些评论，包括来自旧平台如Usenet的评论，过去可能不会被视为有毒。然而，这种差异并不显著影响我们的分析，我们的分析集中于当前的毒性标准。另一方面，语言特征的变化可能会产生一些影响：在1990年代经常使用的某些词语和表达方式，今天的语言中却很少出现，这可能会使Perspective在分类包含这些词语的短文本时效果不佳。因此，我们继续评估这种可能情景对我们结果的影响。考虑到上述情况，我们将被标记为有毒的文本视为正确分类；相反，我们假设存在一个固定概率*p*，使得某条评论可能被错误标记为非毒性。因此，我们随机指定*p*比例的非毒性评论，将其重新标记为有毒，并在变更后的数据集上计算毒性与对话规模趋势（图[2](/articles/s41586-024-07229-y#Fig2)）跨不同*p*值模拟500种不同趋势，收集其回归斜率以获得空数据集的分布。为了评估错误概率是否会导致观察到的趋势显著差异，我们计算斜率的分数*f*，它们落在区间(−|*s*|,|*s*|)之外的比例，其中*s*是观察到的趋势斜率。我们在不同的*p*值下报告了补充表[9](/articles/s41586-024-07229-y#MOESM1)的结果。与我们先前的分析一致，如果*f*小于0.05，则假设斜率与从随机数据中获得的斜率显著不同。

我们观察到，仅有Usenet Talk数据集对小错误概率显示出敏感性，而其他数据集则未显示出显著差异。因此，我们的结果表明，尽管可能存在语言和文化转变可能影响分类器在旧文本中的可靠性，但Perspective API适用于我们分析中的Usenet数据。

### 短对话的毒性

我们的研究聚焦于用户参与与对话毒性之间的关系，特别是在参与度高或持续时间长的讨论中。一个潜在的关注点是，专注于较长的线程可能忽视了由于早期毒性而迅速终止的对话，因此可能会使我们的分析产生偏差。为了解决这个问题，我们分析了每个数据集中包含的6到20条评论的较短对话。特别是，我们计算了每个线程中前三条和最后三条评论的毒性评分分布。这种方法有助于确保我们的分析涵盖了各种对话长度和毒性发展模式，从而更全面地理解其中的动态。如《补充图表》[3](/articles/s41586-024-07229-y#MOESM1) 所示，在短对话中，毒性评分的分布显示出高度相似性，这意味着在最后的评论中毒性并不显著高于初始评论，表明上述潜在影响并不会削弱我们的结论。关于我们对长线程的分析，我们在这里注意到参与数量可能会在各种情况下产生类似的趋势。例如，高参与可以因为许多用户参与对话，但也可以是少数用户在时间上平等贡献的小组。或者在非常大的讨论中，个别异常值的贡献可能会被隐藏。通过测量参与度，这些边缘情况和其他统计上高度可能的讨论动态可能不会有明显区别，但最终，这种缺乏辨别力对我们的发现或我们所得出的结论的有效性没有任何影响。

### 汇报摘要

关于研究设计的更多信息可在链接到本文的《自然出版集团汇报摘要》[Nature Portfolio Reporting Summary](/articles/s41586-024-07229-y#MOESM2) 中找到。
