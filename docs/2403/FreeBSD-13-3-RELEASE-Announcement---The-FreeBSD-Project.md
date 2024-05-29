<!--yml
category: 未分类
date: 2024-05-27 14:40:53
-->

# FreeBSD 13.3-RELEASE Announcement | The FreeBSD Project

> 来源：[https://www.freebsd.org/releases/13.3R/announce/](https://www.freebsd.org/releases/13.3R/announce/)

FreeBSD 13.3-RELEASE is now available for the amd64, i386, powerpc, powerpc64, powerpc64le, powerpcspe, armv6, armv7, aarch64, and riscv64 architectures.

FreeBSD 13.3-RELEASE can be installed from bootable ISO images or over the network. Some architectures also support installing from a USB memory stick. The required files can be downloaded as described in the section below.

SHA512 and SHA256 hashes for the release ISO, memory stick, and SD card images are included at the bottom of this message.

PGP-signed checksums for the release images are also available at:

A PGP-signed version of this announcement is available at:

The purpose of the images provided as part of the release are as follows:

dvd1

This contains everything necessary to install the base FreeBSD operating system, the documentation, debugging distribution sets, and a small set of pre-built packages aimed at getting a graphical workstation up and running. It also supports booting into a "livefs" based rescue mode. This should be all you need if you can burn and use DVD-sized media.

Additionally, this can be written to a USB memory stick (flash drive) for the amd64 architecture and used to do an install on machines capable of booting off USB drives. It also supports booting into a "livefs" based rescue mode.

As one example of how to use the memstick image, assuming the USB drive appears as /dev/da0 on your machine something like this should work:

```
# dd if=FreeBSD-13.3-RELEASE-amd64-dvd1.iso \
    of=/dev/da0 bs=1m conv=sync
```

Be careful to make sure you get the target (of=) correct.

disc1

This contains the base FreeBSD operating system. It also supports booting into a "livefs" based rescue mode. There are no pre-built packages.

Additionally, this can be written to a USB memory stick (flash drive) for the amd64 architecture and used to do an install on machines capable of booting off USB drives. It also supports booting into a "livefs" based rescue mode. There are no pre-built packages.

As one example of how to use the memstick image, assuming the USB drive appears as /dev/da0 on your machine something like this should work:

```
# dd if=FreeBSD-13.3-RELEASE-amd64-disc1.iso \
    of=/dev/da0 bs=1m conv=sync
```

Be careful to make sure you get the target (of=) correct.

bootonly

This supports booting a machine using the CDROM drive but does not contain the installation distribution sets for installing FreeBSD from the CD itself. You would need to perform a network based install (e.g., from an HTTP or FTP server) after booting from the CD.

Additionally, this can be written to a USB memory stick (flash drive) for the amd64 architecture and used to do an install on machines capable of booting off USB drives. It also supports booting into a "livefs" based rescue mode. There are no pre-built packages.

As one example of how to use the memstick image, assuming the USB drive appears as /dev/da0 on your machine something like this should work:

```
# dd if=FreeBSD-13.3-RELEASE-amd64-bootonly.iso \
    of=/dev/da0 bs=1m conv=sync
```

Be careful to make sure you get the target (of=) correct.

memstick

This can be written to a USB memory stick (flash drive) and used to do an install on machines capable of booting off USB drives. It also supports booting into a "livefs" based rescue mode. There are no pre-built packages.

As one example of how to use the memstick image, assuming the USB drive appears as /dev/da0 on your machine something like this should work:

```
# dd if=FreeBSD-13.3-RELEASE-amd64-memstick.img \
    of=/dev/da0 bs=1m conv=sync
```

Be careful to make sure you get the target (of=) correct.

mini-memstick

This can be written to a USB memory stick (flash drive) and used to boot a machine, but does not contain the installation distribution sets on the medium itself, similar to the bootonly image. It also supports booting into a "livefs" based rescue mode. There are no pre-built packages.

As one example of how to use the mini-memstick image, assuming the USB drive appears as /dev/da0 on your machine something like this should work:

```
# dd if=FreeBSD-13.3-RELEASE-amd64-mini-memstick.img \
    of=/dev/da0 bs=1m conv=sync
```

Be careful to make sure you get the target (of=) correct.

FreeBSD/arm SD card images

These can be written to an SD card and used to boot the supported arm system. The SD card image contains the full FreeBSD installation, and can be installed onto SD cards as small as 512Mb.

For convenience for those without console access to the system, a `freebsd` user with a password of `freebsd` is available by default for `ssh(1)` access. Additionally, the `root` user password is set to `root`, which it is strongly recommended to change the password for both users after gaining access to the system.

To write the FreeBSD/arm image to an SD card, use the `dd(1)` utility, replacing *KERNEL* with the appropriate kernel configuration name for the system.

```
# dd if=FreeBSD-13.3-RELEASE-arm-armv6-KERNEL.img \
    of=/dev/da0 bs=1m conv=sync
```

Be careful to make sure you get the target (of=) correct.

FreeBSD 13.3-RELEASE can also be purchased on DVD from several vendors. One of the vendors that will be offering FreeBSD 13.3-based products is:

Pre-installed virtual machine images are also available for the amd64 (x86_64), i386 (x86_32), AArch64 (arm64), and RISCV (riscv64) architectures in `QCOW2`, `VHD`, and `VMDK` disk image formats, as well as raw (unformatted) images.

FreeBSD 13.3-RELEASE amd64 is also available on these cloud hosting platforms:

```
  ap-south-2 region: ami-089060a45633ba665
  ap-south-1 region: ami-02dcc1601d891f579
  eu-south-1 region: ami-096876ff505a88735
  eu-south-2 region: ami-0f79768daa8a3ba50
  me-central-1 region: ami-02d3f18699b9044e8
  ca-central-1 region: ami-0ce4529f93eafe727
  eu-central-1 region: ami-0164778b543e5eb63
  eu-central-2 region: ami-0175ef9bad67508c5
  us-west-1 region: ami-08224a1938d8317b7
  us-west-2 region: ami-0249e4148b539d9eb
  af-south-1 region: ami-06779d1b6ac99d8a3
  eu-north-1 region: ami-0eecde4f1b297b2f4
  eu-west-3 region: ami-0e9aecbeff62a6964
  eu-west-2 region: ami-080e8e554b9e7f71a
  eu-west-1 region: ami-0a9749ae5c79baeee
  ap-northeast-3 region: ami-028e5798c648ba909
  ap-northeast-2 region: ami-04597ef6da3c0ecc8
  me-south-1 region: ami-0a84a1af1dfd0d260
  ap-northeast-1 region: ami-031c3791531713063
  sa-east-1 region: ami-08ec9cb0dbb7e607b
  ap-east-1 region: ami-0e175b993757d40e1
  ap-southeast-1 region: ami-074e797559eee5372
  ap-southeast-2 region: ami-09575879989825e87
  ap-southeast-3 region: ami-060d04faa6c867a5c
  us-east-1 region: ami-0dd9c166b4640d88d
  us-east-2 region: ami-027cde133a99701ec
```

These AMI IDs can be retrieved from the Systems Manager Parameter Store in each region using the keys:

```
/aws/service/freebsd/amd64/base/ufs/13.3/RELEASE
```

FreeBSD/arm64 Amazon® EC2™:
AMIs are available in the following regions:

```
  ap-south-2 region: ami-039f1df5455411b20
  ap-south-1 region: ami-0936dfeedfb80ca34
  eu-south-1 region: ami-01bfd433755596cf7
  eu-south-2 region: ami-018a8a15d1142acf9
  me-central-1 region: ami-087714379c015bce7
  ca-central-1 region: ami-09457792921858205
  eu-central-1 region: ami-010f1ca390ff93032
  eu-central-2 region: ami-09177d81d898f4019
  us-west-1 region: ami-0e8701784b2fc0612
  us-west-2 region: ami-0111db642946be5d5
  af-south-1 region: ami-0a21399a27229150f
  eu-north-1 region: ami-011ec12eea0adafdd
  eu-west-3 region: ami-0f6906e72786bb4a5
  eu-west-2 region: ami-071c9be1f0abbb967
  eu-west-1 region: ami-03e0415f3cb413977
  ap-northeast-3 region: ami-0c9e7434790d501cd
  ap-northeast-2 region: ami-0495294876a8fd7fd
  me-south-1 region: ami-031acbfed308316a7
  ap-northeast-1 region: ami-06a4f41d6812e3bd7
  sa-east-1 region: ami-035f731b7d29f80ef
  ap-east-1 region: ami-0773ec8775ce85f82
  ap-southeast-1 region: ami-08cd1a98b1595a824
  ap-southeast-2 region: ami-076121bdf01ee8cfe
  ap-southeast-3 region: ami-0f54e915d216d5f2c
  us-east-1 region: ami-011496bffb848eee0
  us-east-2 region: ami-0d3d876c2693efbdd
```

These AMI IDs can be retrieved from the Systems Manager Parameter Store in each region using the keys:

```
/aws/service/freebsd/arm64/base/ufs/13.3/RELEASE
```

```
      % gcloud compute instances create INSTANCE \
        --image freebsd-13-3-release-amd64 \
        --image-project=freebsd-org-cloud-dev
      % gcloud compute ssh INSTANCE
```

*   Microsoft® Azure™:
    FreeBSD virtual machine images will be available on the Azure Marketplace once they have completed certification for publication.

*   Hashicorp/Atlas® Vagrant™:
    Instances can be deployed using the `vagrant` utility:

```
      % vagrant init freebsd/FreeBSD-13.3-RELEASE
      % vagrant up
```