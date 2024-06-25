<!--yml

category: 未分类

date: 2024-05-27 15:08:27

-->

# Rust 在 Espressif 芯片上的应用 - 2024 年 01 月 24 日

> 来源：[`mabez.dev/blog/posts/esp-rust-24-01-2024/`](https://mabez.dev/blog/posts/esp-rust-24-01-2024/)

这是 esp-rs 努力的下一个季度更新，详细说明了 2023 年第四季度的进展。

## Rust 编译器

### 上游

在第四季度，我们看到上游 RISC-V 目标的许多改进。首先，现在所有裸机非原子（不是 `a`）目标都启用了原子加载/存储代码生成！这对于 ESP32C3、ESP32C2 和其他 RISC-V 目标的用户来说是一个巨大的痛点。目前仍处于夜间版本，但随着 1.76 版本的发布，它将在稳定版上可用。其次，我们添加了 `riscv32imafc-unknown-none-elf` 目标，为 [ESP32P4](https://www.espressif.com/en/news/ESP32-P4) 做好准备，这是 Espressif 的第一个没有无线电的芯片。我们还将所有当前裸机 RISC-V 目标提升到二级，并在此过程中填补了缺失的 [平台支持文档](https://github.com/rust-lang/rust/pull/117874)。最后，我们为 ESP32P4 添加了一个 `espidf` 目标，[riscv32imafc-esp-espidf](https://github.com/rust-lang/rust/pull/119738)。

### Xtensa

Espressif 的 LLVM 团队继续改进和重新基于 Xtensa LLVM 后端，该后端最近已经基于 LLVM 17 重新构建。不幸的是，我们目前没有任何上游进展可报告。

## esp-hal - no_std

在第四季度，`v0.13`、`v0.14` 和 `v0.15` 版本的 esp-hal 发布。重点包括为 PARL_IO、I2S 和 RMT 新增的新的异步驱动程序；以及更新为期待已久的 `v1.0` 的 embedded-hal！我们通过统一 API 跨所有芯片扩展了我们的低功耗支持，还为 ESP32S2 和 ESP32S3 添加了 ULP（超低功耗）核心支持。最后，我们实现的一个很酷的功能是为 ESP32C6 和 ESP32H2 实现的 `flip-link`，`flip-link` 提供了零成本的堆栈溢出检测，对于这些没有硬件上可以捕获此类错误的芯片来说，这真的很不错。查看完整的 [更新日志](https://github.com/esp-rs/esp-hal/blob/main/CHANGELOG.md) 获取所有详细信息和更改。

一个重要的突破性变化需要注意的是，我们不再支持原子模拟，因为上游 Rust 现在支持非原子目标上的原子加载和存储。使用 ESP32S2、ESP32C3 或 ESP32C2 的应用程序用户，请阅读 [此文档](https://github.com/esp-rs/esp-hal/blob/main/CHANGELOG.md#breaking-1) 以便更轻松地升级。对于基于 CAS 的原子操作，您应该切换到 [portable-atomic](https://github.com/taiki-e/portable-atomic) crate。非常感谢 [@taiki-e](https://github.com/taiki-e) 为我们的目标添加了对 portable-atomic 的支持，并支持 crates，并推动了原子加载/存储编译器的更改！

## esp-wifi - no_std

在 Q4，我们为 ESP32H2 添加了 BLE 支持，修复了 ESP32 上的并存支持，并添加了一个基准示例来测试 WiFi 吞吐量。此外，我们最终在 crates.io 上发布了 `esp-wifi`！我们花了很长时间来完善和文档化，以达到这一点，我们很高兴它已经发布了。

### espflash

Q4 发布了 `v2.1.0` 版本的 espflash，一个功能较小的发布版本，新增了一些新功能，用于擦除扇区、区域和分区。然后我们开始了 `v3.0` 的开发周期，其中将默认包含[基于 Rust 的烧录存根](https://github.com/esp-rs/esp-flasher-stub)！查看[changelog](https://github.com/esp-rs/espflash/blob/main/CHANGELOG.md#added)的未发布部分，了解到目前为止新增了什么。

### probe-rs

在 Q4，probe-rs 和乐鑫芯片取得了大量进展。这一切都受到我们想要开始使用真实硬件进行 CI 测试的驱动，称为 Hardware In Loop（HIL）测试。

新的乐鑫芯片都是基于 RISC-V 的，probe-rs 已经对此有很好的支持。然而，ESP32、ESP32S2 和 ESP32S3 是基于 Xtensa 架构的，因此为了在 CI 中测试这些芯片，我们需要[probe-rs 中的 Xtensa 架构支持](https://github.com/probe-rs/probe-rs/issues/2001)，这也是我们着手做的事情。我很高兴地报告，我们在 probe-rs 中已经有了足够的 Xtensa 支持，可以连接、检查和烧录 ESP32S2 和 ESP32S3 了！这也带来了一些巨大的速度改进，感谢 probe-rs 中 USB-SERIAL-JTAG 和 FTDI 驱动程序的改进。所有这些都离不开 probe-rs 团队的帮助，当然还有一个特别的个人的辛勤工作，[Dániel Buga](https://github.com/bugadani)一直在推动这项工作。

## RFC

### esp-openthread

我们正在寻求关于下一步开发 esp-openthread 的意见，如果您计划在项目或产品中使用 thread，请查看[此 RFC 问题](https://github.com/esp-rs/esp-openthread/issues/4)。

## 未来的工作

要了解我们本季度将要开展的工作，请查看 Jesse Braham 的[帖子](https://beta7.io/posts/esp-rs-quarterly-planning-q1-2024/)！

* * *
