<!--yml

category: 未分类

date: 2024-05-27 15:15:17

-->

# MicroZed Alveo Edition: 安装 XDMA 驱动程序。

> 来源：[`www.adiuvoengineering.com/post/microzed-alveo-edition-installing-xdma-drivers`](https://www.adiuvoengineering.com/post/microzed-alveo-edition-installing-xdma-drivers)

在本系列中到目前为止，我们已经介绍了 Alveo 卡的范围，并创建了我们的第一个应用程序，基于标准的卡管理解决方案。

在本博客中，我们将在包含 U55C 的 Linux 机器上安装软件驱动程序，并运行一个简单的测试以确保驱动程序已正确安装。这将允许我们查看创建额外的软件，以便利用 DMA。

作为 CMS 参考设计的一部分实例化的 DMA 使用了 XDMA，因此我们需要为我们的主机计算机安装 XDMA 驱动程序。

要开始使用这个，我们将首先克隆位于[这里](https://github.com/Xilinx/dma_ip_drivers/tree/2022.1.5)的 XDMA GitHub 仓库，一旦目录被克隆，我们就能够编译源代码，为我们的主机机器创建必要的驱动程序。

为此，我们需要在 XDMA 路径下编译源代码，从克隆的 repo 中，我们可以通过运行 make 文件来编译源代码。

但是，在我们执行此操作之前，我们需要仔细检查文件 XDMA_MOD.c 中是否包含了 U55C 卡的 PCi 标识符。该文件包含了与供应商 ID 相关联的设备 PCi ID 的声明。

我们可以通过使用以下命令获取 U55C 的 PCi 标识符

这将报告安装的卡的 PCI ID，如果此 PCI ID 不在 XMDA_MOD.c 文件中，则将其添加到从第 47 行开始的列表中

将 PCi 标识符添加到源代码中后，我们现在可以使用 make 命令编译驱动程序，我们需要在 XDMA 源目录中执行此操作。

创建并安装了驱动程序后，下一步是编译我们将用于测试编译和安装的工具。切换到工具目录并运行以下命令

现在我们可以检查我们是否正确安装了驱动程序，并运行测试以确保我们能够与 Alveo U55C 板通信。记得首先使用硬件管理器对板进行编程，使用 CMS 示例设计。

要将驱动程序加载到内核中，我们需要切换到测试目录，并运行脚本加载驱动程序的命令。

当运行此操作时，您应该期望看到以下输出。

要双重检查 XDMA 驱动程序是否已正确安装并绑定到 Alveo U55X，我们可以运行以下命令

这清楚地显示了 XDMA 正在作为内核驱动程序和模块使用。

最后一步是运行测试目录中也可用的 XDMA 测试脚本，要执行脚本，请发出以下命令。

这将在主机处理器和 Alveo U55C 之间使用 XDMA 驱动程序运行一系列测试。

正如我们所见，这些测试都如预期地通过了，因此我们能够说我们已经正确安装了驱动程序，并且它与我们的 CMS 示例设计一起工作。

在下一篇博客中，我们将探讨如何创建使用此驱动程序的自己的软件。

如果您喜欢这篇博客，不妨看看我们多年来制作的免费网络研讨会、工作坊和培训课程。重点包括

想要了解如何从零开始设计嵌入式系统吗？看看我们关于创建嵌入式系统的书籍。这本书将引导您完成所有阶段的要求、架构、元件选择、原理图、布局和 FPGA/软件设计。我们设计并制造了书中核心的板子！原理图和布局可在 Altium 中获取[这里](https://www.e3designers.com/altium-365)了解有关板子的更多信息（请参阅先前博客中的[引导](https://www.adiuvoengineering.com/post/microzed-chronicles-configuring-zynq-on-a-custom-board)、[DDR 验证](https://www.adiuvoengineering.com/post/microzed-chronicles-validating-your-custom-zynq-board-memory)、[USB](https://www.adiuvoengineering.com/post/microzed-chronicles-smart-sensor-iot-board-getting-usb-up-and-running)、[传感器](https://www.adiuvoengineering.com/post/microzed-chronicles-petalinux-i2c-in-the-ps-and-axi-iic))，并查看原理图[此处](https://www.adiuvoengineering.com/post/sensorsthink-board-schematic)。 
