# ROC7

Onboarded 2026-11-07 on Betaflight 4.5.3, upgraded the same day to 2026.6.2.

## Hardware

| Part | Detail |
|---|---|
| Flight controller | HAKRC F722 V2. Betaflight target `HAKRCF722V2`, STM32F722 |
| Sensors detected | Gyro, accelerometer, barometer, GPS |
| Compass | Supported by the build, not enabled (`mag_hardware = NONE`) |
| Receiver | ExpressLRS **900 MHz**, CRSF on UART2. Radio model ID 1 on the TX15 internal module |
| GPS | UART6, `gps_auto_baud = ON` |
| Video | Digital VTX on UART4 (MSP DisplayPort) |
| ESCs | DShot300 bidirectional, ESC telemetry on UART5, props out (`yaw_motors_reversed = ON`), idle 4.5% (`motor_idle = 450`) |
| Board alignment | `align_board_yaw = 90` |
| LEDs | 4-LED strip (`led 0-3`) |

No camera tilt servo.

## Serial ports

| UART | Use | `serial` line |
|---|---|---|
| UART2 | Serial RX (CRSF) | `serial UART2 64 ...` |
| UART4 | MSP + VTX (MSP DisplayPort) | `serial UART4 131073 ...` |
| UART5 | ESC sensor (telemetry) | `serial UART5 1024 ...` |
| UART6 | GPS | `serial UART6 2 ...` |

## Firmware

Betaflight **2026.6.2**, cloud build:

| Option | Value |
|---|---|
| Radio protocol | CRSF |
| Telemetry protocol | Automatically included |
| OSD protocol | Digital |
| Motor protocol | DSHOT |
| Other options | Acro Trainer, Altitude Hold, GPS, LED Strip, Magnetometers, Pin IO, Position Hold, VTX, Chirp |

## Modes

Standard layout shared with every quad, see [the root README](../README.md#standard-switch-layout).

| Mode | Channel | Range | Radio switch (TX15) |
|---|---|---|---|
| ARM | AUX1 (ch5) | 1700-2100 | SE down |
| ALT HOLD + POS HOLD | AUX3 (ch7) | 1300-1700 | SB middle |
| GPS RESCUE | AUX3 (ch7) | 1700-2100 | SB down |
| BEEPER | AUX4 (ch8) | 1700-2100 | SC down |
| HORIZON | AUX5 (ch9) | 1300-1700 | SD middle |
| ANGLE | AUX5 (ch9) | 1700-2100 | SD down (SD up = Acro) |
| FLIP OVER AFTER CRASH (turtle) | AUX6 (ch10) | 1700-2100 | SF, hold |

Changed 2026-11-07 to the standard layout: PREARM removed (SE alone arms), ARM and BEEPER
ranges 1800 to 1700, **ANGLE moved from SD up to SD down**, HORIZON and ALT/POS HOLD added.
Position Hold has no compass here: it shows `POS HOLD FAIL` until GPS has worked out the
heading from a stretch of straight forward flight.

## Radio (EdgeTX)

RadioMaster TX15, model `ROC7` = `MODELS/model3.yml` on the SD card, copy in
[`radio-model.yml`](radio-model.yml). Cloned 2026-11-07 from the STELLAR model: same channel map
and low-battery alerts (L1 < 3.5 V/cell, L2 < 3.3 V/cell, LiPo; quiet with no pack via L3); see
[stellar25/README.md](../stellar25/README.md#radio-edgetx). Model ID 1 (900 MHz).

Long-range screens:

| Page | Layout | Left | Right |
|---|---|---|---|
| 1 (main) | 2x4 | RxBt (cell V), Capa (mAh), Curr (A), RQly | Dist, Alt, GSpd, FM |
| 2 | 1x1 | Outputs | |
| 3 | 2x3 | 1RSS (dBm), RSNR (dB), TPWR (mW) | Sats, GPS (position), RFMD (packet rate) |

`Dist` is a calculated sensor (index 23, `FORMULA_DIST`, GPS source sensor 0, no altitude):
ground distance from the first GPS fix after the radio powers up or telemetry is reset, so get
a fix at the take-off spot. On `Layout2x3` the right column is zones 5 (top), 3 (middle),
4 (bottom).

Restore: radio in USB Storage mode, copy `radio-model.yml` to `MODELS/model3.yml`, eject.

## Failsafe and GPS Rescue

```
set failsafe_procedure = GPS-RESCUE
set failsafe_switch_mode = STAGE2
rxfail 3 a          # throttle: auto (the default on 2026.6, so not in the diff)
rxfail 6 s 2000     # AUX3 (GPS Rescue) high on signal loss
rxfail 8 s 2000     # AUX5 high on signal loss = Angle on
set gps_rescue_return_alt = 50
set gps_rescue_min_start_dist = 30
set gps_rescue_ground_speed = 1000
set gps_rescue_ascend_rate = 650
set gps_rescue_descend_rate = 100
set ap_max_angle = 35          # was gps_rescue_max_angle
set ap_throttle_max = 1600     # was gps_rescue_throttle_max
set gps_rescue_allow_arming_without_fix = ON
```

On 2026.6 `ap_max_angle` and `ap_throttle_max` are shared by every autopilot mode (GPS Rescue,
Position Hold). `gps_rescue_allow_arming_without_fix = ON` lets it take off before a home point
is set; Rescue then has nowhere to return to.

## Telemetry

`feature TELEMETRY` on, `beacon RX_LOST` and `beacon RX_SET` on, `report_cell_voltage = ON`
(`RxBt` on the radio is the average cell voltage, which the shared radio alerts expect).

## Tune

Custom PIDs (`simplified_pids_mode = OFF`, master multiplier 165). Rates: RC rate 6, super rate
62, expo 20 on all axes.

## Migration: Betaflight 4.5.3 to 2026.6.2 (done 2026-11-07)

The 4.5.3 config is in [`archive/2026-11-07_bf-4.5.3_original.txt`](archive/2026-11-07_bf-4.5.3_original.txt).

| 4.5.3 | 2026.6.2 | Why |
|---|---|---|
| `d_roll = 76`, `d_min_roll = 69` | `d_roll = 69`, `d_max_roll = 76` | D-min became D-max, values swap |
| `d_pitch = 108`, `d_min_pitch = 98` | `d_pitch = 98`, `d_max_pitch = 108` | same |
| `simplified_dmax_gain = 30` | `simplified_d_max_gain = 30` | renamed |
| `dshot_idle_value = 450` | `motor_idle = 450` | renamed, same 0.01% units |
| `gps_rescue_max_angle = 35` | `ap_max_angle = 35` | moved to shared autopilot settings |
| `gps_rescue_throttle_max = 1600` | `ap_throttle_max = 1600` | same |
| `debug_mode = GYRO_SCALED` | (dropped) | mode no longer exists; blackbox debug only |
| `feature SERVO_TILT`, `feature CHANNEL_FORWARDING` | (dropped) | no servo fitted, Servos not in the build |
| `vbat_max_cell_voltage = 435` | same, under `battery_profile 0` | battery profiles added |
| `serial 1 64 ...` | `serial UART2 64 ...` | ports listed by name |

No custom gyro alignment on this board, so no alignment change was needed.
