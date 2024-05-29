<!--yml
category: 未分类
date: 2024-05-29 13:20:47
-->

# Almost a fully open-source boot chain for Rockchip's RK3588!

> 来源：[https://www.collabora.com/news-and-blog/blog/2024/02/21/almost-a-fully-open-source-boot-chain-for-rockchips-rk3588/](https://www.collabora.com/news-and-blog/blog/2024/02/21/almost-a-fully-open-source-boot-chain-for-rockchips-rk3588/)

Eugen Hristev
February 21, 2024

***Now included in our Debian images & available via our GitLab, you can build a complete, working BL31 (Boot Loader stage 3.1), and replace the closed binary blob with an open-source binary that anyone can compile.***

In resuming our efforts in getting Rockchip's RK3588 supported upstream, we can see that recently the boot-chain has improved in the sense that the open-source BL31 (Boot Loader stage 3.1) from TF-A is now included in our Debian images, which are published on our [GitLab](https://gitlab.collabora.com/hardware-enablement/rockchip-3588). Previously, to build U-Boot, the stage 2 SPL (Secondary Program Loader) and stage 3 U-Boot proper, it was mandatory to include a closed-source DDR training binary blob and also a pre-built BL31 blob from the vendor.

TF-A is the Trusted-Firmware for Cortex-A cores (which are also the types of cores used by the RK3588). Currently this project is the defacto standard trusted firmware for ARM SoCs, but it does not support the RK3588\. However Rockchip have sent a few patches to the TF-A project [here](https://review.trustedfirmware.org/c/TF-A/trusted-firmware-a/+/21840) to support this product. We had the chance to look at these patches, try them out, and put them together in a [dedicated repository](https://gitlab.collabora.com/hardware-enablement/rockchip-3588/trusted-firmware-a) on our RK3588 enablement effort's page. From TF-A we can now build a complete working BL31 and replace the closed binary blob with an open-source binary that we can compile ourselves.

Here is a log of RK3588 booting using the open-source TF-A BL31 stage:

```
  U-Boot SPL 2024.01-g5557bfdc (Jan 21 2024 - 02:43:56 +0000) 
  Trying to boot from MMC2
  ## Checking hash(es) for config config-1 ... OK 
  ## Checking hash(es) for Image atf-1 ... sha256+ OK 
  ## Checking hash(es) for Image u-boot ... sha256+ OK 
  ## Checking hash(es) for Image fdt-1 ... sha256+ OK 
  ## Checking hash(es) for Image atf-2 ... sha256+ OK 
  NOTICE:  BL31: v2.10.0  (release):002d8e8 
  NOTICE:  BL31: Built : 02:43:49, Jan 21 2024
```

This is an excellent step towards making the boot chain completely open, more reliable, and easier to tweak. Previously, closed source binary blobs were opaque, nobody (except Rockchip) knew what the software was doing and nobody (except Rockchip) could apply bug fixes, updates, or changes. With this new possibility, we can solve issues, improve security , provide new features, and much more due to the fact that everyone has access and can review the code in TF-A.

There are still some missing parts and the most important that is remaining right now is the DDR training blob, which is still closed source.

At the moment of writing this article, we have identified a few differences from the binary blob previously used, which we can highlight as following:

There could be more issues that are unknown at the moment and users should be aware of it.

As always, our Notes repository holds all the information and a tutorial on how to build and flash the bootloader images on boards like the Radxa's Rock-5B [here](https://gitlab.collabora.com/hardware-enablement/rockchip-3588/notes-for-rockchip-3588/-/blob/main/upstream_uboot.md?ref_type=heads). If you are keen on trying prebuilt images, our Debian recipes repository builds them every day with the latest development and [you can download them directly from here](https://gitlab.collabora.com/hardware-enablement/rockchip-3588/debian-image-recipes/-/pipelines)!