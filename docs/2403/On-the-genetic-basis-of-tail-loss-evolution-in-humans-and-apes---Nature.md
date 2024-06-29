<!--yml

category: 未分类

date: 2024-05-27 14:29:31

-->

# 论人类和类人猿尾部丧失进化的遗传基础 | 自然

> 来源：[https://www.nature.com/articles/s41586-024-07095-8](https://www.nature.com/articles/s41586-024-07095-8)

### 比较基因组学分析尾部发育相关基因

类人猿的进化代表灵长类进化中的一个延长阶段，涉及许多表型变化和广泛的基因组序列变化。因此，跨基因组查询类人猿特异性突变体产生数千万候选者，其中大多数位于非编码区域。我们使用以下标准定义候选变体可能有助于类人猿尾部丧失进化：(1) 必须是类人猿特异性的，这意味着变体序列或氨基酸是类人猿物种独有的，不与任何其他具有尾部的物种共享；(2) 相关基因的功能与尾部发育有关。脊椎动物的尾部发育相关基因收集自MGI表型数据库和MGI数据库未涵盖的额外文献。初始分析主要涵盖从MGI术语MP0003456“缺失尾部”表型提取的基因，共计31个基因。其他分析包括来自MP0002632“退化尾部”（[https://www.informatics.jax.org/vocab/mp_ontology/MP:0002632](https://www.informatics.jax.org/vocab/mp_ontology/MP:0002632)）和MP0003456“短尾部”（[https://www.informatics.jax.org/vocab/mp_ontology/MP:0000592](https://www.informatics.jax.org/vocab/mp_ontology/MP:0000592)）的基因。因此，涉及脊椎动物尾部发育的基因最终清单包括140个基因（截至2023年2月MGI更新），其中报道的突变与尾部减少表型相关（附录数据 [1](/articles/s41586-024-07095-8#MOESM3)）。

从 Ensembl 109 的 BioMart ([https://useast.ensembl.org/info/data/biomart/index.html](https://useast.ensembl.org/info/data/biomart/index.html)) 下载了 140 个基因的基因结构注释。每个基因选择了具有最多外显子的最长转录本。从 UCSC Genome Browser 下载了跨 27 个灵长类物种的 Multiz30way 基因组序列比对。我们选择了所有六种类人类物种（hg38、gorGor5、panTro5、panPan2、ponAbe2 和 nomLeu3）来计算一个类人共识序列，并使用两种非人类人种（猕猴和海豹猴）作为外类群。使用 bedtools (v.2.30.0)^([52](/articles/s41586-024-07095-8#ref-CR52 "Quinlan, A. R. & Hall, I. M. BEDTools: a flexible suite of utilities for comparing genomic features. Bioinformatics 26, 841–842 (2010).")) 从 Multiz30way 比对中提取了这 8 种物种中 140 个基因的同源区域以及上游和下游各 10,000 bp 的序列。使用以下参数鉴定了类人类特异性变异：被六种类人类物种共享但两个外类群猴种中的任何一个不同的 SNVs 或替换被鉴定为可能的类人类特异性 SNVs（补充资料 [2](/articles/s41586-024-07095-8#MOESM4)）；在所有六种类人类物种中存在但两个外类群猴种中不存在的 DNA 序列被鉴定为类人类特异性插入（补充资料 [3](/articles/s41586-024-07095-8#MOESM5)）；在所有六种类人类物种中不存在但两个外类群猴种中存在的 DNA 序列被鉴定为类人类特异性缺失（补充资料 [4](/articles/s41586-024-07095-8#MOESM6)）。值得注意的是，我们分析类人类特异性变异的标准可能包括一小部分假阳性，这些假阳性是外类群特异性变异。

我们使用Ensembl变异效应预测器（集成在Ensembl 109中）^([53](/articles/s41586-024-07095-8#ref-CR53 "McLaren, W. et al. The ensembl variant effect predictor. Genome Biol. 17, 122 (2016)."))来推断检测到的人猿特异性SNV、插入和缺失的潜在功能影响。由于缺乏祖先基因组作为参考序列，变异效应预测器的预测是反向进行的，使用人类/人猿基因组序列作为参考等位基因，外群序列作为替代等位基因。在SIFT评分中标记为‘有害’（<0.05）或在PolyPhen评分中标记为‘有害’（>0.446）的SNV（53个实例）、影响蛋白质序列的插入（6个实例）或缺失（2个实例）被收集进行进一步的跨物种手动检查比较。这些额外的检查是在UCSC基因组浏览器比较基因组模块中的241个物种的Cactus比对中进行的^([51](/articles/s41586-024-07095-8#ref-CR51 "Kent, W. J. et al. The human genome browser at UCSC. Genome Res. 12, 996–1006 (2002)."))。这些检查发现，大多数可能影响宿主基因功能的注释变异可以归为三类：（1）外群特异性变异；（2）在次要转录本中错误注释的变体功能；以及（3）在人猿物种中是错义变异，但在至少一种其他有尾物种中共享相同的氨基酸。这些变异不被认为是可能影响人猿类尾巴丧失进化的候选者。除了这些变异外，我们识别出九个变异作为真正的人猿特异性编码区突变，包括七个SNV和两个插入缺失（补充数据[1](/articles/s41586-024-07095-8#MOESM3)）。在确定首要候选者后，从NCBI数据库下载了代表性脊椎动物物种的蛋白质序列比对，并使用MEGA X软件和默认设置中的MUSCLE算法进行了分析^([54](/articles/s41586-024-07095-8#ref-CR54 "Kumar, S., Stecher, G., Li, M., Knyaz, C. & Tamura, K. MEGA X: molecular evolutionary genetics analysis across computing platforms. Mol. Biol. Evol. 35, 1547–1549 (2018)."))。

### RNA二级结构预测

使用RNAfold（v.2.6.0）通过ViennaRNA Web服务器（[http://rna.tbi.univie.ac.at/](http://rna.tbi.univie.ac.at/)）进行人类*TBXT*外显子5-内含子5-外显子6-内含子6-外显子7序列的RNA二级结构预测^([35](/articles/s41586-024-07095-8#ref-CR35 "Lorenz, R. et al. ViennaRNA Package 2.0\. Algorithms Mol. Biol. 6, 26 (2011)."))。该算法使用默认参数的最小自由能矩阵计算折叠概率。此外，计算还包括分区函数和碱基配对概率矩阵。值得注意的是，人类*TBXT*转录本的5'非翻译区外显子标记使其外显子编号与包括小鼠在内的大多数其他物种不同。为简化起见，我们将人类*TBXT*的第一个编码外显子称为外显子1，因此将其称为外显子6的备选剪接外显子，与小鼠*Tbxt*一致。RNA二级结构预测使用从外显子5到外显子7的DNA序列按此顺序。

### 人类ES细胞培养与分化

人类ES细胞（WA01，也称为H1，来自WiCell Research Institute）由分销商WiCell使用短串联重复序列分析鉴定细胞系。人类ES细胞在不需饲养层的条件下，培养在涂有人类ES细胞合格Geltrex（Gibco, A1413302）的组织培养级板上。Geltrex以1:100稀释于DMEM/F-12培养基（Gibco, 11320033），并添加1× Glutamax（100X, Gibco, 35050061）和1% 青霉素-链霉素（Gibco, 15070063）。在接种人类ES细胞之前，板子在组织培养孵育箱（37 °C 和 5% CO[2]）中至少处理1小时，使用Geltrex工作溶液。

根据制造商的方案，使用StemFlex培养基（Gibco, A3349401）在无饲养层条件下维持和培养人类ES细胞。简而言之，StemFlex完全培养基由StemFlex基础培养基（450 ml）和StemFlex补充剂（10×，50 ml）加上1% 青霉素-链霉素组成。每个涂有Geltrex的6孔板孔中接种20万个细胞，以在3-4天内达到约80%的充实度。人类ES细胞在PSC Cryomedium（Gibco, A2644601）中冷冻保存。当细胞经历压力条件，如冻结-解冻或核转染时，培养基中添加1× RevitaCell（100×，Gibco, A2644501，也包含在PSC Cryomedium套件中）。RevitaCell补充培养基在第二天替换为常规StemFlex完全培养基。在RevitaCell条件下培养的人类ES细胞可能会拉伸，但回到正常的StemFlex完全培养基后会恢复。在我们例行的基于定量PCR的支原体检测中，所有人类ES细胞系均测试为阴性。

用于诱导原始条纹状态基因表达模式的人 ES 细胞分化试验，改编自之前发表的方法^([36](/articles/s41586-024-07095-8#ref-CR36 "Xi, H. et al. In vivo human somitogenesis guides somite development from hPSCs. Cell Rep. 18, 1573–1585 (2017)."))。在第-1天，新鲜培养的人 ES 细胞群体被 Versene 缓冲液（含 EDTA，Gibco, 15040066）分解成团块（2–5 个细胞）。分解的细胞在 Geltrex 包被的 6 孔组织培养板上播种，每平方厘米 25,000 个细胞（在 6 孔板的每孔 0.25 M）。分化至原始条纹状态从第二天（第 0 天）开始，通过将 StemFlex 完整培养基转换为基础分化培养基。基础分化培养基（50 ml）由 48.5 ml DMEM/F-12、1% Glutamax（500 µl）、1% ITS-G（500 µl，Gibco, 41400045）和 1% 青霉素-链霉素（500 µl）制成，并添加 3 µM GSK3 抑制剂 CHIR99021（15 mM DMSO 溶液中的 10 µl；Tocris, 4423）。在分化的第 1 到 3 天收集细胞用于下游实验，确认在 3 天分化期间中胚层基因（*TBXT* 和 *MIXL1*）的表达波动^([36](/articles/s41586-024-07095-8#ref-CR36 "Xi, H. et al. In vivo human somitogenesis guides somite development from hPSCs. Cell Rep. 18, 1573–1585 (2017)."))（扩展数据图 [3](/articles/s41586-024-07095-8#Fig7)）。

### 鼠 ES 细胞培养和分化

鼠 ES 细胞系（MK6），源自 C57BL/6J 鼠株，由纽约大学朗格尼健康啮齿动物基因工程实验室提供。此野生型 MK6 鼠 ES 细胞系验证其在依赖饲养层条件下培养后注射到囊胚中具有贡献能力。本研究中使用的 MK6 鼠 ES 细胞以实验目的为基础，同时在依赖和无饲养层条件下培养。我们常规的基于定量 PCR 的支原体检测结果显示所有鼠 ES 细胞系均为阴性。对于依赖饲养层的鼠 ES 细胞培养条件，将鼠 ES 细胞播种在预先播种的鼠胚成纤维细胞（MEF）细胞（CellBiolabs, CBA-310）的单层上。MEF 包被板通过在涂有 0.1% 明胶溶液（EMD Millipore, ES-006-B）的组织培养板上播种 50,000 个细胞/cm² 制备而成。MEF 培养基由 DMEM（Gibco, 11965118）、10% FBS（GeminiBio, 100–500）、0.1 mM 非必需氨基酸（Gibco, 11140050）、1× Glutamax（Gibco, 35050061）和 1% 青霉素-链霉素（Gibco, 15070063）制成。鼠 ES 细胞培养基由 Knockout DMEM（Gibco, 10829018）、15%（体积/体积）FBS（Hyclone, SH30070.03）、0.1 mM β-巯基乙醇（Gibco, 31350010）、1× MEM 非必需氨基酸（Gibco, 11140050）、1× Glutamax（Gibco, 35050061）、1× 核苷（Millipore, ES-008-D）和 1,000 units ml^(–1) LIF（EMD Millipore, ESG1107）制成。

对于无饲养层小鼠ES细胞培养条件，细胞生长在预先用小鼠ES细胞合格的0.1%明胶（EMD Millipore, ES-006-B）预涂的组织培养板上，室温下至少30分钟。在播种小鼠ES细胞之前，向涂有明胶的板中加入无饲养层小鼠ES细胞培养基，并在37°C和5% CO[2]孵育箱中预热至少30分钟。无饲养层小鼠ES细胞培养基，也称为‘80/20’培养基，体积组成为上述小鼠ES细胞培养基的80%和2i培养基的20%。2i培养基由Advanced DMEM/F-12（Gibco, 12634010）和Neurobasal-A（Gibco, 10888022）1:1混合制成，然后添加1× N2 supplement（Gibco, 17502048）、1× B-27 supplement（Gibco, 17504044）、1× Glutamax（Gibco, 35050061）、0.1 mM β-巯基乙醇（Gibco, 31350010）、1,000 units ml^(–1) LIF（Millipore, ESG1107）、1 µM MEK1/2抑制剂（Stemgent, PD0325901）和3 µM GSK3抑制剂CHIR99021（Tocris, 4423）制备而成。

用于诱导*Tbxt*基因表达的小鼠ES细胞分化方案是从先前描述的一种无饲养层细胞条件下的方法进行调整^([37](/articles/s41586-024-07095-8#ref-CR37 "Pour, M. et al. Emergence and patterning dynamics of mouse-definitive endoderm. iScience 25, 103556 (2022)."))。细胞首先在明胶涂层的6孔板上用80/20培养基培养24小时，然后转换至不含LIF或2i的N2/B27培养基中进行另外2天的培养。N2/B27培养基（50 ml）由18 ml Advanced DMEM/F-12、18 ml Neurobasal-A、9 ml Knockout DMEM、2.5 ml Knockout Serum Replacement（Gibco, 10828028）、0.5 ml N2 supplement、1 ml B27 supplement、0.5 ml Glutamax（100×）、0.5 ml核苷（100×）和0.1 mM β-巯基乙醇制成。然后，N2/B27培养基中加入3 µM GSK3抑制剂CHIR99021诱导分化（第0天）。细胞在分化第-3天收集进行下游实验，显示*Tbxt*基因表达在3天分化期间波动一致。

### CRISPR靶向

所有CRISPR实验的引导RNA均使用集成在UCSC基因组浏览器中的CRISPOR算法设计^([55](/articles/s41586-024-07095-8#ref-CR55 "Concordet, J.-P. & Haeussler, M. CRISPOR: intuitive guide selection for CRISPR/Cas9 genome editing experiments and screens. Nucleic Acids Res. 46, W242–W245 (2018)."))。引导RNA被克隆到pX459V2.0-HypaCas9质粒（AddGene, plasmid 62988）或其定制衍生物中，通过替换抗蓝霉素基因与抗梭菌素基因。本研究中的引导RNA设计成成对，以删除插入序列。小鼠*Tbxt*（*Tbxt*^(*insASAY*)）的AluSx1和AluY插入位点选定，基于引导RNA质量和与人类*TBXT*基因结构的相对距离。RCS元素（*Tbxt*^(*insRCS*)）的插入位点与AluY元素的插入位点相同。CRISPR靶向位点和引导RNA序列列在补充表[2](/articles/s41586-024-07095-8#MOESM1)中。

所有寡核苷酸（加上Golden-Gate装配悬臂）均由Integrated DNA Technologies（IDT）合成，并遵循标准Golden Gate装配协议，使用BbsI限制酶（NEB, R3539）连接到空的pX459V2.0载体中。构建的质粒从3 ml *大肠杆菌*培养物中纯化，使用ZR Plasmid MiniPrep纯化试剂盒（Zymo Research, D4015）进行序列验证。用于转入ES细胞的质粒从250 ml *大肠杆菌*培养物中纯化，使用PureLink HiPure Plasmid Midiprep纯化试剂盒（Invitrogen, K210005）。为了通过核转染促进DNA递送到ES细胞，纯化的质粒在三羟甲基氨基甲烷-乙二胺四乙酸缓冲液（pH 7.5）中溶解，至少浓度为1 µg µl^(–1)，在无菌罩内操作。

### DNA递送

用于CRISPR–Cas9靶向的人类或小鼠ES细胞的DNA递送是使用Nucleofector 2b装置（Lonza, BioAAB-1001）完成的。分别使用Human Stem Cell Nucleofector kit 1（VPH-5012）和小鼠ES细胞Nucleofector kit（Lonza, VVPH-1001）将DNA递送至人类和小鼠ES细胞中。在核转染实验前一天对ES细胞进行双倍喂养以保持优良条件。

在人类ES细胞进行核转染之前，使用0.5 µg cm^(–2) rLaminin-521处理了6厘米组织培养皿，在37°C和5% CO[2]培养箱中至少2小时。rLaminin-521处理的培养皿在单个细胞种植人类ES细胞时提供了最佳的存活率。培养的人类ES细胞用DPBS洗涤，并使用TrypLE Select酶（无酚红; Gibco, 12563011）分离为单个细胞。使用A-023程序，使用Nucleofector 2b设备根据制造商的说明对100万个人类ES细胞单细胞进行了核转染。转染细胞转移到预热的StemFlex完全培养基补充了1× RevitaCell但不含青霉素-链霉素的rLaminin-521处理的6厘米培养皿上。在核转染后24小时使用泊霉素（最终浓度为0.8 μg ml^(–1); Gibco, A1113802）进行抗生素选择。

小鼠ES细胞使用StemPro Accutase（Gibco, A1110501）分离为单个细胞，根据制造商的指示使用A-023程序转染了500万个细胞。小鼠ES细胞中的Exon 6缺失是在无饲料条件下培养的细胞中进行的。核转染细胞被接种在0.1%明胶处理的10厘米培养皿上，随后在核转染后24小时使用噻菌素（最终浓度为7.5 µg ml^(–1); Gibco, A1113903）进行抗生素选择。在依赖饲料条件下，使用小鼠ES细胞在培养基上的MEF细胞单层上接种了Alu*Sx1-AluY对和insRCS工程。小鼠ES细胞被接种在0.1%明胶处理的10厘米培养皿上，随后在核转染后24小时进行抗生素选择。

与用于核转染的pX459V2.0-HypaCas9-gRNA质粒一起，单链DNA寡核苷酸被共同传递，用于微同源诱导的靶向序列删除^([56](/articles/s41586-024-07095-8#ref-CR56 "Chen, F. et al. High-frequency genome editing using ssDNA oligonucleotides with zinc-finger nucleases. Nat. Methods 8, 753–755 (2011)."))。这些单链DNA序列由IDT通过其Ultramer DNA Oligo服务合成，每端包括三个磷硫酸酯键修饰。这些长单链DNA寡核苷酸的详细序列信息列在补充表[3](/articles/s41586-024-07095-8#MOESM1)中。

对于*Tbxt*^(*insASAY*)和*Tbxt*^(*insRCS*)工程，包括选择标记基因*puro*-*ΔTK*（用于阳性选择的贫血霉素抗性基因和阴性选择的简化的单纯疱疹病毒第1型胸苷激酶*ΔTK*，如延伸数据图 [7](/articles/s41586-024-07095-8#Fig11) 所示的那样的同源修复模板质粒），与pX459V2.0-HypaCas9-gRNA质粒一同转染。核转染和抗生素选择（从核转染第二天开始，使用0.8 μg/ml贫血霉素进行3天），随后挑选单克隆，并通过PCR基因分型检测CRISPR–Cas9靶向位点（外显子6缺失，插入AluY，插入AluSx1或插入RCS）。基因分型PCR引物详见附表 [4](/articles/s41586-024-07095-8#MOESM1)。

经PCR基因分型确认的克隆进一步使用捕获测序（见下文）进行验证，确认基因型并排除质粒DNA的任意随机整合可能性。随后，瞬时引入Cre重组酶去除选择标记*puro*-*ΔTK*。细胞用250 nM甘氨酸西韦处理，以逆向选择*ΔTK*阴性细胞作为选择标记基因耗尽的细胞。在没有选择标记基因的*Tbxt*^(*insASAY*)和*Tbxt*^(*insRCS*)小鼠ES细胞中分离出单克隆后，这些克隆用于下游实验，包括体外分化实验和胚胎囊注射以生成小鼠模型。

### 捕获测序基因分型

捕获测序（Capture-seq）或感兴趣位点的靶向测序如先前所述进行^([39](/articles/s41586-024-07095-8#ref-CR39 "Brosh, R. et al. A versatile platform for locus-scale genome rewriting and verification. Proc. Natl Acad. Sci. USA 118, e2023952118 (2021)."))。概念上，捕获测序使用定制的生物素化探针从标准全基因组测序文库中拉下感兴趣的基因组位点序列，从而能够以更高深度进行特定基因组位点的测序，同时降低成本。

从小鼠ES细胞或感兴趣的小鼠的耳部取样中，使用Zymo Quick-DNA Miniprep Plus kit (D4068)按照制造商的方案纯化基因组DNA。使用适用于Illumina测序仪的DNA测序文库，按照标准方案制备。简而言之，使用Covaris LE220 (450 W，10% duty factor，200 cycles per burst和90-s处理时间)在96孔微板中将1 µg的gDNA剪切至500–900碱基对，随后使用DNA Clean and Concentrate-5 kit (Zymo Research, D4013)纯化。剪切和纯化的DNA接受端修复酶混合物处理 (T4 DNA polymerase, Klenow DNA polymerase和T4 polynucleotide kinase, NEB, M0203, M0210和M0201)，并使用Klenow 3′−5′ exo-enzyme (NEB, M0212)加上A尾。随后，将Illumina测序文库接头连接到DNA末端，然后使用KAPA 2X Hi-Fi Hotstart Readymix (Roche, KR0370)进行PCR扩增。

使用BAC DNA和/或质粒作为模板，通过Nick Translation制备定制的生物素化探针作为诱饵。这些探针全面覆盖整个位点。我们使用了从BACPAC Genomics购买的BAC系列RP24-88H3和RP23-159G7，生成覆盖小鼠*Tbxt*位点及其上下游约200 kb序列的诱饵探针。将全基因组测序文库混合物与溶液中的生物素化诱饵杂交，并使用streptavidin涂覆的磁珠纯化。在拉伸下，使用Qubit 3.0 Fluorometer (Invitrogen, Q33216)和dsDNA HS Assay kit (Invitrogen, Q32851)对DNA测序文库进行定量。随后在Illumina NextSeq 500测序仪上进行成对末端测序。

使用Illumina bcl2fastq (v.2.20)对测序结果进行解多重复用，需要与索引BC序列完全匹配。使用Trimmomatic (v.0.39)修剪低质量的reads或碱基和Illumina适配器序列。然后使用bwa (v.0.7.17)将reads映射到小鼠基因组(mm10)上。通过在UCSC Genome Browser镜像版本中可视化来检查*Tbxt*位点及其周围的覆盖和突变。

### 小鼠实验和*Tbxt* ^(*Δexon6/+*)小鼠的生成

所有小鼠实验均遵循NYULH的动物方案指南，并在NYU Langone Health Rodent Genetic Engineering Laboratory进行。小鼠放置在NYU Langone Health BSL1屏障设施中，采用12小时光照和12小时黑暗循环，环境温度和湿度条件。所有实验程序均获得NYU Langone Health的动物护理和使用委员会批准。采用C57BL/6J野生型（品系000664）小鼠，从The Jackson Laboratory获得。

*Tbxt*^(*Δexon6/+*) 杂合小鼠模型是通过将 CRISPR 试剂注入野生型 C57BL/6J 合子（Jackson 实验室品系 000664）进行光合作用微注射而生成的，采用了先前发表的方法^([57](/articles/s41586-024-07095-8#ref-CR57 "Yang, H., Wang, H. & Jaenisch, R. Generating genetically modified mice using CRISPR/Cas-mediated genome engineering. Nat. Protoc. 9, 1956–1968 (2014)."))。简言之，*Cas9* mRNA（MilliporeSigma, CAS9MRNA）、合成的 guide RNA 和单链 DNA 寡核苷酸同时注入到 1 细胞阶段合子中，遵循描述的步骤^([57](/articles/s41586-024-07095-8#ref-CR57 "Yang, H., Wang, H. & Jaenisch, R. Generating genetically modified mice using CRISPR/Cas-mediated genome engineering. Nat. Protoc. 9, 1956–1968 (2014)."))。合成的 guide RNA 是从 Synthego 定制的 CRISPRevolution sgRNA EZ kit 中订购的，具有与小鼠 ES 细胞的 CRISPR 删除实验中使用的相同靶向位点（AUUUCGGUUCUGCAGACCGG 和 CAAGAUGCUGGUUGAACCAG）。同时注射的单链 DNA 寡核苷酸与上述描述的相同。注射的胚胎随后在体外培养至囊胚阶段，然后移植到假孕寄养母体内。在光合作用微注射和移植后，根据它们异常的尾部表型筛选出创始小鼠幼仔。约在第 21 天采集 DNA 样本，进行 PCR 基因分型和 Capture-seq 验证，以排除 *Tbxt* 基因座的非靶向作用。

在确认基因型后，*Tbxt*^(*Δexon6/+*) 创始小鼠与野生型 C57B/6J 小鼠进行了回交，以产生杂合 F[1] 小鼠。由于尾部表型的多样性，F[1] 杂合子之间进行了两类交配：第一类交配包括至少一个父母没有尾巴或尾巴短的情况，而第二类交配是在两个长尾 F[1] 杂合子之间进行的（见表 [1](/articles/s41586-024-07095-8#Tab1)）。如表 [1](/articles/s41586-024-07095-8#Tab1) 所总结的，这两种交配类型都产生了F[2] *Tbxt*^(*Δexon6/+*) 小鼠的异质性尾部表型，从而确认了尾部表型的不完全穿透性和无纯合子（*Tbxt*^(*Δexon6/Δexon6*)）的存在。成年小鼠（>12周）被麻醉，使用 Bruker In-Vivo Xtreme IVIS 成像系统对椎骨进行 X 射线成像。为确认纯合子的胚胎表型，从定期怀孕的母鼠中取出 E11.5 孕期阶段的胚胎，使用标准协议解剖。

### 生成*Tbxt* ^(*insASAY*) 和*Tbxt* ^(*insRCS2*) 小鼠

通过注入经过工程处理的*Tbxt*^(*insASAY*)和*Tbxt*^(*insRCS*)小鼠ES细胞，分别注入到C57BL/6J-白化（查尔斯河实验室，品系493）胚胎囊胚以产生嵌合体F[0]创始小鼠，或注入到B6D2F1/J（C57BL/6J雌性与DBA/2J雄性的F[1]杂交，杰克逊实验室品系100006）四倍体胚胎囊胚以生产纯合子F[0]创始小鼠。四倍体补充策略旨在在F[0]代产生具有提议的基因型的纯合子小鼠^([58](/articles/s41586-024-07095-8#ref-CR58 "王志强, Kiefer, F., Urbánek, P. & Wagner, E. F. 通过四倍体胚胎注射生成完全胚胎干细胞衍生突变小鼠。Mech. Dev. 62, 137–145 (1997)."))。通过多次使用两种小鼠ES细胞系的注射试验，我们仅成功获得了一个*Tbxt*^(*insASAY/insASAY*) F[0]创始小鼠（雄性），但没有*Tbxt*^(*insRCS*)小鼠。然而，在*Tbxt*^(*Δexon6/+*)创始小鼠的基因型筛查期间，我们意外地发现了一只雄性灰色小鼠，其内含子5中插入了一个杂合插入物。基因型分析显示，插入物是*Tbxt*的内含子6的220 bp DNA序列（染色体17：8439335–8439554，mm10），以反向互补的方式插入到设计好的CRISPR靶向位点内含子5（染色体17：8438386，mm10）。因此，在内含子5中插入的序列insRCS2与其原始序列在内含子6中形成了一个220 bp的倒置互补序列对，类似于设计的*Tbxt*^(*insRCS*)和*Tbxt*^(*insASAY*)基因结构。因此，这种基因型被称为*Tbxt*^(*insRCS2*)。这只*Tbxt*^(*insRCS2/+*)小鼠的Capture-seq基因分型证实了*Tbxt*^(*insRCS2*)等位基因是C57BL/6背景，而其*Tbxt*^(*insRCS2/+*)创始小鼠的野生型*Tbxt*基因座来自DBA/2J背景。因此，这只*Tbxt*^(*insRCS2/+*)小鼠被回交到C57BL/6野生型小鼠，并在F[1]杂合子之间进一步杂交以产生F[2]代的纯合子（*Tbxt*^(*insRCS2/insRCS2*)）。*Tbxt*^(*insRCS2/insRCS2*)小鼠的Capture-seq分析证实了它们在*Tbxt*基因座的C57BL/6背景（扩展数据图[8](/articles/s41586-024-07095-8#Fig12)）。我们还比较了年龄匹配的C57BL/6和DBA/2J小鼠的尾部表型，并未发现差异（数据未显示），这表明这两种品系之间的任何遗传背景差异不会影响尾长。因此，这些*Tbxt*^(*insRCS2*)小鼠（杂合子和纯合子）被用于尾部表型分析。

分别将*Tbxt*^(*insASAY*)和*Tbxt*^(*insRCS2*)创始小鼠，均为雄性，与野生型C57B/6J小鼠分开回交以生成杂合子F[1]幼仔，随后在F[1]杂合子之间进行回交以生成F[2]纯合子。有了所有基因型后，跨基因型和性别组别每月测量小鼠尾长。此外，通过在不同*Tbxt*^(*Δexon6/+*)小鼠创始系的后代中执行两种类型的配对，*Tbxt*^(*insRCS2/+*) × *Tbxt*^(*Δexon6/+*)和*Tbxt*^(*insRCS2/insRCS2*) × *Tbxt*^(*Δexon6/+*)，分析尾部表型。这些结果总结并显示在表格[2](/articles/s41586-024-07095-8#Tab2)中。

为了分析小鼠*Tbxt*在胚胎尾芽区的同工型表达模式，采用*Tbxt*^(*insRCS2/+*) × *Tbxt*^(*insRCS2/+*)、*Tbxt*^(*insASAY/+*) × *Tbxt*^(*insASAY/+*)的杂交实验从E10.5胚胎阶段剖析野生型、杂合子和纯合子胚胎。收集每个胚胎的尾芽以分离总RNA，并与用于基因分型的胚胎组织一起。这些结果显示在图[4e](/articles/s41586-024-07095-8#Fig4)中。

### 剪接同工型检测

使用标准柱基净化试剂盒（Qiagen RNeasy Kit, 74004）分别从人类和小鼠ES细胞的未分化和分化细胞，以及胚胎尾芽样本中收集总RNA。在RNA提取过程中进行DNase处理以去除任何可能的DNA污染。提取后，通过基于核糖体RNA完整性的电泳评估RNA质量。每个样品使用1 µg高质量总RNA进行反转录，使用High-Capacity RNA-to-cDNA kit (Applied Biosystems, 4387406)。用于RT-PCR和/或定量RT-PCR的DNA寡核苷酸列在补充表格[1](/articles/s41586-024-07095-8#MOESM1)中。

### 分化小鼠ES细胞的转录组分析

从野生型、*Tbxt*^(*insASAY/insASAY*)、*Tbxt*^(*Δexon6*/*+*)和*Tbxt*^(*Δexon6*/*Δexon6*)基因型的体外分化的小鼠ES细胞中分离的总RNA样本，用于批量RNA测序分析。使用标准柱基净化试剂盒（Qiagen RNeasy kit, 74004）准备RNA样本。每种小鼠ES细胞基因型准备了两个生物学重复样本，其中两个*Tbxt*^(*Δexon6*/*Δexon6*)小鼠ES细胞样本来自不同的克隆。使用NEBNext Ultra II Directional RNA Library Prep kit (NEB, E7765L)通过其Poly(A) mRNA测序工作流程准备RNA测序文库，使用NEBNext Poly(A) mRNA Magnetic Isolation Module (NEB, E7490L)。

使用STAR（v.2.7.2a）比对器将原始测序读数映射到小鼠基因组（mm10）^([59](/articles/s41586-024-07095-8#ref-CR59 "Dobin, A. et al. STAR: ultrafast universal RNA-seq aligner. Bioinformatics 29, 15–21 (2013)."))。所有样本的结果与链特异性读数已整合到矩阵中，以进行下游分析。使用DESeq2（v.1.40.2）检测差异表达基因^([60](/articles/s41586-024-07095-8#ref-CR60 "Love, M. I., Huber, W. & Anders, S. Moderated estimation of fold change and dispersion for RNA-seq data with DESeq2\. Genome Biol. 15, 550 (2014)."))，使用其默认的双侧Wald检验，截断log[2]（折叠表达变化） > 0.5和多重测试调整后的*P* 值 < 0.05。从DESeq2的顶部500个变量基因中跨所有样本进行主成分分析。*Tbxt*靶基因来源于先前的出版物^([41](/articles/s41586-024-07095-8#ref-CR41 "Lolas, M., Valenzuela, P. D. T., Tjian, R. & Liu, Z. Charting Brachyury-mediated developmental pathways during early mouse embryogenesis. Proc. Natl Acad. Sci. USA 111, 4478–4483 (2014)."))，通过在体分化的小鼠ES细胞中检测到显著的*Tbxt* ChIP-seq峰信号来定义。将*Tbxt*靶基因集与每个突变样本中的野生型对照中显著差异表达的基因进行交叉，并将其汇总以生成分析过的小鼠ES细胞系中不同表达的*Tbxt*靶基因的整体集。使用热图可视化这些不同表达的*Tbxt*靶基因，其后跟随log[10]转换的归一化转录物矩阵，跨样本进行*z* 分数标准化。

### 报告摘要

更多研究设计信息请参阅与本文链接的《自然投稿摘要》（[Nature Portfolio Reporting Summary](/articles/s41586-024-07095-8#MOESM2)）。
