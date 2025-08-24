# APEC SoM IO assignments

## RP2350

| Peripheral | Assignment            |
| ---------- | --------------------- |
| GPIO0		 | PSRAM CS              |
| GPIO1-7    | Pins                  |
| GPIO8      | ESP32 TX              |
| GPIO9      | ESP32 RX              |
| GPIO10     | ESP32 CTS             |
| GPIO11     | ESP32 RTS             |
| GPIO11-26  | Pins                  |
| GPIO27     | Settings Flash CS     |
| GPIO28     | Internal SPI bus MISO |
| GPIO29     | ADS124S08 DRDY        |
| GPIO30     | Internal SPI bus SCLK |
| GPIO31     | Internal SPI bus MOSI |
| GPIO32     | ADS124S08 CS          |
| GPIO33-37  | Pins                  |
| GPIO38     | Internal I2C bus SDA  |
| GPIO39     | Internal I2C bus SCL  |
| GPIO40-47  | Pins                  |
| SPI1       | Internal SPI bus      |
| I2C1       | Internal I2C bus      |
| UART1      | ESP32 bus             |

## ESP32-C6

| Peripheral | Assignment              |
| ---------- | ----------------------- |
| GPIO0-1    | Pins                    |
| GPIO2      | RP2350 QSPI SD3 *       |
| GPIO3      | RP2350 QSPI SD2 *       |
| GPIO4-6    | Pins (JTAG)             |
| GPIO7      | Pins (JTAG)             |
| GPIO8      | 3V3                     |
| GPIO9      | Pins (Boot mode select) |
| GPIO12-13  | USB                     |
| GPIO14     | RP2350 QSPI SD1 *       |
| GPIO15     | Pins (JTAG select)      |
| GPIO16-17  | Pins (UART0)            |
| GPIO18     | RP2350 RTS              |
| GPIO19     | RP2350 CTS              |
| GPIO20     | RP2350 RX               |
| GPIO21     | RP2350 TX               |
| GPIO22     | RP2350 Bootsel          |
| GPIO23     | RP2350 EN               |

### Notes

* RP2350 UART pins are described from the RP2350's point of view, i.e. RP2350 RX is the pin that the ESP32 should TX on.
* Assignments to QSPI SD1-3 are passed through 3 state buffers, so as to not interrupt ordinary operations. OEn is connected to GPIO14, which is pulled up through a 10K pull-up. The purpose of these pins are intended to communicate with the RP2350 UART Boot (Data sheet 5.8) interface.