# APEC SoM Engineering Notes

Things a carrier-board designer or firmware author needs to know when using the APEC SoM (EV3). GPIO assignments for the RP2350 and ESP32-C6 are in [IO.md](IO.md); the castellation pinout is at the end of this file.

## Power

* **Input voltage:** VIN on J1 pins 8–9, **6–17 V**. The two TPS82130 buck modules are rated for 3–17 V (20 V absolute maximum).
* **No input protection on the module.** There is no reverse-polarity, surge or TVS protection on VIN. The carrier must provide it. The module has 2 × 10 µF of ceramic input capacitance; add bulk capacitance on the carrier if the supply leads are long.
* **Rails the module generates** (regulator ratings; the module's own load comes out of these):

  | Rail | Castellations | Source | Rating | Notes |
  | ---- | ------------- | ------ | ------ | ----- |
  | +3V3 | J1 11–12 | TPS82130 buck from VIN | 3 A | Starts first |
  | +6V | J1 5–6 | TPS82130 buck from VIN | 3 A | Nominally **5.76 V**, not 6.0 V |
  | +5V | J1 2–3 | NCP59800 LDO from +6V | 1 A | |
  | +5VA | J7 4 | ADM7170 LDO from +6V | 500 mA | Also the ADC's analog supply (AVDD); keep carrier loads on it quiet |

* **Sequencing:** +3V3 comes up first. +6V, +5V and +5VA are enabled by the +3V3 power-good signal.
* **Low input voltage:** below about 6.2 V the +6V buck runs at 100 % duty cycle and +6V tracks VIN. +5V and +5VA keep working but with less headroom and more noise.
* **Don't drive the output rails from the carrier.** +3V3, +6V, +5V and +5VA are regulator outputs.

## Grounding and carrier layout

* There are nine GND castellations: J1 14, J1 31, J2 12, J2 31, J6 9, J6 10, J6 13, J6 16, J7 5. Connect all of them to a solid ground plane under the module.
* The power group on J1 (pins 2–12) has only one adjacent ground pad (J1 14). Give the return current a low-impedance path on the carrier, with a solid plane near those pins.
* Unconnected castellations: J1 and J2 pins 1 and 36, J6 and J7 pins 1 and 20 (the corners), and J1 pins 4, 7, 10 and 13 (spacers between the power rails).

## RP2350

* **BOOTSEL** (QSPI_SS) is on J1 19, through a 1 kΩ resistor on the module (R25). A carrier BOOTSEL button can go straight to GND. Hold it low while releasing reset to enter the USB bootloader.
* **Reset** (RUN, active low) is on J1 22, with a 10 kΩ pull-up on the module.
* **SWD:** SWCLK on J1 20, SWDIO on J1 21.
* **The ESP32-C6 can reset the RP2350 or force BOOTSEL** through a dual MOSFET (ESP32 GPIO23 holds RUN low, GPIO22 holds BOOTSEL low). 1.5 kΩ pull-downs keep both inactive while the ESP32 is in reset or unprogrammed. ESP32 firmware must drive these pins low (or leave them as inputs) in normal operation.
* **UART boot from the ESP32:** the ESP32 can load code into a blank RP2350 over the RP2350's UART boot interface, through buffers on the QSPI lines enabled by ESP32 GPIO14. See IO.md.
* **Used on the module:** GPIO0 (PSRAM chip select), GPIO8–11 (UART to the ESP32), GPIO27–32 (SPI bus with the settings flash and ADC, including their chip selects and the ADC's data-ready) and GPIO38–39 (I2C). The SPI and I2C lines are also brought out for the carrier; see "Shared buses" below. GPIO40–47 are the RP2350's ADC-capable pins.
* **Erratum E9:** if the fitted RP2350 is stepping A2, a Bank 0 GPIO input held between logic levels can source current and hold itself at about 2.2 V. If a carrier input relies on a pull-down, use an external pull-down of 8.2 kΩ or less. Later steppings fix this.

## Shared buses exposed to the carrier

* **SPI** (INT_SPI_MOSI/SCLK/MISO on J1 28/29/30) is shared with the settings flash (U2) and the ADS124S08. Both have their own chip selects with 10 kΩ pull-ups. Carrier SPI devices need a chip select from one of the exported GPIOs and must tri-state MISO when not selected.
* **I2C** (QWIIC_SDA/SCL on J2 2/3) is the RP2350's I2C1 bus. The FDC1004 sits on it at the fixed 7-bit address **0x50**, so carrier devices must avoid that address. The module has 10 kΩ pull-ups to 3.3 V. For 400 kHz operation or long cables, add stronger pull-ups on the carrier.

## USB hub (FE1.1s)

* Upstream port on J6 17/18. The RP2350 and ESP32-C6 use two of the hub's four downstream ports; the other two are on J6 14/15 (USB_PORT3) and J6 11/12 (USB_PORT4).
* **VBUS sense** (J6 19) must connect to the upstream port's VBUS. It is 5 V tolerant and has a 100 kΩ pull-down on the module; with no upstream VBUS the hub powers down.
* The hub runs self-powered. It has **no port power switching or over-current reporting**: the carrier supplies and current-limits VBUS on its downstream ports.
* There is **no USB ESD protection on the module**. Add it on the carrier at the connectors, and route the pairs as 90 Ω differential.

## ESP32-C6 (MINI-1U)

* The U variant has a U.FL connector and **needs an external antenna**.
* Several **strapping pins** are on the castellations: GPIO9 (boot mode, J6 3), GPIO15 (JTAG source, J6 4), MTMS/GPIO4 (J6 5) and MTDI/GPIO5 (J6 6). Their level at reset selects boot behaviour (for example, GPIO9 low enters download mode). Don't load them on the carrier in a way that changes the boot mode; see the strapping-pin section of Espressif's ESP32-C6 datasheet. GPIO8 has a 10 kΩ pull-up on the module.
* EN (J6 2) has a 10 kΩ / 1 µF power-on delay on the module. Pull it low to reset the ESP32.
* UART0 is on J2 32 (TX) / 33 (RX); JTAG is on J6 5–8.

## ADC (ADS124S08)

* AVDD is +5VA (5.0 V). **AINCOM is tied to GND on the module**, so single-ended measurements are referenced to ground. Differential measurements between any two inputs also work.
* **References:** the internal 2.5 V reference is decoupled on the module but not brought out. External reference 0 is on J7 3 (REFP0) / J7 2 (REFN0); AIN6/AIN7 double as reference 1. Leave REFN0 floating or tied to GND when unused, since the internal low-side switch can short it to ground.
* AIN0–7 are on J7 6–13. AIN8–11 are on J1 32–35 and **double as the ADC's GPIO0–3**, whose logic levels follow AVDD (5 V). Don't connect them to 3.3 V logic. Don't leave them floating if they're configured as digital inputs.
* **No input filtering or ESD protection on the module.** TI recommends an RC filter on each input and a C0G capacitor across each differential pair, close to the ADC. Put these on the carrier, plus ESD protection for off-board sensors.
* START/SYNC and CLK are tied to ground on the module. Conversions are started and stopped by SPI commands only, and the ADC always runs from its internal 4.096 MHz oscillator.
* On the carrier, route the analog inputs away from digital lines, over unbroken ground.

## Capacitive sensing (FDC1004)

* CIN1–4 are on J7 15–18, shield 1 (SHLD1) on J7 14 and shield 2 (SHLD2) on J7 19.
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

Generated from the EV3 schematic. Net names match the schematic. The repository also contains a carrier-side symbol (`APEC_SoM.kicad_sym`) and footprint (`CastellatedEdges.pretty/APEC_SoM_SMD.kicad_mod`); these have not been re-checked against EV3, so verify them against these tables before use.

### J1 (36 pins)

| Pin | Signal | Pin | Signal |
| --- | ------ | --- | ------ |
| 1 | — | 19 | RP2040_BOOTSEL_EXT (BOOTSEL through 1 kΩ) |
| 2 | +5V | 20 | RP2040_SWCLK |
| 3 | +5V | 21 | RP2040_SWD |
| 4 | — | 22 | RP2040_RESETn (RUN) |
| 5 | +6V | 23 | RP_GP33 |
| 6 | +6V | 24 | RP_GP34 |
| 7 | — | 25 | RP_GP35 |
| 8 | VIN | 26 | RP_GP36 |
| 9 | VIN | 27 | RP_GP37 |
| 10 | — | 28 | INT_SPI_MOSI |
| 11 | +3V3 | 29 | INT_SPI_SCLK |
| 12 | +3V3 | 30 | INT_SPI_MISO |
| 13 | — | 31 | GND |
| 14 | GND | 32 | ADC_AIN11 |
| 15 | RP_GP23 | 33 | ADC_AIN10 |
| 16 | RP_GP24 | 34 | ADC_AIN9 |
| 17 | RP_GP25 | 35 | ADC_AIN8 |
| 18 | RP_GP26 | 36 | — |

### J2 (36 pins)

| Pin | Signal | Pin | Signal |
| --- | ------ | --- | ------ |
| 1 | — | 19 | RP_GP7 |
| 2 | QWIIC_SDA | 20 | RP_GP12 |
| 3 | QWIIC_SCL | 21 | RP_GP13 |
| 4 | RP_GP40 | 22 | RP_GP14 |
| 5 | RP_GP41 | 23 | RP_GP15 |
| 6 | RP_GP42 | 24 | RP_GP16 |
| 7 | RP_GP43 | 25 | RP_GP17 |
| 8 | RP_GP44 | 26 | RP_GP18 |
| 9 | RP_GP45 | 27 | RP_GP19 |
| 10 | RP_GP46 | 28 | RP_GP20 |
| 11 | RP_GP47 | 29 | RP_GP21 |
| 12 | GND | 30 | RP_GP22 |
| 13 | RP_GP1 | 31 | GND |
| 14 | RP_GP2 | 32 | ESP32_U0TXD |
| 15 | RP_GP3 | 33 | ESP32_U0RXD |
| 16 | RP_GP4 | 34 | ESP32_GP1 |
| 17 | RP_GP5 | 35 | ESP32_GP0 |
| 18 | RP_GP6 | 36 | — |

### J6 (20 pins)

| Pin | Signal | Pin | Signal |
| --- | ------ | --- | ------ |
| 1 | — | 11 | USB_PORT4_DN |
| 2 | ESP32_EN | 12 | USB_PORT4_DP |
| 3 | ESP32_GPIO9 (strapping) | 13 | GND |
| 4 | ESP32_JTAG_SELECT (GPIO15, strapping) | 14 | USB_PORT3_DN |
| 5 | ESP32_TMS (GPIO4, strapping) | 15 | USB_PORT3_DP |
| 6 | ESP32_TDI (GPIO5, strapping) | 16 | GND |
| 7 | ESP32_TCK | 17 | USB_UPSTREAM_DN |
| 8 | ESP32_TDO | 18 | USB_UPSTREAM_DP |
| 9 | GND | 19 | USB_VBUS (upstream VBUS sense) |
| 10 | GND | 20 | — |

### J7 (20 pins)

| Pin | Signal | Pin | Signal |
| --- | ------ | --- | ------ |
| 1 | — | 11 | ADC_AIN2 |
| 2 | ADC_REFN0 | 12 | ADC_AIN1 |
| 3 | ADC_REFP0 | 13 | ADC_AIN0 |
| 4 | +5VA | 14 | C_SHIELD_1 (FDC1004 SHLD1) |
| 5 | GND | 15 | CIN1 |
| 6 | ADC_AIN7 | 16 | CIN2 |
| 7 | ADC_AIN6 | 17 | CIN3 |
| 8 | ADC_AIN5 | 18 | CIN4 |
| 9 | ADC_AIN4 | 19 | C_SHIELD_2 (FDC1004 SHLD2) |
| 10 | ADC_AIN3 | 20 | — |
