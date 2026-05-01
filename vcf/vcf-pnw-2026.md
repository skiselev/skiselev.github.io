# Modern PC/XT ISA Cards
## Audio Cards:
### Snark Barker  by TubeTime
[Website](https://tubetime.us/index.php/2019/01/19/announcing-the-snark-barker-a-100-compatible-sb-1-0-replica/) · [Source code](https://github.com/schlae/snark-barker/)

A 100% compatible clone of the famed SB 1.0 “Killer Card” sound card from 1989. It implements all the features, including the digital sound playback and recording, Ad Lib compatible synthesis, the joystick/MIDI port, and the CMS chips.

### ISA SoundBlaster 1.5 Clone  by David Larsson
[Purchase](https://www.tindie.com/products/kdehl/8-bit-isa-sound-blaster-15-clone-with-cms/)

A 100% compatible clone of the Sound Blaster 1.0 and 1.5. All old components have been switched out for modern surface-mounted chips, with the exception of the OPL2 and C/MS chips, which are kept original.

### Sonic Buster 8  by Labs LV
[Website](https://labs-lv.github.io/sb8/)

Based on a reverse-engineered SB2.0 firmware, written from scratch for an AVR MCU. It implements all the internal workings of the original card, and is fully playback-compatible. It features high-quality analog output, with significantly less background noise and aliasing.


### SAAYM  by TexElec
[Purchase](https://texelec.com/product/saaym/)

A fully functional clone of the original GameBlaster or Creative Music System (CMS) card, this card has the necessary circuitry to allow the CMS driver to load and be used in games, and high quality line-level output.

### AdLib (Clone)  by TubeTime
[Website](https://tubetime.us/index.php/2016/07/22/a-reproduction-adlib-sound-card/) · [Source code](https://github.com/schlae/adlib/)

A (nearly) exact clone of the 1990 version of the famous AdLib sound card. An OPL2 FM synthesizer connected to the ISA bus. Based on the ISA OPL2 by Sergey Kiselev, it matches the PCB layout and components as closely as possible to the original.

### ISA OPL2  by Sergey Kiselev
[Source code](https://github.com/skiselev/isa-opl2/)

A sound card based on the Yamaha OPL2 chip, the YM3812. This card is compatible with the AdLib Music Synthesizer Card that was very popular in late 80s and early 90s. Relatively easy to build and the components are widely available.

### Resound OPL3  by TexElec
[Purchase](https://texelec.com/product/resound-opl3-4-channel-opl3-sound-card-8-bit-isa/)

This card uses the famous Yamaha OPL3 chipset, used by many popular sound cards in the late 80s and early 90s. It is backwards compatible with the OPL2, so works for any AdLib-supporting game. It also features 4-channel surround sound.

## Floppy Disk Controllers
### Monster FDC  by Sergey Kiselev
[Source code](https://github.com/skiselev/monster-fdc/) · [Purchase](https://www.tindie.com/products/weird/monster-fdc/)

ISA card with two floppy disk controllers, supporting up to eight floppy drives, with configurable IRQ and DMA channels for the secondary FDC. It includes a configurable UART serial port, and can be built without some components for partial functionality.


### Quad-Flop  by TexElec
[Purchase](https://texelec.com/product/quad-flop-four-port-isa-floppy-controller/)

A floppy disk controller supporting up to four floppy drives, including 8” drives. It features an extra AVR microcontroller to display the read status on a 3-digit display. It also has an edge connector for drives 0/1, for use in older systems.


## IDE Disk Controllers:
### ISA IDE to SD Adapter  by TexElec
[Purchase](https://texelec.com/product/isa-ide-to-sd-adapter/)

Functionally equivalent to the Lo-Tech ISA CompatFlash adapter, a combination of an XTIDE-based adapter and a IDE to SD adapter, all in one card. Works reliably on older computers, including those that struggle with CompactFlash cards.

### XT-IDE  by Glitch Works
[Website](https://www.glitchwrks.com/xt-ide/) · [Source code](https://github.com/glitchwrks/xt_ide/)

An 8 bit ISA adapter for attaching modern(ish) hard drives, DOMs, and CompactFlash cards to the ISA bus. When paired with the XT-IDE Universal BIOS, it allows IBM PC compatibles to boot from IDE drives otherwise not supported.


## Miscellaneous Cards
### 8-Bit Ethernet Controller  by Sergey Kiselev
[Source code](https://github.com/skiselev/isa8_eth/) · [Purchase](https://www.tindie.com/products/weird/isa-8-bit-ethernet-controller/)

An open-source Network Interface Controller (NIC) card, designed specifically to be used in computers with an 8-bit ISA bus, such as PC/XT compatibles. It is based on Realtek RTL8019AS Ethernet controller and is NE2000-compatible.

### RTC8088  by Sergey Kiselev & Aitor Gómez García
[Source code](https://github.com/skiselev/RTC8088/) · [Purchase](https://www.tindie.com/products/weird/rtc8088-isa-real-time-clock/)

A Real Time Clock for your PC/XT. Very low profile, simple and open hardware. Originally designed by Aitor Gómez García for use in Micro 8088 systems, updated to add I/O port configuration switches and a DOS device driver.
