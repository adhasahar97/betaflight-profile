# betaflight-profile

Betaflight configuration backups for my quads, one folder per craft.

| Craft | Board (Betaflight target) | Firmware | Config |
|---|---|---|---|
| [STELLAR25](stellar25/) | JHEMCU GHF405AIO-HD (`JHEF405PRO`) | Betaflight 2026.6.2 | [`stellar25/diff-all.txt`](stellar25/diff-all.txt) |
| [ROC7](roc7/) | HAKRC F722 V2 (`HAKRCF722V2`) | Betaflight 2026.6.2 | [`roc7/diff-all.txt`](roc7/diff-all.txt) |

Open work: [TODO.md](TODO.md).

## Standard switch layout

Every quad uses the same modes on the same switches (RadioMaster TX15, EdgeTX). Each quad has its
own radio model, cloned from the same template, so the channel order is identical.

| Switch | Channel | Up | Middle | Down |
|---|---|---|---|---|
| SE (2-pos) | ch5, AUX1 | disarmed | | ARM |
| SA (3-pos) | ch6, AUX2 | | | spare (CHIRP + BLACKBOX during autotune) |
| SB (3-pos) | ch7, AUX3 | | ALT HOLD + POS HOLD | GPS RESCUE |
| SC (3-pos) | ch8, AUX4 | | spare | BEEPER |
| SD (3-pos) | ch9, AUX5 | ACRO | HORIZON | ANGLE |
| SF (button) | ch10, AUX6 | | | TURTLE (hold) |

Ranges: down = 1700-2100, middle = 1300-1700. No PREARM. Spare channels: ch11-12 (P1, P2 pots).

Radio models on the TX15: `model1.yml` STELLAR, `model3.yml` ROC7 (clone of STELLAR). Each
craft folder has a copy as `radio-model.yml`.

## ELRS bands

The TX15's internal ELRS module (LR1121) runs 2.4 GHz or 900 MHz, one at a time. ELRS keeps its
settings (packet rate, so the band) per **model ID**, the receiver number of the internal module
in each EdgeTX model. Selecting the model on the radio therefore selects the band.

| Craft | Receiver band | EdgeTX model | Model ID (internal module) |
|---|---|---|---|
| STELLAR25 | 2.4 GHz | `model1.yml` STELLAR | 0 (default, not written in the file) |
| ROC7 | 900 MHz | `model3.yml` ROC7 | 1 (`header.modelId.0.val`) |

A new craft gets the next free model ID. After creating its model, select it on the radio and set
the packet rate for its band once in the ExpressLRS Lua script.

```
aux 0 0 0 1700 2100 0 0    # ARM          AUX1
aux 1 1 4 1700 2100 0 0    # ANGLE        AUX5 down
aux 2 46 2 1700 2100 0 0   # GPS RESCUE   AUX3 down
aux 3 13 3 1700 2100 0 0   # BEEPER       AUX4 down
aux 4 35 5 1700 2100 0 0   # TURTLE       AUX6
aux 5 2 4 1300 1700 0 0    # HORIZON      AUX5 middle
aux 6 3 2 1300 1700 0 0    # ALT HOLD     AUX3 middle
aux 7 11 2 1300 1700 0 0   # POS HOLD     AUX3 middle
```

## Layout

```
<craft>/
├── README.md      hardware, wiring, firmware build options, modes, notes
├── diff-all.txt   CURRENT config: output of `diff all`, for the firmware named in the README
└── archive/       configs captured on older firmware, named <date>_bf-<version>_<note>.txt
```

`diff-all.txt` keeps the same name on purpose: overwrite it after every change and commit,
so `git log -p <craft>/diff-all.txt` is the change history. Configs from an older firmware go
into `archive/`, because a `diff all` only restores cleanly on the version it came from.

## Back up

1. Connect in Betaflight Configurator, open the **CLI** tab.
2. Run `diff all` and save the output (or use *Save to File*).
3. Overwrite `<craft>/diff-all.txt` with it, commit.

## Restore

1. Flash the firmware version and cloud-build options listed in the craft's README, with
   **Full chip erase** on.
2. Open the **CLI** tab, paste the whole of `diff-all.txt`, press Enter.
3. If the CLI reports any error the batch refuses to `save`. Fix the line and paste again.
4. Run the pre-flight checks in the craft's README before flying.

Props off whenever the craft is powered on the bench.
