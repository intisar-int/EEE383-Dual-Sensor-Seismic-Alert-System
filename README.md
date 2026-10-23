# Dual-Sensor Hybrid Seismic Alert System

EEE383 Electronic System Design, Section 01, BRAC University.
Project assigned 30 June 2026. Report submitted 01 September 2026.

## Objective

An early seismic warning device that sounds an alarm only when two independent
sensors agree that seismic activity is occurring, so that it does not trigger on
door slams, passing trains or trucks, or nearby construction. A piezoelectric
disc and an MPU-6050 accelerometer are each read, conditioned, and frequency-
analyzed on an ESP32, and the result is checked against real earthquake data
before the alarm fires.

## Architecture

Two physical PCBs communicate over ESP-NOW:

- **Sensing PCB**: ESP32-S3-DEVKITC-1U-N8R8, MPU-6050 breakout (I2C), RBD-2229
  piezo vibration module, LM358N two-stage conditioning (charge amplifier +
  Sallen-Key low-pass at 15.92 Hz), TP4056 charger feeding an LM2596 buck
  regulator module.
- **Alarm/Gateway PCB**: classic ESP32 DevKitC, three discrete through-hole
  LEDs (red/green/blue, direct GPIO drive), a passive piezo buzzer (PWM
  driven), the same TP4056 + LM2596 power chain, and (planned) polling of the
  EMSC LastQuake and USGS GeoJSON feeds for a Stage 2 cross-check.

Subsystems, as numbered in the report:

| # | Subsystem | Folder in this repo |
| --- | --- | --- |
| 3.1 | Primary Power Regulation and Over-Current Protection | `hardware/schematics/3_1_...`, `docs/` |
| 3.2 | Dual-Axis Seismic Sensing (MEMS + Piezo) | `hardware/schematics/3_2_...` |
| 3.3 | Active Hardware Filter and Signal Conditioning | `hardware/schematics/`, `hardware/capture-screenshots/` (Proteus traces) |
| 3.4 | FFT-Based Signal Processing | `matlab/`, `firmware/fft_pipeline_proteus_sim/` |
| 3.5 | Microcontroller and IoT Communication | `hardware/schematics/3_5_...` |
| 3.6 | Alarm and Visual Indicator | `hardware/schematics/3_6_...` |

## What is actually running right now

The hardware demonstration differs from the two-PCB design on a few points.
This is documented in Section 7 of the report and repeated here so the code
and schematics in this repo are not misread:

- **One breadboard, not two PCBs.** Sensing, filtering, consensus logic, and
  alarm output all run on a single ESP32 today. Both PCBs are etched and
  drilled (see `hardware/pcb-photos/`) but unpopulated, and ESP-NOW between
  two physical nodes has not been tested yet.
- **RMS deployed, FFT designed.** The firmware in
  `firmware/deployed_rms_alarm/` uses live-calibrated RMS/amplitude
  thresholding with detection weight shifted onto the MPU-6050, because the
  piezo disc responds to rate of deformation and stayed flat during slow
  shaking in calibration testing. The 256-point FFT pipeline in
  `firmware/fft_pipeline_proteus_sim/` and `matlab/` is validated in Proteus
  and MATLAB but not yet merged into the deployed sketch. See Section 3.4 and
  7.2 of the report.
- **USB power, not battery.** The breadboard runs from a regulated USB
  supply into the LM2596 buck stage. This is why the LM2596 headroom problem
  below has not shown up on the bench yet.

## Known open items

- **LM2596 cannot regulate from a single LiPo cell.** Its dropout puts the
  minimum input near 4.8 to 5 V, while a single-cell LiPo only supplies 3.0
  to 4.2 V. USB's ~5 V clears this, which is why USB bring-up works and
  battery bring-up has not been tried. Resolution paths from the report:
  swap in a low-dropout linear regulator, add a second cell in series, or
  accept and bench-verify a reduced rail voltage. Not yet decided.
- **F1 polyfuse footprint is unwired.** The schematic no longer routes
  through an actual fuse there; it needs a wire or a 0 ohm resistor bridge
  before the board is trusted as protected, or an actual fuse needs to be
  fitted.
- **VREF bias measured low on the breadboard.** Designed for 0.889 V
  (R5 = 27k, R6 = 10k divider), measured at 0.22 V during bring-up, which
  causes asymmetric clipping on the piezo's negative half-cycles. The
  firmware in `firmware/deployed_rms_alarm/EEE383_RMS_Alarm.ino` flags this
  at boot and logs a clipping percentage, but does not correct it in
  hardware.
- **Battery/LM2596 chain unverified on a real cell.** Calculation-only until
  bench-tested, per Section 3.1 of the report.
