<!--yml

category: 未分类

date: 2024-05-27 13:26:39

-->

# 代谢酶演化中分形几何的出现 | 自然

> 来源：[https://www.nature.com/articles/s41586-024-07287-2](https://www.nature.com/articles/s41586-024-07287-2)

### 分子克隆

*S.* *elongatus* PCC 7942 中编码 CS 的基因通过 PCR（Q5 高保真 2× Master Mix，New England Biolabs）从基因组 DNA 扩增，并通过 Gibson 克隆（Gibson Assembly Master Mix，New England Biolabs）引入 pLIC 表达载体^([38](/articles/s41586-024-07287-2#ref-CR38 "Yang, J., Zhang, Z., Zhang, X. A. A ligation-independent cloning method using nicking DNA endonuclease. Biotechniques 49, 817–821 (2010)."))。所有其他现存和祖先 CS 序列均从 Twist Bioscience 获得并通过 Gibson 克隆引入相同的表达载体。所有 CS 序列均标记有 C 端聚组氨酸标签以进行纯化（标签序列：LE-HHHHHH-Stop）。对于单点突变和 CS 序列的删除，使用 KLD 酶混合物（New England Biolabs）。使用 NEBasechanger 设计突变引物并用于 PCR 扩增将要更改的基因的载体。生成的 PCR 产物添加到 KLD 酶混合物中，随后转化。在实验使用之前，所有克隆的基因均通过 Sanger 测序（Microsynth）进行验证。

所有纯化蛋白质的 DNA 序列和所有现存序列的 NCBI 标识符见补充表 [2](/articles/s41586-024-07287-2#MOESM1)。

### 蛋白质纯化

对于异源过表达，含有感兴趣基因的载体被转化为化学竞争力*大肠杆菌* BL21 (DE3)细胞。 转化的菌落用于接种由LB培养基制成的表达培养物（500 ml），添加12.5 g/l乳糖（Fisher Chemical）。 培养在30°C和200 r.p.m条件下过夜。 细胞通过离心（4,500*g*，15 min，4°C）收集，用缓冲液A（20 mM Tris，300 mM NaCl和20 mM imidazole，pH 8）重新悬浮并新鲜添加DNA酶I（3 units/µl^(–1)，Applichem）。 细胞使用Microfluidizer（Microfluidics）在15,000 psi的条件下分裂3个周期，并离心以沉淀细胞碎片和聚集物（30,000*g*，30 min，4°C）。 澄清的裂解物使用蠕动泵（Hei-FLOW 06，Heidolph）加载到预装的镍NTA柱（5 ml Nuvia IMAC Ni-Charged，Bio-Rad），这些柱已与缓冲液A预平衡。 装载柱首先用缓冲液A洗涤7个柱体积，然后用缓冲液A中10%（v/v）的缓冲液B（20 mM Tris，300 mM NaCl和500 mM imidazole，pH 8）洗涤7个柱体积。 结合的蛋白质用缓冲液B洗脱，或者用PD-10脱盐柱（Cytiva）将其换成PBS或20 mM Tris，200 mM NaCl，pH 7.5，或通过分子排阻色谱（SEC）进一步纯化。 对于SEC，将蛋白质注射到ENrich SEC 650柱（Bio-Rad）中，使用PBS作为运行缓冲液，使用NGC色谱系统（Bio-Rad）。 通过SDS-PAGE分析蛋白质的纯度。 在脱盐或SEC后，纯化的蛋白质被液氮快速冷冻并存储在−20°C，以备进一步使用。

### 系统发育分析和祖先序列重建

从NCBI参考序列数据库中收集了84个蓝细菌和海洋γ变形菌门的CS基因的氨基酸序列作为外类群，并使用MUSCLE (v.3.8.31)^([39](/articles/s41586-024-07287-2#ref-CR39 "Edgar, R. C. MUSCLE: multiple sequence alignment with high accuracy and high throughput. Nucleic Acids Res. 32, 1792–1797 (2004).")) 进行了序列比对。最大似然（ML）进化树从多序列比对（MSA）中推断，使用raxML (v.8.2.10)^([40](/articles/s41586-024-07287-2#ref-CR40 "Stamatakis, A. RAxML version 8: a tool for phylogenetic analysis and post-analysis of large phylogenies. Bioinformatics 30, 1312–1313 (2014)."))。使用LG替代矩阵^([41](/articles/s41586-024-07287-2#ref-CR41 "Le, S. Q. & Gascuel, O. An improved general amino acid replacement matrix. Mol. Biol. Evol. 25, 1307–1320 (2008)."))，由自动最佳模型选择确定，以及固定的碱基频率和速率异质性的gamma模型。ML树拓扑的稳健性通过raxML推断了100个非参数Bootstrap树，使用BOOSTER ([https://booster.pasteur.fr](https://booster.pasteur.fr)) 获取了Felsenstein和传递Bootstrap值。我们还使用PhyML (3.0)^([42](/articles/s41586-024-07287-2#ref-CR42 "Guindon, S. et al. New algorithms and methods to estimate maximum-likelihood phylogenies: assessing the performance of PhyML 3.0\. Syst. Biol. 59, 307–321 (2010).")) 推断了近似似然比测试^([43](/articles/s41586-024-07287-2#ref-CR43 "Anisimova, M. & Gascuel, O. Approximate likelihood-ratio test for branches: a fast, accurate, and powerful alternative. Syst. Biol. 55, 539–552 (2006).")) 用于统计评估进化树中的分支支持度。

基于CS树和MSA，使用PAML (v.4.9)^([44](/articles/s41586-024-07287-2#ref-CR44 "Yang, Z. PAML 4: phylogenetic analysis by maximum likelihood. Mol. Biol. Evol. 24, 1586–1591 (2007).")) 中的codeML包推断了祖先序列。为了调整CS序列中的间隙和N端的不同长度，使用PAUP (4.0a) 基于MSA的二进制版本进行了最简派系推断（1 = 氨基酸，0 = 间隙，无残基）。然后将树中每个节点的状态分配（氨基酸或间隙）应用于推断的祖先序列。

初步重建关键氨基酸替换q18L在祖先ancB和ancA中存在歧义。我们确定这种情况是因为在*Planktothrix*类群和*S.* *elongatus*中存在这个L残基，但在*Cyanobium/Prochlorococcus*类群中却不存在。因此，这个L残基要么在*Cyanobium/Prochlorococcus*的演化过程中被获得一次然后丢失，要么在*Planktothrix*和*S.* *elongatus*中发生了两次同源性的获得。因此，我们将来自蓝细菌*Prochlorothrix hollandica*的CS序列加入到了序列比对中，该序列已被多项研究证实是*S.* *elongatus*和*Cyanobium/Prochlorococcus*类群的姐妹群^([45](#ref-CR45 "Moore, K. R. et al. An expanded ribosomal phylogeny of cyanobacteria supports a deep placement of plastids. Front. Microbiol. 10, 1612 (2019)."),[46](#ref-CR46 "Schirrmeister, B. E., Antonelli, A. & Bagheri, H. C. The origin of multicellularity in cyanobacteria. BMC Evol. Biol. 11, 45 (2011)."),[47](/articles/s41586-024-07287-2#ref-CR47 "Pinevich, A., Velichko, N. & Ivanikova, N. Cyanobacteria of the genus Prochlorothrix. Front. Microbiol. 3, 173 (2012)."))。之前由于无法确定其在树上的位置而被省略。我们手动在树上添加了一个分支，将*P.* *hollandica*放置为*S.* *elongatus*和*Cyanobium/Prochlorococcus*类群的姐妹群。使用raxML重新优化了分支长度，并使用PAML重复进行了祖先重建。结果强烈支持替换q18L在ancB和ancA中存在（扩展数据图[8b](/articles/s41586-024-07287-2#Fig13)），因为*P.* *hollandica*在位置18处也包含L。这使得在*Cyanobium/Prochlorococcus*类群中一次获取和随后丢失的假设比在*Planktothrix*、*P.* *hollandica*和*S.* *elongatus*中三次独立获取的假设更加可信。

### MP分析

测量使用OneMP质量光度计（Refeyn）进行。可重复使用的硅胶垫（CultureWellTM, CW-50R-1.0，直径50-3 mm × 深度1 mm）设置在清洁的显微镜盖玻片（1.5 H, 24 × 60 mm，Carl Roth）上，并使用浸油（IMMOIL-F30CC，Olympus）装配在质量光度计的舞台上。胶垫中填充19 µl缓冲液（PBS或20 mM Tris, 200 mM NaCl pH 7.5）以聚焦仪器。然后，加入1 µl预稀释的蛋白溶液（1 µM）到缓冲液液滴中，并彻底混合。测量期间蛋白质的最终浓度为50 nM，除非另有说明。使用AcquireMP（Refeyn, v.1.2.1）以100帧每秒的速度获取60秒数据。使用DiscoverMP（Refeyn, v.2.5.0）处理和分析生成的电影。通过高斯曲线拟合，绘制相应分子量的蛋白复合物作为直方图，并显示单个寡聚体状态的种群。在每次测量会话期间至少校准一次仪器，使用商业标准（NativeMark未染蛋白标准，Thermo Fisher）或具有已知复合物大小的蛋白质的自制校准标准。

对于底物滴定，预稀释的蛋白样品（2 µM）与相应底物浓度共孵育10分钟。同样的底物浓度也加入了用于聚焦的胶垫缓冲液中。每种底物浓度进行三次独立测量。对于pH滴定，蛋白样品稀释至具有相应pH值的缓冲液中（20 mM Tris, 200 mM NaCl pH 7–9.5）。稀释因子至少为200，包括预稀释和胶垫中的最终稀释。每个pH值进行两次独立测量。

### 原位质谱

经纯化的蛋白样品通过离心过滤装置（Amicon Ultra）和三轮浓缩与稀释进行缓冲物质交换到200 mM 醋酸铵中。蛋白质浓度通过紫外吸收（NanoDrop 分光光度计，Thermo Fisher）测定，并以适当的单体浓度稀释成小分子。纳米电喷在Q Exactive UHMR 质谱仪（Thermo Fisher）上正离子模式下进行，使用内部制备的金涂层毛细管和施加适度的背压（约0.5 mbar）。硫化氟气体被引入碰撞“HCD” 电池以提高传输效率，并且仪器在分辨率为6,250（在200 *m/z*处）、“高检测器优化”以及HCD 电池中设置的捕获压力为4的条件下操作。其余参数针对每个样品进行了优化，具体范围包括：1.2–1.5 kV 的毛细管电压；100–250 °C 的毛细管温度；从-15到-150 V 的源内捕获；50–100 ms 的注射时间；以及1–10 次微扫描。质谱图谱通过UniDec^([48](/articles/s41586-024-07287-2#ref-CR48 "Marty, M. T. et al. Bayesian deconvolution of mass and ion mobility spectra: from binary interactions to polydisperse ensembles. Anal. Chem. 87, 4370–4376 (2015)."))进行去卷积处理。

### 动力酶测定法

对于CS动力学测定，基于5,5′-二硝基苯二硫酸（DTNB）的巯基分组的比色定量方法被采用^([49](/articles/s41586-024-07287-2#ref-CR49 "Ellman, G. L. Tissue sulfhydryl groups. Arch. Biochem. Biophys. 82, 70–77 (1959)."),[50](/articles/s41586-024-07287-2#ref-CR50 "Srere, P. A., Brazil, H., Gonen, L. & Takahashi, M. The citrate condensing enzyme of pigeon breast muscle and moth flight muscle. Acta Chem. Scand. 17, 129–134 (1963)."))。在25 °C下，在50 mM Tris pH 7.5，10 mM KCl，0.1 mg ml^(–1) DTNB和25 nM蛋白质浓度下进行了光谱反应。为了测量*K*[m]值，一个底物被饱和并加入到反应混合物中（1 mM草酸乙酰或0.5 mM乙酰辅酶A）。变化的浓度和最后加入反应的另一个底物开始测量动力学测定。对于非饱和底物浓度的动力学测定，蛋白质仅在反应开始前立即稀释，以防止复合物的解离并最后加入反应混合物。通过在412 nm处测量2-硝基-5-硫代苯甲酸的出现（在板读取器Infinite M Nano+，Tecan上使用Tecan i-control（v.3.9.1）中的消光系数14.150 M^(−1) cm^(−1)）。使用GraphPad Prism（v.8.4.3）进行数据分析和确定Michaelis–Menten动力学参数。对于cys4变体的动力学测定，蛋白质在含有谷胱甘肽氧化还原系统的缓冲液中透析，以诱导半胱氨酸残基的二硫键形成（50 mM Na[2]HPO[4]，150 mM NaCl，1 mM谷胱甘肽和0.5 mM谷胱甘肽二硫酸，pH 8）。过夜透析后，部分蛋白质样品用于动力学测定。剩余部分通过在4 °C下与10 mM二硫苏糖醇孵育3 h后还原，并再次用于动力学测定。为了排除处理本身的额外影响，WT SeCS按照相同方式处理（在氧化还原缓冲液中透析和与二硫苏糖醇还原），并进行动力学测定以供比较。

### 盒计数

为了量化分形缩放，我们使用了固定网格扫描。18mer和54mer组装体的类平均图像与非重叠的常规网格（Adobe Illustrator，v.24.0.2）叠加。手动计数了填充结构所需的正方形。这个过程对网格的九个不同框大小（85–17 px）重复进行。对于两种组装体，这个过程在三种不同的网格方向上重复。使用GraphPad Prism（v.8.4.3）进行了线性回归分析。

### *S.* *elongatus*的培养和代谢组学分析样品制备

*S.* *elongatus* PCC 7942 通过同源重组被基因改造以携带 CS 的变种，方法如先前描述^([51](/articles/s41586-024-07287-2#ref-CR51 "Clerico, E. M., Ditty, J. L. & Golden, S. S. Specialized techniques for site-directed mutagenesis in cyanobacteria. Methods Mol. Biol. 362, 155–171 (2007).")). 标准载体 pSyn_6（Thermo Fisher Scientific）作为骨架。通过 PCR 从 WT *S.* *elongatus* PCC 7942 的基因组 DNA 中扩增和提取 CS 基因和相邻同源区域的 1,000 bp 构建同源片段。这些被引入包含用于选择转化子的新霉素抗性基因的 pSyn_6 载体中。将 CS 的相应序列变化（L18Q）引入此载体，以创建相应的同源片段。构建的同源片段（WT，L18Q）被转化到 WT *S.* *elongatus* PCC 7942 中，并在含有 10 µg ml^(–1) 新霉素的 BG11 平板上进行选育。转化子在新 BG11 平板上连续重划，并通过提取基因组 DNA 分析结果表明成功整合。所有菌株通过引入 DNA 区域外部结合的引物进行 PCR 扩增和 Sanger 测序验证。同源片段的所有序列见附表 [3](/articles/s41586-024-07287-2#MOESM1)。

*S.* *elongatus* PCC 7942 培养物和基因改造菌株在 30 °C，100 r.p.m.，环境 CO[2] 浓度和交替光照条件下生长：12 小时光照（光子通量为 120 μmol m^(−2) s^(−1)），12 小时黑暗。在生长实验之前，预培养在昼夜节律条件下同步细胞 5 天。然后从 3 个独立的预培养中建立了 3 个主要培养物（50 ml），接种至光密度 OD[750] 为 0.025 或 0.05\. 为了方便通过注射阀分离培养液，样品用特定烧瓶培养，导致生长行为比标准烧瓶慢。样品在光和暗周期后的 6 个不同时间点（第 3、5 和 7 天）取样进行代谢组学分析。

对于氮缺乏条件下的恢复实验，*S.* *elongatus* 菌株在 BG11 培养基中在全光下生长至光密度 OD[750] 为 0.5，并进行三次重复。然后将细胞转移到无氮源培养基中。为此，细胞用无硝酸盐的 BG11 洗涤两次，然后在无氮的 BG11 中持续培养。细胞在随后的日子里发生叶绿素减退并完全漂白。在第 14 和 20 天，对相应培养物进行序列稀释后，在 BG11 琼脂平板上点斑并继续孵育 7 天进行恢复。

### 代谢组学分析的样品准备

从摇瓶中取出1 ml培养液，并通过注射器立即置于预冷的-80°C甲醇中进行淬灭，甲醇浓度为70%。样品混合后进行离心（10 min，-10°C，13,000*g*）。上清液去除，沉淀物在-80°C保存，直至进行内代谢组提取。每个时间点，使用Coulter计数器（Multisizer 4e，Beckman Coulter）测量每个培养物的细胞数量和大小。计算每个细胞沉淀的生物体积，并用于推断每个样品的标准化提取液量（提取液 = 20,00 × 生物体积）。所有代谢组提取步骤均在冰上和预冷（-20°C）试剂条件下进行。为了提取代谢产物，将计算的提取液量（50%（v/v）甲醇，50%（v/v）TE缓冲液pH 7.0）与相同量的氯仿一起加入到细胞沉淀中。样品在摇动条件下在4°C孵育2小时。然后通过离心（10 min，-10°C，13,000*g*）分离相。上相用注射器提取，并再次添加同等量的氯仿。混合后，样品再次离心（10 min，-10°C，13,000*g*）以去除残留的细胞碎片和色素。上相分离后，加入LC–MS小瓶中，并存储在-20°C直到分析。

### 通过LC–MS/MS对*S.* *elongatus*的细胞内代谢产物进行定量分析。

使用LC–MS/MS对乙酰辅酶A和柠檬酸进行定量测定。色谱分离在Agilent Infinity II 1290 HPLC系统（Agilent）上进行，使用Kinetex EVO C18柱（150×2.1 mm，3μm粒径，100Å孔径，Phenomenex），连接到相似特异性的防护柱（20×2.1 mm，3μm粒径，Phenomenex）。对于乙酰辅酶A，使用恒定流速为0.25 ml/min的移动相，其中相A为50 mM醋酸铵水溶液，pH 8.1，相B为25°C时的100%甲醇。注射体积为1 µl。移动相配置包括以下步骤和线性梯度：0–0.5 min，5% B；0.5–6.5 min，从5%到80% B；6.5–7.5 min，80% B恒定；7.5–7.6 min，从80%到5% B；7.6–10 min，5% B恒定。使用Agilent 6470质谱仪（Agilent），正离子模式，电喷雾离子源（ESI），条件为：ESI喷雾电压4500 V；喷嘴电压1500 V；鞘气温度400°C，流速11 l/min；雾化压力30 psi；干燥气体温度250°C，流速11 l/min。目标分析物根据两个特定质量过渡（810.1 → 428和810.1 → 302.2），在35 V碰撞能量下进行识别，并通过其保留时间与标准物质进行比较。

对于柠檬酸，使用0.1%甲酸水溶液（相A）和0.1%甲酸甲醇（相B）在25°C下的恒定流速为0.2 ml/min。注射体积为10 µl。移动相配置包括以下步骤和线性梯度：0-5 min保持在0% B；5-6 min从0到100% B；6-8 min保持在100% B；8-8.1 min从100到0% B；8.1-12 min保持在0% B。使用Agilent 6495离子漏斗质谱仪（Agilent）在负模式下，带ESI源和以下条件进行操作：ESI喷雾电压为2,000 V；喷嘴电压为500 V；气鞘气体温度为260°C，流量为10 l/min；雾化器压力为35 psi；干燥气体温度为100°C，流量为13 l/min。目标分析物根据标准的两个特定质量转换（191 → 111.1 和 191 → 85.1），在碰撞能量为11和14 V以及其保留时间确定。

色谱图是使用MassHunter软件（Agilent）进行积分的。绝对浓度基于在样品基质中制备的外部校准曲线计算。

### 负染EM

碳包覆铜网格（400网目）通过氩气放电（PELCO easiGlow, Ted Pella）进行亲水处理。接下来，将450 nM的蛋白悬浮液加在亲水化网格上，并在用双蒸馏水短暂冲洗后，用2%醋酸铀进行染色。样品使用加速电压为120 kV的JEOL JEM-2100透射电子显微镜进行分析。使用2k F214 FastScan CCD相机（TVIPS）进行图像采集。或者，使用操作电压为80 kV的JEOL JEM1400 TEM，并配备4k TVIPS TemCam XF416相机。对于2D类平均，手动拍摄图像并使用cisTEM^([52](/articles/s41586-024-07287-2#ref-CR52 "Grant, T., Rohou, A. & Grigorieff, N. CisTEM, user-friendly software for single-particle image processing. eLife 7, e35383 (2018)."))进行处理。以下是各类平均的颗粒数量：18mers平均为1,491颗粒；54mers平均为200颗粒；36mers平均为186颗粒。36mer和54mer颗粒来自于一个扩展数据集，专门寻找更大的组装体。由于具有非常强的取向偏好，大于18mer的复合物的确切百分比难以估算。大多数颗粒似乎不是落在三角形的面上，而是落在其边缘或甚至其顶端之一（扩展数据图 [1](/articles/s41586-024-07287-2#Fig6)）。为了获得估算值，我们收集了另一个150个显微图的数据集，没有偏向更大的组装体。对这些显微图的所有颗粒进行手动计数，并包括那些落在其边缘并呈现为矩形的组装体。通过测量边长，我们可以将它们归类为36mer（30 nm）或54mer（40 nm）。分析显示，在负染色TEM条件下（450 nM），约92.8%的检测到的组装体被鉴定为18mer（1,773颗粒），3.5%为36mer（66颗粒），3.8%为54mer（72颗粒）。对于SeCS的H369R变体，使用了450 nM的蛋白浓度，并平均了136个颗粒，以生成扩展数据图 [6d](/articles/s41586-024-07287-2#Fig11) 中的18mer的2D类平均图。

### 晶体学和结构测定

使用坐滴法在20°C的250 nl滴（Crystal Gryphon, Art Robbins Instruments）中结晶化，包含相等部分的蛋白质和沉淀溶液（Swissci 3 Lens结晶板）。250 µM的蛋白质溶液与5 mM乙酰辅酶在室温下孵育10分钟，诱导其解离成六聚体。结晶条件为0.1 M柠檬酸pH 5.5和2.0 M硫酸铵。在采集数据之前，晶体在液氮中闪冻，使用母液加入20%（v/v）甘油的冷冻溶液。在德国电子同步辐射装置P13下在低温条件下采集数据。数据使用XDS处理并使用XSCALE缩放^([53](/articles/s41586-024-07287-2#ref-CR53 "Kabsch, W. XDS. Acta Crystallogr. Sect. D Biol. Crystallogr. 66, 125–132 (2010).")). 所有结构都通过PHASER（v.1.19.2）进行分子替代，手动在WinCOOT (v.0.9.6)^([55](/articles/s41586-024-07287-2#ref-CR55 "Emsley, P. & Cowtan, K. Coot: model-building tools for molecular graphics. Acta Crystallogr. Sect. D Biol. Crystallogr. 60, 2126–2132 (2004)."))建模，并使用PHENIX (v.1.19.2)^([56](/articles/s41586-024-07287-2#ref-CR56 "Adams, P. D. et al. PHENIX: a comprehensive Python-based system for macromolecular structure solution. Acta Crystallogr. Sect. D Biol. Crystallogr. 66, 213–221 (2010)."))进行精制。结构图像使用PyMOL (v.2.5.2)生成。

### 冷冻电子显微镜

在冷冻电子显微镜样品制备中，取4.5 µl的蛋白样品（22.5 µM），涂抹在预处理的Quantifoil 2/1网格上，使用Vitrobot Mark III（Thermo Fisher）在100%湿度和4°C条件下以力量4轻轻拍干4秒，并且在液氮中冷冻的乙烷中快速冻结。使用FEI Titan Krios透射电子显微镜（Thermo Fisher）和SerialEM软件^([57](/articles/s41586-024-07287-2#ref-CR57 "Mastronarde, D. N. Automated electron microscope tomography using robust prediction of specimen movements. J. Struct. Biol. 152, 36–51 (2005)."))获取了冷冻电子显微镜数据。电影帧以×29,000倍的名义放大率使用K3直接电子探测器（Gatan）录制。每Å²约55电子的总电子剂量在像素大小为1.09 Å的30帧中分布。微图像在从-0.5到-3.0 μm的偏差范围内记录。

### 图像处理、分类和精制

对于SeCS 18mer，所有处理步骤均在cryoSPARC（v.3.2.0）^([58](/articles/s41586-024-07287-2#ref-CR58 "Punjani, A., Rubinstein, J. L., Fleet, D. J. & Brubaker, M. A. CryoSPARC: algorithms for rapid unsupervised cryo-EM structure determination. Nat. Methods 14, 290–296 (2017)."))中完成。共有1,408个电影使用补丁运动校正工具进行了对齐，使用补丁CTF工具确定了对比度传递函数（CTF）参数。通过几轮斑点拾取、2D分类和模板拾取获得了初始的10,173个粒子，用于训练Topaz卷积神经网络粒子拾取模型^([59](/articles/s41586-024-07287-2#ref-CR59 "Bepler, T., Kelley, K., Noble, A. J. & Berger, B. Topaz-Denoise: general deep denoising models for cryoEM and cryoET. Nat. Commun. 11, 5208 (2020)."))。从所有校正后的显微图像中，使用Topaz提取工具和训练模型提取了350x350像素的框大小共273,259个粒子，像素大小为1.09 Å。通过2D分类去除了质量差的粒子后，选择了总共224,041个粒子进行原子模型的初步重建。然后使用异质性精炼工具对初始密度图进行了三维分类和精炼，得到了三个类别。主要类别（占56.7%的粒子）经过另一轮异质性精炼后，分为两个类别。对主类别（占79.8%的粒子）进行了带有C3对称性的三维非均匀精炼，随后进行局部CTF精炼，最终获得了3.93 Å的最终分辨率（GSFSC = 0.143），用于模型构建。使用局部分辨率估计工具计算了密度图的局部分辨率。

对于∆2–6样本，如果它们通过了选择标准（iciness < 1.05，漂移 0.4 Å < *x* < 70 Å，除焦距 0.5 µm < *x* < 5.5 µm，估计的CTF分辨率 < 6 Å），则使用Focus软件包^([60](/articles/s41586-024-07287-2#ref-CR60 "Biyani, N. et al. Focus: the interface between data collection and data processing in cryo-EM. J. Struct. Biol. 198, 124–133 (2017)."))实时处理冷冻电镜微图像。微图像帧使用MotionCor2（ref. ^([61](/articles/s41586-024-07287-2#ref-CR61 "Zheng, S. Q. et al. MotionCor2: anisotropic correction of beam-induced motion for improved cryo-electron microscopy. Nat. Methods 14, 331–332 (2017).")))进行对齐，并使用GCTF^([62](/articles/s41586-024-07287-2#ref-CR62 "Zhang, K. Gctf: real-time CTF determination and correction. J. Struct. Biol. 193, 1–12 (2016)."))确定对齐帧的CTF。从5,419张获取的微图像中，使用crYOLO的Phosaurus神经网络架构挑选出1,687,951个颗粒^([63](/articles/s41586-024-07287-2#ref-CR63 "Wagner, T. et al. SPHIRE-crYOLO is a fast and accurate fully automated particle picker for cryo-EM. Commun. Biol. 2, 218 (2019)."))，像素框大小为256，缩小到96，并使用RELION（v.3.1）^([64](/articles/s41586-024-07287-2#ref-CR64 "Scheres, S. H. W. A Bayesian view on cryo-EM structure determination. J. Mol. Biol. 415, 406–418 (2012)."))提取。颗粒经历了几轮无参考2D分类。总体而言，选择了1,271,457个颗粒（∆2–6），使用256的框大小重新提取，并导入Cryosparc（v.2.3）^([58](/articles/s41586-024-07287-2#ref-CR58 "Punjani, A., Rubinstein, J. L., Fleet, D. J. & Brubaker, M. A. CryoSPARC: algorithms for rapid unsupervised cryo-EM structure determination. Nat. Methods 14, 290–296 (2017)."))。对每个样本生成初探模型，并通过异质分类和精化。选择的颗粒重新导入RELION并经历了几轮精化、CTF精化（估计各向异性放大倍数、每幅图的除焦和散斜校正、束斜估计）和贝叶斯抛光^([65](/articles/s41586-024-07287-2#ref-CR65 "Scheres, S. H. W. Beam-induced motion correction for sub-megadalton cryo-EM particles. eLife 3, e03665 (2014)."))。最终的C1精化生成了估计分辨率为3.1 Å的模型（∆2–6的金标准FSC分析，两个独立半集在0.143截止处）。使用RELION和“Remote 3DFSC Processing Server”网络界面^([66](/articles/s41586-024-07287-2#ref-CR66 "Zi Tan, Y. et al. Addressing preferred specimen orientation in single-particle cryo-EM through tilting. Nat. Methods 14, 793–796 (2017)."))分别计算了局部分辨率和3D FSC图。

对于 H369R SeCS 54mer 和 18mer，所有处理步骤均在 cryoSPARC（v.4.4.0）中完成。总共有 29,126 个电影使用 patch 运动校正工具进行对齐，使用 patch CTF 工具确定 CTF 参数。随后，选择了 8,583 个估计的 CTF 适合度 ≤ 3.5 Å 的显微图进行后续分析。通过运行几轮 Topaz train 和 Topaz extract 从初始手动挑选的 150 个粒子中生成了 Topaz 粒子挑选模型。使用训练模型挑选了总共 95,268 个粒子，并在像素大小为 0.79 Å 的 1,200 x 1,200 像素的框中提取。在进行 2D 分类之前，将粒子下采样到像素大小为 1.58 Å。选择了对应于 SeCS 54mer 的 2D 类别来重建两个密度图，使用从头重建工具。所有提取的粒子通过重新对齐和运行异质细化工具进行 3D 分类，使用与完整 54mer 对应的密度图作为参考。通过非均匀细化进一步细化了 3D 类别（18.0% 粒子），最终分辨率为 5.91 Å（GSFSC = 0.143），用于模型构建。为了重建突变体 18mer，使用对应于 18mer 的 2D 类别作为模板挑选了 899,109 个粒子，并在像素大小为 0.79 Å 的 500 x 500 像素的框中提取。从 2D 分类中选择了 552,353 个粒子生成了 3 个初始地图。对主要类别进行了 3D 分类和对齐，随后进行非均匀细化以生成最终的 18mer 密度为 3.34 Å（GSFSC = 0.143）。使用局部分辨率估计工具计算了密度图的局部分辨率，并使用方位诊断工具评估了优选方向。

对于18meric SeCS，初始模型是根据其蛋白序列分别使用alphaFold^([67](/articles/s41586-024-07287-2#ref-CR67 "Jumper, J. et al. Highly accurate protein structure prediction with AlphaFold. Nature 596, 583–589 (2021)."))生成的，然后作为刚性体结构装配到密度中使用UCSF Chimera。该模型是通过WinCoot（v.0.9.6）^([55](/articles/s41586-024-07287-2#ref-CR55 "Emsley, P. & Cowtan, K. Coot: model-building tools for molecular graphics. Acta Crystallogr. Sect. D Biol. Crystallogr. 60, 2126–2132 (2004)."))手动重建的。非晶体对称性约束在PHENIX（v.1.19.2）^([56](/articles/s41586-024-07287-2#ref-CR56 "Adams, P. D. et al. PHENIX: a comprehensive Python-based system for macromolecular structure solution. Acta Crystallogr. Sect. D Biol. Crystallogr. 66, 213–221 (2010)."))中手动定义，以便在一个六聚体中每个单体与另外两个六聚体中的两个相应单体（对应于18mer的C3对称性细化）之间建立链接。对于Δ2–6六聚体，从18mer模型中提取了一个六聚体亚单位作为精细化的起始模型。该模型首先刚性体适配到密度中，然后在WinCoot（v.0.9.6）^([55](/articles/s41586-024-07287-2#ref-CR55 "Emsley, P. & Cowtan, K. Coot: model-building tools for molecular graphics. Acta Crystallogr. Sect. D Biol. Crystallogr. 60, 2126–2132 (2004)."))中手动进行了精细化。这两个模型都在相应密度图上使用PHENIX（v.1.19.2）的phenix.real_space_refine进行了实空间细化。结构图像使用PyMOL（v.2.5.2）生成。对于SeCS H369R的54mer结构，我们使用从WT 18mer结构中提取的二聚体作为起始模型，并将它们作为刚性体装配到密度中使用UCSF Chimera。然后，使用PHENIX（v.1.19.2）^([56](/articles/s41586-024-07287-2#ref-CR56 "Adams, P. D. et al. PHENIX: a comprehensive Python-based system for macromolecular structure solution. Acta Crystallogr. Sect. D Biol. Crystallogr. 66, 213–221 (2010)."))中的pdbtools截断了所有侧链。然后，结构经过PHENIX中默认参数的实空间细化一轮。对于SeCS H369R的18mer结构，我们还使用18mer SeCS结构作为起始模型。首先，将各个二聚体作为刚性体适配到密度中使用UCSF Chimera。然后，结构经过Namdinator服务器^([68](/articles/s41586-024-07287-2#ref-CR68 "Kidmose, R. T. et al. Namdinator—automatic molecular dynamics flexible fitting of structural models into cryo-EM and crystallography experimental maps. IUCrJ 6, 526–531 (2019)."))中默认参数的一轮柔性拟合，随后使用默认参数在PHENIX中进行了精细化。然后，在WinCoot（v.0.9.6）^([55](/articles/s41586-024-07287-2#ref-CR55 "Emsley, P. & Cowtan, K. Coot: model-building tools for molecular graphics. Acta Crystallogr. Sect. D Biol. Crystallogr. 60, 2126–2132 (2004)."))中手动重建了模型。在该模型中，由于我们地图中分辨率不佳，使得在精细化过程中很难不引入寄存器错误，我们截断了所有链中不属于分形界面的底物盖（残基220–312）。

### SAXS数据的收集与分析

在ESRF的BM29束线上进行了SAXS实验^([69](/articles/s41586-024-07287-2#ref-CR69 "Kieffer, J. et al. New data analysis for BioSAXS at the ESRF. J. Synchrotron Radiat. 29, 1318–1328 (2022)."))，使用了PILATUS3X 2M光子计数探测器（DECTRIS），固定距离为2,827 m。蛋白样品在25 mM Tris-HCl缓冲液pH 7.5和200 mM NaCl的稀释系列中准备。通过透析实现缓冲匹配，所有测量在20 °C进行。样品输送和测量使用了1 mm直径的石英毛细管，这是BioSAXS自动样品更换单元（Arinax）的一部分。每个样品测量前后测量并平均相应的缓冲液。每个样品取10帧（每秒1帧）。所有实验均使用以下参数进行：200 mA的束流电流；样品位置的2.6 × 1,012光子/s的通量；1 Å波长；估计的200 × 200 µm束斑大小。收集的SAXS数据的处理和分析使用了ScÅtter IV^([70](/articles/s41586-024-07287-2#ref-CR70 "Tully, M. D., Tarbouriech, N., Rambo, R. P. & Hutin, S. Analysis of SEC–SAXS data via EFA deconvolution and Scatter. J. Vis. Exp. 167, e61578 (2021)."))。*R*[g]值由Guinier近似确定。使用BioXTAS RAW绘制了SAXS曲线和Guinier区域^([71](/articles/s41586-024-07287-2#ref-CR71 "Hopkins, J. B., Gillilan, R. E. & Skou, S. BioXTAS RAW: improvements to a free open-source program for small-angle X-ray scattering data reduction and analysis. J. Appl. Crystallogr. 50, 1545–1553 (2017)."))。

### 使用18mer构建54mer的原子模型

我们在PyMOL（v.2.5.2）内使用了align、translate和rotate命令来模拟54mer复合物的组装，如果应用了18mer结构中观察到的4.0°和4.2°二聚体旋转以及二聚体之间的60°二面角。旋转应用于构成54mer的三个18mer子复合物的连接二聚体。为此，六聚体的副本被旋转了120°，以便将角二聚体叠加到边二聚体上。随后，两个18mer副本通过两步结构对准连接到旋转角的角。这样，应该形成第三界面的残基彼此之间的距离为210 Å（扩展数据图[2g](/articles/s41586-024-07287-2#Fig7)）。

### *R* [g]值的计算

计算使用GROMACS 2022.2模拟包中的gmx gyrate工具从6mer、18mer和54mer的原子模型中完成*R*[g]值的计算^([72](/articles/s41586-024-07287-2#ref-CR72 "Abraham, M. J. et al. GROMACS: high performance molecular simulations through multi-level parallelism from laptops to supercomputers. SoftwareX 1–2, 19–25 (2015)."))。

### 原子模型的位移向量、旋转轴和二面角

对称轴由AnAnaS^([73](/articles/s41586-024-07287-2#ref-CR73 "Pagès, G. & Grudinin, S. AnAnaS: software for analytical analysis of symmetries in protein structures. Methods Mol. Biol. 2165, 245–257 (2020)."))生成，旋转轴和角度使用PyMOL（v.2.5.2）及其兼容脚本计算。位移向量使用对象参数和cgo-arrow在对齐结构的Cα原子之间绘制。跨分形界面的二聚体之间的二面角在PyMOL（v.2.5.2）中计算。首先使用com命令计算两个二聚体及每个二聚体中的一个单体的质心。然后沿着由两个二聚体质心定义的轴使用get_dihedral计算二面角。

### 报告摘要

研究设计的更多信息请参阅链接到本文的[Nature Portfolio报告摘要](/articles/s41586-024-07287-2#MOESM2)。
