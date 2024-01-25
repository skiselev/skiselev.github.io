# Designing Homebrew Retro Computers: General Notes

*Initially published on 2024-01-25*

These are my notes on designing your own homebrew retro computer. I use the "homebrew retro computers" term for modern hobbyists designs that typically based on older, 1970's-1980's processors and other components. Often newer components used as well out of convenience or necessity.

## Introduction

Designing and building homebrew retro computers lately became a popular hobby. The community around this hobby is quite diverse. From professional electrical engineers to software people (like me) to people that don't have much to do with electronics or computers but would like to experiment. Therefore, the designs and their goals vary significantly. And as it would be expected, many people base their projects on projects and computer systems that existed previously. Unfortunately, often people replicate inefficient or poor designs, other times people ask questions that were already answered elsewhere. In this and subsequent articles, I will try to describe some of the best known-methods and some of the pitfalls to avoid.

## Clock Sources

In older 1970's-1980's designs it was common to build oscillators using logic gates and quartz crystal resonators. Integrated oscillators were not existent or expensive. Nowadays, integrated oscillators with TTL/CMOS level outputs are common and come in convenient to use "full can" - DIP-14 or "half can" - DIP-8 packages, as well as in a variety of SMD packages. I would **strongly recommend** using these integrated oscillators and avoid building your own oscillators.

### Building Oscillator Using Logic Gates

If you do insist on building your own oscillator there are several things to be aware of:

