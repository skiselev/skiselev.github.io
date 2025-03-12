# Flash-based Dial-up Router

## Introduction

Long time ago I had only a dial-up connection to the Internet, and wanted to share this connection in my home network. I tried various setups, like using dial-on-demand on my "main" computer, or using Cisco 1005 router with an external modem, but wasn't satisfied. Therefore I decided to build a router myself. One of the requirements was using flash memory to store firmware, because I didn't want to have rotating and noisy hard drive in this device.

## Hardware

I used a 386SX based WH386SX motherboard with an FPU and 16MB of RAM installed. For connectivity, I used US Robotics Sportster 33.6 internal modem, and an NE2000 NIC.
I built a custom ISA board, containing 2.5MiB of flash memory and one serial port for the console.
The flash storage is implemented using five 4Mbit AM29F040B flash chips (in this design it possible to have up to 8 such devices, but I've got just 5). The flash memory is mapped through 64KiB window (at 0xd0000-0xdffff, could be changed by switches). Lower 6-bit of the 0x310 I/O register are used to select a 64KiB page that is mapped to the mentioned above memory addresses. Higher 2 bits are used to control the LEDs.
The serial port uses a 16C550 UART and MAX232 drivers.

## Software 

The router is running linux-2.4.22 kernel with busybox. I wrote a MTD (memory technology device) driver for my flash storage (based on Arcom code), the patch can be found in the attachments. The driver presents two MTD partitions:

* 512KiB - used to store kernel
* 2048KiB - used for JFFS2 filesystem, containing all the binaries and the data.

JFFS2 partition is normaly mounted read-only, and remounted read-write only to commit configuration changes.
I also write a set of simple (shell-based) CGI scripts, to control and configure the router.

## Downloads

* Flash ROM disk schematic: [PDF](flash.pdf), [EAGLE EDA](flash.sch)
* RS232 controller schematic: [PDF](rs232isa.pdf), [EAGLE EDA](rs232isa.sch)
* Linux 2.4.22 MTD patch for the ROM disk: [linux-2.4.22-asmflash.patch.bz2](linux-2.4.22-asmflash.patch.bz2)

## Photos

![Flash ROM and serial card](flash_rs232_card.jpg Flash ROM and serial card)

![WH386SX motherboard](WH386SX_motherboard.jpg WH386SX motherboard)

![Complete and working router in the rack](router_in_rack.jpg Complete and working router in the rack)
