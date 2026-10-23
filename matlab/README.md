# MATLAB

`EEE383_FFT_Validation.m` mirrors the firmware's FFT parameters (100 Hz
sample rate, 256-sample buffer, Hamming window, band energy summed over
bins 3 to 25, corresponding to 1.17 to 9.77 Hz) and runs them against five
signals to validate the frequency-domain discrimination approach:

| Signal | Source | Band energy (bins 3-25) | Rejected by |
| --- | --- | --- | --- |
| Real earthquake (ObsPy BW.RJOB) | `quake_signal.txt` | 4.296 | Passes, correctly |
| Construction / pile driving, 10 Hz | synthetic, Eq. 8 in the report | 29.64 | Dual-sensor agreement (frequency content alone cannot reject it) |
| Door slam, 20 Hz | synthetic, Eq. 10 | 0.0091 | Frequency-domain filter |
| Train vibration, 15.18 Hz | synthetic, Eq. 11 | 0.0000003 | Frequency-domain filter |
| Body massager, 53 Hz | synthetic sanity check | 0.0000567 | Frequency-domain filter (easy case) |

The construction case is the important result: its band energy is higher
than the real earthquake's, because a clean 10 Hz tone concentrates all its
energy right at the edge of the passband. That is why this system does not
rely on FFT band energy alone and requires the second sensor's independent
vote (see Section 3.4 of the report and `engineering-learnings` in project
memory for the reasoning).

## Running it

The script expects `quake_signal.txt` (850 samples, 8.5 s, tab or
comma-separated time/amplitude columns) in the same folder. That file is
not included in this repo; re-export it from ObsPy against the BW.RJOB
station, or substitute another seismogram with the same sample count. If
the file is missing or fails to parse, the script falls back to a
placeholder Gaussian-windowed burst so the rest of the pipeline can still be
exercised.

Output: a comparison figure (time domain + FFT magnitude per signal) and a
printed band-energy summary table. A copy of this figure, as generated
during the actual project run, is at
`../docs/figures/FFT_Detection_Pipeline_Signal_Comparison.png`.
