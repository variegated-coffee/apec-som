# APEC SoM Manufacturing Notes

How to fabricate, assemble and bring up the APEC SoM, revision R1A. R1A hasn't been built yet. All values below come from the R1A design files (KiCad 10). Order parameters are given in JLCPCB's terms, since the project's production files are set up for JLCPCB, but any fab and assembler with the same capabilities will do.

For the pinout and carrier design, see [ENGINEERING-NOTES.md](ENGINEERING-NOTES.md).

## The panel

The board file is already a complete 70 × 70 mm panel. Order it as a single board; don't let the fab panelize it.

* The module itself is 49.53 × 29.21 mm. It sits in the middle of a frame, separated from it by 3 mm milled slots.
* The module is held only at its four corners, by mouse-bite tabs MB1–MB4. That's why the corners have no castellations.
* The frame has four more mouse-bite perforations (MB5–MB8) and the four global fiducials (FID1–FID4).
* Mouse-bite holes are 0.5 mm, non-plated.

## PCB fabrication

| Parameter | Value |
| --------- | ----- |
| Layers | 6 |
| Size | 70 × 70 mm (the panel) |
| Thickness | 1.6 mm |
| Material | FR-4 |
| Copper weight | 1 oz on all layers |
| Surface finish | ENIG |
| Castellated holes | **Yes**, on all four edges of the module |
| Via covering | **Epoxy filled and capped** (see below) |
| Vias | Through-hole only; no blind, buried or micro vias |
| Smallest via | 0.3 mm pad, 0.2 mm drill (413 of the 440 vias) |
| Smallest hole | 0.2 mm |
| Smallest track and space | 0.09 mm (3.5 mil) |
| Impedance control | Not specified in the design |

**Stackup** (from the board file, top to bottom):

| Layer | Use | Thickness |
| ----- | --- | --------- |
| F.Cu | Components, signals | 35 µm |
| Prepreg 3313 | | 0.1 mm |
| In1.Cu | Power | 35 µm |
| Core FR-4 | | 0.54 mm |
| In2.Cu | Ground | 35 µm |
| Prepreg 2116 | | 0.1 mm |
| In3.Cu | Signals (analog and capacitive-sensing inputs run here) | 35 µm |
| Core FR-4 | | 0.54 mm |
| In4.Cu | Ground | 35 µm |
| Prepreg 3313 | | 0.1 mm |
| B.Cu | Signals, test pads | 35 µm |

**Castellations.** Each of the 104 castellations has a 0.65 mm plated hole centred on the module edge, which becomes a plated half-hole when the slot is milled. A second 0.65 mm plated hole sits 0.77 mm further in, in the same copper land. Ask the fab for castellated (plated half) holes. The pads aren't marked as castellated in the design files, so the Gerbers won't flag them on their own.

**Via-in-pad.** 153 of the 478 copper SMD pads, on 70 parts, have a via inside the pad. These include 0201 passives, the ESP32-C6 module, the flash and PSRAM, and several exposed pads. Without filling and capping, solder wicks down these vias and leaves joints starved or open. The board must therefore be manufactured with epoxy filled and capped vias (at JLCPCB: "Via Covering: Epoxy Filled & Capped"). The board file marks every via that way.

## Assembly

* **Top side only.** The bottom has only eight bare test pads and the OSHW logo, so no bottom-side assembly or stencil is needed.
* **Parts:** 100 to place, all with LCSC part numbers in the schematic. The test points TP1–TP8 are in the BOM but are bare pads; leave them out of the assembly order.
* **Finest features:**
  * 69 parts in 0201 (35 capacitors, 34 resistors)
  * RP2350B (U4): QFN-80, 0.4 mm pitch, thermal vias in the exposed pad
  * ESP32-C6-MINI-1U (U7): LGA module
  * TPS82130 (U1, U3): MicroSiP LGA with thermal vias
  * ADS124S08 (U6): 32-pin, 0.5 mm pitch
  * FE1.1s (U5): QFN-24
  * Flash, PSRAM and LDOs: DFN/USON/SON
