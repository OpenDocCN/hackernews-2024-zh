<!--yml

分类：未分类

日期：2024-05-27 14:48:08

-->

# 人类活动对北半球雪量减少的证据 | 自然

> 来源：[https://www.nature.com/articles/s41586-023-06794-y](https://www.nature.com/articles/s41586-023-06794-y)

我们采用两种方法评估人为气候变化对春季积雪的影响。首先，我们遵循一种归因方法，使用观测到的历史积雪趋势与气候模型模拟的趋势之间的相关性。其次，我们采用数据-模型融合方法，生成一个大型基于观测的历史积雪集合，并估算在没有人为影响的情况下，冷季温度和降水发生变化后的三月积雪量。前者表明半球性积雪受到的强迫性变化，后者表明水文相关尺度上的强迫性积雪变化。

### 数据

我们的SWE观测集合由来自欧洲中期天气预报中心（ECMWF）的五个长期网格化数据集组成，包括 ERA5-Land 重新分析^([45](/articles/s41586-023-06794-y#ref-CR45 "Muñoz-Sabater, J. et al. ERA5-Land: a state-of-the-art global reanalysis dataset for land applications. Earth Syst. Sci. Data 13, 4349–4383 (2021)."))；日本气象厅的 JRA-55 重新分析^([46](/articles/s41586-023-06794-y#ref-CR46 "Kobayashi, S. et al. The JRA-55 reanalysis: general specifications and basic characteristics. J. Meteorol. Soc. Jpn Ser. II 93, 5–48 (2015)."))；NASA 的 MERRA-2 重新分析^([47](/articles/s41586-023-06794-y#ref-CR47 "Gelaro, R. et al. The Modern-Era Retrospective Analysis for Research and Applications, Version 2 (MERRA-2). J. Clim. 30, 5419–5454 (2017)."))；欧洲空间局的 Snow-CCI, Version 2.0^([48](/articles/s41586-023-06794-y#ref-CR48 "Luojus, K. et al. ESA Snow Climate Change Initiative (Snow_cci): snow water equivalent (SWE) level 3 C daily global climate research data package (CRDP)(1979 –2020), version 2.0\. NERC EDS Centre for Environmental Data Analysis.                    https://doi.org/10.5285/4647cc9ad3c044439d6c643208d3c494                                     (2022)."))；以及 TerraClimate^([49](/articles/s41586-023-06794-y#ref-CR49 "Abatzoglou, J. T., Dobrowski, S. Z., Parks, S. A. & Hegewisch, K. C. TerraClimate, a high-resolution global dataset of monthly climate and climatic water balance from 1958–2015\. Sci. Data 5, 170191 (2018)."))。具有次月时间分辨率的产品是对所有可用的三月值进行平均。我们关注三月，因为在气候学上，这是北半球雪量最大的月份^([20](/articles/s41586-023-06794-y#ref-CR20 "Mudryk, L. et al. Historical Northern Hemisphere snow cover trends and projected changes in the CMIP6 multi-model ensemble. Cryosphere 14, 2495–2514 (2020)."))，并且在三月有大量的现场测量数据可用于我们评估结果。因为基于卫星遥感的 Snow-CCI 产品在山地地形上被屏蔽，所以我们遵循参考文献的方法^([20](/articles/s41586-023-06794-y#ref-CR20 "Mudryk, L. et al. Historical Northern Hemisphere snow cover trends and projected changes in the CMIP6 multi-model ensemble. Cryosphere 14, 2495–2514 (2020)."))，将山地单元格的 SWE 值填充为其他四个数据源的平均值。对于非山地网格单元，我们使用未经修改的 Snow-CCI 数据。此外，我们还使用了来自西部美国的雪盖遥测网络（SNOTEL）的现场 SWE 数据^([50](/articles/s41586-023-06794-y#ref-CR50 "Snowpack Telemetry Network (SNOTEL) (USDA Natural Resources Conservation Service, 2022)."))；加拿大历史雪水当量数据集（CanSWE）^([51](/articles/s41586-023-06794-y#ref-CR51 "Vionnet, V., Mortimer, C., Brady, M., Arnal, L. & Brown, R. Canadian historical Snow Water Equivalent dataset (CanSWE, 1928–2020). Earth Syst. Sci. Data 13, 4603–4619 (2021)."))；以及 NH-SWE 数据集，这是一个半球性数据集，利用一个经过验证的模型将更丰富的雪深观测转换为 SWE^([52](/articles/s41586-023-06794-y#ref-CR52 "Fontrodona-Bach, A., Schaefli, B., Woods, R., Teuling, A. J. & Larsen, J. R. NH-SWE: Northern Hemisphere Snow Water Equivalent dataset based on in situ snow depth time series. Earth Syst. Sci. Data 15, 2577–2599 (2023)."))。只有在 1981 年至 2020 年之间至少有 35 年记录的现场观测才被保留，结果是 SNOTEL 的 550 条记录，CanSWE 的 341 条记录和 NH-SWE 的 2,119 条记录。

降水格点数据来自欧洲中期天气预报中心（ECMWF）的ERA5再分析^([53](/articles/s41586-023-06794-y#ref-CR53 "Hersbach, H. et al. ERA5全球再分析。《皇家气象学会季刊》146，1999–2049（2020）。"))；全球降水气候中心（GPCC）^([54](/articles/s41586-023-06794-y#ref-CR54 "Schneider, U. et al. GPCC全数据再分析版本6.0，0.5°：基于GTS和历史数据的雨量计的月度陆地表面降水。全球降水气候中心 https://doi.org/10.5676/DWD_GPCC/FD_M_V7_050 (2011)."))；MERRA-2^([47](/articles/s41586-023-06794-y#ref-CR47 "Gelaro, R. et al. 现代-时代回顾性研究和应用第2版（MERRA-2）。《气候杂志》30，5419–5454（2017）。"))；多源加权集合降水（MSWEP），第2版^([55](/articles/s41586-023-06794-y#ref-CR55 "Beck, H. E. et al. MSWEP V2全球3小时0.1°降水：方法和定量评估。《美国气象学会公报》100，473–500（2019）。"))；以及TerraClimate^([49](/articles/s41586-023-06794-y#ref-CR49 "Abatzoglou, J. T., Dobrowski, S. Z., Parks, S. A. & Hegewisch, K. C. TerraClimate，1958–2015年全球月气候和气候水平的高分辨率数据集。《科学数据》5，170191（2018）。")). 格点温度数据来自伯克利地球（BEST）^([56](/articles/s41586-023-06794-y#ref-CR56 "Rohde, R. A. & Hausfather, Z. 伯克利地球陆地/海洋温度记录。《地球系统科学数据》12，3469–3479（2020）。"))；美国国家海洋和大气管理局（NOAA）气候预测中心（CPC）全球统一温度^([57](/articles/s41586-023-06794-y#ref-CR57 "CPC全球统一温度。NOAA气候预测中心（2023）。"))；ERA5^([53](/articles/s41586-023-06794-y#ref-CR53 "Hersbach, H. et al. ERA5全球再分析。《皇家气象学会季刊》146，1999–2049（2020）。"))；和MERRA-2^([47](/articles/s41586-023-06794-y#ref-CR47 "Gelaro, R. et al. 现代-时代回顾性研究和应用第2版（MERRA-2）。《气候杂志》30，5419–5454（2017）。")). 每日格点径流数据来自欧洲中期天气预报中心的全球洪水预警系统（GloFAS）^([58](/articles/s41586-023-06794-y#ref-CR58 "Harrigan, S. et al. GloFAS-ERA5运行全球河流流量再分析1979–现在。《地球系统科学数据》12，2043–2060（2020）。")). 分析中使用的所有数据集的详细信息请参见扩展数据表[1](/articles/s41586-023-06794-y#Tab1)。

对于基于气候模型的归因和基于观测的重建，我们分别使用保守的重采样将所有数据重采样为2°×2°和0.5°×0.5°水平分辨率。对于除径流之外的所有数据，如果三月份的雪水当

我们还使用来自12个模型的气候模型输出，这些模型存档了来自工业前控制（PIC）、历史（HIST）、历史自然（HIST-NAT）和SSP2-4.5 CMIP6实验的月度雪水当量（‘snw’）数据，以及来自历史（HIST）、历史自然（HIST-NAT）和SSP2-4.5实验的月度气温（‘tas’）和降水（‘pr’）数据^[27](/articles/s41586-023-06794-y#ref-CR27 "Gudmundsson, L., Seneviratne, S. I. & Zhang, X. Anthropogenic climate change detected in European renewable freshwater resources. Nat. Clim. Change 7, 813–816 (2017)."),[28](/articles/s41586-023-06794-y#ref-CR28 "Padrón, R. S. et al. Observed changes in dry-season water availability attributed to human-induced climate change. Nat. Geosci. 13, 477–481 (2020).")。所有模型输出都与网格观测数据一样被重新网格化和遮蔽。与检测和归因模型比较项目（DAMIP）协议一致，历史模拟在2014年结束，通过使用SSP2-4.5情景将其延伸至2020年^[59](/articles/s41586-023-06794-y#ref-CR59 "Gillett, N. P. et al. The Detection and Attribution Model Intercomparison Project (DAMIP v1.0) contribution to CMIP6\. Geosci. Model Dev. 9, 3685–3697 (2016).")。为简化起见，“历史”（HIST）始终指代这些扩展的时间序列。模型详情请参见扩展数据表[2](/articles/s41586-023-06794-y#Tab2)。

为了在决策意义上提供水文量的估算，我们使用全球径流数据中心的世界主要河流流域数据库^[44](/articles/s41586-023-06794-y#ref-CR44 "GRDC Major River Basins of the World (Federal Institute of Hydrology, 2020).")，将网格级别的数据聚合到河流流域尺度。所有经验估算的雪水当量（SWE）、降水和径流（以毫米或等效的千克每平方米表示）的网格单元值，在计算流域尺度的质量（以千克表示）之前，需乘以网格单元的面积（以平方米表示），然后将流域内所有网格单元的值相加。流域和半球平均温度由所有覆盖雪的网格单元的面积加权平均温度给出。

所有流域人口的估算均使用NASA的社会经济数据和应用中心的2020年值，该数据来自15弧分世界人口格网，第4版（GPWv4）数据集^[60](/articles/s41586-023-06794-y#ref-CR60 "Gridded Population of the World, Version 4 (GPWv4): Population Count (NASA Socioeconomic Data and Applications Center, 2016);                    https://doi.org/10.7927/H4X63JVC                                    .")。

### 将雪水当量趋势归因于人为强迫。

我们的半球性归因方法测试观测到的和气候模式模拟的受迫雪水当量（SWE）趋势之间的相似性是否超过了仅由自然气候变异可能导致的程度^([26](#ref-CR26 "钱成 & 张晓红. 人类活动对中高纬度陆地区温度季节性变化的影响. 气候期刊. 28, 5908–5921 (2015)."),[27](#ref-CR27 "古德蒙松, L., 塞内维拉特尼, S. I. & 张晓红. 在欧洲可再生淡水资源中检测到的人为气候变化. 自然气候变化. 7, 813–816 (2017)."),[28](#ref-CR28 "帕德隆, R. S. 等人. 观察到的人为气候变化对干季水资源供应的影响. 自然地球科学. 13, 477–481 (2020)."),[29](/articles/s41586-023-06794-y#ref-CR29 "格兰特, L. 等人. 将全球湖泊系统变化归因于人为驱动因素. 自然地球科学. 14, 849–854 (2021)."))。为了评估零假设，即HIST模拟中SWE趋势的模式可能仅是自然变异的结果，我们计算每个模型的HIST模拟中1981年至2020年3月SWE趋势的空间模式，并针对这些模型的无迫雨PIC模拟中的每个独特的40年期间生成相同模式的趋势地图（例如，对于500年的PIC模拟，我们生成了461张40年趋势地图）。所有趋势都使用Theil–Sen估计器计算，这是一种更能适应偏斜数据或包含异常值的数据的线性趋势的非参数技术，比使用普通最小二乘回归计算的趋势更稳健。然后，我们计算HIST和PIC趋势的空间地图之间的Spearman（等级）相关系数，以量化模式相似性。结果得到的78601个相关系数的经验分布（图[2](/articles/s41586-023-06794-y#Fig2)上的背景直方图）代表了历史强迫模拟中的模式可能仅来自自然变异的可能性。

我们通过计算观测到的积雪水当量趋势图案与模型估计的受强迫响应的相似度，采用了Spearman空间相关性，即将每个观测产品的趋势图与HIST模拟的多模型平均图进行比较（图[2e](/articles/s41586-023-06794-y#Fig2)中的红色符号）。 对于这个分析，原位观测被聚合到与网格观测和气候模型相同的2° × 2°网格中，方法是计算每个网格单元内所有站点的平均趋势（图[2a](/articles/s41586-023-06794-y#Fig2)）。 如果观测与HIST模拟之间的相关性大于几乎所有HIST和PIC模拟之间的相关性，我们可以拒绝零假设，即观察到的历史模式可能仅仅是由自然变异引起的，并声称观察到的模式存在对历史强迫的响应。 此外，如果我们不能通过观测与仅具有太阳和火山强迫的HIST-NAT模拟之间的相关性来拒绝零假设，那么观察到的模式不太可能是自然辐射强迫的结果。 总的来说，这两条证据线强烈表明，人为强迫是导致观察到的雪量水当量趋势的原因。

### 基于观测的雪量重建

作为衡量历史性积雪水量（SWE）变化的另一种手段，并更好地理解其模式和驱动因素，以更符合雪量减少的影响尺度，我们生成了一个基于大量观测数据的历史性三月份SWE的集合，包括人为影响和不包括人为影响的情况。我们通过使用常见的随机森林机器学习算法来实现这一点，该算法对数据的自举样本拟合随机回归树，并将它们的预测结果平均在一起。决策树框架特别适合捕捉非线性交互作用，例如在雪的背景下温度和降水之间的交互作用，以及相关的预测因子。随机森林算法已被应用于重建各种受温度、降水及其相互作用影响的生物地球物理变量，包括历史径流^([61](/articles/s41586-023-06794-y#ref-CR61 "Ghiggi, G., Humphrey, V., Seneviratne, S. I. & Gudmundsson, L. GRUN: an observation-based global gridded runoff dataset from 1902 to 2014\. Earth Syst. Sci. Data 11, 1655–1674 (2019).")), 农作物产量^([62](/articles/s41586-023-06794-y#ref-CR62 "Vogel, E. et al. The effects of climate extremes on global agricultural yields. Environ. Res. Lett. 14, 054010 (2019).")) 和气候诱导的物种分布范围变化^([63](/articles/s41586-023-06794-y#ref-CR63 "Lawler, J. J., White, D., Neilson, R. P. & Blaustein, A. R. Predicting climate-induced range shifts: model differences and model reliability. Glob. Change Biol. 12, 1568–1584 (2006)."))。在每种情况下，随机森林模型的性能均明显优于其他机器学习算法和更传统的方法，如线性回归。此外，对于重建历史雪储的这种特定应用，该模型不对雨雪分割或雪融化的温度阈值做任何先验假设，这些阈值在空间上可能有很大变化，并且本身就是模拟SWE估计中的不确定性因素之一^([42](/articles/s41586-023-06794-y#ref-CR42 "Jennings, K. S., Winchell, T. S., Livneh, B. & Molotch, N. P. Spatial variation of the rain–snow temperature threshold across the Northern Hemisphere. Nat. Commun. 9, 1148 (2018)."),[64](/articles/s41586-023-06794-y#ref-CR64 "Kim, R. S. et al. Snow Ensemble Uncertainty Project (SEUP): quantification of snow water equivalent uncertainty across North America via ensemble land surface modeling. Cryosphere 15, 771–791 (2021)."))。我们将三月份SWE建模为从前一年十一月到当年三月的平均月温度和累计月降水的函数：

$${{\rm{SWE}}}_{y,i}=f({T}_{y,11,i},{P}_{y,11,i},{T}_{y,12,i},{P}_{y,12,i},{T}_{y,1,i},{P}_{y,1,i},{T}_{y,2,i},{P}_{y,2,i},{T}_{y,3,i},{P}_{y,3,i})$$

(1)

其中SWE[*y*,*i*]是水年（10月至次年9月）*y*在网格单元*i*处的平均三月积雪水当量，*f*是随机森林模型，*T*[*y*,*m*,*i*]是水年*y*和网格单元*i*的月份*m*的平均温度，*P*[*y*,*m*,*i*]是水年*y*和网格单元*i*的月份*m*的总降水量。我们使用0.5° × 0.5°的网格数据的完整时空面板（即，从1981年到2020年的所有网格单元年份）拟合模型，然后将预测的网格值聚合到河流流域尺度。我们发现，在全面板数据上训练单一模型相比在更局部的数据上训练多个模型（例如，每个流域一个模型）有两个主要优势。首先是，在许多人口稠密的中纬度流域（如美国西部、西欧和高山亚洲）中，全面板模型的样本外预测技能显著更高；局部模型在不到20%的流域中更具技能，在人口稀少的高纬度流域中更具集中性，在这些流域中，全面板模型的技能已经很高（扩展数据图[3](/articles/s41586-023-06794-y#Fig7)）。其次，在整个半球的数据上训练单一模型提供了更大的统计稳定性，用于对输入变量进行大幅扰动的预测，例如添加一个世纪末的气候变化信号（扩展数据图[8](/articles/s41586-023-06794-y#Fig12)），这可能超出了局部历史观测的支持，因为记录以不断增加的速度消失^([65](/articles/s41586-023-06794-y#ref-CR65 "Coumou, D., Robinson, A. & Rahmstorf, S. Global increase in record-breaking monthly-mean temperatures. Climatic Change 118, 771–782 (2013)."),[66](/articles/s41586-023-06794-y#ref-CR66 "Rahmstorf, S. & Coumou, D. Increase of extreme events in a warming world. Proc. Natl Acad. Sci. USA 108, 17905–17909 (2011).")）。

为了充分采样和量化雪储量、温度和降水的观测不确定性，并创建足够广泛的可能的雪水当量（SWE）值集合，我们对所有 6 个 SWE（5 个网格化 + 实地观测）组合、4 个温度和 5 个降水数据集（扩展数据表 [1](/articles/s41586-023-06794-y#Tab1)）重复这个过程，从 1981 年到 2020 年提供了盆地尺度 3 月份 SWE 的 120 个估计值（6 × 4 × 5）。我们的集合方法基于两个主要考虑因素。首先，很难确定在水文相关尺度上代表“真实”雪储量的是什么。所有估计空间分布雪储量的方法（例如，遥感或再分析）都有其固有限制，导致对雪量、其变异性和长期趋势存在高水平的分歧^([5](/articles/s41586-023-06794-y#ref-CR5 "Gottlieb, A. R. & Mankin, J. S. Observing, measuring, and assessing the consequences of snow drought. Bull. Am. Meteorol. Soc. 103, E1041–E1060 (2022)."),[6](/articles/s41586-023-06794-y#ref-CR6 "Mortimer, C. et al. Evaluation of long-term Northern Hemisphere snow water equivalent products. Cryosphere 14, 1579–1594 (2020)."))，正如我们在图 [1](/articles/s41586-023-06794-y#Fig1) 中展示的那样。实地观测可能代表收集到的位置上的真相，但很难推广，特别是在复杂地形中。因此，使用这些点观测来裁决哪些网格化产品（其值代表几十到数万公里的平均值）最接近“真相”是具有挑战性的。鉴于无法知道雪储量的真实状态或严格排除其各种网格化估计中的任何一个，我们选择将这些观测产品视为同样有效的真实估计，在其中我们可以尝试识别共享响应。其次，集合方法使我们能够捕捉 SWE 如何对温度和降水变化的结构性不确定性，这些变化本身也受到数据不确定性的影响（补充图 [2](/articles/s41586-023-06794-y#MOESM1)）。使用所有数据集组合，我们可以对 SWE、温度和降水及它们彼此之间的协方差的不确定性进行采样和表征。这种方法已被用于估计地球系统组成部分的受迫变化，其中感兴趣的因变量和自变量本身也是不确定的^([32](/articles/s41586-023-06794-y#ref-CR32 "Yao, F. et al. Satellites reveal widespread decline in global lake water storage. Science 380, 743–749 (2023)."),[67](/articles/s41586-023-06794-y#ref-CR67 "Zumwald, M. et al. Understanding and assessing uncertainty of observational climate datasets for model evaluation using ensembles. Wiley Interdiscip. Rev. Clim. Change 11, e654 (2020).")）。

我们通过此过程生成的模型预测时间序列与训练模型的观测SWE产品进行比较，使用常见的*R*²和RMSE指标（扩展数据图[4](/articles/s41586-023-06794-y#Fig8)）。此外，由于分析的重点是SWE的长期趋势，我们将重建的趋势与研究期间的观察到的趋势进行比较，并发现我们的模型相当好地复制了趋势的空间模式和幅度相当好，所有数据产品的相关性都在0.9和0.97之间（扩展数据图[3](/articles/s41586-023-06794-y#Fig7)）。此外，建设模型预测的RMSE在1981年至2020年的10个最冷、10个最暖和20个“平均”年份中是可比的，表明重建即使在极端年份中也是稳定的（补充图[4](/articles/s41586-023-06794-y#MOESM1)）。

作为模型技能的额外测试，我们使用仅在格网观测产品上训练的模型来预测SNOTEL、CanSWE和NH-SWE数据集中2961个现场站点的完全超出样本的三月SWE。我们的重建能够相当好地捕捉现场SWE的年际变化，站点的中位数*R*²为0.59，RMSE约为22%（扩展数据图[5](/articles/s41586-023-06794-y#Fig9)）。重建模型预测同样能够巧妙地捕捉现场站点的长期SWE趋势，模式相关性为0.72（扩展数据图[5](/articles/s41586-023-06794-y#Fig9)）。最后，至关重要的是，我们确认我们的重建相对于现场观测没有系统性的时间偏差趋势（补充图[5](/articles/s41586-023-06794-y#MOESM1)），表明重建模型相当忠实地捕捉了雪堆的实际变化率。

### 对事实雪堆重建

为了确定人为气候变化如何在影响相关尺度上改变了春季雪盖，我们将基于观测的重建与气候模型模拟相结合，这些重建在捕捉历史上影响相关尺度上的积雪水当量趋势方面非常有技巧性，从而使我们能够估计温度和降水的受迫变化。这种数据-模型融合方法已被用于将人为影响的变化归因于各种系统，包括物理系统（例如，土壤湿度^([8](/articles/s41586-023-06794-y#ref-CR8 "Williams, A. P. et al. Large contribution from anthropogenic warming to an emerging North American megadrought. Science 368, 314–318 (2020)."),[31](/articles/s41586-023-06794-y#ref-CR31 "Williams, A. P., Cook, B. I. & Smerdon, J. E. Rapid intensification of the emerging southwestern North American megadrought in 2020–2021\. Nat. Clim. Change 12, 232–234 (2022).")), 火灾^([30](/articles/s41586-023-06794-y#ref-CR30 "Abatzoglou, J. T. & Williams, A. P. Impact of anthropogenic climate change on wildfire across western US forests. Proc. Natl Acad. Sci. USA 113, 11770–11775 (2016).")) 和湖泊储水量^([32](/articles/s41586-023-06794-y#ref-CR32 "Yao, F. et al. Satellites reveal widespread decline in global lake water storage. Science 380, 743–749 (2023)."))) 以及社会经济系统（例如，作物保险赔偿^([33](/articles/s41586-023-06794-y#ref-CR33 "Diffenbaugh, N. S., Davenport, F. V. & Burke, M. Historical warming has increased U.S. crop insurance losses. Environ. Res. Lett. 16, 084025 (2021).")) 和气候损失^([34](/articles/s41586-023-06794-y#ref-CR34 "Callahan, C. W. & Mankin, J. S. National attribution of historical climate damages. Climatic Change 172, 40 (2022).")))。

我们计算人为强迫响应的温度作为每月HIST和HIST-NAT运行的30年滚动平均温度之间的差异。对于降水，我们计算强迫响应作为HIST与HIST-NAT中30年滚动平均月降水的百分比差异。通过从同一模型的实验中进行差异化，我们希望限制气候温度和降水模型偏差的影响，因为每个模型都是基准其自身的气候学。然而，模型模拟的趋势中的系统偏差（例如，过快的变暖或湿润）可能导致过高或过低估计强迫响应。为了解决这种可能性，我们评估1981年至2020年冬季温度和降水的模型偏差与观测趋势的差异，方法是计算CMIP6 HIST集合平均值与每个数量的观测产品平均值之间的差异（扩展数据图[6](/articles/s41586-023-06794-y#Fig10)）。为了测试观察到的和模拟的趋势是否一致，我们询问观察到的趋势是否落在强迫加内部变异的合理范围内，即CMIP6 HIST趋势的2.5-97.5百分位数范围。只有1%（3%）的网格单元落在这个范围之外的温度（降水）表明气候模型捕捉到了真实的历史气候趋势。

通过估算网格温度和降水的人为迫使变化，我们通过将输出降解到观测集合的0.5° × 0.5°分辨率，使用保守的重新网格化，并从每个网格温度和降水数据集的每个模型实现中删除强制响应，创建温度和降水的反事实时间序列。通过从观测中减去强迫变化来调整温度，通过强制百分比变化来调整降水。然后，我们使用历史数据训练的重建模型（方程式([1](/articles/s41586-023-06794-y#Equ1))）来使用反事实温度和降水数据预测三月的SWE，从而估计在没有人为气候变化的情况下SWE将会是什么样子。此外，我们通过仅从观测中删除温度或降水之一的强制响应，而将另一个保持在其观察到的历史值，来分别隔离温度和降水的强迫变化的影响。然后，这些网格反事实重建类似地聚合到流域尺度，并使用Theil–Sen估计器计算这些反事实场景的SWE的线性趋势。温度和降水单独强迫变化的效应（图[3c,d](/articles/s41586-023-06794-y#Fig3)）和组合效应（图[3e](/articles/s41586-023-06794-y#Fig3)）被计算为每个历史趋势和相同SWE–温度–降水数据集组合的反事实趋势之间的差异。对于120个重建集合成员中的每一个，我们对人为影响估计有101个（来自每个气候模型实现的一个；扩展数据表[2](/articles/s41586-023-06794-y#Tab2)），每个流域总共有12,120个估计。仅使用每个气候模型的第一个实现，而不是所有可用的运行，产生几乎相同的结果（补充图[6](/articles/s41586-023-06794-y#MOESM1)）。

为进一步测试利用温度和降水的强迫性变化来估计反事实 SWE 的方法的有效性，我们在‘完美模型’框架中重复此协议，仅使用气候模型输出。对于每个模型，我们使用来自 CMIP6 HIST 模拟在 1981 年至 2020 年期间的 SWE、温度和降水数据，而不是观测数据，来拟合方程 ([1](/articles/s41586-023-06794-y#Equ1)) 中描述的经验模型。然后，我们使用在这些 HIST 数据上训练的随机森林来使用 HIST-NAT 模拟的温度和降水来预测反事实 SWE。最后，我们将从重建方法计算的强制性（HIST 减去 HIST-NAT）趋势与使用来自 HIST 和 HIST-NAT 气候模型实验的直接 SWE 输出计算的‘真实’强制性趋势进行比较（扩展数据图 [9](/articles/s41586-023-06794-y#Fig13) 和补充图 [7](/articles/s41586-023-06794-y#MOESM1)）。‘真实’和重建的强制性响应模式的强烈相似性表明，使用移除了温度和降水强迫变化的观测产生了强制性 SWE 变化的合理估计。

### 不确定性量化

上述详细方法产生了对 169 个主要流域中 3 月雪盖趋势的气候变化影响的 12,120 个估计值。这些估计的扩展涉及四个主要不确定性来源：（1）用于重建的 SWE 数据产品的不确定性；（2）温度和降水数据产品以及它们与 SWE 的关系的不确定性；（3）由于气候模型之间的结构差异而导致的温度和降水强制响应的差异；以及（4）由于温度和降水的内部气候变异引起的不确定性。

为了量化每个来源引入的不确定性的大小，我们计算了在单个维度上强制 SWE 趋势的标准偏差，将其他所有维度保持在其均值。例如，由于模型结构差异导致的强制温度和降水响应的不确定性由 12 个气候模型（仅考虑每个模型的第一个实现）的强制 SWE 趋势的标准偏差给出，考虑所有 SWE–温度–降水数据集组合的平均值。

为了孤立温度和降水内部变异的不确定性，我们使用 MIROC6 模型^([68](/articles/s41586-023-06794-y#ref-CR68 "Tatebe, H. et al. Description and basic evaluation of simulated mean state, internal variability, and climate sensitivity in MIROC6\. Geosci. Model Dev. 12, 2727–2765 (2019).")) 中的 50 对 HIST 和 HIST-NAT 模拟，这些模拟仅在初始条件上有所不同。我们对所有 50 个实现的强制 SWE 趋势进行标准偏差计算，并在所有 SWE、温度和降水数据产品组合上取平均值。

与以前在不确定性分区中的工作一致，我们认为流域*b*中受迫SWE趋势的总不确定性*U*是所有四个来源的总和：

$${U}_{b}={S}_{b}+{{\rm{T}}{\rm{P}}}_{b}+{M}_{b}+{I}_{b}$$

(2)

其中，*S*是SWE观测的不确定性，TP是温度和降水观测的不确定性，*M*是模型结构的不确定性，*I*是内部变异的不确定性。为了评估每个流域不确定性的主要来源，我们考虑每个来源的分数不确定性（例如，*S*[*b*]/*U*[*b*]表示流域*b*归因于SWE观测不确定性的不确定性比例）。这种分数不确定性报告在附图12中。对于每个来源，我们对不确定性量不足以改变受迫SWE趋势的集合平均估计的符号的流域进行阴影处理（即，信噪比>1）。

### 积雪覆盖的温度敏感性

为了更好地理解SWE的异质空间响应的驱动因素以及随着进一步升温可能的未来变化，我们评估了原位观测、网格观测、我们的流域尺度重建和气候模型中气候学冬季温度梯度下三月SWE的温度敏感性。额外加热程度的边际效应，∂SWE/∂*T*或*β*[1]，计算为三月SWE对冷季（11月至3月）温度的回归系数：

$${{\rm{SWE}}}_{y,i}={\beta }_{0,i}+{\beta }_{1,i}{T}_{y,i}$$

(3)

在水年*y*中，SWE[*y*,*i*]表示单元*i*（原位站点、网格单元或流域）的三月积雪水当量，而*T*[*y*,*i*]表示相同单元中的平均冷季温度。我们在每个原位位置、20种网格化SWE和温度产品的所有组合、所有12个气候模型（使用HIST模拟）以及所有120个流域尺度的重建中运行此回归。然后，我们在滚动的5°温度窗口中计算给定类型数据（原位、网格观测、气候模型和流域尺度重建）的所有系数的平均值和标准偏差，以生成图4a中的曲线。因此，不确定性估计包括参数和数据不确定性。

### 受积雪覆盖的径流变化

为了评估人为造成的积雪减少对水资源安全的差异性影响，我们量化由于强制性3月SWE变化而引起的春季（4月–7月）径流变化。我们再次使用随机森林算法，将4月–7月的径流建模为前一年11月至7月的月温度和降水的函数：

$${Q}_{y,b}=f({{\rm{SWE}}}_{y,b},{T}_{y,11,b},{P}_{y,11,b},{T}_{y,12,b},{P}_{y,12,b},\ldots ,{T}_{y,7,b},{P}_{y,7,b})$$

(4)

其中*Q*[*y*,*b*]是水年（10月–9月）*y*中*水文年*（4月–7月）的总径流，SWE[*y*,*b*]是水年*y*中*水文年*（3月平均）*b*盆地的积雪水当量（SWE重建使用盒格点级别进行拟合并汇总到盆地尺度，而径流模型则使用盆地尺度数据进行拟合），*T*[*y*,*m*,*b*]是水年*y*中*月* *m*的*面积加权*盆地平均温度，*P*[*y*,*m*,*b*]是水年*y*中*月* *m*的*总*盆地尺度降水量。我们使用所有120种SWE–温度–降水数据集组合和GloFAS径流数据（扩展数据表[1](/articles/s41586-023-06794-y#Tab1)）来拟合此模型。我们使用与验证我们的SWE重建相同的方法来评估模型技能（扩展数据图[10](/articles/s41586-023-06794-y#Fig14)）。

类似于上述盆地尺度的3月积雪水量归因，由于对积雪的强迫性温度和降水变化的影响而引起的春季径流变化由估算的历史SWE和估算的强制性温度和降水变化的效应之间的差异给出。

### 未来积雪和径流变化

为了更好地理解未来由气候变暖引起的积雪水量变化对水资源的影响，我们将我们的统计模型和未来温度和降水变化的预测结合起来，以在SSP2-4.5强迫情景下产生本世纪末（2070–2099年）的积雪量估算。具体来说，我们使用一种“增量”方法，通过将每个月的观测气候与气候模型中的本世纪末和历史（1981–2020年）气候之间的差异进行调整来调整气候学。我们通过加法调整温度，并按照历史和未来气候之间的百分比变化调整降水。然后，我们使用调整后的数据和在历史数据上训练的方程中描述的模型（方程（[1](/articles/s41586-023-06794-y#Equ1)））对未来气候学积雪进行预测。

由于SWE变化引起的未来径流变化通过方程（[4](/articles/s41586-023-06794-y#Equ4)）计算，但替换历史SWE的未来SWE气候学估计，同时保持温度和降水在其观测历史气候学值。

### 积雪的主导作用

要预先确定在图[1](/articles/s41586-023-06794-y#Fig1)中被认为是以雪为主的河流流域，我们使用了水年（十月至九月）累积降雪量与径流的比值 *R*^([1](/articles/s41586-023-06794-y#ref-CR1 "Barnett, T. P., Adam, J. C. & Lettenmaier, D. P. Potential impacts of a warming climate on water availability in snow-dominated regions. Nature 438, 303–309 (2005)."))，该比值是从 ERA5-Land^([45](/articles/s41586-023-06794-y#ref-CR45 "Muñoz-Sabater, J. et al. ERA5-Land: a state-of-the-art global reanalysis dataset for land applications. Earth Syst. Sci. Data 13, 4349–4383 (2021).")) 计算得出的。 平均 *R* 大于 0.5 的流域被认为是雪融水为主。
