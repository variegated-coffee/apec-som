The footprints in this directory come from Raspberry Pi's RP2350 minimal design: the 2024 KiCad files accompanying "Hardware design with RP2350" (https://datasheets.raspberrypi.com/rp2350/hardware-design-with-rp2350.pdf). Raspberry Pi's terms for these design resources permit their use "solely in conjunction with the Raspberry Pi products"; the APEC SoM is built around a Raspberry Pi RP2350.

The RP2350 symbol in `../RP2350.kicad_sym` is treated as part of the same kit.

Changes made in this repository: the RP2350 footprint's thermal vias are covered by solder mask on the bottom; 3D model paths point to KiCad's standard models; unused footprints and the RP2350A symbol are removed.
