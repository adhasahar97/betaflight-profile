# ROC7

Onboarded 2026-11-07 as found. Nothing on the quad was changed.

## Hardware

| Part | Detail |
|---|---|
| Flight controller | HAKRC F722 V2. Betaflight target `HAKRCF722V2`, STM32F722 |
| Sensors detected | Gyro, accelerometer, barometer, GPS |
| Compass | Supported by the build, not enabled (`mag_hardware = NONE`) |
| Receiver | ExpressLRS, CRSF on UART2 |
| GPS | UART6, `gps_auto_baud = ON` |
| Video | Digital VTX on UART4 (MSP DisplayPort) |
| ESCs | DShot300 bidirectional, ESC telemetry on UART5, props out (`yaw_motors_reversed = ON`), idle 4.5% |
| Board alignment | `align_board_yaw = 90` |
| LEDs | 4-LED strip (`led 0-3`) |

Features also enabled: `SERVO_TILT`, `CHANNEL_FORWARDING`.

## Serial ports

| UART | Use | `serial` line |
|---|---|---|
| UART2 | Serial RX (CRSF) | `serial 1 64 ...` |
| UART4 | MSP + VTX (MSP DisplayPort) | `serial 3 131073 ...` |
| UART5 | ESC sensor (telemetry) | `serial 4 1024 ...` |
| UART6 | GPS | `serial 5 2 ...` |

## Firmware

Betaflight **4.5.3** (built 2025-11-23). Build options: CRSF, GPS, Magnetometer, LED Strip,
OSD (SD and HD), Pin IO, Servos, VTX, DShot.

## Modes

| Mode | Channel | Range |
|---|---|---|
| ARM | AUX1 (ch5) | 1800-2100 |
| PREARM | AUX2 (ch6) | 1700-2100 |
| GPS RESCUE | AUX3 (ch7) | 1700-2100 |
| BEEPER | AUX4 (ch8) | 1800-2100 |
| ANGLE | AUX5 (ch9) | 900-1200 (low) |
| FLIP OVER AFTER CRASH (turtle) | AUX6 (ch10) | 1700-2100 |

Arming needs PREARM (AUX2 high) first.

## Failsafe and GPS Rescue

```
set failsafe_procedure = GPS-RESCUE
set failsafe_switch_mode = STAGE2
rxfail 3 a          # throttle: auto
rxfail 6 s 2000     # AUX3 (GPS Rescue) high on signal loss
rxfail 8 s 2000     # AUX5 high on signal loss (Angle is 900-1200, so Angle off)
set gps_rescue_return_alt = 50
set gps_rescue_min_start_dist = 30
set gps_rescue_ground_speed = 1000
set gps_rescue_ascend_rate = 650
set gps_rescue_descend_rate = 100
set gps_rescue_max_angle = 35
set gps_rescue_throttle_max = 1600
set gps_rescue_allow_arming_without_fix = ON
```

`gps_rescue_allow_arming_without_fix = ON` lets it take off before a home point is set. Rescue
then has nowhere to return to.

## Telemetry

`feature TELEMETRY` on, `beacon RX_LOST` and `beacon RX_SET` on. `report_cell_voltage` is not set,
so `RxBt` on the radio is pack voltage.

## Tune

Custom PIDs (`simplified_pids_mode = OFF`, master multiplier 165). Rates: RC rate 6, super rate
62, expo 20 on all axes.

## If upgrading to Betaflight 2025.12+

Same traps as STELLAR25 (see [stellar25/README.md](../stellar25/README.md#migration-notes-betaflight-45-to-20266)),
minus the gyro alignment one (no custom sensor alignment here, only `align_board_yaw = 90`):

| 4.5.3 | 2025.12+ |
|---|---|
| `d_roll = 76`, `d_min_roll = 69` | `d_roll = 69`, `d_max_roll = 76` |
| `d_pitch = 108`, `d_min_pitch = 98` | `d_pitch = 98`, `d_max_pitch = 108` |
| `simplified_dmax_gain = 30` | `simplified_d_max_gain = 30` |
| `debug_mode = GYRO_SCALED` | (dropped) |
