# APEC SoM Engineering Notes

Things a carrier-board designer or firmware author needs to know when using the APEC SoM (R1A). GPIO assignments for the RP2350 and ESP32-C6 are in [IO.md](IO.md); the castellation pinout is at the end of this file.

## Power

* **Input voltage:** VIN on pins 79–80, **6–17 V**. The two TPS82130 buck modules are rated for 3–17 V (20 V absolute maximum).
* **No input protection on the module.** There is no reverse-polarity, surge or TVS protection on VIN. The carrier must provide it. The module has 2 × 10 µF of ceramic input capacitance; add bulk capacitance on the carrier if the supply leads are long.
* **Rails the module generates** (regulator ratings; the module's own load comes out of these):

  | Rail | Pins | Source | Rating | Notes |
  | ---- | ------------- | ------ | ------ | ----- |
  | +3V3 | 76–77 | TPS82130 buck from VIN | 3 A | Starts first |
  | +6V | 82–83 | TPS82130 buck from VIN | 3 A | Nominally **5.76 V**, not 6.0 V |
  | +5V | 85–86 | NCP59800 LDO from +6V | 1 A | |
  | +5VA | 50 | ADM7170 LDO from +6V | 500 mA | Also the ADC's analog supply (AVDD); keep carrier loads on it quiet |

* **Sequencing:** +3V3 comes up first. +6V, +5V and +5VA are enabled by the +3V3 power-good signal.
* **Low input voltage:** below about 6.2 V the +6V buck runs at 100 % duty cycle and +6V tracks VIN. +5V and +5VA keep working but with less headroom and more noise.
* **Don't drive the output rails from the carrier.** +3V3, +6V, +5V and +5VA are regulator outputs.

## Grounding and carrier layout

* There are nine GND pins: 5, 24, 49, 57, 74, 90, 93, 96 and 97. Connect all of them to a solid ground plane under the module.
* The power group (pins 76–86) has only one adjacent ground pad (pin 74). Give the return current a low-impedance path on the carrier, with a solid plane near those pins.
* Pins 75, 78, 81 and 84 are unconnected (spacers between the power rails).

## RP2350

* **BOOTSEL** (QSPI_SS) is on pin 69, through a 1 kΩ resistor on the module (R25). A carrier BOOTSEL button can go straight to GND. Hold it low while releasing reset to enter the USB bootloader.
* **Reset** (RUN, active low) is on pin 66, with a 10 kΩ pull-up on the module.
* **SWD:** SWCLK on pin 68, SWDIO on pin 67.
* **The ESP32-C6 can reset the RP2350 or force BOOTSEL** through a dual MOSFET (ESP32 GPIO23 holds RUN low, GPIO22 holds BOOTSEL low). 1.5 kΩ pull-downs keep both inactive while the ESP32 is in reset or unprogrammed. ESP32 firmware must drive these pins low (or leave them as inputs) in normal operation.
* **UART boot from the ESP32:** the ESP32 can load code into a blank RP2350 over the RP2350's UART boot interface, through buffers on the QSPI lines enabled by ESP32 GPIO14. See IO.md.
* **Used on the module:** GPIO0 (PSRAM chip select), GPIO8–11 (UART to the ESP32), GPIO27–32 (SPI bus with the settings flash and ADC, including their chip selects and the ADC's data-ready) and GPIO38–39 (I2C). The SPI and I2C lines are also brought out for the carrier; see "Shared buses" below. GPIO40–47 are the RP2350's ADC-capable pins.
* **Erratum E9:** if the fitted RP2350 is stepping A2, a Bank 0 GPIO input held between logic levels can source current and hold itself at about 2.2 V. If a carrier input relies on a pull-down, use an external pull-down of 8.2 kΩ or less. Later steppings fix this.

## Shared buses exposed to the carrier

* **SPI** (INT_SPI_MOSI/SCLK/MISO on pins 60/59/58) is shared with the settings flash (U2) and the ADS124S08. Both have their own chip selects with 10 kΩ pull-ups. Carrier SPI devices need a chip select from one of the exported GPIOs and must tri-state MISO when not selected.
* **I2C** (QWIIC_SDA/SCL on pins 34/33) is the RP2350's I2C1 bus. The FDC1004 sits on it at the fixed 7-bit address **0x50**, so carrier devices must avoid that address. The module has 10 kΩ pull-ups to 3.3 V. For 400 kHz operation or long cables, add stronger pull-ups on the carrier.

## USB hub (FE1.1s)

* Upstream port on pins 89 (D−) / 88 (D+). The RP2350 and ESP32-C6 use two of the hub's four downstream ports; the other two are on pins 92/91 (USB_PORT3) and 95/94 (USB_PORT4), D− first.
* **VBUS sense** (pin 87) must connect to the upstream port's VBUS. It is 5 V tolerant and has a 100 kΩ pull-down on the module; without upstream VBUS the hub won't connect to the host.
* The hub runs self-powered. It has **no port power switching or over-current reporting**: the carrier supplies and current-limits VBUS on its downstream ports.
* There is **no USB ESD protection on the module**. Add it on the carrier at the connectors, and route the pairs as 90 Ω differential.

## ESP32-C6 (MINI-1U)

* The U variant has a U.FL connector and **needs an external antenna**.
* Several **strapping pins** are on the castellations: GPIO9 (boot mode, pin 103), GPIO15 (JTAG source, pin 102), MTMS/GPIO4 (pin 101) and MTDI/GPIO5 (pin 100). Their level at reset selects boot behaviour (for example, GPIO9 low enters download mode). Don't load them on the carrier in a way that changes the boot mode; see the strapping-pin section of Espressif's ESP32-C6 datasheet. GPIO8 has a 10 kΩ pull-up on the module.
* EN (pin 104) has a 10 kΩ / 1 µF power-on delay on the module. Pull it low to reset the ESP32.
* UART0 is on pins 4 (TX) / 3 (RX); JTAG is on pins 98–101.

## ADC (ADS124S08)

* AVDD is +5VA (5.0 V). **AINCOM is tied to GND on the module**, so single-ended measurements are referenced to ground. Differential measurements between any two inputs also work.
* **References:** the internal 2.5 V reference is decoupled on the module but not brought out. External reference 0 is on pins 51 (REFP0) / 52 (REFN0); AIN6/AIN7 double as reference 1. Leave REFN0 floating or tied to GND when unused, since the internal low-side switch can short it to ground.
* AIN0–7 are on pins 41–48 (AINn on pin 41 + n). AIN8–11 are on pins 53–56 and **double as the ADC's GPIO0–3**, whose logic levels follow AVDD (5 V). Don't connect them to 3.3 V logic. Don't leave them floating if they're configured as digital inputs.
* **No input filtering or ESD protection on the module.** TI recommends an RC filter on each input and a C0G capacitor across each differential pair, close to the ADC. Put these on the carrier, plus ESD protection for off-board sensors.
* START/SYNC and CLK are tied to ground on the module. Conversions are started and stopped by SPI commands only, and the ADC always runs from its internal 4.096 MHz oscillator.
* On the carrier, route the analog inputs away from digital lines, over unbroken ground.

## Capacitive sensing (FDC1004)

* CIN1–4 are on pins 39, 38, 37 and 36, shield 1 (SHLD1) on pin 40 and shield 2 (SHLD2) on pin 35.
* On the module, CIN1/CIN2 are routed inside SHLD1's shield and CIN3/CIN4 inside SHLD2's. What the shields do depends on the measurement mode:

  | Mode | SHLD1 | SHLD2 | Consequence |
  | ---- | ----- | ----- | ----------- |
  | Single-ended, CAPDAC off | Tied to SHLD2 inside the chip | Tied to SHLD1 | All four channels are shielded |
  | Single-ended, CAPDAC on | Follows the selected channel | **Floating** | Only CIN1/CIN2 are properly shielded |
  | Differential CINn − CINm (n < m) | Follows CINn | Follows CINm | CIN1–CIN3 and CIN2–CIN4 work as routed; CIN1–CIN2 and CIN3–CIN4 put one channel inside the wrong shield |

  **Recommendation:** for single-ended measurements with CAPDAC (offset cancellation) enabled, use CIN1 and CIN2. Don't short SHLD1 to SHLD2 on the carrier unless the firmware never uses differential mode, since both shields are actively driven then.
* **Limits:** ±15 pF measurement range, up to 100 pF of offset per channel (cancelled by CAPDAC), and at most 400 pF load on each shield output. The module's shields contribute about 0.4 pF; the rest of the budget is for the carrier and sensor cabling.
* **Carrier layout,** following TI: put the sensor close to the module and keep the CIN traces short. Run each CIN trace between shield traces connected to the matching SHLD pin. Keep ground away from the CIN traces. If a shielded cable is used, connect its shield to the matching SHLD pin.

## Environment

* The FE1.1s USB hub is rated for **0–70 °C**, which limits the whole module. Its crystal is rated −30 to 85 °C. Mount the module away from boilers and other heat sources inside the machine.

## Castellation pinout

Generated from the R1A schematic; net names match the schematic. Pin 1 (ESP32_GP0) is marked on the module, and the numbering runs clockwise viewed from the top. There are no castellations at the board corners, so the 104 pins are numbered without gaps. The module schematic splits them into four connector symbols, one per edge.

For carrier boards, use the `APEC_SoM` symbol in `APEC_SoM.kicad_sym` and one of the footprints in `APEC_SoM.pretty`: `APEC_SoM_SMD`, `APEC_SoM_SMDTHT`, `APEC_SoM_THT` or `APEC_SoM_FlexyPin`. They use the same numbering as these tables. `APEC_SoM.pretty` also holds a 3D model of the module, `APEC_SoM.step` and `APEC_SoM.stl`. It was generated from EV2; R1A differs only in a few small parts. Symbols drawn from a copy of `APEC_SoM.kicad_sym` older than R1A have pins 35–40 in reverse order; update them from the library and check the wiring on those pins.

### Pins 1–34: Top edge, left to right (J2 on the module schematic)

| Pin | Signal | Pin | Signal |
| --- | ------ | --- | ------ |
| 1 | ESP32_GP0 | 18 | RP_GP6 |
| 2 | ESP32_GP1 | 19 | RP_GP5 |
| 3 | ESP32_U0RXD | 20 | RP_GP4 |
| 4 | ESP32_U0TXD | 21 | RP_GP3 |
| 5 | GND | 22 | RP_GP2 |
| 6 | RP_GP22 | 23 | RP_GP1 |
| 7 | RP_GP21 | 24 | GND |
| 8 | RP_GP20 | 25 | RP_GP47 |
| 9 | RP_GP19 | 26 | RP_GP46 |
| 10 | RP_GP18 | 27 | RP_GP45 |
| 11 | RP_GP17 | 28 | RP_GP44 |
| 12 | RP_GP16 | 29 | RP_GP43 |
| 13 | RP_GP15 | 30 | RP_GP42 |
| 14 | RP_GP14 | 31 | RP_GP41 |
| 15 | RP_GP13 | 32 | RP_GP40 |
| 16 | RP_GP12 | 33 | QWIIC_SCL |
| 17 | RP_GP7 | 34 | QWIIC_SDA |

### Pins 35–52: Right edge, top to bottom (J7)

| Pin | Signal | Pin | Signal |
| --- | ------ | --- | ------ |
| 35 | C_SHIELD_2 (FDC1004 SHLD2) | 44 | ADC_AIN3 |
| 36 | CIN4 | 45 | ADC_AIN4 |
| 37 | CIN3 | 46 | ADC_AIN5 |
| 38 | CIN2 | 47 | ADC_AIN6 |
| 39 | CIN1 | 48 | ADC_AIN7 |
| 40 | C_SHIELD_1 (FDC1004 SHLD1) | 49 | GND |
| 41 | ADC_AIN0 | 50 | +5VA |
| 42 | ADC_AIN1 | 51 | ADC_REFP0 |
| 43 | ADC_AIN2 | 52 | ADC_REFN0 |

### Pins 53–86: Bottom edge, right to left (J1)

| Pin | Signal | Pin | Signal |
| --- | ------ | --- | ------ |
| 53 | ADC_AIN8 | 70 | RP_GP26 |
| 54 | ADC_AIN9 | 71 | RP_GP25 |
| 55 | ADC_AIN10 | 72 | RP_GP24 |
| 56 | ADC_AIN11 | 73 | RP_GP23 |
| 57 | GND | 74 | GND |
| 58 | INT_SPI_MISO | 75 | — |
| 59 | INT_SPI_SCLK | 76 | +3V3 |
| 60 | INT_SPI_MOSI | 77 | +3V3 |
| 61 | RP_GP37 | 78 | — |
| 62 | RP_GP36 | 79 | VIN |
| 63 | RP_GP35 | 80 | VIN |
| 64 | RP_GP34 | 81 | — |
| 65 | RP_GP33 | 82 | +6V |
| 66 | RP2040_RESETn (RUN) | 83 | +6V |
| 67 | RP2040_SWD | 84 | — |
| 68 | RP2040_SWCLK | 85 | +5V |
| 69 | RP2040_BOOTSEL_EXT (BOOTSEL through 1 kΩ) | 86 | +5V |

### Pins 87–104: Left edge, bottom to top (J6)

| Pin | Signal | Pin | Signal |
| --- | ------ | --- | ------ |
| 87 | USB_VBUS (upstream VBUS sense) | 96 | GND |
| 88 | USB_UPSTREAM_DP | 97 | GND |
| 89 | USB_UPSTREAM_DN | 98 | ESP32_TDO |
| 90 | GND | 99 | ESP32_TCK |
| 91 | USB_PORT3_DP | 100 | ESP32_TDI (GPIO5, strapping) |
| 92 | USB_PORT3_DN | 101 | ESP32_TMS (GPIO4, strapping) |
| 93 | GND | 102 | ESP32_JTAG_SELECT (GPIO15, strapping) |
| 94 | USB_PORT4_DP | 103 | ESP32_GPIO9 (strapping) |
| 95 | USB_PORT4_DN | 104 | ESP32_EN |
