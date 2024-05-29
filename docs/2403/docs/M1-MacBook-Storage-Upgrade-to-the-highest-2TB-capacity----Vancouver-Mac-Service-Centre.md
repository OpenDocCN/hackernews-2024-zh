<!--yml
category: 未分类
date: 2024-05-27 14:43:24
-->

# M1 MacBook Storage Upgrade to the highest 2TB capacity! - Vancouver Mac Service Centre

> 来源：[https://vancouvermac.ca/repair/macbook-storage-upgrade/](https://vancouvermac.ca/repair/macbook-storage-upgrade/)

## Background

Once ago, MacBook storage upgrades were a simple, 10 minute job, involving swapping out a standard 2.5″ SATA drive from the computer’s storage drive bay and putting in any commonly available replacement part. In 2012, Apple switched to a proprietary PCIe connector for their newer blade SSD drives, further complicating the situation. However, swapping drives was still simple, taking only a few minutes. In 2015, Apple started integrating the SSD into the logic board with their 12″ MacBook model.

As a result, the transfer speeds increased and the MacBook could be made thinner than even. However, it caused a variety of problems for the consumer. Most critically, it became impossible to retrieve data if the logic board was broken, which causes lost data in the even of a logic board failure.

## Not enough disk space

MacBook Storage upgrades were once a common procedure for when they run out of disk space, and due to these design changes, the only option available to consumers was to upgrade the entire MacBook to a different one with more built-in storage capacity. Thankfully, due to advanced soldering techniques, it is now possible to upgrade the storage space of the newest MacBooks to their highest configuration, for prices lower than what is originally quoted by Apple at purchase.

## Liquid Damage Failure

On MacBook models after 2018, the power delivery system is particularly susceptible to causing catastrophic failure in the event of liquid damage. Firstly, the SSD is usually placed on the outer borders of the MacBook, making it the most accessible to liquid ingress. Secondly, the power delivery system involves a buck converter circuit controller by a BGA IC. This means that high voltages are placed within millimetres of the sensitive lower-voltage lines that power the SSD’s chips. Liquid can easily cause a short circuit that sends higher voltage to the SSD, destroying the chip and preventing any possibility of data recovery. This failure mode is particularly common on the 16″ 2019 MacBook Pro A2141 due to these design characteristics.

## Working around custom firmware

MacBook Storage upgrades can be performed on all MacBook Air and MacBook Pro models, but on MacBook models containing a SoC chip, such as the T2 chip in 2018-2020 MacBooks, there are additional complications due to the custom configuration firmware loaded onto the SSD NAND chips. These NAND chips (between 2-8 depending on the board and capacity) work together on the main logic board to create an SSD array, but their configuration is encoded into each chip such that position of the chips cannot be changed. Therefore, tools such as the [JC P13](https://www.jcprogrammer.com/product/p13-nand-programmer-hard-disk-unbind-and-purple-screen-repair-for-ios-jcid/) are used at Vancouver Mac Service Centre to transfer the custom firmware to the replacement SSD array.

MacBooks with Apple Silicon SoCs, such as the M1, M2 or M3 series chips also have custom firmware that encodes the array configuration like in T2-equipped Macs. However, unlike on T2 equipped Macs, the Apple Configurator utility can re-program the SSD automatically through its restore function. This greatly improved repairability and possibility for MacBook storage upgrades since the custom firmware dumps do not have to be available for each boards and storage capacity.

## MacBook Storage Upgrade Procedure

The upgrade procedure is relatively simple – the main logic board must be removed, and the NAND chips are removed and replaced with the replacement parts. This video below outlines the procedure with a focus on the soldering technique required to safely transfer the chips.

After the chips are replaced, Apple Configurator is run on the machine through another Mac via USB-C to reprogram the SSD configuration, and correctly link the CPU to the new storage drive, reassigning the encryption parameters.

## Upgrade your MacBook’s Storage

Please [contact](https://vancouvermac.ca/contact/) us if you would like your MacBook’s storage upgraded!