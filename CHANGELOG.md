# Changelog

## EV1 to EV2

* Changed C58 (APS6404L decoupling capacitor) to 1 uF per spec sheet
* Changed R13 and R15 (RP2350 EN and Bootsel override pulldowns) to 1K5 resistors, to overpower ESP32-C6 WPU at startup
* Moved capacitive traces into a stripline layer
* Moved analog traces into a stripline layer
* Changed power supply seqencing from 3V3 -> 6V -> {5V, 5VA} to 3V3 -> {6V, 5V, 5VA}
  * This allows for 5V rail operation (albeit with more noise and lower voltages) when VIN < 6.2V
* Added rails and V-cuts for ease of manufacturing, and ease of depaneling