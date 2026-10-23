# Firmware

Two sketches, covering the two states described in the main README: what is
deployed today, and the FFT design that has been validated separately but
not yet merged in.

## deployed_rms_alarm/EEE383_RMS_Alarm.ino

Target: ESP32-S3 (breadboard demo, single board running sensing, filtering,
consensus, and alarm together).

Libraries (install via Arduino Library Manager):
- `Adafruit MPU6050`
- `Adafruit Unified Sensor`

Pin map:

| Signal | Pin | Note |
| --- | --- | --- |
| Piezo analog in | GPIO34 | ADC1 channel, chosen to coexist with Wi-Fi/ESP-NOW |
| MPU-6050 | I2C (default SDA/SCL) | via Adafruit_MPU6050 library |
| LED Red | GPIO25 | common-cathode, active-HIGH on breadboard |
| LED Green | GPIO26 | |
| LED Blue | GPIO27 | |
| Buzzer | GPIO14 | PWM via `ledcAttach`, 3 kHz |

Behavior: on boot, holds still for 2 seconds to record a rest baseline for
both sensors, then asks for 5 seconds of deliberate shaking to record a peak
and set a trigger threshold at 40% of the way from rest to peak. Detection
uses a 20-sample RMS window per sensor; alarm requires both piezo and accel
to have tripped within the last 500 ms (consensus window). Alarm latches
until serial command `c`; serial command `r` re-runs calibration.

A boot-time note logs that VREF was measured at 0.22 V against a 0.889 V
design target (see the main README's known open items); the firmware flags
ADC clipping against the 0V floor as a percentage but does not correct the
bias in software.

## fft_pipeline_proteus_sim/EEE383_FFT_Pipeline_ProteusSim.ino

Target: Arduino Mega 2560, standing in for the ESP32 in Proteus (the ESP32
was not reliably modeled in Proteus at the time of this project).

Library: `arduinoFFT` v2 (`ArduinoFFT<float>` template, `FFTWindow::Hamming`,
`FFTDirection::Forward`).

This sketch reads the MPU-6050 via raw `Wire` register access rather than
the Adafruit library, since it targets the Proteus wiring rather than the
deployed board, and implements the full 256-sample, 100 Hz, Hamming-windowed
FFT with band energy summed over bins 3 to 25. A three-stage alarm state
machine (STANDBY, STAGE1_LOCAL, STAGE2_CONFIRMED) is included, with the
Stage 2 gateway confirmation currently simulated with a timer
(`DEBUG_SIMULATE_GATEWAY_CONFIRM_AFTER_MS`) rather than a real ESP-NOW link.

This is the design target. It is not what runs on the deployed breadboard
today; see the main README for why.
