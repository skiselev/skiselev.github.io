# Z80 Compatible CPUs Type Detection

## Notes

* This document uses 'F' suffix to indicate flags, e.g., CF for carry flags. That is to distinguish flags for registers.

## MME U880 / Thesys Z80 CPUs Test

MME U880 is an East German Z80 clone, which later after reunification of Germany was briefly produced and sold as Thesys Z80.
MME U880 is highly compatible with the Zilog Z80 NMOS CPUs. There is a known difference in the CF flags behavior with OUTI instruction.

The OUTI instruction reads a byte from (HL) and writes it to the (C) port. HL is then incremented, and B is decremented.
According to the [Zilog Z80 documentation](https://www.zilog.com/docs/z80/um0080.pdf), page 309, this instruction does not affect CF flag.
In practice, when the B value prior to executing OUTI is 1, (HL value prior to executing OUTI is 0FFFFh), most Z80 processors will clear the CF.
U880 does not clear the CF in this case.

* Note: Test if that is B = 0 or HL = 0 that clears the CF.

Sample test code:
```
MOV  HL,0FFFFh
MOV  BC,0100h
SCF
OUTI
JP  C,U880
; Not a U880 CPU
Z80:
...
; A U880 CPU
U880:
...
```

It have been [suggested](https://www.cpushack.com/2021/01/26/the-story-of-the-soviet-z80-processor/) that it is likely that soviet/post-soviet T34VM1 / KR1858VM1 CPUs were manufactured using either U880 dies or masks.
Two KR1858VM1 CPUs, dated 9305 and 9306 do not pass the test above, and work more like the original Z80, which suggests an independent design or reverse engineering.

## NMOS vs. CMOS CPUs Test

The behavior of the undocumented instruction with the op code **0EDh, 71h** is different between most NMOS and CMOS Z80 processors.
* On NMOS CPUs, **0EDh, 71h** outputs 0 to the I/O port (C), performing OUT (C),0 instruction
* On most CMOS CPUs, **0EDh, 71h** outputs 0FFh to the I/O port (C), performing OUT (C),0FFh instruction
* Exception: Sharp LH5080A - the CMOS Z80 implementation, performs OUT (C),0, just like NMOS CPUs

Sample test code
```
MOV	C,82h  ; note - the port needs to be a read/write port, that can be read back
DB	0EDh, 071h	; undocumented OUT (C),<0|0FFH> instruction
IN  A,(82h)  ; read back the register
CP  0  ; is it zero?
JP  Z,NMOS
; A CMOS CPU
CMOS:
...
; An NMOS CPU
NMOS
...
```

## Undocumented Flags Behavior

Z80 FLAGS register has two officially unused flag bits:
* FLAGS.5 - often called YF
* FLAGS.3 - often called XF

The behavior of these bits has been researched by many people and documented in several places online. Although, through my research I found that most of the
documentation is either incomplete or incorrect. More specifically, on some CPU types in certain cases these flags are not set up deterministically and are possibly set to whatever a floating signal inside a processor reads at that moment.

### ALU Instructions

Some people [theorized](https://github.com/redcode/Z80_XCF_Flavor) that there is a hidden Q register, that keeps the result ALU operations. And for the ALU instructions, FLAGS.5 and FLAGS.3 bits will be set to the corresponding bits of that result, Q register. For example, OR 28h will set both bits, while AND 0D7h will clear both.

The things get more interesting for the instructions that do not produce an 8-bit result, particularly:

### SCF and CCF Instructions

The effect SCF and CCF instructions on FLAGS.5 and FLAGS.3 bits is known to be different between different Z80 CPU types and manufacturers.

There seem to be several cases here:

#### SCF or CCF instruction follows an instruction that sets FLAGS

All tested CPUs produce a consistent and reproducible result when SCF or CCF follow another instruction that sets FLAGS, excluding POP AF. In this case, the resulting flags seem simply to be a copy of the corresponding A register (accumulator) bits.

#### SCF or CCF instruction when both FLAGS.5 and A.5, or FLAGS.3 and A.3 are either set or not set

In the case were SCF or CCF is executed after an instruction that does not modify flags, for example POP AF, and both FLAGS.5 and A.5 or FLAGS.3 and A.3 are set or not set, the result seems to be a logic AND between the corresponding bits, that is:
FLAGS.5 = FLAGS.5 & A.5
FLAGS.3 = FLAGS.3 & A.3

#### SCF or CCF instruction when FLAGS.5 or FLAGS.3 are set, but A.5 and A.5 are not set

Behavior in this case depends on the CPU type. Here is the list of known/tested CPUs and their corresponding behaviors:

* NMOS Zilog Z80: Z80 CPU, Z0840006PSC; CMOS Zilog Z80 - Z84C0010PEG; Most NMOS Z80 clones, Most CMOS Z80 clones
  * FLAGS.5 = FLAGS.5
  * FLAGS.3 = FLAGS.3
* CMOS Sharp LH5080A
  * FLAGS.5 = 0
  * FLAGS.3 = 0
* CMOS NEC D70008AC-6
  * FLAGS.5 = FLAGS.5
  * FLAGS.3 = random, it seems to randomly affected by other set bits in FLAGS
* Overclocked KR1858VM1 (@8 MHz)
  * FLAGS.5 = 0
  * FLAGS.3 = 0

#### SCF or CCF instruction when FLAGS.5 or FLAGS.3 are not set, but A.5 and A.5 are set

Behavior in this case depends on the CPU type. Here is the list of known/tested CPUs and their corresponding behaviors:

* NMOS Zilog Z80: Z80 CPU, Z0840006PSC; CMOS Zilog Z80 - Z84C0010PEG, Thesys Z80H, 
  * FLAGS.5 = 1
  * FLAGS.3 = 1
* CMOS Toshiba TMPZ84C00AP, CMOS ST Z84C00AB6, CMOS Sharp LH5080A
  * FLAGS.5 = 0
  * FLAGS.3 = 1
* CMOS Sharp LH5080A
  * FLAGS.5 = 0
  * FLAGS.3 = 0
* CMOS NEC D70008AC-6
  * FLAGS.5 = 0
  * FLAGS.3 = random, it seems to randomly affected by other set bits in FLAGS
* NMOS NEC D780C, NMOS Sharp LH0080A, UD880D (T3, possibly an older die?!), KR1858VM1 (at least one of two I have)
  * FLAGS.5 = 1
  * FLAGS.3 = random, often 1, possibly affected by other set bits in FLAGS
* NMOS GoldStar Z8400B
  * FLAGS.5 = random, often 1, possibly affected by other set bits in FLAGS
  * FLAGS.3 = random, often 1, possibly affected by other set bits in FLAGS
* Overclocked KR1858VM1 (@8 MHz)
  * FLAGS.5 = random, often 0, possibly affected by other set bits in FLAGS
  * FLAGS.3 = 0

## 16-bit Instructions, MEMPTR and BIT n,(HL)

BIT n,(HL) sets YF and XF to what appear to be bits 13 and 11 of an internal register that is typically refereced as MEMPTR.
The value of MEMPTR is typically modified by the instructions that use 16-bit values, for example: LD A,(rp); JP, ADD rp1,rp2; OUT (C),A, etc.

[Supposedly](https://gist.github.com/drhelius/8497817) the behavior of KR1858VM1 for LD (rp),A and OUT (port),A differs from that of other Z80 CPUs, but in my tests I wasn't able to find any differences.

## References

1. [Z80 Undocumented](http://www.myquest.nl/z80undocumented/z80-documented-v0.91.pdf)
2. [The Story of the Soviet Z80 Processor](https://www.cpushack.com/2021/01/26/the-story-of-the-soviet-z80-processor/)
3. [Z80 MEMPTR](https://gist.github.com/drhelius/8497817)
4. [Z80_XCF_Flavor](https://github.com/redcode/Z80_XCF_Flavor)
