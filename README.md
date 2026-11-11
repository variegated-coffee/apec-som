# All Purpose Espresso Controller System-on-Module

Espresso electronics are usually proprietary affairs with little to no expandability. The purpose of this project is to give everyone an open source alternative with lots of features and flexibility. 

The APEC SoM is a 108 pin, 1.27 mm pitch compact castellated module containing pretty much everything you need to build an espresso machine controller. The module is intended to be paired with a machine specific carrier board, containing things like connectors, FETs and passives.

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

The current hardware revision is EV2. This is an engineering validation revision. It hasn't been manufactured yet, and you shouldn't be the first one to do it.

## Disclaimer

Anything you do with this, you do at your own risk. This hardware hasn't even been manufactured by it's creator yet, and it's definitiely not certified. Your machine uses both line voltage power, high pressured hot water, steam and other dangerous components. You *really* shouldn't be using this yet; in fact, you shouldn't even consider it. Risks include but are not limited to, damaging your machine, personal injury, property damage and summoning dead Cthulhu from his sleep at R'lyeh.