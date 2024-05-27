<!--yml
category: 未分类
date: 2024-05-27 13:04:03
-->

# Release 0.4.0 of the seL4 Device Driver Framework - Devel - lists.sel4.systems

> 来源：[https://lists.sel4.systems/hyperkitty/list/devel@sel4.systems/thread/6QKUK5C5PNN6CUR2XEUR2SSFLTCTTXLL/](https://lists.sel4.systems/hyperkitty/list/devel@sel4.systems/thread/6QKUK5C5PNN6CUR2XEUR2SSFLTCTTXLL/)

Hello all

This is a message to let people know that version 0.4.0 of the seL4 Device Driver Framework (sDDF) has been released.

The sDDF aims to provide a collection of interfaces, libraries and tools for writing device drivers for seL4 that allow accessing devices securely and with low overhead. Some may be familiar with the sDDF from previous seL4 Summit talks.

Since the pre-release at the 2023 Summit, we have made significant improvements to the networking sub-system, generally rationalising interfaces and further improving performance, the Ethernet device-class interfaces are now mature. We have also introduced preliminary specs and prototype implementations for serial, I2C, block, audio and graphics device classes.

You can find information about the release and the accompanying documentation here: [https://github.com/au-ts/sddf/releases/tag/0.4.0](https://github.com/au-ts/sddf/releases/tag/0.4.0)

Thanks, Ivan