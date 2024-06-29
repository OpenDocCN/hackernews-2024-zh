<!--yml

类别：未分类

日期：2024-05-27 14:42:03

-->

# MicroZed编年史：频率域

> 来源：[https://www.adiuvoengineering.com/post/microzed-chronicles-the-frequency-domain](https://www.adiuvoengineering.com/post/microzed-chronicles-the-frequency-domain)

最近，我发布了一些关于信号处理或滤波器的博客和项目。我认为深入了解这些博客和项目中常用的元素之一——傅里叶变换，会是一个不错的主意。

我们应该能够在时间和频率域内自如地工作，因此希望这篇博客能为那些可能不熟悉的人提供一个良好的起点。

#### **时间域或频率域**

作为工程师，我们可以在时间域或频率域内分析和操作信号，知道何时选择其中之一是项目中需要工程师的核心原因。

时间域使工程师能够分析信号随时间的变化。通常，在电子系统中，所讨论的信号是由传感器输出或系统的另一部分生成的变化电压、电流或频率。在这个领域内，我们可以测量信号的振幅、频率和周期，以及信号的上升和下降时间等更有趣的参数。如果我们希望在实验室环境中观察时间域信号，通常会使用示波器或逻辑分析仪。

然而，有些信号参数需要在频率域内分析，以获取其中包含的信息。这些参数存在于频率域中，我们可以在那里识别信号的频率成分、它们的振幅以及每个频率的相位。在频率域内工作还使信号操作变得更简单，因为在该域内可以轻松执行卷积。类似于时间域分析，如果我们希望在实验室环境中观察频率域信号，必须使用频谱分析仪。

对于某些应用程序，我们希望在时间域内工作，比如监测电压或温度等系统。虽然噪声可能是一个问题，但平均取样数在许多情况下就足够了。然而，对于其他应用程序，最好在频率域内工作。例如，这可能包括需要从一个信号中滤波或从噪声源中分离信号的信号处理应用。

当然，仅在时间域内工作需要对量化数字信号进行较少的处理。另一方面，在频率域内工作首先需要对量化数据应用变换以从时间域转换。类似地，我们需要再次执行逆变换将频率域中的后处理数据输出回时间域。

#### **从时间域到频率域**

根据信号类型（即重复或非重复，离散或非离散），有多种方法可用于在时间和频率域之间转换，例如傅里叶级数、傅里叶变换或Z变换。在电子信号处理和特别是FPGA应用中，最常用的方法是离散傅里叶变换（DFT），它是傅里叶变换的子集。DFT用于分析周期性和离散的信号，这意味着它们由一个采样频率下均匀间隔的N位样本组成，在许多应用中由系统中的ADC提供。

简而言之，DFT将输入信号分解为两个输出信号，表示该信号的正弦和余弦分量。因此，对于N个样本的时间域序列，DFT将返回两组N/2+1个余弦和正弦波样本，分别称为实部和虚部，如下图一所示。对于n位输入信号宽度，实部和虚部样本宽度也将为N/2。

计算DFT的算法非常直接，我们可以使用下面显示的方程。

其中**x[i]**是时间域信号，i的范围从0到N-1，k的范围从0到N/2。上述算法称为相关方法，它将输入信号与该迭代的正弦或余弦波形相乘以确定其幅度。

在我们的应用程序中的某个时刻，我们将希望从频率域转换回时间域。为此，我们可以使用合成方程，该方程结合了实部和虚部波形以重新创建时间域信号。

**ReX ̅**和**ImX ̅**是余弦和正弦波的缩放版本，因此我们需要对它们进行缩放。在所有情况下，**ImX[k]**或**ReX[k]**除以N/2以确定**ReX ̅**和**ImX ̅**的值，除非在**ReX[0]**和**ReX[N/2]**的情况下，此时它们除以N。

出于明显的原因，这被称为逆离散傅里叶变换或简称IDFT。

我们可以使用Python、MATLAB甚至Excel等工具对捕获的数据执行DFT计算，许多实验室工具如示波器可以根据请求执行DFT。

在继续之前，值得指出上述的DFT和IDFT都被称为实DFT和实IDFT。这意味着输入是实数而不是复数。很快就会明白为什么我们需要知道这一点。

我们可以使用Python在JupyterLab中演示DFT算法，使用以下代码。我们还可以生成下面显示10 MHz信号和采样率为100 Msps的实部元素的图形。

```
def dft(signal):
    """
    Compute the Discrete Fourier Transform (DFT) of the given signal.
    """
    N = len(signal)
    n = np.arange(N)
    k = n.reshape((N, 1))
    e = np.exp(-2j * np.pi * k * n / N)
    return np.dot(e, signal)

# Compute the DFT
Y = dft(signal)
n = len(signal)
freq = np.arange(n) / (n*1/sampling_rate)  # Frequency bins

# Plot the signal
plt.figure(figsize=(12, 6))
plt.subplot(2, 1, 1)
plt.plot(t, signal)
plt.title('Sample Signal')
plt.xlabel('Time')
plt.ylabel('Amplitude')

# Plot the magnitude spectrum
plt.subplot(2, 1, 2) [#plt](https://www.adiuvoengineering.com/blog/hashtags/plt).plot(freq, np.abs(Y))
plt.plot(freq[:n // 2], np.abs(Y)[:n // 2] * 1/n)  # Plot only the positive frequencies

plt.title('Magnitude Spectrum')
plt.xlabel('Frequency (Hz)')
plt.ylabel('Magnitude') [#plt](https://www.adiuvoengineering.com/blog/hashtags/plt).xlim([-2e7,2e7])
plt.show()
```

#### **基于FPGA的实现**

上述描述的DFT和IDFT的实现通常作为嵌套循环实现，每个循环执行N次计算，如Python示例所示。因此，实现DFT计算所需的时间如下。

其中 Kdft 是要执行的每次迭代的处理时间，这可能会变得相当耗时。在高性能桌面上运行速度会非常快，但在边缘处理器上则会慢得多。

因此，在 FPGA 中实现 DFT 通常使用称为快速傅里叶变换（FFT）的算法来计算 DFT。由于其对多个行业的影响，这种算法经常被称为“我们生活中最重要的算法”。FFT 与之前解释的 DFT 算法略有不同，它计算复杂的 DFT。这意味着它期望实部和虚部的时域信号，并在频域中生成结果，宽度为 n 位，而不是 n/2。更进一步解释，当我们希望计算实数 DFT 时，必须先将虚部设为零，并将时域信号移入实部。如果我们希望在 AMD FPGA 中实现 FFT，有两种选择：一种是使用我们选择的 HDL 从头编写 FFT，另一种是使用 Vivado IP 目录中提供的 FFT IP。除非有无法使用 IP 的紧急原因，我建议使用 IP 核以减少开发时间。

FFT 的基本方法是将时域信号分解成多个单点时域信号。这通常被称为比特反转，因为样本被重新排序。如果不使用比特反转算法作为捷径，创建这些单点时域信号所需的阶段数由 Log2 N 计算，其中 N 是比特数。

然后，这些单点时域信号用于计算每个点的频谱。这非常直接，因为频谱等于单点时域。

FFT 算法在重新组合这些单频点时变得复杂，我们必须逐个阶段重新组合这些频谱点。这与时域分解相反，再次需要 Log2 N 个阶段来重建频谱。这就是著名的 FFT 蝴蝶效应发挥作用的地方。

与 DFT 执行时间相比，FFT 花费

这大大提高了计算 DFT 的执行时间。

在我们的 FPGA 中实现 FFT 时，我们还必须考虑 FFT 的大小，因为这将决定噪声底线，低于这个底线我们无法看到潜在感兴趣的信号。FFT 的大小还将决定频率箱的间距。下面的公式用于计算这一点。

**FFT噪声底线（dB）=6.02n+1.77+10log10（FFT大小/2）**

其中 n 是时域内量化位数，FFT 大小是 FFT 大小。对于基于 FPGA 的实现，这通常是 2 的幂次方（例如，256、512、1024 等）。频率箱将均匀间隔。

要举一个非常简单的例子，具有 100 MHz 的 FS 和 FFT 大小为 128 的信号会有 0.39 Hz 的频率分辨率。这意味着相隔 0.39 Hz 的频率是无法区分的。

再次，我们可以在 JupyterLabs 中查看 FFT，并观察与 DFT 相同配置的输出。

```
# Compute the FFT
n = len(signal)  # Length of the signal
freq = np.fft.fftfreq(n, d=1/sampling_rate)  # Frequency bins
Y = np.fft.fft(signal)/n  # FFT and normalization

# Plot the signal
plt.figure(figsize=(12, 6))
plt.subplot(2, 1, 1)
plt.plot(t, signal)
plt.title('Sample Signal')
plt.xlabel('Time')
plt.ylabel('Amplitude')

# Plot the magnitude spectrum
plt.subplot(2, 1, 2)
plt.plot(freq, np.abs(Y))
plt.title('Magnitude Spectrum')
plt.xlabel('Frequency (Hz)')
plt.ylabel('Magnitude')
plt.xlim([-2e7,2e7])
plt.show()
```

当然，既然我们现在对 FFT 和 DFT 有了更多的理解，我们可能希望在我们的 FPGA 中实现它们。如果您有 PYNQ 板，您可以找到关于如何为 MPSoC 或 Zynq 7000 板执行详细的逐步说明书。

这应该帮助您了解如何在 FPGA 中实现 FFT，并为进一步使用它们提供一个良好的起点。

如果您喜欢这篇博客，不妨看看我们多年来创建的免费网络研讨会、工作坊和培训课程。重点包括

想要了解更多关于从零开始设计嵌入式系统的内容吗？查看我们的关于创建嵌入式系统的书籍。本书将引导您完成所有需求、架构、组件选择、原理图、布局以及 FPGA / 软件设计的各个阶段。书中的核心板我们进行了设计和制造！原理图和布局可在 Altium 中查看 [此处](https://www.e3designers.com/altium-365)，了解有关板卡的更多信息（参见之前关于 [引导](https://www.adiuvoengineering.com/post/microzed-chronicles-configuring-zynq-on-a-custom-board)、[DDR 验证](https://www.adiuvoengineering.com/post/microzed-chronicles-validating-your-custom-zynq-board-memory)、[USB](https://www.adiuvoengineering.com/post/microzed-chronicles-smart-sensor-iot-board-getting-usb-up-and-running)、[传感器](https://www.adiuvoengineering.com/post/microzed-chronicles-petalinux-i2c-in-the-ps-and-axi-iic))，并在 [此处](https://www.adiuvoengineering.com/post/sensorsthink-board-schematic) 查看原理图。
