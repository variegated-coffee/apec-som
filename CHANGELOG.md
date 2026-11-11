# Changelog

## EV1 to EV2

* Changed C58 (APS6404L decoupling capacitor) to 1 uF per spec sheet
* Changed R13 and R15 (RP2350 EN and Bootsel override pulldowns) to 1K5 resistors, to overpower ESP32-C6 WPU at startup
* Moved capacitive traces into a stripline layer
* Moved analog traces into a stripline layer
* Changed power supply seqencing from 3V3 -> 6V -> {5V, 5VA} to 3V3 -> {6V, 5V, 5VA}
  * This allows for 5V rail operation (albeit with more noise and lower voltages) when VIN < 6.2V
* Added rails and mousebites for ease of manufacturing, and ease of depaneling
* Increased the capacitance of the 5VA output capacitor (4.7 uF to 10 uF)
* Added additional output caps for the 3V3 (100 nF), 5V (1 uF) and 6V (100 nF) output rails

## EV2 to R1A

* Tie Start/Sync and Clk on the ADS124S08 to DGND
* Added 10K pull-ups on the settings flash CS (R21) and the ADS124S08 CS (R24), so both stay deselected during power-up
* Added a 1K series resistor (R23) between Q7 and QSPI_SS (Bootsel), per the RP2350 hardware design guide
* Added a 1K series resistor (R25) between pin 69 (carrier Bootsel) and QSPI_SS, for the same reason
* Added 100K pull-downs on FE1.1s DRV (R20) and VBUSM (R6), so charging-hub mode and upstream VBUS sense don't float
* ESP32-C6 GPIO8 strap is now pulled up through 10K (R22) instead of being tied directly to 3V3
* Changed R8 from 12K to 12.1K, lowering the 6V rail from 5.8 V to 5.76 V
  * Keeps the worst-case rail within the NCP59800's 6.0 V input limit
* Changed Y3 (USB hub crystal) to a 12 MHz, 20 pF, 50 Ω max part (Yajingxin TAXM12M4RLBDDT2T), to meet the FE1.1s crystal requirements (16-20 pF, 50 Ω max ESR)
  * Footprint changed from 2.5x2.0 mm to 3.2x2.5 mm
* Corrected the U13 MP field to ADM7170ACPZ-5.0-R7
* Moved the +1V1 routing off the In1 power plane onto the outer layers
* Added thermal vias to the U13 (ADM7170) exposed pad
* Renumbered the castellations 1-104, clockwise from pin 1 (ESP32_GP0) viewed from the top, and removed the corner pins (which have no castellation) from the J1/J2/J6/J7 symbols
* Fixed the carrier-side APEC_SoM symbol: pins 35-40 (C_SHIELD_2, CIN4, CIN3, CIN2, CIN1, C_SHIELD_1) were in reverse order
* All vias are now specified as epoxy filled and capped, since many SMD pads have vias in them
* The RP2350 thermal vias are now covered by solder mask on the bottom side