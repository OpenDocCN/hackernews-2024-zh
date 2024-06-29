<!--yml

category: 未分类

date: 2024-05-27 14:32:04

-->

# 使用深度强化学习避免聚变等离子体撕裂不稳定性 | 自然

> 来源：[https://www.nature.com/articles/s41586-024-07024-9](https://www.nature.com/articles/s41586-024-07024-9)

### DIII-D

美国圣迭戈的通用原子公司（General Atomics）设有DIII-D国家聚变设施，是一所领先的研究机构，致力于通过实验和理论研究推动聚变能领域的发展。该设施拥有DIII-D托卡马克，是美国最大、最先进的磁约束聚变装置。DIII-D的主半径和次半径分别为1.67米和0.67米。其环向磁场可达2.2特斯拉，等离子体电流可达2.0兆安培，外部加热功率可达23兆瓦。DIII-D配备了高分辨率的实时等离子体诊断系统，包括汤姆逊散射系统^([45](/articles/s41586-024-07024-9#ref-CR45 "Carlstrom, T. N. et al. Design and operation of the multipulse Thomson scattering diagnostic on DIII-D (invited). Rev. Sci. Instrum. 63, 4901–4906 (1992)."))、电荷交换复合谱学^([46](/articles/s41586-024-07024-9#ref-CR46 "Seraydarian, R. P. & Burrell, K. H. Multichordal charge-exchange recombination spectroscopy on the DIII-D tokamak. Rev. Sci. Instrum. 57, 2012–2014 (1986)."))和EFIT磁流体力学重建^([37](/articles/s41586-024-07024-9#ref-CR37 "Shousha, R. et al. Machine learning-based real-time kinetic profile reconstruction in DIII-D. Nucl. Fusion 64, 026006 (2024)."),[39](/articles/s41586-024-07024-9#ref-CR39 "Ferron, J. et al. Real time equilibrium reconstruction for tokamak discharge control. Nucl. Fusion 38, 1055 (1998)."))。这些诊断工具允许对电子密度、电子温度、离子温度、离子旋转、压力、电流密度和安全因子进行实时分析。此外，DIII-D能够通过对八个不同方向的中性束进行可靠高频调制，实现灵活的总束功率和扭矩控制。因此，DIII-D是验证和利用我们的AI控制器的最佳实验设备，该控制器能够实时观测等离子体状态并操作执行器。

### 等离子体控制系统

DIII-D 托卡马克的独特特点之一是其先进的 PCS^([47](/articles/s41586-024-07024-9#ref-CR47 "Margo, M. et al. Current state of DIII-D plasma control system. Fusion Eng. Des. 150, 111368 (2020)."))，这使得研究人员能够实时精确控制和操作等离子体。这使得研究人员能够在广泛的条件下研究等离子体的行为，并测试控制和稳定等离子体的想法。PCS 包括一个分层结构的实时控制器，从磁控制系统（低级控制）到轮廓控制系统（高级控制）。我们的撕裂避免算法也实现在 DIII-D PCS 的这个分层结构中，并与现有的低级控制器集成，如等离子体边界控制算法^([39](/articles/s41586-024-07024-9#ref-CR39 "Ferron, J. et al. Real time equilibrium reconstruction for tokamak discharge control. Nucl. Fusion 38, 1055 (1998)."),[41](/articles/s41586-024-07024-9#ref-CR41 "Barr, J. et al. Development and experimental qualification of novel disruption prevention techniques on DIII-D. Nucl. Fusion 61, 126019 (2021).")) 和个体束控制算法^([40](/articles/s41586-024-07024-9#ref-CR40 "Boyer, M., Kaye, S. & Erickson, K. Real-time capable modeling of neutral beam injection on NSTX-U using neural networks. Nucl. Fusion 59, 056008 (2019)."))。

### 撕裂不稳定性

磁重连接是指在磁化等离子体中，由于等离子体电阻性扩散导致磁场线被撕裂并重新连接的现象。这种磁重连接是一种普遍事件，发生在多种环境中，如太阳大气层、地球磁层、等离子体推进器以及托卡马克等实验室等离子体。在托卡马克的嵌套磁场结构中，当磁流函数 *q* 变成有理数时，在表面处的磁重连接会导致分离的磁场线形成磁岛。当这些岛屿增长并变得不稳定时，被称为撕裂不稳定性。撕裂不稳定性的增长速率经典上依赖于撕裂稳定性指数 *Δ*′，如方程式 ([2](/articles/s41586-024-07024-9#Equ2)) 所示。

$${\varDelta }^{{\prime} }\equiv {\left[\frac{1}{\psi }\frac{{\rm{d}}\psi }{{\rm{d}}x}\right]}_{x=0-}^{x=0+}$$

(2)

当*Δ*′为正时，磁拓扑变得不稳定，允许（经典）撕裂不稳定性发展。然而，即使*Δ*′为负（经典撕裂不稳定性不会增长），由于几何效应或带电粒子的漂移效应，‘新经典’撕裂不稳定性也可能产生，这些效应可以放大种子扰动。随后，改变的磁拓扑可以要么达到饱和，无法进一步生长^([48](/articles/s41586-024-07024-9#ref-CR48 "Escande, D. & Ottaviani, M. Simple and rigorous solution for the nonlinear tearing mode. Phys. Lett. A 323, 278–284 (2004)."),[49](/articles/s41586-024-07024-9#ref-CR49 "Loizu, J. et al. Direct prediction of nonlinear tearing mode saturation using a variational principle. Phys. Plasmas 27, 070701 (2020)."))，或者可以与其他磁流体力学事件或等离子体湍流耦合^([50](#ref-CR50 "Muraglia, M. et al. Generation and amplification of magnetic islands by drift interchange turbulence. Phys. Rev. Lett. 107, 095003 (2011)."),[51](#ref-CR51 "Hornsby, W. A. et al. On seed island generation and the non-linear self-consistent interaction of the tearing mode with electromagnetic gyro-kinetic turbulence. Plasma Phys. Control. Fusion 57, 054018 (2015)."),[52](#ref-CR52 "Agullo, O. et al. Nonlinear dynamics of turbulence driven magnetic islands. I. Theoretical aspects. Phys. Plasmas 24, 042308 (2017)."),[53](/articles/s41586-024-07024-9#ref-CR53 "Choi, G. J. & Hahm, T. S. Long term vortex flow evolution around a magnetic island in tokamaks. Phys. Rev. Lett. 128, 225001 (2022)."))。理解和控制这些撕裂不稳定性对于在托卡马克中实现稳定和可持续的聚变反应至关重要^([54](/articles/s41586-024-07024-9#ref-CR54 "Sauter, O. et al. Marginal β-limit for neoclassical tearing modes in JET H-mode discharges. Plasma Phys. Control. Fusion 44, 1999 (2002)."))。

### ITER基线情景

ITER基线方案（IBS）是为ITER设计的运行条件，旨在实现聚变功率*P*[fusion] = 500 MW，并且聚变增益*Q* ≡ *P*[fusion]/*P*[external] = 10，持续时间超过300 s（参见 ^([12](/articles/s41586-024-07024-9#ref-CR12 "Shimada, M. et al. Progress in the ITER physics basis—chapter 1: overview and summary. Nucl. Fusion 47, S1 (2007)."))）。与当前托卡马克实验相比，IBS条件以其显著较低的边缘安全因子（*q*[95] ≈ 3）和扭矩而闻名。在PCS的帮助下，DIII-D比其他设备更容易进入这种IBS状态；然而，观察到许多IBS实验被破碎性撕裂不稳定性终止^([19](/articles/s41586-024-07024-9#ref-CR19 "Turco, F. et al. The causes of the disruptive tearing instabilities of the ITER baseline scenario in DIII-D. Nucl. Fusion 58, 106043 (2018)."))。这是因为当*q*[95]较低时，在*q* = 2的表面附近出现的撕裂不稳定性很容易锁定在壁上，当等离子体旋转频率低时会导致破碎。因此，在本研究中，我们进行了实验，以测试在*q*[95] ≈ 3和低扭矩（≤1 Nm）条件下，AI撕裂控制器易于被激发的破碎性撕裂不稳定性。

然而，除了撕裂不稳定性是关键问题的IBS外，还有其他情景，如ITER的混合和非感应情景^([12](/articles/s41586-024-07024-9#ref-CR12 "Shimada, M. et al. Progress in the ITER physics basis—chapter 1: overview and summary. Nucl. Fusion 47, S1 (2007)."))。这些不同的情景不太可能因撕裂而中断，但每种情景都有其挑战，如无壁稳定限制或最小化感应电流。因此，值得进一步开发通过修改的观察、驱动和奖励设置训练的AI控制器，以解决这些不同的挑战。此外，在DIII-D中使用的执行器和传感器的灵活性将不同于ITER和反应堆中的情况。未来还需要开发在更有限的感知和执行条件下的控制策略。

### 撕裂不稳定性预测的动态模型

为了预测DIII-D中的撕裂事件，我们首先根据实验中 *n* = 1 Mirnov线圈信号标记每个阶段是否撕裂稳定（0或1）。利用这些标记的实验数据，我们训练了一个基于DNN的多模态动态模型，该模型以各种等离子体剖面和托卡马克作动为输入，并预测25 ms后撕裂可能性作为输出。经过训练的动态模型输出一个介于0和1之间的连续值（所谓的撕裂性），数值越接近1表示25 ms后撕裂不稳定性发生的可能性越高。该模型的结构显示在扩展数据图 [1](/articles/s41586-024-07024-9#Fig5) 中。关于动态预测模型的输入输出变量和超参数的详细描述可以在参考文献 ^([5](/articles/s41586-024-07024-9#ref-CR5 "Seo, J. et al. Multimodal prediction of tearing instabilities in a tokamak. In 2023 International Joint Conference on Neural Networks (IJCNN) 1–8 (IEEE, 2023).")) 中找到。虽然这个动态模型是一个黑盒子，不能明确提供诱导撕裂不稳定性的根本原因，但它可以作为对稳定性响应的代理，避免昂贵的真实世界实验。例如，这个动态模型被用作该工作中撕裂避免控制器的RL训练环境。在RL训练过程中，动态模型从AI控制器确定的给定等离子体条件和执行器值中预测未来的 *β*[N] 和撕裂性。然后，根据方程 ([1](/articles/s41586-024-07024-9#Equ1)) 预测的状态估计奖励并作为反馈提供给控制器。

图 [4b–d](/articles/s41586-024-07024-9#Fig4) 显示了在我们控制实验的给定等离子体条件下可能的束流功率的预估撕裂性轮廓图。由AI控制的实际束流功率用黑色实线表示。虚线是每个放电设定的阈值轮廓线，大致代表每一点处束流功率的稳定限制。图表明，经过训练的AI控制器在不稳定警告之前积极避免触及撕裂性阈值。

对电子温度和密度诊断误差的撕裂敏感性显示在扩展数据图 [2](/articles/s41586-024-07024-9#Fig6) 中。扩展数据图 [2](/articles/s41586-024-07024-9#Fig6) 中的填充区域表示在分别从193280测量中增加和减少电子温度和密度10%时，撕裂预测的范围。由于电子温度误差，撕裂稳定性的不确定性平均约为10%，而电子密度误差的不确定性约为20%。然而，即使考虑到诊断误差，随时间变化的撕裂稳定性趋势仍然可以观察到保持一致。

### 撕裂避免的RL训练

用于预测未来撕裂不稳定动态的动态模型已集成到OpenAI Gym库^([55](/articles/s41586-024-07024-9#ref-CR55 "Brockman, G. et al. OpenAI Gym. Preprint at                    https://arxiv.org/abs/1606.01540                                     (2016)."))，使其能够作为训练环境与控制器进行交互。另一个DNN模型，撕裂避免控制器，是使用深度确定性策略梯度^([56](/articles/s41586-024-07024-9#ref-CR56 "Lillicrap, T. P. et al. Continuous control with deep reinforcement learning. Preprint at                    https://arxiv.org/abs/1509.02971                                     (2015)."))方法进行训练，使用Keras-RL实现（[https://keras.io/](https://keras.io/)）^([57](/articles/s41586-024-07024-9#ref-CR57 "keras-rl2\. GitHub                    https://github.com/inarikami/keras-rl2                                     (2019).")）。

观测变量包括在磁通坐标的33个等距分布的网格上映射的5种不同等离子体剖面：电子密度、电子温度、离子旋转、安全因子和等离子体压力。当等离子体被转移时，安全因子（*q*）可以发散到无穷大。因此，观测变量使用了1/*q*来减少数值困难^([42](/articles/s41586-024-07024-9#ref-CR42 "Abbate, J., Conlin, R. & Kolemen, E. Data-driven profile prediction for DIII-D. Nucl. Fusion 61, 046027 (2021)."))。动作变量包括总束功率和等离子体边界的三角性，它们的可控范围被限制为与DIII-D的IBS实验一致。AI控制的等离子体边界形状已被ITER的极向磁场线圈系统证实可实现，如扩展数据图 [3](/articles/s41586-024-07024-9#Fig7) 所示。

AI 控制器的 RL 训练过程在扩展数据图 [4](/articles/s41586-024-07024-9#Fig8) 中有所描述。每次迭代中，观察变量（五种不同的配置文件）从实验数据中随机选择。AI 控制器根据这些观察结果确定所需的束流功率和等离子体三角形。为了减少局部优化的可能性，在训练过程中向控制动作添加基于 Ornstein–Uhlenbeck 过程的动作噪声。然后，动态模型根据给定的等离子体配置文件和执行器数值预测 25 毫秒后的 *β*[N] 和撕裂性。奖励根据方程式 ([1](/articles/s41586-024-07024-9#Equ1)) 使用预测状态进行评估，并作为反馈提供给 AI 控制器的 RL 训练。当控制器和动态模型观察等离子体配置文件时，即使由于墙壁条件或杂质等不可预测因素导致等离子体配置文件变化，也能反映撕裂稳定性的变化。此外，尽管本文专注于 IBS 条件下的撕裂不稳定性关键问题，但 RL 训练本身并未限制于任何特定的实验条件，确保其在所有条件下的适用性。训练完成后，基于 Keras 的控制器模型使用 Keras2C 库^([58](/articles/s41586-024-07024-9#ref-CR58 "Conlin, R., Erickson, K., Abbate, J. & Kolemen, E. Keras2c: a library for converting Keras neural networks to real-time compatible C. Eng. App. Artif. Intell. 100, 104182 (2021).")) 转换为 C 以进行 PCS 集成。

之前，一项相关工作^([17](/articles/s41586-024-07024-9#ref-CR17 "Fu, Y. et al. Machine learning control for disruption and tearing mode avoidance. Phys. Plasmas 27, 022501 (2020).")) 使用简单的 bang-bang 控制方案仅使用束流功率处理撕裂性。虽然我们的控制性能在 *β*[N] 方面与该工作看似相似，但在考虑其他操作条件时并非如此。在 ITER 和未来的聚变设备中，具有稳定核心不稳定性的高归一化聚变增益（*G* ∝ *Q*）至关重要。这需要高 *β*[N] 和小 *q*[95]，因为 \(G\propto {\beta }_{{\rm{N}}}/{q}_{95}^{2}\)。同时，由于加热能力有限，高 *G* 必须通过弱等离子体旋转（或束流扭矩）实现。在这里，高 *β*[N]、小 \({q}_{95}^{2}\) 和低扭矩都是撕裂不稳定性的不稳定条件，突显撕裂不稳定性是 ITER 的重大瓶颈。

如扩展数据图 [5](/articles/s41586-024-07024-9#Fig9) 所示，我们的控制实现了比参考文献^([17](/articles/s41586-024-07024-9#ref-CR17 "Fu, Y. et al. Machine learning control for disruption and tearing mode avoidance. Phys. Plasmas 27, 022501 (2020)."))中测试实验更高*G*的撕裂稳定操作。这是通过在更易产生撕裂不稳定性的条件下（从4 → 3）保持更高（或相似）的*β*[N]来实现的。此外，这是在更弱扭矩的情况下实现的，进一步突显了我们的RL控制器在更严苛条件下的能力。因此，这项工作展示了更符合ITER相关的性能，为未来设备中实现高融合增益和强撕裂避免提供了更接近和更清晰的路径。

此外，当考虑*β*[N]对撕裂不稳定性的非单调效应时，RL控制在实现高融合能力方面的表现可以进一步突出。与*q*[95]或扭矩不同，增加或减少*β*[N]均可能使撕裂不稳定性加剧。这导致存在最佳融合增益（如*G* ∝ *β*[N]），它使得撕裂稳定操作变得更加复杂。这里，扩展数据图 [6](/articles/s41586-024-07024-9#Fig10) 显示了RL控制器在融合增益与时间空间中的放电轨迹，轮廓颜色说明了撕裂性。这清楚地表明RL控制器成功地通过撕裂性谷驱动等离子体，确保稳定运行，并显示其在如此复杂系统中的显著性能。

RL相较于传统方法具有的优势使得这种卓越性能成为可能，以下是其描述。

1.  (1)

    通过采用RL技术的“多执行器（光束和形状）多目标（低撕裂性和高*β*[N]）”控制器，我们能够进入更高的*β*[N]区域，同时保持可接受的撕裂性。如扩展数据图 [5](/articles/s41586-024-07024-9#Fig9) 所示，我们控制的放电（193280）显示出比先前工作（176757）更高的*β*[N]和*G*。我们控制器的这一优势在于同时调整光束和等离子体形状，以实现增加*β*[N]和降低撕裂性。值得注意的是，我们的放电在*β*[N]和撕裂稳定性方面都处于更不利的条件（较低的*q*[95]和较低的扭矩）。

1.  (2)

    先前的撕裂模型基于当前零维测量评估了撕裂可能性，没有考虑即将进行的激励控制。然而，我们的模型考虑了一维详细配置文件以及即将进行的激励，然后预测未来撕裂响应对未来控制的响应。这可以在控制方面提供更灵活的适用性。我们的RL控制器经过训练，可以理解这种撕裂响应，并考虑未来的影响，而先前的控制器只能看到当前的稳定性。通过考虑未来的响应，我们提供了更长期的更优化激励，而不是贪婪的方式。

这使得在我们实验之外的更通用情况中应用成为可能。例如，如扩展数据图[7a](/articles/s41586-024-07024-9#Fig11)所示，撕裂性是*β*[N]的非线性函数。在某些情况下（如扩展数据图[7b](/articles/s41586-024-07024-9#Fig11)所示），这种关系也是非单调的，使得增加束流功率成为减少撕裂性的期望指令（如扩展数据图[7b](/articles/s41586-024-07024-9#Fig11)中所示的向右箭头）。这是由于撕裂不稳定源的多样性，如*β*[N]限制，*Δ*′和电流井。在这些情况下，使用在参考文献中显示的简单控制^([17](/articles/s41586-024-07024-9#ref-CR17 "Fu, Y. et al. Machine learning control for disruption and tearing mode avoidance. Phys. Plasmas 27, 022501 (2020)."))可能导致振荡激励甚至进一步的不稳定。在RL控制的情况下，振荡较少，并且在阈值以下更快地控制，通过多作动器控制实现更高的*β*[N]，如扩展数据图[7c](/articles/s41586-024-07024-9#Fig11)所示。

### 控制等离子体的三角度

等离子体形状参数是影响各种类型等离子体不稳定性的关键控制旋钮。在DIII-D中，形状参数如三角度和延伸可以通过近距离控制进行调节^([41](/articles/s41586-024-07024-9#ref-CR41 "Barr, J. et al. Development and experimental qualification of novel disruption prevention techniques on DIII-D. Nucl. Fusion 61, 126019 (2021)."))。在本研究中，我们将顶部三角度作为AI控制器的行动变量之一。在我们的实验中，底部三角度保持不变，因为它直接与内壁上的打击点相关联。

我们还注意到，通过AI控制改变顶部三角度与典型调整相比非常大。因此，有必要验证在ITER磁线圈能力允许的情况下是否允许如此大的等离子体形状变化。如扩展数据图[3](/articles/s41586-024-07024-9#Fig7)所示的额外分析证实，ITER的重新缩放等离子体形状可以在线圈电流限制内实现。

### 在不同条件下保持撕裂性的稳健性

图 [3b](/articles/s41586-024-07024-9#Fig3) 和 [4a](/articles/s41586-024-07024-9#Fig4) 中的实验表明，通过适当的基于AI的控制可以维持撕裂性。然而，需要验证在添加额外执行器和等离子体条件变化时，是否能稳健地维持低撕裂性。特别是ITER计划不仅使用50 MW的束流，还将使用10–20 MW的射频执行器。电子回旋共振射频加热直接改变电子温度分布，稳定性可能会敏感变化。因此，我们进行了实验，看看AI控制器是否能成功在添加射频加热的新条件下，保持低撕裂性。在放电 193282 中（扩展数据图 [8](/articles/s41586-024-07024-9#Fig12) 中的绿线），预先编程将1.8 MW的射频加热稳定应用于背景，同时通过AI控制束流功率和等离子体三角性。这里的射频加热是朝向等离子体核心的，而撕裂位置处的电流驱动微乎其微。

然而，由于在 *t* = 3.1 s 时等离子体电流控制突然丧失，*q*[95] 从3增加到4，并且随后的放电未能在ITER基线条件下继续。需要注意的是，这种等离子体电流控制的变化是无意的，与AI控制无直接关系。这种等离子体电流波动急剧提高了撕裂性，在 *t* = 3.2 s 时超过了阈值，但立即通过持续的AI控制稳定下来。尽管最终因等离子体电流不足而在预先编程的平顶结束之前被打断，但这次意外的实验展示了基于AI的撕裂性控制在额外加热执行器、更广 *q*[95] 范围和意外电流波动方面的稳健性。

在正常等离子体实验中，控制参数通过前馈设置保持不变，使每次放电成为单个数据点。然而，在我们的实验中，放电过程中等离子体和控制参数都在变化。因此，一个放电包含多个控制周期。因此，与标准固定控制等离子体实验相比，我们的结果比预期更重要，支持控制方案的可靠性。

此外，预测的等离子响应由于 RL 控制对从实验数据库随机选择的 1,000 个样本，其中包括不仅仅是 IBS 而是所有实验条件，显示在扩展数据图 [9a,b](/articles/s41586-024-07024-9#Fig13) 中。当 *T* > 0.5（不稳定，顶部）时，控制器试图减少 *T* 而不是影响 *β*[N]，而当 *T* < 0.5（稳定，底部）时，它试图增加 *β*[N]。这与方程式 ([1](/articles/s41586-024-07024-9#Equ1)) 中所显示的奖励预期响应相匹配。在不稳定相中的 98.6% 中，控制器降低了撕裂性，并且在稳定相中的 90.7% 中，控制器增加了 *β*[N]。

扩展数据图 [9c](/articles/s41586-024-07024-9#Fig13) 显示了我们实验会话的放电序列的实现时间积分 *β*[N]。直到 193276 号之前，RL 控制未应用或控制开始前发生撕裂不稳定，而 193277 号之后应用了 RL 控制。在 RL 控制之前，除了一个（193266 号：低 *β*[N] 参考，见图 [3b](/articles/s41586-024-07024-9#Fig3)）之外，所有射击都受到干扰，但在应用 RL 控制后，仅有两个（193277 号和 193282 号）受到干扰，这些情况前面已经讨论过。在 RL 控制应用后，平均时间积分 *β*[N] 也增加了。此外，控制放电的输入特征范围与扩展数据图 [10](/articles/s41586-024-07024-9#Fig14) 中的训练数据库分布进行了比较，表明我们的实验既不过于集中（模型没有过度拟合我们的实验条件），也不过于分散（确认我们的控制器在实验中的可用性）。
