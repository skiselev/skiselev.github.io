# Z80 Compatible CPUs Type Detection

## Notes

* This document uses 'F' suffix to indicate flags, e.g., CF for carry flags. That is to distinguish flags for registers.

## MME U880 / Thesys Z80 CPUs Test

MME U880 is an East German Z80 clone, which later after reunification of Germany was briefly produced and sold as Thesys Z80.
MME U880 is highly compatible with the Zilog Z80 NMOS CPUs. The only known difference so far is the CF flags behavior with OUTI instruction

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

It's been suggested that it is likely that soviet/post-soviet T34VM1 / KR1858VM1 CPUs were manufactured using either U880 dies or masks.
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

The behavior of these bits was researched by many people and documented in several places online.

### ALU Instructions
(Verify) Generally, for ALU instructions, these bits reflect the corresponding bits of the result. E.g., OR 28h will set both bits, while AND 0D7h will clear both.

Things get more interesting for the instructions that do not produce an 8-bit result, particularly:

### SCF and CCF Instructions

SCF and CCF instructions, that supposed only to affect the CF, also tend to change the YF and XF. The way these flags are changed depends on the processor type and manufacturer, and on whether the prior instruction also changed flags or not.
* Zilog CPUs, regardless of the previously executed instruction (verify), do:
  * YF = FLAGS.5 | A.5
  * XF = FLAGS.3 | A.3
* (FIXME: add other CPUs)
* (FIXME: add Q register and behaviors when the previous instruction does or doesn't modify flags)

## 16-bit Instructions, MEMPTR and BIT n,(HL)

BIT n,(HL) sets YF and XF to what appear to be bits 13 and 11 of an internal register that is typically refereced as MEMPTR.
The value of MEMPTR is typically modified by the instructions that use 16-bit values, for example: LD A,(rp); JP, ADD rp1,rp2; OUT (C),A, etc.

Supposedly the behavior of KR1858VM1 for LD (rp),A and OUT (port),A differs from that of other Z80 CPUs, but in my tests I wasn't able to find any differences.

## References

1. [Z80 Undocumented](http://www.myquest.nl/z80undocumented/z80-documented-v0.91.pdf)
2. [The Story of the Soviet Z80 Processor](https://www.cpushack.com/2021/01/26/the-story-of-the-soviet-z80-processor/)
3. [Z80 MEMPTR](https://gist.github.com/drhelius/8497817)
