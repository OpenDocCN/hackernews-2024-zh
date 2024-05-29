<!--yml
category: 未分类
date: 2024-05-27 15:15:17
-->

# MicroZed Alveo Edition: Installing XDMA Drivers.

> 来源：[https://www.adiuvoengineering.com/post/microzed-alveo-edition-installing-xdma-drivers](https://www.adiuvoengineering.com/post/microzed-alveo-edition-installing-xdma-drivers)

So far in this series, we have introduced the range of Alveo cards and created our first application, based on the standard card management solution. 

In this blog we are going to install the software drivers on our Linux machine which contains the U55C and run a simple test to ensure the drivers are correctly installed. This will then allow us to look at creating additional software which allows us to exploit the DMA.

The DMA instantiated as part of the CMS reference design uses the XDMA as such we need to install the XDMA drivers for our host computer.

To get started with this we are going to first clone the XDMA GitHub repository located [here](https://github.com/Xilinx/dma_ip_drivers/tree/2022.1.5), once the directory is cloned we are able to then compile the source to create the necessary drivers for our host machine.

To do this we need to compile the source under the XDMA path, from the cloned repo we are then able to compile the source by running the make file.

However, before we do this we need to double check the U55C cards  PCi identifier is included in the file XDMA_MOD.c. This file has a declaration of device PCi ids which are associated with the vendor ID.

We can obtain the PCi identifier of the U55C by using the command

This will report the PCI ID of the fitted card, if this PCI ID is not within the XMDA_MOD.c file then add it into the list which begins at line 47

With the PCi Identifier added to the source code we are now in a position that we can compile the driver by using the make command, we need to do this from within the XDMA source directory.

Having created and installed the drivers, the next step is to compile the tool we will be using to test the compilation and installation.  Change directory into the tools directory and run the command

We can now check we have the driver installed correctly and run tests to ensure we can communicate with the Alveo U55C board. Remember to program the board first with the CMS example design using hardware manager.

To load the driver into the kernel, we need to change directory into the tests directory and run the script load driver with the command.

When this runs, you should expect to see the output as below.

To double check the XDMA driver was correctly installed and bound to the Alveo U55X we can run the command

This clearly shows the XDMA is in use as the kernel driver and module.

The final step is to run the XDMA test script also available within the tests directory, to execute the script issue the command.

This will run a series of tests between the host processor and the Alveo U55C using the XDMA driver.

As we can see these tests all pass as expected so we are able to say we have correctly installed the driver and it is working with our CMS example design.

In the next blog we will examine how to create our own software which uses this driver.

If you enjoyed the blog why not take a look at the free webinars, workshops and training courses we have created over the years. Highlights include

Do you want to know more about designing embedded systems from scratch? Check out our book on creating embedded systems. This book will walk you through all the stages of requirements, architecture, component selection, schematics, layout, and FPGA / software design. We designed and manufactured the board at the heart of the book! The schematics and layout are available in Altium [here](https://www.e3designers.com/altium-365)Learn more about the board (see previous blogs on [Bring up](https://www.adiuvoengineering.com/post/microzed-chronicles-configuring-zynq-on-a-custom-board), [DDR validation,](https://www.adiuvoengineering.com/post/microzed-chronicles-validating-your-custom-zynq-board-memory) [USB](https://www.adiuvoengineering.com/post/microzed-chronicles-smart-sensor-iot-board-getting-usb-up-and-running), [Sensors](https://www.adiuvoengineering.com/post/microzed-chronicles-petalinux-i2c-in-the-ps-and-axi-iic)) and view the schematics [here](https://www.adiuvoengineering.com/post/sensorsthink-board-schematic).