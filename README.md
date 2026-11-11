# All Purpose Espresso Controller System-on-Module

Espresso electronics are usually proprietary affairs with little to no expandability. The purpose of this project is to give everyone an open source alternative with lots of features and flexibility. 

The APEC SoM is a 104 pin, 1.27 mm pitch compact castellated module containing pretty much everything you need to build an espresso machine controller. The module is intended to be paired with a machine specific carrier board, containing things like connectors, FETs and passives.

## High-level features

* RP2350B microcontroller with 8 MB of program flash, 8 MB of settings flash and 8 MB PSRAM
* ESP32-C6 for Wi-fi/Bluetooth/Zigbee/Thread connectivity
* ADS124S08 ADC
* FDC1004 Capacitance-to-digital
* Five different power domains
  * Vin (6-17V)
  * 6V
  * 5VD
  * 5VA
  * 3V3
* 4-port USB 2.0 hub

## Project status

The current hardware revision is R1A. It hasn't been manufactured yet, and you shouldn't be the first one to do it. The earlier engineering validation revisions, EV1 and EV2, have been manufactured; [CHANGELOG.md](CHANGELOG.md) lists what changed between revisions.

## License

The design files and documentation in this repository are licensed under the CERN Open Hardware Licence Version 2 – Permissive (CERN-OHL-P-2.0); see [LICENSE.txt](LICENSE.txt). This includes the carrier-board symbol and footprints (`APEC_SoM.kicad_sym` and `APEC_SoM.pretty`), so you can use them in your own carrier designs.

Some library files come from third parties and keep their own licences; see [THIRD_PARTY.md](THIRD_PARTY.md).

## Disclaimer

Anything you do with this, you do at your own risk. The current revision hasn't even been manufactured by its creator yet, and it's definitely not certified. Your machine uses both line voltage power, high pressured hot water, steam and other dangerous components. You *really* shouldn't be using this yet; in fact, you shouldn't even consider it. Risks include but are not limited to, damaging your machine, personal injury, property damage and summoning dead Cthulhu from his sleep at R'lyeh.