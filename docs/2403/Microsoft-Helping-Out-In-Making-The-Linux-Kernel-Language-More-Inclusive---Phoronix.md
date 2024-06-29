<!--yml

category: 未分类

date: 2024-05-29 12:49:24

-->

# 微软帮助使Linux内核语言更具包容性 - Phoronix

> 来源：[https://www.phoronix.com/news/Microsoft-Linux-More-Inclusive](https://www.phoronix.com/news/Microsoft-Linux-More-Inclusive)

随着时间的推移，微软在Linux内核方面的贡献已经超出了最初在Hyper-V支持和Azure的其他需求以及Windows子系统（WSL）上的业务重点，扩展到更一般的贡献。微软还雇佣了更多的关键Linux贡献者，并投资于诸如systemd等其他项目。本周早些时候，来自一位在微软工作的工程师的补丁已经在

[Linux内核的Rust语言改进](https://www.phoronix.com/news/Linux-Rust-In-Place-Module-Init)

虽然在结束假日周末时，还有一些补丁使Linux内核语言更具包容性。

由微软Linux工程师Easwar Hariharan发送了一组十四个补丁，他在Azure Linux管道、Azure Cobalt等云硅以及虚拟化事务方面工作。微软这次最新的非核心业务重点的Linux贡献是清理代码中的语言，以使其更具包容性。特别是，根据最新的上游I2C、SMBus和I3C规范调整使用适当的术语。

大部分代码和代码注释中的术语已从主控和从控调整为使用控制器和目标（或客户端）。但是，即使对于这些补丁，一些问题也被提出，因为行业规范倾向于使用新的控制器/目标术语，而不是大多数这些新内核补丁中使用的客户端术语。在上游内核开发人员中，显然尚未就客户端和目标之间的选择达成一致意见。

几乎有四百行代码被这些补丁清理干净，涵盖了从核心子系统代码到AMD和Intel图形驱动程序，各种媒体和FBDEV驱动程序，以及其他I2C/I3C/SMBus代码。

可在以下链接找到微软对

[内核邮件列表](https://lore.kernel.org/dri-devel/20240329170038.3863998-1-eahariha@linux.microsoft.com/T/)

.
