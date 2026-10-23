# Bill of Materials

This is the current, as-built component list, pulled from the Altium
captures for both boards (Section 4 of the report). It supersedes the costed
BOM in the original proposal (kept in `docs/EEE383_Group01_Original_Proposal.pdf`
for the sourcing and cost record).

| Subsystem | Component | Model / Value | Qty |
| --- | --- | --- | --- |
| 3.1 Power (per board) | LiPo battery | 3.7 V, 1000 mAh | 1 |
| 3.1 Power (per board) | Battery charger module | TP4056 (USB, onboard protection) | 1 |
| 3.1 Power (per board) | Buck regulator module | LM2596, sold as MC00100 | 1 |
| 3.2 Sensing | MEMS accelerometer breakout | MPU-6050, sold as OC09600 | 1 |
| 3.2 Sensing | Piezoelectric vibration module | RBD-2229, 4-pin (DO, AO, VCC, GND) | 1 |
| 3.3 Filter | Dual op-amp IC | LM358N (DIP-8) | 1 |
| 3.3 Filter | Charge-amp resistors | R2 = 100k, R3 = 100k | 2 |
| 3.3 Filter | Sallen-Key filter resistors | R1 = 100k, R4 = 100k | 2 |
| 3.3 Filter | Sallen-Key filter capacitors | C1 = 100nF, C3 = 100nF | 2 |
| 3.3 Filter | Input coupling capacitor | C2 = 100nF | 1 |
| 3.3 Filter | VREF bias divider | R5 = 27k, R6 = 10k, C5 = 100nF | 3 |
| 3.4 FFT | No dedicated hardware | arduinoFFT v2 library on the ESP32-S3 | - |
| 3.5 MCU/Comm | Sensing-node MCU | ESP32-S3-DEVKITC-1U-N8R8 | 1 |
| 3.5 MCU/Comm | Alarm/gateway MCU | ESP32 DevKitC (classic ESP32) | 1 |
| 3.6 Alarm | Status LEDs | 3 discrete through-hole LEDs (red, green, blue) | 3 |
| 3.6 Alarm | LED series resistors | 220 ohm | 3 |
| 3.6 Alarm | Buzzer series resistor | 100 ohm | 1 |
| 3.6 Alarm | Piezo buzzer element | CSS-73B16K-SMT | 1 |

Two things changed from the original proposal, both flagged in the report:

1. The regulator on both boards is an LM2596 buck module, not the linear
   AP2112K assumed in the earlier calculation drafts (see the note in the
   main README about the two superseded calculation documents).
2. The status indicator is three separate through-hole LEDs, not a single
   RGB package, which changes the driver current calculation in Subsystem
   3.6 (still a simple `(Vbat - Vf) / R` per channel, now applied three
   times instead of once per color plane).

Sourcing: RoboticsBD primary, TechShopBD and the local Nawabpur Road /
Stadium Market as fallbacks, Daraz and LeetechBD as further fallbacks.
RoboticsBD does not stock discrete resettable polyfuses; those need a
fallback supplier if the F1 footprint is populated.
