<!--yml

category: 未分类

date: 2024-05-27 15:22:09

-->

# RISC-V Bytes：使用 picoprobe 访问 Pinecil UART · Daniel Mangum

> 来源：[`danielmangum.com/posts/risc-v-bytes-accessing-pinecil-uart-picoprobe/`](https://danielmangum.com/posts/risc-v-bytes-accessing-pinecil-uart-picoprobe/)
> 
> 本帖是关于[PINE64](https://pine64.org/) Pinecil 焊接熨斗和开发板的 RISC-V Bytes 子系列的第二篇文章。 可以在[RISC-V Bytes](https://danielmangum.com/categories/risc-v-bytes/)类别下找到之前和之后的帖子。

在[最新的 Pinecil 帖子](https://danielmangum.com/posts/risc-v-bytes-soldering-pinecil-breakout-board/)中，我们介绍了如何在 Pinecil [分机板](https://pine64.com/product/pinecil-break-out-board/)上焊接标题引脚。 通过连接标题引脚，我们现在可以通过多种协议与 Pinecil 的微控制器通信。 今天我们将专注于访问通用异步收发器/转换器（UART）硬件，以便在开发机器（例如笔记本电脑）上接收来自微控制器的串行数据，例如日志消息。

然而，尽管分机板确实允许我们访问 UART 引脚，但你的开发机器可能不会暴露自己的 UART 接口。 相反，大多数机器配备了更常用的外围设备端口，例如[通用串行总线（USB）](https://en.wikipedia.org/wiki/USB)。 为了在 Pinecil 微控制器和开发机器之间进行通信，我们需要某种桥接设备。 虽然有很多可以花几美元购买的专用硬件，例如这个[USB 转串口转换器](https://www.adafruit.com/product/5335)，但自从[参加了 2023 年 Hackaday Supercon](https://danielmangum.com/posts/supercon-2023-day-1/)并且[在会议徽章上进行了黑客攻击](https://danielmangum.com/posts/supercon-2023-day-2/)，我一直在利用我的小型[Raspberry Pi Pico](https://www.raspberrypi.com/documentation/microcontrollers/raspberry-pi-pico.html)收藏来执行此类任务，其中包括将 Pico[安装在 PCB 的背面](https://hackaday.com/2023/10/18/2023-hackaday-supercon-badge-welcome-to-the-vectorscope/#under-the-hood)。

Pico 是第一个使用内部硅芯片的树莓派产品，其中包含了[RP2040](https://www.raspberrypi.com/products/rp2040/)，一个带有双 ARM [Cortex-M0+](https://developer.arm.com/Processors/Cortex-M0-Plus)处理器和高度灵活的[可编程输入/输出（PIO）](https://www.raspberrypi.com/news/what-is-pio/)系统的微控制器。 使用通用微控制器而不是专用硬件的“缺点”是我们需要编写或获取执行所需功能的固件。

幸运的是，Raspbery Pi 明确鼓励使用 Pico 进行这种类型的用例。

[Pico 入门指南](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf)的附录 A 介绍了 Pico 作为编程和调试设备的使用。为了支持这种情况，他们提供了开源固件，名为`picoprobe`，构建在[FreeRTOS](https://github.com/FreeRTOS/FreeRTOS-Kernel)和[Pico C/C++ SDK](https://github.com/raspberrypi/pico-sdk)之上。

尽管该指南主要描述了在使用第二个 Pico 时的使用情况，但`picoprobe`可以与任何暴露 UART 引脚或[串行线调试（SWD）](https://developer.arm.com/documentation/ihi0031/a/The-Serial-Wire-Debug-Port--SW-DP-/Introduction-to-the-ARM-Serial-Wire-Debug--SWD--protocol)端口的设备一起使用。树莓派还提供了一个[调试探针](https://www.raspberrypi.com/products/debug-probe/)，它本质上是一个预先在 RP2040 上预编程了`picoprobe`的 Pico，并具有专用的端口和连接器用于 UART 和 SWD。

`picoprobe` 是使用头文件配置的，然后在[src/picoprobe_config.h](https://github.com/raspberrypi/picoprobe/blob/721b69cf5c8535e57995dbdd2e74f1bbc2f36944/src/picoprobe_config.h#L68)中包含。有两个内置配置，一个是[Pico](https://github.com/raspberrypi/picoprobe/blob/721b69cf5c8535e57995dbdd2e74f1bbc2f36944/include/board_pico_config.h)，另一个是[调试探针](https://github.com/raspberrypi/picoprobe/blob/721b69cf5c8535e57995dbdd2e74f1bbc2f36944/include/board_debugprobe_config.h)。我们可以使用 Pico 配置，因为它是默认的，以下步骤可以用来构建`picoprobe`而不需要任何更新。

1.  确保所有子模块都是最新的。

```
$ git submodule update --init Submodule 'CMSIS_5' (https://github.com/ARM-software/CMSIS_5) registered for path 'CMSIS_5' Submodule 'freertos' (https://github.com/FreeRTOS/FreeRTOS-Kernel) registered for path 'freertos' Cloning into '/home/hasheddan/code/github.com/raspberrypi/picoprobe/CMSIS_5'... Cloning into '/home/hasheddan/code/github.com/raspberrypi/picoprobe/freertos'... Submodule path 'CMSIS_5': checked out 'a65b7c9a3e6502127fdb80eb288d8cbdf251a6f4' Submodule path 'freertos': checked out '2dfdfc4ba4d8bb487c8ea6b5428d7d742ce162b8' 
```

1.  创建一个构建目录并进入。

1.  使用`cmake`生成构建文件，指定 Pico SDK 应该作为操作的一部分进行获取（`PICO_SDK_FETCH_FROM_GIT`）。

```
$ PICO_SDK_FETCH_FROM_GIT=on cmake .. Using PICO_SDK_FETCH_FROM_GIT from environment ('on') Downloading Raspberry Pi Pico SDK PICO_SDK_PATH is /home/hasheddan/code/github.com/raspberrypi/picoprobe/build/_deps/pico_sdk-src Defaulting PICO_PLATFORM to rp2040 since not specified. Defaulting PICO platform compiler to pico_arm_gcc since not specified. -- Defaulting build type to 'Release' since not specified. PICO compiler is pico_arm_gcc -- The C compiler identification is GNU 10.3.1 -- The CXX compiler identification is GNU 10.3.1 -- Detecting C compiler ABI info -- Detecting C compiler ABI info - done -- Check for working C compiler: /usr/bin/arm-none-eabi-gcc - skipped -- Detecting C compile features -- Detecting C compile features - done -- Detecting CXX compiler ABI info -- Detecting CXX compiler ABI info - done -- Check for working CXX compiler: /usr/bin/arm-none-eabi-g++ - skipped -- Detecting CXX compile features -- Detecting CXX compile features - done -- The ASM compiler identification is GNU -- Found assembler: /usr/bin/arm-none-eabi-gcc Build type is Release Defaulting PICO target board to pico since not specified. Using board configuration from /home/hasheddan/code/github.com/raspberrypi/picoprobe/build/_deps/pico_sdk-src/src/boards/include/boards/pico.h -- Found Python3: /usr/bin/python3.10 (found version "3.10.12") found components: Interpreter TinyUSB available at /home/hasheddan/code/github.com/raspberrypi/picoprobe/build/_deps/pico_sdk-src/lib/tinyusb/src/portable/raspberrypi/rp2040; enabling build support for USB. BTstack available at /home/hasheddan/code/github.com/raspberrypi/picoprobe/build/_deps/pico_sdk-src/lib/btstack cyw43-driver available at /home/hasheddan/code/github.com/raspberrypi/picoprobe/build/_deps/pico_sdk-src/lib/cyw43-driver Pico W Bluetooth build support available. lwIP available at /home/hasheddan/code/github.com/raspberrypi/picoprobe/build/_deps/pico_sdk-src/lib/lwip mbedtls available at /home/hasheddan/code/github.com/raspberrypi/picoprobe/build/_deps/pico_sdk-src/lib/mbedtls -- Configuring done -- Generating done -- Build files have been written to: /home/hasheddan/code/github.com/raspberrypi/picoprobe/build 
```

1.  构建固件。

```
$ make [  2%] Built target bs2_default [  4%] Built target bs2_default_padded_checksummed_asm [  5%] Performing build step for 'PioasmBuild' [100%] Built target pioasm [  6%] No install step for 'PioasmBuild' [  7%] Completed 'PioasmBuild' [ 11%] Built target PioasmBuild [ 12%] Built target picoprobe_probe_pio_h [ 13%] Built target picoprobe_probe_oen_pio_h [ 14%] Performing build step for 'ELF2UF2Build' [100%] Built target elf2uf2 [ 15%] No install step for 'ELF2UF2Build' [ 15%] Completed 'ELF2UF2Build' [ 19%] Built target ELF2UF2Build [ 20%] Building C object CMakeFiles/picoprobe.dir/src/led.c.obj [ 21%] Building C object CMakeFiles/picoprobe.dir/src/main.c.obj [ 22%] Building C object CMakeFiles/picoprobe.dir/src/usb_descriptors.c.obj [ 23%] Building C object CMakeFiles/picoprobe.dir/src/probe.c.obj [ 24%] Building C object CMakeFiles/picoprobe.dir/src/cdc_uart.c.obj [ 25%] Building C object CMakeFiles/picoprobe.dir/src/sw_dp_pio.c.obj [ 25%] Building C object CMakeFiles/picoprobe.dir/src/tusb_edpt_handler.c.obj [ 26%] Building C object CMakeFiles/picoprobe.dir/CMSIS_5/CMSIS/DAP/Firmware/Source/DAP.c.obj [ 27%] Building C object CMakeFiles/picoprobe.dir/CMSIS_5/CMSIS/DAP/Firmware/Source/JTAG_DP.c.obj [ 28%] Building C object CMakeFiles/picoprobe.dir/CMSIS_5/CMSIS/DAP/Firmware/Source/DAP_vendor.c.obj [ 29%] Building C object CMakeFiles/picoprobe.dir/CMSIS_5/CMSIS/DAP/Firmware/Source/SWO.c.obj [ 30%] Linking CXX executable picoprobe.elf [100%] Built target picoprobe 
```

构建完成后，我们应该在`build`目录中看到一些构件。

```
$ ls CMakeCache.txt  cmake_install.cmake  elf2uf2          generated  picoprobe.bin  picoprobe.elf      picoprobe.hex  pico-sdk  probe_oen.pio.h CMakeFiles      _deps                FREERTOS_KERNEL  Makefile   picoprobe.dis  picoprobe.elf.map  picoprobe.uf2  pioasm    probe.pio.h 
```

有多种方法可以编程 Pico，但最简单的方法是在连接到开发机器时按住引导选择（`BOOTSEL`）按钮。这将导致 Pico 出现为[USB 大容量存储设备](https://en.wikipedia.org/wiki/USB_mass_storage_device_class)，这意味着您可以从您机器的文件系统访问它。

```
$ ls /media/hasheddan/RPI-RP2/ INDEX.HTM  INFO_UF2.TXT 
```

将`picoprobe.uf2`文件复制到挂载的目录中将导致 RP2040 重新启动并开始运行固件。

```
$ cp picoprobe.uf2 /media/hasheddan/RPI-RP2/ 
```

Pico 现在应该显示为一个 USB 串行设备。

```
$ ls /dev/ | grep "ACM\|USB" ttyACM0 
```

我们可以使用`minicom`来监视串行输出。最初，我们不应该看到任何输出，因为`picoprobe`默认将其调试信息定向到`UART0`。我们最终在 USB 串行上看到的输出将是 Pinecil 微控制器的桥接输出。

```
$ minicom -D /dev/ttyACM0 -b 2000000 
```

Pinecil v2 运行在[Bouffalo Lab BL706](https://en.bouffalolab.com/product/?type=detail&id=8)芯片组上，其中包括基于[SiFive E24 核心](https://sifive-china.oss-cn-zhangjiakou.aliyuncs.com/Standard%20Core%20IP/e24_core_complex_manual_21G2.pdf)的[32 位 RISC-V 处理器](https://riscv.org/)。它受到开源项目[IronOS](https://github.com/Ralim/IronOS)的支持，最初是作为[TS100](https://www.ifixit.com/products/ts100-soldering-iron)铁的替代固件开发的。IronOS 是 Pinecil 的默认固件。

IronOS 也是基于 FreeRTOS 构建的，并利用了`bl_mcu_sdk`，它已经演变成了[bouffalo_sdk](https://github.com/bouffalolab/bouffalo_sdk)。根据[数据手册](http://download.bl602.fun/BL702_%E5%AE%98%E6%96%B9%E8%8A%AF%E7%89%87%E6%89%8B%E5%86%8C.pdf)，BL706 有两个内置 UART，并且 IronOS [配置](https://github.com/Ralim/IronOS/blob/6e2bca9699347d9d1381087149ab70ca0f6fcb4c/source/Core/BSP/Pinecilv2/peripheral_config.h#L82) `UART0` 的波特率为 `2000000`，这就是我们在上面提供 `-b 2000000` 给 `minicom` 的原因。

```
#if defined(BSP_USING_UART0) #ifndef UART0_CONFIG #define UART0_CONFIG \  { .id = 0, .baudrate = 2000000, .databits = UART_DATA_LEN_8, .stopbits = UART_STOP_ONE, .parity = UART_PAR_NONE, .fifo_threshold = 0, } #endif #endif 
```

IronOS 有一个优秀的[基于 Docker 的构建流程](https://github.com/Ralim/IronOS/blob/6e2bca9699347d9d1381087149ab70ca0f6fcb4c/Documentation/Development.md#building-pinecil-v20)，这消除了安装自己的 RISC-V 工具链的必要性。以下步骤可用于为 Pinecil v2 构建英文二进制文件。

1.  启动构建容器。

```
$ ./scripts/deploy.sh   ====>>>> Firing up & starting container...  * type "./scripts/deploy.sh clean" to delete created container (but not cached data)   ====>>>> /usr/bin/docker  compose  -f /home/hasheddan/code/github.com/Ralim/IronOS/./scripts/../Env.yml  run  --rm  builder   /build/ironos # 
```

1.  运行构建脚本。

```
/build/ironos # cd source/ && ./build.sh -l EN -m Pinecilv2   ********************************************  IronOS Firmware builder for Miniware + Pine64   by Ralim ******************************************** Available languages : BE BG CS DA DE EL EN ES ET FI FR HR HU IT JA_JP LT NB NL NL_BE PL PT RO RU SK SL SR_CYRL SR_LATN SV TR UK VI YUE_HK ZH_CN ZH_TW Requested languages : EN ******************************************** Available models : TS100 TS80 TS80P Pinecil MHP30 Pinecilv2 S60 TS101 Requested models : Pinecilv2 ******************************************** Cleaning previous builds  [Success] ******************************************** Building firmware for Pinecilv2 in EN ...  [Success] ********************************************  -- Firmwares successfully generated -- End... 
```

构建完成后，您可以键入 `exit` 退出容器。构建产物应该存在于 `source/Hexfile/` 中。

```
$ ls source/Hexfile/ Pinecilv2_EN.bin  Pinecilv2_EN.dfu  Pinecilv2_EN.elf  Pinecilv2_EN.elf.map  Pinecilv2_EN.hex 
```

通过按住标有`-`的按钮，同时将 USB 电缆插入开发计算机，即可在连接至 Breakout 板时对 Pinecil 进行编程。Pine64 为其 Bouffalo Lab 设备提供了一个名为`blisp`的[系统内编程工具（ISP）](https://en.wikipedia.org/wiki/In-system_programming)，可以从该项目的[发布页面](https://github.com/pine64/blisp/releases)下载。

安装后，可以使用以下命令将 IronOS 固件刷新到设备上。

```
$ sudo ./blisp write -c bl70x --reset Pinecilv2_EN.bin Sending a handshake... Handshake successful! Getting chip info... BootROM version 1.0.2.7, ChipID: 0000C41CD8FDD7C4 0b / 59200 (0.00%) 4092b / 59200 (6.91%) ... 59200b / 59200 (100.00%) Sending a handshake... Handshake with eflash_loader successful. Input file identified as a .bin file Erasing flash to flash boot header Flashing boot header... Erasing flash for firmware, this might take a while... Flashing the firmware 188440 bytes @ 0x00002000... 0b / 188440 (0.00%) 2052b / 188440 (1.09%) ... 188440b / 188440 (100.00%) Checking program... Program OK! Resetting the chip. Flash complete! 
```

## 将 Pinecil 连接到 Pico *链接到标题*

*查看 Pinecil 上运行的 IronOS 固件发出的串行输出的最后一步是将 Breakout 板上的 UART 引脚连接到运行 `picoprobe` 的 Pico 上。 Pico 上的 `UART1` 用于 USB-UART 桥接器，引脚 6 和 7 分别对应 `UART1 TX` 和 `UART1 RX`。 Pinecil Breakout UART 头上的 `RX` 引脚应连接到 Pico 上的引脚 6，而 `TX` 应连接到引脚 7。 最后，Breakout 板上的 UART `GND`（地）引脚应连接到 Pico 上的 `GND` 引脚，例如引脚 3。下面提供了完整的映射。

| **Pinecil Breakout** | **Pico** |
| --- | --- |
| UART RX | 引脚 6 (UART1 TX) |
| UART TX | 引脚 7 (UART1 RX) |
| UART GND | 引脚 3 (GND) |

> 要查看映射的可视化表示，请查看本文的封面图片，其中包含了布线的引脚准确描述。

完成布线，将 Pico 连接到开发机器，并运行`minicom`后，我们可以通过连接板将 Pinecil 连接到电源源。在我们的`minicom`会话中，我们应该看到以下输出。

```
dynamic memory init success,heap size = 48 Kbyte show flash cfg: jedec id   0x000000 mid            0xC2 iomode         0x11 clk delay      0x01 clk invert     0x01 read reg cmd0  0x05 read reg cmd1  0x00 write reg cmd0 0x01 write reg cmd1 0x00 qe write len   0x02 cread support  0x00 cread code     0x00 burst wrap cmd 0x77 ------------------- Enable IRQ's Hello from Pinecil via picoprobe! Pine64 Pinecilv2 Starting BLE Starting  BLE Starting...Done BLE Starting advertising 
```

这些信息在启动时通过[调用`bflb_platform_init()`](https://github.com/Ralim/IronOS/blob/6e2bca9699347d9d1381087149ab70ca0f6fcb4c/source/Core/BSP/Pinecilv2/bl_mcu_sdk/bsp/bsp_common/platform/bflb_platform.c#L72)来发出。有了构建、编程和监控固件的能力，我们可以开始在 Pinecil 硬件上进行更改和测试。在我们的下一篇文章中，我们将逐步解释 Bouffalo Lab SDK 和 IronOS 的确切操作，然后看看我们如何修改它们以包含新功能。
