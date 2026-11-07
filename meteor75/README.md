# Meteor75 Pro

BetaFPV Meteor75 Pro, 1S whoop. Onboarded 2026-11-08 on Betaflight 4.5.1 (BetaFPV factory
build, Dec 2025), upgraded the same day to 2026.6.2.

## Hardware

| Part | Detail |
|---|---|
| Flight controller | BetaFPV G473. Betaflight target `BETAFPVG473` (the original, **not** `_V2` or `_V3`, which are wired differently), STM32G474 |
| Gyro | BMI270. The target fixes it at `CW180`; this quad needs `CW90` overall, so `align_board_yaw = 270` |
| Barometer | None. Detection on a fresh 2026.6.2 found nothing on I2C, so `baro_hardware = NONE` |
| GPS, compass | None |
| Blackbox | 16 MB onboard flash (`blackbox_device = SPIFLASH`) |
| Receiver | ExpressLRS **2.4 GHz**, CRSF on UART3 (target default). Radio model ID 2 on the TX15 internal module |
| Video | Digital VTX on UART4 (MSP DisplayPort, HD OSD 53 x 20) |
| ESCs | DShot300 bidirectional, 12-pole motors, idle 5% (`motor_idle = 500`) |
| Battery | 1S LiHV (`vbat_max_cell_voltage = 440`, warning 3.45 V), current sensor on (`ibata_scale = 550`) |

## Serial ports

| UART | Use | `serial` line |
|---|---|---|
| UART2 | Nothing (2026.6.2 defaults it to SmartAudio, cleared) | `serial UART2 0 ...` |
| UART3 | Serial RX (CRSF) | target default, not in the diff |
| UART4 | MSP + VTX (MSP DisplayPort) | `serial UART4 131073 ...` |

## Firmware

Betaflight **2026.6.2**, cloud build:

| Option | Value |
|---|---|
| Radio protocol | CRSF |
| Telemetry protocol | Automatically included |
| OSD protocol | Digital |
| Motor protocol | DSHOT |
| Other options | Acro Trainer, Altitude Hold, Chirp, LED Strip, VTX |

Altitude Hold has no barometer to work with on this quad; it is only in the build to match the
others. Chirp is needed for autotune.

**Flashing needs the BOOT button.** The software reboot into DFU (the App's, or CLI `bl`) makes
the board drop off USB without a DFU device appearing. Unplug, hold BOOT, plug USB in, release:
the board shows up as `DFU in FS Mode`, then flash from the App.

## Modes

Standard layout shared with every quad, see [the root README](../README.md#standard-switch-layout).
No GPS and no barometer, so GPS RESCUE, ALT HOLD and POS HOLD (`aux 2`, `aux 6`, `aux 7`) are left
out and SB does nothing on this quad.

| Mode | Channel | Range | Radio switch (TX15) |
|---|---|---|---|
| ARM | AUX1 (ch5) | 1700-2100 | SE down |
| BEEPER | AUX4 (ch8) | 1700-2100 | SC down |
| HORIZON | AUX5 (ch9) | 1300-1700 | SD middle |
| ANGLE | AUX5 (ch9) | 1700-2100 | SD down (SD up = Acro) |
| FLIP OVER AFTER CRASH (turtle) | AUX6 (ch10) | 1700-2100 | SF, hold |
| CHIRP + BLACKBOX (autotune test only) | AUX2 (ch6) | 1700-2100 | SA down |

Changed 2026-11-08 to the standard layout: **ANGLE moved from SD up to SD down**. Slots
`aux 0, 1, 3, 4, 5` are the standard block, `aux 8-9` the chirp test.

Failsafe: Betaflight default (drop), no GPS Rescue.

## Radio (EdgeTX)

RadioMaster TX15, model `METEOR75` = `MODELS/model4.yml` on the SD card, copy in
[`radio-model.yml`](radio-model.yml). Cloned 2026-11-08 from the STELLAR model: same channel map
and low-battery alerts (L1 < 3.5 V/cell for 3 s, L2 < 3.3 V/cell for 2 s); see
[stellar25/README.md](../stellar25/README.md#radio-edgetx). ELRS model ID 2 (2.4 GHz). The SB down
`gps` voice call-out is removed, because SB does nothing here.

Main screen (`Layout2x4`, Value widgets):

| Left | Right |
|---|---|
| RxBt (average cell voltage) | FM (flight mode) |
| Capa (mAh used) | 1RSS (dBm) |
| Curr (A) | TPWR (mW) |
| RQly (link quality) | RFMD (packet rate) |

Restore: radio in USB Storage mode, copy `radio-model.yml` to `MODELS/model4.yml`, eject.

## Telemetry

`feature TELEMETRY` (set explicitly: the Chirp build has it off by default, and without it the
radio reads 0 V and the low-battery alert fires), `beacon RX_LOST` and `beacon RX_SET` on,
`report_cell_voltage = ON` (`RxBt` on the radio is the average cell voltage, which the shared
radio alerts expect).

## Autotune (chirp)

Same test setup as STELLAR, currently in `diff-all.txt`:

```
set debug_mode = CHIRP
set blackbox_high_resolution = ON
aux 8 55 1 1700 2100 0 0   # CHIRP mode on AUX2 (SA down)
aux 9 26 1 1700 2100 0 0   # BLACKBOX on AUX2: log only while SA is down (16 MB flash)
```

The blackbox flash was erased 2026-11-08 (it was full of logs from the 4.5.1 firmware). Flight
procedure and the revert afterwards: [stellar25/README.md](../stellar25/README.md#autotune-chirp).

## Tune

BetaFPV's tune, PID profile `GF 40mm`, sliders on (master 120). Rates: RC rate 5, super rate
72 / 72 / 54, expo 45 on all axes, `thr_mid = 45`, `thr_expo = 50`.

## Migration: Betaflight 4.5.1 to 2026.6.2 (done 2026-11-08)

The 4.5.1 config is in [`archive/2026-11-08_bf-4.5.1_original.txt`](archive/2026-11-08_bf-4.5.1_original.txt).

| 4.5.1 | 2026.6.2 | Why |
|---|---|---|
| `gyro_1_sensor_align = CW90`, `gyro_1_align_yaw = 900` | `align_board_yaw = 270` | User gyro alignment was removed; the target fixes the gyro at CW180, and CW180 + 270 = CW90 |
| `d_roll = 48`, `d_min_roll = 43` | `d_roll = 43`, `d_max_roll = 48` | D-min became D-max, values swap |
| `d_pitch = 67`, `d_min_pitch = 58` | `d_pitch = 58`, `d_max_pitch = 67` | same |
| `simplified_dmax_gain = 40` | `simplified_d_max_gain = 40` | renamed |
| `dshot_idle_value = 500` | `motor_idle = 500` | renamed, same units |
| `debug_mode = GYRO_SCALED` | (dropped, now `CHIRP` for the test) | mode no longer exists |
| `serial 3 131073 ...` | `serial UART4 131073 ...`, plus `serial UART2 0 ...` | ports listed by name; UART2 now defaults to SmartAudio |
| `vbat_max_cell_voltage = 440`, `vbat_warning_cell_voltage = 345` | same, under `battery_profile 0` | battery profiles added |
| `aux 1 1 4 900 1300 0 0` (ANGLE on SD up) | `aux 1 1 4 1700 2100 0 0` | standard layout |

OSD element positions came across unchanged.
