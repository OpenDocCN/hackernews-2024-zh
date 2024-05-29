<!--yml
category: 未分类
date: 2024-05-27 14:41:12
-->

# BitLocker encryption broken in 43 seconds with sub-$10 Raspberry Pi Pico — key can be sniffed when using an external TPM | Tom's Hardware

> 来源：[https://www.tomshardware.com/pc-components/cpus/youtuber-breaks-bitlocker-encryption-in-less-than-43-seconds-with-sub-dollar10-raspberry-pi-pico](https://www.tomshardware.com/pc-components/cpus/youtuber-breaks-bitlocker-encryption-in-less-than-43-seconds-with-sub-dollar10-raspberry-pi-pico)

Bitlocker is one of the most easily accessible encryption solutions available today, being a built-in feature of Windows 10 Pro and [Windows 11](https://www.tomshardware.com/tag/windows-11) Pro that's designed to secure your data from prying eyes. However, [YouTuber stacksmashing](https://www.youtube.com/watch?v=wTl4vEednkQ) demonstrated a colossal security flaw with Bitlocker that allowed him to bypass [Windows Bitlocker](https://www.tomshardware.com/news/windows-software-bitlocker-slows-performance) in less than a minute with a cheap sub-$10 [Raspberry Pi Pico](https://www.tomshardware.com/how-to/raspberry-pi-pico-setup), thus gaining access to the encryption keys that can unlock protected data. After creating the device, the exploit only took 43 seconds to steal the master key.

To do this, the YouTuber took advantage of a known design flaw found in many systems that feature a [dedicated Trusted Platform Module](https://www.tomshardware.com/news/where-to-buy-tpm-2.0-for-windows-11), or TPM. For some configurations, Bitlocker relies on an external TPM to store critical information, such as the Platform Configuration Registers and Volume Master Key (some CPUs have this built-in). For external TPMs, the TPM key communications across an LPC bus with the CPU to send it the encryption keys required for decrypting the data on the drive.

Stacksmashing found that the communication lanes (LPC bus) between the CPU and external TPM are completely unencrypted on boot-up, enabling an attacker to sniff critical data as it moves between the two units, thus stealing the encryption keys. You can see his method in the video below. 

With this in mind, the YouTuber decided to test an attack on a ten-year-old laptop with Bitlocker encryption. His specific laptop's LPC bus is readable through an unpopulated connector on the motherboard, located right next to one of the laptop's M.2 ports. This same type of attack can be used on newer motherboards that leverage an external TPM, but these typically require more legwork to intercept the bus traffic.

To read data off the connector, the YouTuber created a cheap Raspberry Pi Pico device that could connect to the unsecured connector just by making contact with the metal pads protruding from itself. The Pico was programmed to read the raw 1s and 0s off from the TPM,  granting access to the Volume Master Key stored on the module.

Stacksmashing's work demonstrates that Windows Bitlocker, as well as external TPMs, aren't as safe as many think because the data lanes between the TPM and CPU are unencrypted. The good news is that this attack method, which has been known for some time, is relegated to discrete TPMs. If you have a CPU with a built-in TPM, like the ones in modern Intel and AMD CPUs, you should be safe from this [security](https://www.tomshardware.com/tag/security) flaw since all TPM communication occurs within the CPU itself. 

Get Tom's Hardware's best news and in-depth reviews, straight to your inbox.