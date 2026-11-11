# APEC SoM carrier library

KiCad footprints and a 3D model for using the APEC SoM on a carrier board. Use them with the `APEC_SoM` symbol in `../APEC_SoM.kicad_sym`.

| File | Contents |
| ---- | -------- |
| `APEC_SoM_SMD.kicad_mod` | Land pattern for reflow-soldering the module's castellations |
| `APEC_SoM_THT.kicad_mod` | Through-hole variant |
| `APEC_SoM_SMDTHT.kicad_mod` | Combined SMD and through-hole variant |
| `APEC_SoM_FlexyPin.kicad_mod` | Variant for FlexyPin sockets |
| `APEC_SoM.step`, `APEC_SoM.stl` | 3D model of the module (generated from EV2; R1A differs only in a few small parts) |
| `Femtoprobe*.kicad_mod`, logo footprints | Supporting footprints |

Pins are numbered 1–104, clockwise from pin 1 (ESP32_GP0) viewed from the top. The pinout is in [ENGINEERING-NOTES.md](../ENGINEERING-NOTES.md).

## License

Licensed under the CERN Open Hardware Licence Version 2 – Permissive (CERN-OHL-P-2.0), like the rest of the APEC SoM; see [LICENSE.txt](../LICENSE.txt). You can use these files in your own carrier designs, including closed ones.
