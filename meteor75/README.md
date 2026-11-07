# Meteor75 Pro

BetaFPV Meteor75 Pro, 1S whoop. Onboarded 2026-11-08 on Betaflight 4.5.1 (BetaFPV factory
build, Dec 2025). `diff-all.txt` is the config as it arrived.

## Hardware

| Part | Detail |
|---|---|
| Flight controller | BetaFPV G473. Betaflight target `BETAFPVG473` (the original, **not** `_V2` or `_V3`), STM32G474 |
| Gyro | BMI270. Target orientation `CW180`, this quad is set to `CW90` (`gyro_1_sensor_align`, `gyro_1_align_yaw = 900`) |
| Barometer | Off (`baro_hardware = NONE`). Whether the board has one gets checked on the next firmware |
| GPS, compass | None |
| Blackbox | 16 MB onboard flash |
| Receiver | ExpressLRS **2.4 GHz**, CRSF on UART3 (target default) |
| Video | Digital VTX on UART4 (MSP DisplayPort, HD OSD 53 x 20) |
| ESCs | DShot300 bidirectional, 12-pole motors, idle 5% (`dshot_idle_value = 500`) |
| Battery | 1S LiHV (`vbat_max_cell_voltage = 440`, warning 3.45 V), current sensor on (`ibata_scale = 550`) |
| Tune | BetaFPV's, PID profile `GF 40mm`, sliders on |

## Serial ports

| UART | Use | `serial` line |
|---|---|---|
| UART3 | Serial RX (CRSF) | target default, not in the diff |
| UART4 | MSP + VTX (MSP DisplayPort) | `serial 3 131073 ...` |

UART1 and UART2 are free.

## Modes as received

| Mode | Channel | Range | Radio switch (TX15) |
|---|---|---|---|
| ARM | AUX1 (ch5) | 1700-2100 | SE down |
| ANGLE | AUX5 (ch9) | 900-1300 | SD up |
| HORIZON | AUX5 (ch9) | 1300-1700 | SD middle |
| BEEPER | AUX4 (ch8) | 1700-2100 | SC down |
| FLIP OVER AFTER CRASH (turtle) | AUX6 (ch10) | 1700-2100 | SF, hold |

Plan: Betaflight 2026.6.2 and the [standard switch layout](../README.md#standard-switch-layout)
(ANGLE moves to SD down). Steps in [TODO.md](../TODO.md).

## Next firmware: 2026.6.2 cloud build

| Option | Value |
|---|---|
| Board | `BETAFPVG473` |
| Radio protocol | CRSF |
| Telemetry protocol | Automatically included |
| OSD protocol | Digital |
| Motor protocol | DSHOT |
| Other options | Acro Trainer, Altitude Hold, LED Strip, VTX, Chirp |

Altitude Hold only works if a barometer turns up. Chirp is there for autotune later.
