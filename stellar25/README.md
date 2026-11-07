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
| Other options | Acro Trainer, GPS, LED Strip, Pin IO, VTX, Chirp |
| Race Pro | off |

The build also pulls in Altitude Hold and Position Hold with GPS. No magnetometer support
(none fitted).

## Modes

Standard layout shared with every quad, see [the root README](../README.md#standard-switch-layout).

| Mode | Channel | Range | Radio switch (TX15) |
|---|---|---|---|
| ARM | AUX1 (ch5) | 1700-2100 | SE down |
| ALT HOLD + POS HOLD | AUX3 (ch7) | 1300-1700 | SB middle |
| GPS RESCUE | AUX3 (ch7) | 1700-2100 | SB down |
| BEEPER | AUX4 (ch8) | 1700-2100 | SC down |
| HORIZON | AUX5 (ch9) | 1300-1700 | SD middle (SD up = Acro) |
| ANGLE | AUX5 (ch9) | 1700-2100 | SD down |
| FLIP OVER AFTER CRASH (turtle) | AUX6 (ch10) | 1700-2100 | SF, hold (momentary) |
| CHIRP + BLACKBOX (autotune test only) | AUX2 (ch6) | 1700-2100 | SA down |

Slots `aux 0-7` are the standard block, `aux 8-9` the chirp test. ALT/POS HOLD added
2026-11-07; Position Hold has no compass here and shows `POS HOLD FAIL` until GPS has worked out
the heading from a stretch of straight forward flight.

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

Special functions: voice call-outs on SB2 (`gps`), SD0 (`acrmod`), SE2 (`armed`); SF2 also runs
the `setGyro` script; RGB LED always on. Removed: the template's TRAINER function on SC2 (SC down
is the beeper) and the SA1 `preact` call-out (no PREARM in the standard layout).
This model is the template for every quad's radio model (ROC7 is `model3.yml`).

Main screen (`Layout2x4`, Value widgets), page 2 Outputs, page 3 RSSI:

| Left | Right |
|---|---|
| RxBt (average cell voltage) | FM (flight mode) |
| Capa (mAh used) | Sats |
| Curr (A) | Alt |
| RQly (link quality) | GPS (position) |

Low-battery alerts, on average cell voltage (`report_cell_voltage = ON` on the quad):

| Logical switch | Condition | Special functions |
|---|---|---|
| L1 | RxBt < 3.5 V for 3 s | speak RxBt every 10 s, light haptic every 10 s |
| L2 | RxBt < 3.3 V for 2 s | `Wrn2` tone every 5 s, strong haptic every 5 s |

Not tied to the arm switch: EdgeTX already treats telemetry-based logical switches as off when
telemetry is not streaming, so the alerts also warn about a low pack left plugged in on the bench.
Thresholds match Betaflight's `vbat_warning_cell_voltage` (3.5) and `vbat_min_cell_voltage` (3.3).

Restore: radio in USB Storage mode, copy `radio-model.yml` to `MODELS/model1.yml`, eject.

## Telemetry and lost-quad beeper

```
feature TELEMETRY   # CRSF telemetry to the radio: battery, mAh, flight mode, GPS position
set report_cell_voltage = ON   # RxBt on the radio = average cell voltage, not pack voltage
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

## Autotune (chirp)

The Betaflight App's **Autotune** tab analyses a blackbox log recorded with `debug_mode = CHIRP`
and recommends gains. Needs the **Chirp** cloud-build option. Test setup currently in
`diff-all.txt`:

```
set debug_mode = CHIRP
set blackbox_high_resolution = ON
aux 8 55 1 1700 2100 0 0   # CHIRP mode on AUX2 (SA down)
aux 9 26 1 1700 2100 0 0   # BLACKBOX on AUX2: log only while SA is down (8 MB flash)
```

Flight: open space, fresh pack, hover 2-3 m in Angle mode. SA down: OSD shows `CHIR`, the
quad shakes on roll; hold position, wait for `CHIRP IS FINISHED`, SA up. Repeat for pitch,
then yaw (each SA down moves to the next axis). SA up stops it at once.

Then: Blackbox tab, *Save flash to file*; Autotune tab, *Import* the `.bbl`; review and apply.
Afterwards remove the test lines (`debug_mode`, `blackbox_high_resolution`, aux 8/9) and erase
the flash.

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
