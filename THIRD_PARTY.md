# Third-party content

Everything in this repository is licensed under the CERN Open Hardware Licence Version 2 – Permissive (CERN-OHL-P-2.0, see [LICENSE.txt](LICENSE.txt)), except the content listed below, which keeps its original licence.

## Library files included in this repository

| Files | Source | Licence | Changes made here |
| ----- | ------ | ------- | ----------------- |
| `Espressif.kicad_sym`, `Espressif.pretty/`, `espressif.3dshapes/` | Espressif's KiCad libraries, <https://github.com/espressif/kicad-libraries> | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) with the same exception as the KiCad libraries: designs that use them aren't covered by the licence ([licence file](https://github.com/espressif/kicad-libraries/blob/main/LICENSE.md)) | Only the ESP32-C6-MINI-1/U symbol, the ESP32-C6-MINI-1U footprint and its STEP model are kept. The footprint's 3D model path points to `espressif.3dshapes/`. |
| `ADM7170ACPZ-1/SON50P300X300X80-9N.kicad_mod` | SnapMagic (formerly SnapEDA), <https://www.snapeda.com/parts/ADM7170ACPZ-1.8-R7/Analog+Devices/view-part/> | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) with SnapMagic's Design Exception 1.0, subject to SnapMagic's [terms of service](https://www.snapeda.com/about/terms/) | Exposed pad renumbered from 9 to EP; re-saved in a newer KiCad format. |
| `RP2350_80QFN_minimal.pretty/`, `RP2350.kicad_sym` | Raspberry Pi's RP2350 minimal design (the 2024 KiCad files accompanying [Hardware design with RP2350](https://datasheets.raspberrypi.com/rp2350/hardware-design-with-rp2350.pdf)). The origin of `RP2350.kicad_sym` wasn't recorded; it is treated as part of the same kit. | Raspberry Pi's terms for its design resources, which permit their use "solely in conjunction with the Raspberry Pi products". The APEC SoM is built around a Raspberry Pi RP2350. | RP2350 footprint: thermal vias covered by solder mask on the bottom. 3D model paths point to KiCad's standard models. Unused footprints and the RP2350A symbol removed. |

Each of these directories also contains a `LICENSE.md` with the same information.

## Content embedded in the design files

The schematic and PCB contain copies of symbols, footprints and 3D model references from these libraries. The library files themselves aren't included.

| Library | Licence |
| ------- | ------- |
| KiCad standard libraries | [CC BY-SA 4.0 with the KiCad libraries exception](https://www.kicad.org/libraries/license/): designs that use them, and files generated from them, aren't covered by the licence. |
| Alternate KiCad Library (AKL) by Dawid Cisło, used for Q7, U10 and U14, <https://github.com/DawidCislo/Alternate-KiCad-Library> | CC BY-SA 4.0 with the same exception as the KiCad libraries; no attribution is required in designs. |