1. There are two popular oscillator circuits: parallel resonant oscillator circuit also known as Pierce oscillator, and series resonant oscillator circuit. [This article](http://www.z80.info/uexosc.htm) give a good description of both circuits. A few things to be aware of:
  * The choice between parallel and series circuit often depends on the logic gate family used: the parallel circuit works better with CMOS logic, while the series circuit works better with TTL/TTL-LS logic.
  * In both cases avoid using Schmitt trigger gates, as they lack the linear region the oscillator needs to operate. If using CMOS, ideally use unbuffered gates, such as 74HCU04, although 74HC04 and even 74HCT might work as well.
  * Crystal resonators come in parallel and series varieties and the resonator type should match the circuit type. Otherwise, the circuit's frequency will be slightly off.
2. Use appropriate load capacitors for parallel resonant oscillator circuits. There is an equation to calculate the proper values, for example, it is documented in [this article](https://microchip.my.site.com/s/article/Calculating-crystal-load-capacitor).
3. Crystal oscillator circuits are sensitive to EMI. They generally don't work well when assembled on a breadboard. When designing a PCB, to improve the stability, keep the crystal as close as possible to the logic IC, and have ground fill around the crystal.
4. Some CPUs, SoC, and peripheral controllers, such as clock generators, UARTs, floppy disk controllers, and real time clocks come with a built-in oscillator circuit. Such ICs will typically have two pins to connect a quartz
resonator labeled X1 and X2. The considerations described above also apply to these ICs.
  * Most modern ICs are CMOS based and implement parallel resonant oscillator circuits. Use parallel crystals and appropriate load capacitors. Keep the crystal as close to the IC as possible.
  * Some ICs, particularly these released in late 1970's and early 1980's, came in TTL, NMOS or CMOS versions. The crystal resonator circuit for the same IC implemented using different logic families will be different. For example:
    * The [8085 microprocessor](https://www.jameco.com/Jameco/Products/ProdDS/52062.pdf) was originally built using NMOS logic. It's CMOS version - [80C85](https://www.jameco.com/jameco/products/prodds/51705oki.pdf) was released later. Both have a parallel resonant circuit, but their recommended crystal oscillator circuits differ slightly.
    * The [8284 clock generator](https://www.jameco.com/Jameco/Products/ProdDS/52871.pdf) for 8086/8088 microprocessors was originally built using TTL-LS logic and uses a series resonant oscillator circuit. The [82C84](https://www.renesas.com/us/en/document/dst/82c84a-datasheet), a CMOS version of 8284, uses a parallel resonant circuit.
  * Consult the IC's datasheet for the oscillator selection and circuit implementation.
  * Most of these ICs can also accept TTL/CMOS input clock instead of using the built-in crystal oscillator. That might be a good choice if you have several ICs that need the same clock, or otherwise have the clock already available.

## Reset Circuit

A computer system needs a reset circuit to reset the CPU and other devices' registers and other memory cells to known default values and enable correct operation. The reset circuit typically provides a delayed reset signal after the power has been applied to the system. This delay allows the power supply voltages to stabilize, the oscillators to start running, and the processors and other peripheral controllers to run for a several clock cycles to reset their memory cells. In many cases, the reset circuit also provides a reset switch, that allows the user to reset the system at any time.

Older designs commonly use an RC delay circuit with Schmitt trigger gates to obtain the properly formed logic signal. While such an RC circuit remains valid approach, nowadays there is a variety of specialized reset and CPU supervisory ICs that provide several benefits over an RC circuit:

* The main advantage is the power supply voltage monitoring. There are two cases where its useful:
  * System power-on: When the power is applied to the system, the reset ICs waits a preset time after the supply voltage raises above certain predefined threshold, allowing the oscillators and the ICs to reset themselves properly. This is not the case with a native RC reset circuit, and although it is possible to select the RC values to provide a delay long enough to allow for a proper reset, this won't always work, e.g. if the power supply voltage ramps up too slowly. It will also be annoying to have longer reset delays when using manual reset switch.
  * Brown out detection: A "brown out" is an event when the supply voltage drops below the threshold needed for system components to operate properly. A naive RC circuit will not do anything in this case. While most reset and CPU supervisor ICs monitor supply voltage and will activate the reset signal if the supply voltage falls below certain threshold.
* Reset and CPU supervisor ICs typically have well-defined threshold voltages and delays. The designer can select a reset or CPU supervisor IC matching the system reset specifications. Consult the ICs datasheets for these specifications.
* Some CPU supervisor ICs provide additional features, such as battery backup, watchdog, and a power fail interrupt. I personally find the battery backup feature useful for my designs that implement NVRAM.

### Suggested Reset and CPU Supervisory ICs

This list is not exhaustive. It includes some of the CPU supervisory ICs already used in hobbyists' projects.

* [Maxim DS1233](https://www.analog.com/media/en/technical-documentation/data-sheets/ds1233.pdf) and other ICs in EconoReset series. Affordable, minimal, easy to use.
* [Texas Instruments TL77xxA Series](https://www.ti.com/lit/ds/symlink/tl7705a.pdf). Come in variety of voltages. Provide both active low and active high reset outputs.
* [Maxim MAX691A/MAX693A/MAX800L/MAX800M](https://www.analog.com/media/en/technical-documentation/data-sheets/MAX691A-MAX800M.pdf). More expensive, but more versatile CPU supervisory circuits. In addition to the reset signal, they provide power fail detection, a watchdog, and a battery backup option for SRAM.
* [Maxim MAX690A/MAX692A/MAX802L/MAX802M/MAX805L](https://www.analog.com/media/en/technical-documentation/data-sheets/max690a-max805l.pdf). Similar to above, but without the battery backup option.

### Implementing an RC-based Reset Circuit

Here are some recommendations for implementing an RC based reset circuit:

* Calculate the proper R and C values, based on the power supply voltage ramp-up characteristics and the time required for the system components to reset themselves. It is not a bad idea to err on larger delays.
* In an RC circuit with a reset switch, the switch typically provides a low resistance path to discharge the capacitor. It should be noted that when the switch is pressed, momentarily the current flowing through the switch can be high and exceed the switch's current specifications, resulting is sparking and degradation of the switch. It is a good idea to connect 100 - 470 ohm resistor in series with the switch to limit the said current.
* It is a good idea to have a reverse biased diode connected in parallel with the resistor. When power is turned off, this diode will provide a discharge path for the capacitor. This will keep the voltage of the reset signal close to the power supply rail voltage and prevent damage to the ICs. It will also increase the probability of the reset circuit working then power is switched back on quickly after turning it off.
* Buffer the RC reset circuit with a Schmitt trigger gate. Unbuffered signal will have a slow rising/falling timing, which will lead to unpredictable delays and might cause reset malfunctions. Some CPUs and CPU clock generators, e.g., 8085/80C85, 8224, and 8284/82C84 have a built-in Schmitt trigger at the reset input and provide conditioned reset output signal for the rest of the system.

## Power Supplies and Power Distribution

Currently regulated, compact and powerful switch mode 5V power supplies are cheaply available, and they are the preferred way to power homebrew retro computer systems. It should be noted that while USB chargers might appear
a good power supply option, some of them do not provide good output regulation, and can output slightly higher than 5V voltages. My recommendation is to avoid these USB chargers.

If the design requires other than 5V voltages, a DC-DC converter can be used. There are ready made DC-DC converters available, or alternatively a DC-DC converter can be implemented using ICs such as [MC34063A](https://www.onsemi.com/pdf/datasheet/mc34063a-d.pdf)

A few notes about the power distribution on the system's board:

* Please do use decoupling capacitors for each logic IC. 0.1uF is a common value for lower speed (around 1 MHz - 20 MHz) systems. Position these capacitors as close as possible to the IC's VCC (or other) power inputs.
* Use copper fill for the ground. Commonly the homebrew retro systems use two layer PCBs, in such case, it is a good idea to have the ground copper fill on both layers, and connect these layers with vias in multiple points.
Avoid "floating ground" where the ground path is too long.
* Use wider PCB traces for power supply rails. Keep them shorter. This is especially important if the auto-routing PCB EDA software feature is used. Some EDA software does poor job of proper power signals routing. In such a case, it might be possible to route power supply traces manually, and let the software route the rest of the signals.
* Similar routing considerations also apply to clock signals. Keep their traces wider, shorter, with less vias. For higher frequencies, it might be a good idea to terminate the clock and other high frequency signals using pull-up or pull-down resistors at the far from the clock source or buffer end. Alternatively a low resistance, e.g., 33 ohm, resistors can be used in series right after the signal output to reduce signal ringing.

## Serial Ports

Many retro home brew computer systems use [RS-232](https://en.wikipedia.org/wiki/RS-232) serial ports as to interact with a user as a console. There are several considerations when implementing RS-232 serial ports.

### UART Selection

UART selection depends on the design goals, CPU family used and other parameters. Here are some popular choices, their advantages, and drawbacks:
* [8250](https://en.wikipedia.org/wiki/8250_UART)/16450/16550A/[16C550](https://www.ti.com/product/TL16C550C)/[16C750](https://www.ti.com/product/TL16C750) and other 8250 compatible UARTs:
  * Pros: Built-in clock oscillator. Built-in bit rate generator. Built-in FIFO and DMA support in 16550A and newer UARTs. Some newer 16C550 and 17C550 versions support automatic RTS/CTS flow control. Flexible bus interface that works with both Intel 8xxx and Motorola 68xx style busses. Newer UARTs support high bit rates.
  Dual UART 16C552 and Quad UART 16C554 versions are available.
  * Cons: Larger DIP-40 or PLCC-44 package for a single UART. DIP-40 package is out of production (although PLCC-44 can be used in a through hole socket). Might be a bit more difficult to program.
  No built-in Z80 Interrupt Mode 2 support (can be addressed by using Z80 CTC or Z80 PIO as an interrupt controller)
* [8251/82C51 USART](https://en.wikipedia.org/wiki/Intel_8251):
  * Pros: Simple to program. Smaller DIP-28 package. Synchronous operation capability (rarely used). Fits well with 808x CPUs.
  * Cons: No built-in oscillator. Limited bit rate divisor options - divide by 16 and divide by 64 for asynchronous operation. No FIFO. Lower bit rates: 19200 bps for the original NMOS 8251 and 38400 bps for CMOS 82C51
* [Z80 SIO](https://www.zilog.com/docs/z80/ps0183.pdf) and [Z80 DART](https://www.jameco.com/Jameco/Products/ProdDS/2287959.pdf):
  * Pros: Two channels in a single DIP-40 package (SIO/0, SIO/1, SIO/2, DART). Works well with Z80 CPU using interrupt mode 2. Synchronous operation capability for Z80 SIO (rarely used). Small FIFO 4 bytes for receive.
  * Cons: No built-in oscillator. Limited bit rate divisor options - divide by 1, 16, 32 and 64. The interrupts implementation doesn't integrate well with CPUs other than Z80. There are 4 SIO ICs: SIO/0, SIO/1, SIO/2, and SIO/4 with slightly different pinout.
  Requires the designer to pick the suitable version.
* [MC6850/MC68B50 ACIA](https://www.jameco.com/Jameco/Products/ProdDS/43633.pdf):
  * Pros: Smaller DIP-24 package. Integrates well with MC6800 and 6502 CPUs. Supports up to 1 Mbps bitrates.
  * Cons: No longer in production. No built-in oscillator. Limited bit rate divisor options - divide by 1, 16 and 64. Requires additional gates to interface with 80xx and Z80 CPUs control signals.

### RS-232 Signal Levels

Original RS-232 specification uses the input levels of -3V to -15V for logic '1' and +3V to +15V for logic '0'. In computers common output levels used are -10V to -12V for logic '1' and +10V to +12V for logic '0'. Many production systems and older homebrew systems used
power supplies with -12V and +12V outputs in addition +5V used by the logic ICs. Since 1990's RS-232 transmitter ICs with built-in charge pumps capable of operating from a single 5V voltage are available. Common examples include [MAX232](https://www.analog.com/en/products/max232.html), [MAX202](https://www.analog.com/en/products/max202.html), [MAX232A](https://www.analog.com/en/products/max232a.html), [MAX3232](https://www.analog.com/en/products/max3232.html) and similar ICs. Most of these ICs require external capacitors for the charge pump. The capacitance can vary depending on the IC used. Please be sure to check the datasheet for the correct capacitance values.

Another approach common in recent years is to provide unbuffered TTL level signals and to use a [serial-to-USB converter](https://ftdichip.com/products/ttl-232r-5v/) for interfacing the system to a personal computer. It should be noted that UARTs provide limited ESD protection, and this connection is not as robust as using properly buffered RS-232 signals. It is a good idea to at least use resistors in series with the signals to reduce the potential current flowing through the UART IC is case of short-circuits between signals. I am also not a big fan of pin header connectors typically used in this case. These connectors tend to be not polarized and also, they are not rated for multiple connection cycles.

### Flow Control

Simple RS-232 implementation requires only three signals - RX, TX and ground. More complete implementations, particularly those intended to be used with modems, include 2-4 additional modem, line status, and flow control signals. Unless using your system with modems, most of these signals are not required. With that being said, I do recommend implementing RTS and CTS signals. These signals are used for flow control and become really useful when transferring large amounts of "burst" data from a typically faster PC to a slower homebrew vintage systems. Sending files using XMODEM protocol and simply coping and pasting text from the host computer work much better with RTS and CTS flow control and might be a complete PITA without it.

## Extension Busses

Many homebrew vintage systems are modular, and designers would like to have a way to extend their systems in the future. Therefore, these systems will include an extension bus.
There are multiple considerations when designing data busses. Also, there are many industry standard bus specifications already exist, and thoroughly documented. In addition, there are several homebrew busses that appeared recently.
Yet, in some cases, designers of homebrew system will create yet another bus just for their own design.

Before creating your own bus, check the existing busses, their advantages and disadvantages, and check if any of these will be suitable for your design:

### Existing Extension Busses

* [S-100 Bus](https://en.wikipedia.org/wiki/S-100_bus): Originally buffered Intel 8080 CPU signals with later refinements.
  * Pros: Popular - there is an [active S-100 community that provides modern S-100 board designs](http://www.s100computers.com/My%20System%20Index%20Page.htm). Relatively easy to implement for common 8-bit CPUs. Supports multiple CPU and other bus master boards.
  Has a "vintage" feel to it, and rightfully so - the first popular "personal" computer [MITS Altair 8800](https://en.wikipedia.org/wiki/Altair_8800) used this bus.
  * Cons: Bigger, more expensive boards. Card edge connectors. Weird power supply voltages - the boards are expected to have on-board voltage regulators. Might be not as straight forward to implement with processors other than Intel 80xx.
* [ECB](https://en.wikipedia.org/wiki/Europe_Card_Bus): Europe Card Bus - Originally buffered Z80 CPU signals.
  * Pros: Somewhat popular. It is a former industry standard and there are [modern community-led designs](https://retrobrewcomputers.org/doku.php?id=boards:ecb:start) that provide various CPU and peripheral boards. Relatively easy to implement for common 8-bit CPUs, particularly for Z80/Z180 series.
  Uses nice DIN 41612 connectors. Smaller than S-100 boards - 160 x 100 mm.
  * Cons: Z80-centric, requires some additional logic for interfacing with non Z80 CPUs.
* [STD Bus](https://en.wikipedia.org/wiki/STD_Bus): Originally buffered 8085/Z80 CPU signals.
  * Pros: Former industry standard. There are some [new community-led designs](https://bitsofthegoldenage.org/documentation/). Smaller than S-100 boards. The bus is typically buffered.
  * Cons: Card edge connectors. 8085/Z80-centered signaling.
* [STEBus](https://en.wikipedia.org/wiki/STEbus): Non-proprietary, processor-independent 8-bit bus.
  * Pros: Former industry standard. Processor-independent. Typically buffered. Includes a flow control mechanism that allows slower peripherals to work with faster processors. Uses nice DIN 41612 connectors. Smaller than S-100 boards - 160 x 100 mm. 
  * Cons: The flow control mechanism and bus signaling typically requires additional logic.
* [ISA Bus](https://en.wikipedia.org/wiki/Industry_Standard_Architecture): Originally used in IBM PC and XT, later expanded in IBM PC/AT.
  * Pros: Popular - there are both commercially made and community/hobbyists' designs available. Compatible with older PCs. The controller/peripheral boards are simple to implement.
  * Cons: Card edge connectors. 8088/80286-centered signaling. Memory interface assumes 20-bit or 24-bit addresses, which might not work well with 8-bit CPUs.
* [RC2014* Bus](https://rc2014.co.uk/1377/module-template/): Basically, Z80 CPU signals, with the pinout and cards form factor specified by the RC2014* project.
  * Pros: Very popular, at least within RC2014* and current (as for 2024) Z80 homebrew computing community. Very easy to implement. Small, cheap boards. Kits are available for these who want to build a system.
  * Cons:
    * Legal: RC2014* is a registered trademark, and the trademark owner doesn't like when other people mention RC2014* in their design's names ([even if they claim their designs to be compatible](https://groups.google.com/g/rc2014-z80/c/slmvUMmSvDI/m/GorhLWITAQAJ), moreover there is no spec or tests to check for compatibility).
    Also, the trademark owner [doesn't like extensions to the published specifications](https://groups.google.com/g/rc2014-z80/c/gWI5sVqUFyc/m/MzaVrkXTBQAJ).
    * Technical: Z80-centeric, requires some additional logic for interfacing with non Z80 CPUs. The card form factor might be a limitation for bigger boards. Bus signals are not buffered. Still, this appears to work when using relatively low speed (8 MHz or so) CMOS signals.
    Uses unreliable and not polarized pin header connectors. The connector is long and might be difficult to insert and especially extract without bending the pins. The original single row specification doesn't provide sufficient ground connection.
* [RCBus](https://smallcomputercentral.com/rcbus/): A free alternative to RC2014*.
  * Pros: Basic specification is compatible with RC2014* and the modules can be used interchangeably. It seems to solve the legal issues around RC2014*. Some kits are available. Extensions beyond Z80 CPU are specified.
    Specifies several form factors, smaller, RC2014* like and larger boards.
  * Cons: Bus signals are not buffered. Still this appears to work when using relatively low speed (8 MHz or so) CMOS signals.
    Uses unreliable and not polarized pin header connectors. The connector is long, and might be difficult to insert and especially extract without bending the pins.
* [Z50Bus](http://linc.no/products/z50bus/): Like RC2014* and RCBus in spirit.
  * Pros: Uses smaller 50-pin connectors than RC2014* or RCBus. More flexible from a form factor perspective - there are several recommended form factors (but this also can be a drawback). The author doesn't seem to be too concerned about the trademarks (are they?!). 
  * Cons: Bus signals are not buffered. Still, this appears to work when using relatively low speed (8 MHz or so) CMOS signals. Uses unreliable and not polarized pin header connectors. Not as popular as RC2014*/RCBus 

### Bus Implementation Recommendations

Here are my recommendations if for whatever reason you want to create yet another bus for your design

* Use reliable connectors, that are specifically designed as backplane/card/bus connectors. DIN 41612 is one good option. Card edge connectors (such as used in ISA, STD Bus, S-100, PCI, etc.) is not a bad option, but they require expensive "hard gold" plating on the PCB.
The cost of this frequently exceeds the cost of the connectors, with worse reliability.
* Consider the signals and the future extensions you might want to have. Don't try to "fit" your bus for the given connector. Use bigger connector if necessary. It is always better to have some spare pins, than run out of them, and have to respin the bus later.
* Provide ample of ground pins. At least two ground pins. Maybe more.
* Provide enough pins for sufficient power supply current.

## Todo
* Add links for the busses and other specifications.
* Expand the Bus Implementation Recommendation section.

## Trademarks
* "RC2014" is a registered trademark of RFC2795 Ltd.
* Other names and brands may be claimed as the property of others.
