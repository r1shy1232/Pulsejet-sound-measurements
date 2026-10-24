# Pulsejet Sound Meter

A browser-based sound level and frequency meter for measuring pulsejet noise from a phone. It runs entirely in the browser from a single file (`index.html`). Audio is processed on the device and never uploaded.

## Running it

1. Host `index.html` on GitHub Pages (microphone access requires an `https://` page; a local file will not get permission).
2. Open the link in a phone browser (Safari on iPhone), tap **Start** and allow the microphone.

## What the display shows

| Part | Meaning |
|---|---|
| **Status chips** | Whether the browser's echo cancellation, noise suppression and auto gain are off. All three should read "off"; "ON" means the phone is altering levels and readings are unreliable. "Unverified" means the browser did not report the setting. Also shown: sample rate and FFT resolution. |
| **Big number** | Current level, updated every 125 ms (Fast time weighting). |
| **Weighting buttons** | dB(A) approximates human hearing (standard for noise); dB(C) keeps more low-frequency energy; Z is flat (unweighted). |
| **Slow** | Level averaged over about 1 s. |
| **Leq** | Average level since the last reset (energy average, not an average of dB values). |
| **Max** | Highest Fast level since reset. |
| **Peak** | Highest instantaneous C-weighted sample. Always above the averages because it catches brief spikes. |
| **Frequency** | Strongest tone found between 30 Hz and 8 kHz, interpolated to 0.1 Hz, plus up to three further peaks (dB shown relative to the strongest). "No clear tone" appears if nothing is 15 dB above the spectrum average. |
| **Predicted fundamental** | Optional input. If filled in, the app shows the difference between measured and predicted frequency in Hz and %. |
| **Spectrum** | Level against frequency (20 Hz to 10 kHz, log axis, relative dB). Dashed lines mark the dominant frequency and its 2x and 3x multiples; a red dashed line marks the predicted value. |
| **Spectrogram** | Frequency (30 Hz to 4 kHz) against time (about 20 s); brighter is louder. Shows start-up, steady running and pitch drift. |
| **Level history** | Level for the last 2 minutes in the selected weighting. |
| **1/3-octave bands** | Z-weighted level in each standard 1/3-octave band, in the same units as the main level. The lowest bands contain only a few FFT bins, so treat them as approximate. |
| **Red banner** | The microphone is clipping. Levels are invalid until it clears. |

## Why the level is negative

Until calibrated, values are in **dBFS**: decibels relative to digital full scale, where 0 dBFS is the loudest signal the phone can record. Everything quieter is negative (-60 dBFS is 60 dB below the maximum). This is a relative number, not a real sound pressure level. Calibration converts it to **dB SPL**.

dB is logarithmic: +10 dB is ten times the sound power; +3 dB is about double.

## How to use it

1. Open the page, tap **Start**, allow the microphone, and check the chips.
2. **Check frequency accuracy:** play a known tone (e.g. 440 Hz) from another device and confirm the reported frequency.
3. **Calibrate** (optional but needed for dB SPL, see below).
4. **Set up the test:** wear hearing protection, keep the phone out of the exhaust path and heat, use a foam windscreen outdoors, and fix the mic position. Enter the distance in the distance box.
5. Tap **Reset stats and log**, start the engine and tap **Ignition**. Tap **Steady** and **Shutdown** at those moments. Markers are written into the log.
6. If the red banner appears, move the phone further away and repeat the run.
7. Tap **Stop**, then **Download CSV**.

Keep the position, distance and orientation the same between runs so results are comparable. Sound level falls with distance, so always record it.

## Calibrating

1. Play steady noise (or a 1 kHz tone) close to a trusted reference meter or app, with the phone beside it.
2. Select the same weighting in the app as the reference meter uses (usually dB(A)).
3. Type the reference reading into **Reference reading** and tap **Calibrate to reference**.

The offset is applied to every reading and saved in the browser. **Clear calibration** returns to dBFS. The offset can also be typed in directly.

Calibration is only valid near the level and spectrum used to set it. A 94 dB, 1 kHz acoustic calibrator is the better reference if one is available. Quote what you calibrated against, and that the uncertainty was not characterised, when reporting results.

## Reading the CSV

Rows are logged four times per second while the meter runs. The header lines (starting with `#`) record sample rate, FFT resolution, calibration offset, browser audio-processing status, mic distance and your note.

| Column | Meaning |
|---|---|
| `time_s` | Seconds since start or last reset |
| `LAF`, `LCF`, `LZF` | Fast level, A-, C- and Z-weighted |
| `LAS` | Slow level, A-weighted |
| `LAeq` | A-weighted Leq since reset |
| `LCpeak` | Highest C-weighted sample since reset |
| `dominant_Hz` | Strongest tone (blank if none) |
| `clip` | 1 if clipping occurred in the last 4 s |
| `marker` | Event label pressed since the previous row |

Rows with `clip = 1` should be excluded from level analysis. To align with Arduino data, line up the start-up transient (the `Ignition` marker and the jump in level) with the jump in vibration RMS, since the phone and Arduino clocks are independent.

## Limitations

- Phone microphones clip at high sound levels. A pulsejet at close range will likely exceed this, so measure at a distance.
- Phone microphones roll off at low frequency and are not flat. Check the low-frequency response against a known tone before trusting readings near your fundamental.
- Frequency resolution is about 2.9 Hz at 48 kHz (16384-point FFT, about 0.34 s window), which trades off against time resolution.
- iOS Safari may not honour every audio-processing setting; always check the chips.
- Level accuracy depends on calibration and is not guaranteed.
- This is a measurement aid, not a certified sound level meter.

## Safety

Pulsejets are extremely loud. Wear hearing protection, and keep the phone clear of the exhaust and heat.