- **Gateway Stage 2 validation not implemented.** EMSC/USGS polling and the
  two-node ESP-NOW link are still to be written.

<!-- gallery:start -->
## Results and figures

Hardware photos, schematics, PCB layouts and simulation captures. Every raw capture is kept unedited in [`hardware/capture-screenshots/`](hardware/capture-screenshots/).

### Fabricated boards

<table>
<tr><td align="center" width="50%"><img src="hardware/pcb-photos/etched_boards_angle_1.jpg" width="100%" alt="Etched and drilled boards, unpopulated: Sensing PCB (larger) and Alarm/Gateway PCB (smaller)"><br><sub>Etched and drilled boards, unpopulated: Sensing PCB (larger) and Alarm/Gateway PCB (smaller)</sub></td><td align="center" width="50%"><img src="hardware/pcb-photos/etched_boards_angle_2.jpg" width="100%" alt="The same two boards from a second angle"><br><sub>The same two boards from a second angle</sub></td></tr>
</table>

### Subsystem schematics

<table>
<tr><td align="center" width="50%"><img src="hardware/schematics/3_1_power_regulation_schematic.png" width="100%" alt="3.1 Power regulation"><br><sub>3.1 Power regulation</sub></td><td align="center" width="50%"><img src="hardware/schematics/3_2_dual_axis_seismic_sensing.svg" width="100%" alt="3.2 Dual-axis seismic sensing (MPU-6050 and piezo)"><br><sub>3.2 Dual-axis seismic sensing (MPU-6050 and piezo)</sub></td></tr>
<tr><td align="center" width="50%"><img src="hardware/schematics/3_5_esp32_mcu_iot_schematic.png" width="100%" alt="3.5 ESP32 MCU and ESP-NOW link"><br><sub>3.5 ESP32 MCU and ESP-NOW link</sub></td><td align="center" width="50%"><img src="hardware/schematics/3_6_alarm_indicator_schematic.png" width="100%" alt="3.6 Alarm indicator (LEDs and buzzer)"><br><sub>3.6 Alarm indicator (LEDs and buzzer)</sub></td></tr>
</table>

### Sensing PCB (ESP32-S3, MPU-6050, piezo with LM358 conditioning)

<table>
<tr><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260830_110630.png" width="100%" alt="Full schematic"><br><sub>Full schematic</sub></td><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260828_211242.png" width="100%" alt="Piezo and LM358 conditioning stage"><br><sub>Piezo and LM358 conditioning stage</sub></td></tr>
<tr><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260830_110723.png" width="100%" alt="Top-layer routing"><br><sub>Top-layer routing</sub></td><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260830_110731.png" width="100%" alt="Bottom-layer routing"><br><sub>Bottom-layer routing</sub></td></tr>
<tr><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260830_110428.png" width="100%" alt="Component placement view"><br><sub>Component placement view</sub></td><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260830_110742.png" width="100%" alt="3D view"><br><sub>3D view</sub></td></tr>
</table>

### Alarm/Gateway PCB (ESP32 DevKitC, LEDs, buzzer)

<table>
<tr><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260816_232631.png" width="100%" alt="Schematic"><br><sub>Schematic</sub></td><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260816_232711.png" width="100%" alt="Top-layer routing"><br><sub>Top-layer routing</sub></td></tr>
<tr><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260816_233050.png" width="100%" alt="2D board preview"><br><sub>2D board preview</sub></td><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260816_232737.png" width="100%" alt="3D view"><br><sub>3D view</sub></td></tr>
</table>

### Proteus simulation of the FFT pipeline (Arduino Mega 2560 stand-in)

<table>
<tr><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260830_111141.png" width="100%" alt="Simulated circuit"><br><sub>Simulated circuit</sub></td><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260830_111511.png" width="100%" alt="Circuit with scope during a seismic-like burst"><br><sub>Circuit with scope during a seismic-like burst</sub></td></tr>
<tr><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260830_111043.png" width="100%" alt="Scope trace, quiet input"><br><sub>Scope trace, quiet input</sub></td><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260830_111323.png" width="100%" alt="Scope trace, single impulse"><br><sub>Scope trace, single impulse</sub></td></tr>
<tr><td align="center" width="50%"><img src="hardware/capture-screenshots/Screenshot_20260830_111420.png" width="100%" alt="Scope trace, seismic-like burst"><br><sub>Scope trace, seismic-like burst</sub></td><td align="center" width="50%"><img src="docs/figures/FFT_Detection_Pipeline_Signal_Comparison.png" width="100%" alt="FFT detection pipeline, signal comparison (MATLAB)"><br><sub>FFT detection pipeline, signal comparison (MATLAB)</sub></td></tr>
</table>

<!-- gallery:end -->

