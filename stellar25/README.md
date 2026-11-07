# STELLAR25

## Hardware

| Part | Detail |
|---|---|
| Flight controller | JHEMCU GHF405AIO-HD, AIO with 4-in-1 ESC. Betaflight target `JHEF405PRO`, STM32F405, ICM42688P gyro |
| Mounting | FC mounted upside down (`align_board_roll = 180`) |
| Receiver | ExpressLRS, CRSF protocol |
| GPS | BZGNSS BZ-121 (u-blox), `gps_auto_config = ON` |
| Video | Digital VTX on the HD connector, MSP DisplayPort OSD |
| Motors | 12 poles, DShot300 bidirectional (RPM filter) |

Pinout: [ArduPilot JHEMCU F405PRO page](https://ardupilot.org/copter/docs/common-jhemcuf405pro.html).

## Serial ports

| UART | Pads | Use | `serial` line |
|---|---|---|---|
| UART1 | RX1 / TX1 | GPS | `serial UART1 2 ...` |
| UART2 | RX2 / TX2 | unused (nothing wired) | |
| UART3 | RX3 / TX3 | unused | |
| UART4 | RX4 / TX4 | ELRS receiver (Serial RX, CRSF) | `serial UART4 64 ...` |
| UART6 | HD VTX connector | MSP + VTX (MSP DisplayPort) | `serial UART6 131073 ...` |

The HD VTX connector also carries an SBUS line. It is not used (control is ELRS on UART4).

## Firmware

Betaflight **2026.6.2**, cloud build:

| Option | Value |
|---|---|
| Radio protocol | CRSF |
| Telemetry protocol | Automatically included |
| OSD protocol | Digital |
| Motor protocol | DSHOT |
| Other options | Acro Trainer, GPS, LED Strip, Pin IO, VTX |
| Race Pro | off |

The build also pulls in Altitude Hold and Position Hold with GPS. No magnetometer support
(none fitted).

## Modes

| Mode | Channel | Range | Radio switch (TX15) |
|---|---|---|---|
| ARM | AUX1 (ch5) | 1700-2100 | SE down |
| ANGLE | AUX5 (ch9) | 1700-2100 | SD down |
| HORIZON | AUX5 (ch9) | 1300-1700 | SD middle (SD up = Acro) |
| GPS RESCUE | AUX3 (ch7) | 1700-2100 | SB down |
| BEEPER | AUX4 (ch8) | 1700-2100 | SC down |
| FLIP OVER AFTER CRASH (turtle) | AUX6 (ch10) | 1700-2100 | SF, hold (momentary) |

Failsafe procedure: GPS Rescue (`gps_rescue_allow_arming_without_fix = ON`).

## Radio (EdgeTX)

RadioMaster TX15, EdgeTX 3.0.0, internal ELRS module (CRSF, 16 channels). Model `STELLAR` is
`MODELS/model1.yml` on the radio's SD card; copy in [`radio-model.yml`](radio-model.yml).

| Channel | Source | Used for |
|---|---|---|
| CH1-4 | Ail, Ele, Thr, Rud | AETR |
| CH5 (AUX1) | SE (2-pos) | ARM |
| CH6 (AUX2) | SA (3-pos) | unused on the quad |
| CH7 (AUX3) | SB (3-pos) | GPS Rescue |
| CH8 (AUX4) | SC (3-pos) | Beeper |
| CH9 (AUX5) | SD (3-pos) | Acro / Horizon / Angle |
| CH10 (AUX6) | SF (momentary) | Turtle |
| CH11-12 | P1, P2 | unused |
| CH15-16 | TILT_X, TILT_Y | unused |

Special functions: voice call-outs on SA1 (`preact`), SB2 (`gps`), SD0 (`acrmod`), SE2 (`armed`);
SF2 also runs the `setGyro` script; RGB LED always on. The template's TRAINER function on SC2
was removed (SC down is the beeper).

Restore: radio in USB Storage mode, copy `radio-model.yml` to `MODELS/model1.yml`, eject.

## Telemetry and lost-quad beeper

```
feature TELEMETRY   # CRSF telemetry to the radio: battery, mAh, flight mode, GPS position
beacon RX_LOST      # motors beep (DShot beacon) when the radio link is lost
beacon RX_SET       # motors beep when the BEEPER switch (AUX4) is on
```

After a crash out of sight: the radio keeps the last GPS position, and turning the radio off
makes the quad beep.

## Assisted crash flip

Betaflight 2025.12+ feature ([betaflight#13391](https://github.com/betaflight/betaflight/issues/13391),
[PR #13905](https://github.com/betaflight/betaflight/pull/13905)). Needs the accelerometer.

```
set crashflip_rate = 30          # max flip rate 300 deg/s; 0 turns the assist off
set crashflip_auto_rearm = ON    # turtle switch off = back to normal flight, no re-arm
set osd_flip_arrow_pos = 2426    # OSD arrow showing which way to push the stick
```

Use: upside down, turtle switch (AUX6) on, arm, push the stick the way the OSD arrow points
over about half a second. Motors stop on their own once it has flipped. Throttle at zero,
turtle switch off, take off.

Tuning: too weak to flip, try `crashflip_rate = 0`. Flips twice, lower to about 20.

## Pre-flight checks after a restore or reflash

- [ ] Setup tab: nose down, roll right and yaw right each move the 3D model the same way.
- [ ] Setup tab: *Calibrate Accelerometer* on a level surface (last done 2026-11-07 on 2026.6.2).
- [ ] Receiver tab (LiPo in, props off, do not arm): every stick and switch moves the right
      channel the right way.
- [ ] GPS gets satellites outdoors.

## Migration notes: Betaflight 4.5 to 2026.6

Lines from the 4.5.x configs in `archive/` that do not apply on 2025.12+ and what replaced them:

| 4.5.x | 2026.6.2 | Why |
|---|---|---|
| `gyro_1_sensor_align = CW180`, `gyro_1_align_yaw = 1800` | `align_board_yaw = 270` (with `align_board_roll = 180`) | User gyro alignment was removed in 2025.12.1. The board's sensor alignment is now fixed at CW90, so the extra rotation moves into board alignment. Solved with Betaflight's own rotation code, equivalent on every axis. |
| `d_roll = 48`, `d_min_roll = 45` | `d_roll = 45`, `d_max_roll = 48` | D-min became D-max: `d_*` is now the base D and `d_max_*` the peak, so the values swap. |
| `d_pitch = 73`, `d_min_pitch = 68` | `d_pitch = 68`, `d_max_pitch = 73` | Same as roll. |
| `simplified_dmax_gain = 20` | `simplified_d_max_gain = 20` | Renamed. |
| `debug_mode = GYRO_SCALED` | (dropped) | Mode no longer exists; blackbox debug only. |
| `mag_hardware = NONE` | (dropped) | No magnetometer in the build. |
| `vbat_max_cell_voltage = 435` | same, now under `battery_profile 0` | Battery profiles added. |

## History

- **2026-11-07** Found on Betaflight 4.5.2 built with SBUS only, so the ELRS receiver could not
  work. Reflashed 4.5.5 with CRSF; the receiver turned out to be on UART4 and the GPS on UART1
  (the config had them on UART1 and UART2). Then upgraded to 2026.6.2 for the assisted crash
  flip. Archive: [`2026-11-07_bf-4.5.2_original.txt`](archive/2026-11-07_bf-4.5.2_original.txt)
  (as found), [`2026-11-07_bf-4.5.5_elrs.txt`](archive/2026-11-07_bf-4.5.5_elrs.txt) (last 4.5.x
  config, ELRS working, current modes).
