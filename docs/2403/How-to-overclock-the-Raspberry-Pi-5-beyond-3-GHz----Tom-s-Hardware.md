<!--yml
category: 未分类
date: 2024-05-27 15:05:35
-->

# How to overclock the Raspberry Pi 5 beyond 3 GHz! | Tom's Hardware

> 来源：[https://www.tomshardware.com/raspberry-pi/how-to-overclock-the-raspberry-pi-5-beyond-3-ghz](https://www.tomshardware.com/raspberry-pi/how-to-overclock-the-raspberry-pi-5-beyond-3-ghz)

The [Raspberry Pi 5](https://www.tomshardware.com/reviews/raspberry-pi-5) is well known to be the fastest [Raspberry Pi](https://www.tomshardware.com/raspberry-pi), it is the new flagship after all.Originally [we managed to overclock](https://www.tomshardware.com/how-to/overclock-raspberry-pi-5) the Raspberry Pi 5 to 3 GHz, a great boost over the 2.4 GHz stock speed. Many alleged that they overclocked to much higher speeds, but we confirmed with Raspberry Pi that 3 GHz was the limit. 

Well it was, but a new [firmware](https://github.com/raspberrypi/firmware/issues/1876) seems to break that speed limit. Can we push the Raspberry Pi faster? We can, but how fast is in the hands of the [silicon gods](https://mastodon.social/@aallan/112096125218793315) so your mileage may vary.

In our tests we managed a safe and stable overclock of 3.1 GHz. Hitting 3.2 GHz we encountered instability issues when running a Geekbench test. Going further to 3.3 GHz and the system crashed more times than it worked.

YouTuber and Raspberry Pi Expert Jeff Geerling managed 3.14 GHz, but anything over that speed was met with failure.

How fast can your Raspberry Pi 5 go? If you are willing to give it a go know this, if you break it, you bought it.

## For this project you will need

*   [Raspberry Pi 5](https://www.tomshardware.com/reviews/raspberry-pi-5)
*   Active cooling solution
*   Latest Raspberry Pi OS on a micro SD card
*   Spare micro SD card
*   Computer running Windows, MacOS or Linux

Before we make any overclock attempts, we need a good cooling system. At the very least you will need the official Raspberry Pi Active Cooler, Argon’s THRML Active cooler, or a passive cooler such as those from EDATEC. Do not attempt this process without a form of cooling as it may damage your Raspberry Pi. Tom’s Hardware cannot be held responsible if you break your Raspberry Pi 5.

We’ve chosen the Argon THRML 60-RC as a cooler, simply because it is $20 and it is a beast of a cooler. You could purchase the 52Pi water cooling kit, but at $120 it only provides a few degrees extra cooler for the $100 difference.

Get Tom's Hardware's best news and in-depth reviews, straight to your inbox.

1\. On your Windows / Apple / Linux PC, [**download this experimental firmware**](https://github.com/raspberrypi/firmware/files/14604820/rpi-eeprom-recovery.zip)**.**  Note, as per Raspberry Pi’s Alasdair Allan [message on Mastodon](https://mastodon.social/@aallan/112096125218793315) this is not recommended firmware and you do so at your own risk. 

2\. **Download, install and open Raspberry Pi Imager. **

3\. **For Raspberry Pi Device select “No Filtering”,** and then **for Operating System select “Use Custom”.** Then for Storage **select your micro SD card and click Next** to start the write process.

(Image credit: Tom's Hardware)

4\. **Eject the card when prompted, and insert it into a Raspberry Pi 5.** Make sure that your Raspberry Pi 5 is powered off, and that it has a keyboard, mouse and screen attached.

5\. **Power up the Raspberry Pi 5, wait for the screen to go green, then power off** and **remove the micro SD card.**

6. **Insert your Raspberry Pi OS micro SD card** and **power on the Pi to the desktop.** You’ll need the [best Raspberry Pi microSD card](https://www.tomshardware.com/best-picks/raspberry-pi-microsd-cards) to give your Pi an added speed boost. 

7\. **Update the available repositories** and then **upgrade your Raspberry Pi 5**. This will ensure we have the latest software available. It isn’t essential, but it is always prudent to keep your Raspberry Pi up to date.

```
sudo apt update && sudo apt dist-upgrade
```

8\. **Open config.txt for editing**. It’s found in the /boot directory.

```
sudo nano /boot/config.txt
```

9\. **At the bottom of the file make a new line and add these lines** to overclock the CPU to 3.1 GHz.

```
arm_freq=3100
```

10\. **An optional step. Use force turbo to run the CPU and GPU to run at maximum speed.** This overrides any scaling governors and makes the CPU and GPU run at 100%. Active cooling is essential for this to work correctly.

```
force_turbo=1
```

11\. **Add another line to add a little more voltage to the CPU.** This replaces over_voltage=X for providing extra voltage to the Pi. The delta method adds voltage on top of the current level, here we use 50000 to add 0.05V. The older over_voltage method has been deprecated for the Raspberry Pi 5.

```
over_voltage_delta=50000
```

12\. **Save the file by pressing CTRL + X, Y then ENTER.**

13\. **Reboot the Raspberry Pi 5.** If the Raspberry Pi fails to boot, power off the Raspberry Pi. Press and hold the Spacebar. This will bypass the overclocking config and boot the Pi with a default configuration.

14\. **Open a new terminal and use this command** **to see the current CPU speed of the Pi.** Press CTRL + C to stop reading the CPU speed.

```
watch -n 1 vcgencmd measure_clock arm
```

Your Raspberry Pi has now been turbo-charged to 3.1 GHz! If you are brave enough to push it further, we salute you and don’t forget to post a [Geekbench](https://browser.geekbench.com/user/biglesp) score! Give our [stress test benchmark tool](https://www.tomshardware.com/how-to/raspberry-pi-benchmark-vcgencmd) a try, it logs your CPU temperature to a CSV which can be used to graph your Pi 5.