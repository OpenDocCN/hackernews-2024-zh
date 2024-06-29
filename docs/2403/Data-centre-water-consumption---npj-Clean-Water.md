<!--yml

category: 未分类

date: 2024-05-27 14:50:41

-->

# 数据中心水消耗 | npj Clean Water

> 来源：[https://www.nature.com/articles/s41545-021-00101-w](https://www.nature.com/articles/s41545-021-00101-w)

2015年美国总水消耗量为每天1218亿升，其中热电能源使用了5030亿升，灌溉用水量为4460亿升，每天1470亿升用于为美国87%的人口提供饮用水^([13](/articles/s41545-021-00101-w#ref-CR13 "Dieter, C. A. et al. Estimated use of water in the United States in 2015\. Report 1441, US Geological Survey, Reston, VA.                    https://doi.org/10.3133/cir1441                                     (2018)."))。

数据中心通过两个主要方式消耗水资源：间接通过电力生成（传统热电能源）和直接通过冷却。2014年，美国数据中心总计使用了6260亿升水^([4](/articles/s41545-021-00101-w#ref-CR4 "Shehabi, A. et al. United States Data Center Energy Usage Report. Tech. Rep. LBNL-1005775, Lawrence Berkeley National Laboratory, California.                    http://www.osti.gov/servlets/purl/1372902/                                     (2016)."))。在如此高的国家水资源消耗背景下，这只占了很小的比例，但数据中心与其他用户竞争获取本地资源。一个中型数据中心（15兆瓦（MW））的水消耗量相当于三个普通大小的医院，或者超过两个18洞高尔夫球场的用水量^([14](/articles/s41545-021-00101-w#ref-CR14 "FitzGerald, D. Data centers and hidden water use. Wall Street Journal.                    https://www.wsj.com/articles/data-centers-1435168386                                     (2015)."))。尽管部分数据中心已开始使用再循环水和非饮用水，但根据有限的数据显示^([15](/articles/s41545-021-00101-w#ref-CR15 "Realty, D. Environmental performance.                    https://www.digitalrealty.com/environmental-social-and-governance-report-2019-highlights/environmental-performance                                     (2019)."))，一些数据中心运营商超过一半的水资源来自饮用水源（见图[2](/articles/s41545-021-00101-w#Fig2)）。这在水资源紧张地区引发了巨大争议，并突显了理解数据中心如何使用水资源的重要性。

**图2：Digital Realty，一个大型全球数据中心运营商的年度水资源来源。**

饮用水消耗分别为64%（2017年）、65%（2018年）和57%（2019年）^([15](/articles/s41545-021-00101-w#ref-CR15 "Realty, D. Environmental performance.                    https://www.digitalrealty.com/environmental-social-and-governance-report-2019-highlights/environmental-performance                                     (2019).")）。

本节考虑数据中心水消耗的这两个类别。

### 电力发电中的水资源利用

水需求基于提取或消耗进行衡量。消耗指的是丢失的水（通常是通过蒸发），而提取指的是从自然表水、地下水、再生水或处理过的饮用水等水源获取，并稍后返回到水源中^([16](/articles/s41545-021-00101-w#ref-CR16 "Pan, S.-Y., Snyder, S. W., Packman, A. I., Lin, Y. J. & Chiang, P.-C. 冷却水在热电发电中的使用及其解决水能源关系挑战。水能源关系 1, 26–41 (2018).")).

发电厂利用化石燃料如煤和天然气，或核裂变发热，将水转化为蒸汽，从而驱动涡轮机发电。水在这一过程中起着关键作用，需要对源水进行预处理以去除腐蚀性污染物，并进行后处理以去除盐类。一旦加热成蒸汽驱动涡轮机，水会通过蒸发损失，排放为废水或循环利用；有时会同时三者兼有^([16](/articles/s41545-021-00101-w#ref-CR16 "Pan, S.-Y., Snyder, S. W., Packman, A. I., Lin, Y. J. & Chiang, P.-C. 冷却水在热电发电中的使用及其解决水能源关系挑战。水能源关系 1, 26–41 (2018).")).

2015年美国电力发电的平均水强度为每千瓦时2.18升（L/kWh）^([17](/articles/s41545-021-00101-w#ref-CR17 "Lee, U., Han, J., Elgowainy, A. & Wang, M. 美国水力和热电发电区域性水消耗。应用能源 210, 661–672 (2018)."))，但燃料和发电机技术类型对冷却水需求有重大影响。例如，天然气联合循环发电机的干冷却系统消耗和提取0.00–0.02 L/kWh，而燃煤蒸汽涡轮的湿冷却（开放循环）系统消耗0.53 L/kWh，并提取132.5 L/kWh。效率有显著差异，消耗量从0.00到4.4 L/kWh，提取量从0.31到533.7 L/kWh，具体取决于系统特性^([16](/articles/s41545-021-00101-w#ref-CR16 "Pan, S.-Y., Snyder, S. W., Packman, A. I., Lin, Y. J. & Chiang, P.-C. 冷却水在热电发电中的使用及其解决水能源关系挑战。水能源关系 1, 26–41 (2018).")).

水电系统尽管被认为是更清洁的电力来源，但也使用大量水资源。开放水库的水蒸发是主要损失源，特别是在干旱地区和水资源未返送水库或流至下游用户的情况下。美国水电的国家平均水消耗量为每千瓦时16.8升，而热电每千瓦时为1.25升^([17](/articles/s41545-021-00101-w#ref-CR17 "李宇，韩杰，Elgowainy，A. & 王明. 美国水电和热电发电的区域水消耗。应用能源 210, 661–672 (2018)."))。

虽然大部分发电仍来自化石燃料^([18](/articles/s41545-021-00101-w#ref-CR18 "国际能源署。2018-2040年燃料和场景电力生成。https://www.iea.org/data-and-statistics/charts/electricity-generation-by-fuel-and-scenario-2018-2040 (2019)."))，向可再生能源转型对于碳和水的强度都至关重要。只有太阳能和风能在发电过程中不需要水，但在制造和建设过程中仍会消耗水资源^([9](/articles/s41545-021-00101-w#ref-CR9 "联合国教科文组织。联合国世界水发展报告2020：水和气候变化。https://unesdoc.unesco.org/ark:/48223/pf0000372985.locale=en。"))。预计到2030年，转向风能和太阳能可以在英国减少50%的水用量，在美国、德国和澳大利亚减少25%，在印度减少10%^([19](/articles/s41545-021-00101-w#ref-CR19 "国际可再生能源机构。水、能源和食品联合体中的可再生能源。技术报告，IRENA。https://www.irena.org/-/media/Files/IRENA/Agency/Publication/2015/IRENA_Water_Energy_Food_Nexus_2015.pdf。"))。

在数据中心领域，谷歌和微软正在引领向可再生能源的转变。2010年至2018年间，服务器数量增加了6倍，网络流量增加了10倍，存储容量增加了25倍，但能耗仅增长了6%^([6](/articles/s41545-021-00101-w#ref-CR6 "Masanet, E., Shehabi, A., Lei, N., Smith, S. & Koomey, J. Recalibrating global data center energy-use estimates. Science 367, 984–986 (2020)."))。这其中的一个主要贡献者是向云计算的迁移，截至2020年，估计其市场规模达到了2360亿美元^([20](/articles/s41545-021-00101-w#ref-CR20 "Adams, J. & Cser, A. Forrester data: cloud security solutions forecast, 2016 To 2021 (global). Tech. Rep., Forrester.                    https://www.tatacommunications.com/wp-content/uploads/2019/02/Forrester-Report.pdf                                     (2017)."))，并负责管理40%的服务器^([4](/articles/s41545-021-00101-w#ref-CR4 "Shehabi, A. et al. United States Data Center Energy Usage Report. Tech. Rep. LBNL-1005775, Lawrence Berkeley National Laboratory, California.                    http://www.osti.gov/servlets/purl/1372902/                                     (2016)."))。

由于规模经济效应，云服务提供商能够投资高效的运营。尽管经常被批评为效率指标^([21](/articles/s41545-021-00101-w#ref-CR21 "Brady, G. A., Kapur, N., Summers, J. L. & Thompson, H. M. A case study and critical assessment in calculating power usage effectiveness for a data centre. Energy Convers. Manag. 76, 155–161 (2013)."))，低电源使用效率（PUE）比率是其效率的指示器之一。PUE是衡量ICT设备与数据中心基础设施（如冷却）能耗分配的指标^([22](/articles/s41545-021-00101-w#ref-CR22 "ISO. ISO/IEC 30134-2:2016\.                    https://www.iso.org/standard/63451.html                                     (2016)."))，定义如下：

$${\rm{PUE}}=\frac{{\rm{数据中心总能耗}}}{{\rm{ICT设备能耗}}}$$

(1)

PUE与了解间接水消耗相关，因为它指示了一个特定设施在其主要用途上的效率：运行ICT设备。这包括服务器、网络和存储设备。理想的PUE为1.0意味着100%的能源用于运行ICT设备上的有用服务，而不是浪费在冷却、照明和电力分配上。水通过电力生产间接消耗，因此更有效地使用电力意味着更有效地使用水资源。

传统数据中心报告的 PUE 从 2010 年的 2.23 降至 2020 年的 1.93^([6](/articles/s41545-021-00101-w#ref-CR6 "Masanet, E., Shehabi, A., Lei, N., Smith, S. & Koomey, J. Recalibrating global data center energy-use estimates. Science 367, 984–986 (2020)."))。相比之下，最大的“超大规模”云服务提供商报告的 PUE 从 1.25 至 1.18 不等。一些报告甚至表现更好，比如谷歌在 2020 年第二季度全球尾随 12 个月的 PUE 为 1.10^([23](/articles/s41545-021-00101-w#ref-CR23 "Google. Efficiency—data centers.                    https://www.google.com/about/datacenters/efficiency/                                     (2020).")).

随着数据中心效率达到如此高水平，进一步的增益变得更加困难。这已经开始反映在平台化的 PUE 数字^([24](/articles/s41545-021-00101-w#ref-CR24 "Lawrence, A. Data center PUEs flat since 2013\.                    https://journal.uptimeinstitute.com/data-center-pues-flat-since-2013/                                     (2020).")) 上，这意味着未来使用量的预期增加可能很快无法通过效率改进来抵消^([25](/articles/s41545-021-00101-w#ref-CR25 "Shehabi, A., Smith, S. J., Masanet, E. & Koomey, J. Data center growth in the United States: decoupling the demand for services from electricity use. Environ. Res. Lett. 13, 124030 (2018)."))。随着更多设备的部署，以及需要更多数据中心来容纳这些设备，能源需求将增加。如果这些能源不来自可再生能源，间接水消耗将增加。

因此，电力发生源是理解数据中心水消耗的关键元素，其中 PUE 是衡量电力使用效率的指标，但仅仅是第一个类别。直接水使用也很重要——所有这些设备都需要冷却，一些老旧设施中的冷却系统可能占据总数据中心能源需求的高达 30%^([26](#ref-CR26 "Iyengar, M., Schmidt, R. & Caricari, J. Reducing energy usage in data centers through control of room air conditioning units. In 2010 12th IEEE Intersociety Conference on Thermal and Thermomechanical Phenomena in Electronic Systems, 1–11 (IEEE, Las Vegas, NV, 2010)."),[27](#ref-CR27 "Ebrahimi, K., Jones, G. F. & Fleischer, A. S. A review of data center cooling technology, operating conditions and the corresponding low-grade waste heat recovery opportunities. Renew. Sustain. Energy Rev. 31, 622–638 (2014)."),[28](/articles/s41545-021-00101-w#ref-CR28 "Capozzoli, A. & Primiceri, G. Cooling systems in data centers: state of art and emerging technologies. Energy Procedia 83, 484–493 (2015).")).

### 数据中心冷却中的水使用

ICT设备产生热量，因此大多数设备必须有机制来管理它们的温度。将冷空气吹过热金属将热能转移给空气，然后将其推送到环境中。这有效的原因是计算机的温度通常高于周围的空气。

在数据中心中，同样的过程也发生，只是规模更大。ICT设备位于一个房间或大厅内，热量通过排气口排出，然后将空气抽出、冷却并再循环使用。数据中心房间的设计温度范围为20–22°C，下限为12°C^([29](/articles/s41545-021-00101-w#ref-CR29 "Miller, C. 能效指南：数据中心温度。                    https://www.datacenterknowledge.com/archives/2011/03/10/energy-efficiency-guide-data-center-temperature                                     (2011)."))。随着温度的升高，设备故障率也会增加，尽管不一定是线性增加^([30](/articles/s41545-021-00101-w#ref-CR30 "Miller, R. Intel：服务器在外部空气条件下运行良好。                    https://www.datacenterknowledge.com/archives/2008/09/18/intel-servers-do-fine-with-outside-air                                     (2008)."))。

数据中心冷却有几种不同的机制^([27](/articles/s41545-021-00101-w#ref-CR27 "Ebrahimi, K., Jones, G. F. & Fleischer, A. S. A review of data center cooling technology, operating conditions and the corresponding low-grade waste heat recovery opportunities. Renew. Sustain. Energy Rev. 31, 622–638 (2014)."), [28](/articles/s41545-021-00101-w#ref-CR28 "Capozzoli, A. & Primiceri, G. Cooling systems in data centers: state of art and emerging technologies. Energy Procedia 83, 484–493 (2015)."))，但一般的方法包括冷却水通过冷却器降低空气温度——通常降到7–10 °C^([31](/articles/s41545-021-00101-w#ref-CR31 "Frizziero, M. Rethinking chilled water temps bring big savings in data center cooling.                    https://blog.se.com/datacenter/2016/08/17/water-temperatures-data-center-cooling/                                     (2016)."))，然后作为传热机制使用。一些数据中心使用冷却塔，外部空气穿过湿介质使水蒸发。风扇排出热湿空气，冷却水被再循环使用^([32](/articles/s41545-021-00101-w#ref-CR32 "Heslin, K. Ignore data center water consumption at your own peril.                    https://journal.uptimeinstitute.com/dont-ignore-water-consumption/                                     (2016)."))。其他数据中心使用绝热经济器，其中直接喷洒水进入空气流动，或者喷洒到热交换表面，冷却进入数据中心的空气^([33](/articles/s41545-021-00101-w#ref-CR33 "Frizziero, M. Why water use is a key consideration when cooling your data center.                    https://blog.se.com/datacenter/2018/05/10/why-water-use-consideration-cooling-data-center/                                     (2018)."))。使用这些技术之一的小型1 MW数据中心每年可使用约2550万升水^([32](/articles/s41545-021-00101-w#ref-CR32 "Heslin, K. Ignore data center water consumption at your own peril.                    https://journal.uptimeinstitute.com/dont-ignore-water-consumption/                                     (2016)."))。

冷却水是能源消耗的主要来源。将制冷器水温从通常的 7–10 °C 提高到 18–20 °C 可以降低 40% 的费用，因为水与空气之间的温差减少了。成本取决于数据中心位置的季节性环境温度。在较冷的地区，需要较少的冷却，而自由空气冷却可以从外部环境中吸入冷空气^([31](/articles/s41545-021-00101-w#ref-CR31 "Frizziero, M. 重新思考数据中心冷却的冷水温度带来大幅节省。https://blog.se.com/datacenter/2016/08/17/water-temperatures-data-center-cooling/ (2016)."))。这也意味着可以使用更小的制冷器，从而将资本支出降低最多 30%^([31](/articles/s41545-021-00101-w#ref-CR31 "Frizziero, M. 重新思考数据中心冷却的冷水温度带来大幅节省。https://blog.se.com/datacenter/2016/08/17/water-temperatures-data-center-cooling/ (2016)."))。Google^([34](/articles/s41545-021-00101-w#ref-CR34 "Miller, R. Google的无制冷剂数据中心。https://www.datacenterknowledge.com/archives/2009/07/15/googles-chiller-less-data-center (2009)."))和Microsoft^([35](/articles/s41545-021-00101-w#ref-CR35 "Miller, R. Microsoft的无制冷剂数据中心。https://www.datacenterknowledge.com/archives/2009/09/24/microsofts-chiller-less-data-center (2009)."))都建立了没有制冷器的数据中心，但在炎热地区这很困难^([36](/articles/s41545-021-00101-w#ref-CR36 "David, M. P. et al. 通过暖水冷却服务器的高效节能无制冷剂数据中心测试设施的实验表征。在2012年第28届IEEE半导体热量测量和管理研讨会(SEMI-THERM)中232–237 (IEEE, San Jose, CA, 2012)."))。