* **Assembly type:** Standard.
* **Production files:** generated with the Fabrication Toolkit plugin for KiCad, which writes them to `production/` (Gerber zip, BOM, placement file, IPC netlist). Its settings are in `fabrication-toolkit-options.json`.
  * The plugin takes LCSC numbers from the footprints on the PCB. Sync them from the schematic first; see the checklist below.
* **Rotations:** check the placement preview for rotation errors before confirming the order, especially U4, U7, U1/U3, the crystals and the SOT parts.

## Before ordering

1. **Sync part numbers:** in the PCB editor, run Tools → Update PCB from Schematic with "Update footprint fields" enabled, so the LCSC numbers on the footprints match the schematic.
2. **DRC:** refill all zones and run DRC. It should report no errors and no unconnected items.
3. **Outputs:** generate the production files. Check the castellation half-holes and the milled slots in a Gerber viewer before uploading.
4. **Order notes:** mention castellated holes and filled-and-capped vias in the order notes.

## Depaneling

* Break the module out at the four corner tabs (MB1–MB4).
* File or sand the remaining nubs flush. The tabs are about 2.3 mm wide; the nearest castellation is 3.81 mm from each corner, so keep the sanding within the tab.
* Check that no castellation is torn or burred; the half-holes are on the milled edges.

## Bring-up and programming

The module has no connectors of its own. Power and program it through a carrier board, or a fixture that contacts the castellations. Pin numbers below are castellation numbers; see the pinout in ENGINEERING-NOTES.md.

1. **First power-up.** Feed 6–17 V into VIN (pins 79–80) from a current-limited supply, with GND on any ground pin (5, 24, 49, 57, 74, 90, 93, 96, 97). Check the rails:

   | Rail | Pins | Expected |
   | ---- | ---- | -------- |
   | +3V3 | 76–77 | 3.3 V |
   | +6V | 82–83 | 5.76 V |
   | +5V | 85–86 | about 5 V |
   | +5VA | 50 | 5.0 V |

2. **USB.** Connect the hub's upstream port: D− on pin 89, D+ on pin 88, and VBUS on pin 87. Without VBUS on pin 87 the hub won't connect to the host. The RP2350 and the ESP32-C6 then both appear on USB through the hub.
3. **RP2350 (U4).** Hold BOOTSEL (pin 69) low while releasing RUN (pin 66), or while powering up. The RP2350 then enumerates as a USB mass-storage bootloader.
   * SWD is also available: SWCLK on pin 68, SWDIO on pin 67.
   * The ESP32-C6 can also load the RP2350 over its UART boot interface; see ENGINEERING-NOTES.md.
4. **ESP32-C6 (U7).** Program it over USB through the hub, or over UART0 (TX pin 4, RX pin 3).
   * To enter download mode, hold GPIO9 (pin 103) low while releasing EN (pin 104).
   * U7 has a U.FL connector and no antenna of its own. Attach an antenna before testing the radio.
5. **Test pads.** TP1–TP8 on the underside are debug access points, not needed for production test.

   | Pad | Signal |
   | --- | ------ |
   | TP1 | RP_ESP_UART_RTS |
   | TP2 | RP_ESP_UART_CTS |
   | TP3 | RP_ESP_UART_RX |
   | TP4 | RP_ESP_UART_TX |
   | TP5 | ADC_CS |
   | TP6 | ADC_DRDY |
   | TP7 | SET_FLSH_CS |
   | TP8 | FLASH2_SS |

## Mounting on a carrier

* `APEC_SoM.pretty` has a carrier footprint for each mounting style: `APEC_SoM_SMD`, `APEC_SoM_THT`, `APEC_SoM_SMDTHT` and `APEC_SoM_FlexyPin`.
* The castellations can be reflowed or hand-soldered.
* The module's underside has the eight exposed test pads and the bottom ends of the castellation holes. Keep exposed copper on the carrier out of the area under the module, apart from the footprint pads.
