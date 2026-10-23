# Hardware

## schematics/

Per-subsystem schematic exports, matching the crops shown in Section 3 of
the report:

- `3_1_power_regulation_schematic.png`: TP4056 to LM2596 to 3V3 rail chain.
- `3_2_dual_axis_seismic_sensing.svg`: MPU-6050 breakout and piezo module
  wiring, with notes on module regulation, I2C addressing, and why the
  piezo digital output (DO) is left unconnected.
- `3_5_esp32_mcu_iot_schematic.png`: ESP32 DevKitC GPIO assignment
  (alarm/gateway board).
- `3_6_alarm_indicator_schematic.png`: three discrete LEDs and the buzzer,
  direct GPIO drive, no transistor stage.

The Sallen-Key filter and VREF bias network schematic (Subsystem 3.3) and
the full-board sheets (Figures 9.1 and 9.2) are in the report itself; they
are not exported as standalone files in this repo.

## pcb-photos/

Both boards after home etching and drilling, before component population.
Two angles of the same pair of boards (larger: Sensing PCB, smaller:
Alarm/Gateway PCB).

## capture-screenshots/

Raw, dated Altium and Proteus screenshots taken during the build, used as
the source material for several figures in the report (the Section 9
full-board schematics, the Section 3.3 Proteus filter validation traces,
and the Section 10 PCB layout figures). Kept unedited and by original
filename/timestamp for traceability back to when each capture was taken.
