---
layout: default
title: BeagleBoneBlack with our own bootloader
---

<h1>Basics for booting a Soc</h1>

<p1>First we see things to be required for a minimal operation of the Soc</p1>
<ul>
	<li>An address pointed by PC(program counter) to a memory where a set of logical instructions can be accessed</li>
	<li>Clock</li>
	<li>A way for debugging(our one and only prints)</li>
</ul>

<h2>PC address</h2>
<p>
	We all know the instructions executed are fetched from where PC points. So the PC should point to a set of logical instructions in an accessible memory(Where read and write operations can be performed).
	So for many of the Socs this address is fixed also called reset vector address(ARMv7 PC=0x0000_0000) mostly a ROM would be sitting in that address.<br>
</p>

<h2>Clocks</h2>
<p>
As the Socs gets the power supply the Clocks start generating with a predefined frequency. It is not mandatory to change the clock speed. So clocks are started which means processor is already
executing instructions from PC.
<p>

<h2>Intialising peripherals for user interface</h2>
<p>So the processor starts executing instructions but how user intreacts with the soc. There comes the UART. So the one possible routine in the set of logical instructions is the
UART intialization. So after the UART intialized then we could see prints from UART.<br>
</p>

<h2>BeagleBoneBlack Booting sequence</h2>

<ul>
<li>Power on the device. Now PC points to 0x0000_0000(Reset vector address, where ROM is mapped. ROM contains routines for intializing the peripeherals.<br>
Peripherals are intialized so that the user code can be loaded to the device for execution through one of these peripherals. Here BBB provides several modes of peripheral boot(UART, USB, ethernet ... etc).</li>
<img src="/assets/img/bbb_memory_layout.png">
<li>Now as the peripherals are intialized the image can be loaded. But where the image gets loaded is it RAM?. No the ROM code does not intialize the DRAM. So the Soc has a 64kB SRAM(Does not require any initialization) where the image gets loaded.<br>
The SRAM mapped at 0x402F0400 address. The peripherals gets the image at loads at SRAM which is of 64KB. So if u-boot is built for BBB it contains two image MLO(loaded in SRAM) and u-boot.img(loaded in DRAM).
Why does the single image is not sufficient to be loaded in SRAM? Because SRAM is 64KB and well sofisticated bootloaders cannot be loaded in the SRAM. So the code in SRAM intializes the DRAM and then loads the image to DRAM( mapped at 0x8000_0000).
</li>
</ul>

<h2>Let's write our own Bootloader with these information</h2>
<ul>
<li>Are code starts with invalidating and disabling all the caches and tlbs. And then setup the stack pointer for the C code to execute.</li>
<a href="https://github.com/slpp95prashanth/Beaglebone-mBootloader/blob/master/arch/arm/cpu/armv7/start.S">Source code for start.S</a>
<ul>
<li>The execution starts from _start then calls cpu_init_cp15(where disabling caches and tlbs) and then calls cpu_init_crit which intern calls lowlevel_init(sets up stack pointer and calls peripheral init routine).</li>
</ul>

<li>Next disable the watchdog and intialize the UART for prints. From there on just write a minimum shell to communicate with the peripherals.</li>
<a href="https://github.com/slpp95prashanth/Beaglebone-mBootloader/blob/am33xx-integration/board/ti/am33xx/board.c">Source code for peripherals intializing routine</a>
</ul>

<p>Note:<br>
The github repository contains the mBootloader for BBB just clone the repo and checkout to am33xx-integration branch and make to build the image
Load the u-boot.bin to the Soc and can see the shell.
</p>
