# 387SX Floating-Point Unit Benchmark Results

## Test Setup

* Hardware:
  * Motherboard: [ESC WH386SX](https://theretroweb.com/motherboards/s/ecs-wh386sx)
  * CPU: AMD Am386SX/SXL-33 (33 MHz)
  * Cache: 64 KiB
  * DRAM: 16 MiB
  * Cirrus Logic GD5420 512 KiB VGA
* Software:
  * MS-DOS 6.22; benchmarks:
    * COMPTEST 2.59: Double-Precision Kilowhetstones; Double-Precision MFLOPS
    * IEEE-753 Test Vector
  * RedHat 4.2; benchmarks:
    * linpack_new; array size 200 x 200: KFLOPS

## FPUs Tested

* Intel i387SX
  * Part number: N80387SX33
  * Date/Lot: L6365982 A1
* ULSI Systems MathCo
  * Part number: US83S87
  * Date/Lot: 9128A1C
* Chips and Technologies - Super Math
  * Part number: P38700SX A (33)
  * Date/Lot: T9M20 G15, 9230-A HONG KONG
* IIT
  * Part number: XC87SLC-33
  * Date/Lot: 1004FA2F3EC0C CKJ 9523
* Cyrix - Reveal
  * Part number: 387SX
  * Date/Lot: DAE335A

### Not Tested Yet

* Intel i387SL
  * Part number: N80387SL (16~25MHz)
  * Date/Lot: C2390494 A1
* Cyrix - FasMath
  * Part number: CX-83S87-20-KP
  * Date/Lot: A30033

## Benchmark Results 

FPU              | COMPTEST (Kilowhetstones) | COMPTEST (MFLOPS) | linpack_new (KFLOPS)      | IEEE-753 Test Vector Failed Tests | Notes
-----------------|---------------------------|-------------------|---------------------------|-----------------------------------|-----------------------------------------
ULSI 8387?       | 1651.2                    | 0.226             |                           | Division, Multiplication, Scalb   |
Intel i387SX?    | 1603.2                    | 0.225             | 339.934, 339.514, 338.468 | None                              | COMPTEST reports 24.45 MHz FPU frequency
IIT XC87SLC-33   | 1703.9                    | 0.225             | 325.434, 325.049, 324.820 | None                              | COMPTEST reports 47.16 MHz FPU frequency
Cyrix Reveal     | 1834.9                    | 0.229             | 334.144, 334.959, 334.348 | None                              |
ULSI MathCo      | 1654.6                    | 0.226             | 339.095, 340.355, 340.355 | Division, Multiplication, Scalb   |
Chips Super Math |                           |                   | 337.428, 338.677, 338.885 |                                   |
Intel i387SL     |                           |                   |                           |                                   |
Cyrix FasMath    | 1880.1                    | 0.232             |                           |                                   |