## Repository layout

```
docs/                        Full project report, original proposal, and the
                              two earlier calculation documents (see note
                              below on those two).
matlab/                      MATLAB FFT validation script (mirrors firmware).
firmware/
  deployed_rms_alarm/        What is actually running on the breadboard today.
  fft_pipeline_proteus_sim/  FFT + consensus pipeline, validated in Proteus,
                              not yet merged into the deployed sketch.
hardware/
  schematics/                Per-subsystem schematic exports (3.1, 3.2, 3.5, 3.6).
  pcb-photos/                Etched, drilled, unpopulated boards.
  capture-screenshots/       Raw dated Altium/Proteus screenshots used to
                              build the figures in the report.
```

### A note on `docs/Power_System_Analysis_v2_SUPERSEDED.docx` and `docs/Design_Justification_Calculations_PARTIALLY_SUPERSEDED.docx`

These two documents were written against an earlier assumption that the 3.3 V
rail comes from an AP2112K linear LDO and that the status indicator is a
single RGB LED package. The report explicitly corrects both: the real
Altium files use an **LM2596 buck module** on both boards, and the indicator
is **three separate through-hole LEDs**. Their current-budget and polyfuse
numbers are built around the LDO assumption and should not be used as the
current power-budget reference; the LM2596 headroom analysis in Section 3.1
of the report is current. The VREF divider math, Sallen-Key filter sizing,
and IPC-2221 trace-width calculation in these two documents are still valid
and are reused in the final report, since those parts do not depend on which
regulator is in the power stage.

## Firmware

Two Arduino sketches, both requiring the Arduino IDE (or arduino-cli) with
the ESP32 board package installed.

**`firmware/deployed_rms_alarm/EEE383_RMS_Alarm.ino`**
What actually runs today. Requires the `Adafruit_MPU6050` and
`Adafruit_Sensor` libraries. Reads the piezo on GPIO34 (an ADC1 pin, chosen
so it coexists with Wi-Fi/ESP-NOW) and the MPU-6050 over I2C, self-calibrates
a rest baseline and a shake threshold on boot or on serial command `r`,
requires piezo and accelerometer to trip within a 500 ms consensus window,
and drives a common-cathode RGB LED plus a PWM buzzer. Serial command `c`
clears a latched alarm.

**`firmware/fft_pipeline_proteus_sim/EEE383_FFT_Pipeline_ProteusSim.ino`**
The FFT-based design, as validated on an Arduino Mega 2560 in Proteus
(the ESP32 was not reliably modeled in Proteus, so the Mega stood in for
bring-up). Requires the `arduinoFFT` v2 library and reads the MPU-6050 over
raw `Wire` register access rather than a library, since this sketch targets
the Proteus stand-in wiring, not the deployed hardware. 256-sample buffers
at 100 Hz, Hamming window, band energy summed over bins 3 to 25 (1.17 to
9.77 Hz).

## MATLAB

**`matlab/EEE383_FFT_Validation.m`**
Mirrors the firmware's FFT parameters exactly (100 Hz, 256-sample buffer,
Hamming window, bins 3 to 25) and runs them against five test signals: a
real ObsPy-retrieved earthquake recording, a synthetic 10 Hz
construction/pile-driving tone, a decaying 20 Hz door slam, a synthetic
15.18 Hz train signal, and an out-of-band 53 Hz body-massager tone. The
script expects a file named `quake_signal.txt` (850 samples, 8.5 s, from an
ObsPy-retrieved BW.RJOB seismogram) in the same folder; that file is not
included in this repo and needs to be re-exported from ObsPy or supplied by
the team. Without it, the script falls back to a placeholder synthetic burst.
Output figure: `docs/figures/FFT_Detection_Pipeline_Signal_Comparison.png`.

## References

Full IEEE-style reference list is in the report
(`docs/EEE383_Group01_Project_Report.pdf`, final page). Key sources used
across the design justification: Espressif ESP32-S3 datasheet, InvenSense
MPU-6050 datasheet, TI LM2596 datasheet, Diodes AP2112 datasheet (historical,
see note above), IPC-2221, and a 2025 Scientific Reports paper on
high-speed-rail ground vibration used for the synthetic train test signal.

## Contributors

| Name | Primary subsystem |
| --- | --- |
| Jarin | 3.6 |
| Salman | 3.3 |
| Tarteela | 3.1 |
| Ishraq | 3.1 |
| Raihana | 3.5 |
| Bitto | 3.4 |
| Intisar E. | 3.5 |
| Intisar A. | 3.4 |
| Tasin | 3.1 |
| Rehnuma | Project Manager |
| Fabiha | 3.2 |

## License

Released under the MIT License. See [`LICENSE`](LICENSE).
