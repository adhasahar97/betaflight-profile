# TODO

## STELLAR25: chirp autotune (in progress, set up 2026-11-07)

Firmware has the Chirp build option and the test settings are on the quad (see
[stellar25/README.md](stellar25/README.md#autotune-chirp)). Blackbox flash is erased.

- [ ] Setup tab orientation check: nose down, roll right, yaw right all move the 3D model the same way.
- [ ] Fly the chirp test: Angle mode hover, SA down / wait for `CHIRP IS FINISHED` / SA up, three times (roll, pitch, yaw).
- [ ] Blackbox tab: *Save flash to file* (`.bbl`).
- [ ] Autotune tab: *Import* the `.bbl`, review the recommended gains before *Apply*.
- [ ] Remove the test settings and save:
      `set debug_mode = NONE`, `set blackbox_high_resolution = OFF`, `aux 8 0 0 900 900 0 0`, `aux 9 0 0 900 900 0 0`.
- [ ] Erase the blackbox flash, `diff all`, commit the new tune to `stellar25/diff-all.txt`.

## Radio: one-time ELRS band setup (done 2026-11-07, both models link)

- [x] Select model ROC7 (model ID 1): ExpressLRS Lua, set the 900 MHz packet rate used for ROC7. Confirm it links.
- [x] Select model STELLAR (model ID 0): ExpressLRS Lua, confirm the 2.4 GHz packet rate. Confirm it links.
- [x] Switch between the two models once more and check each still links without touching the Lua.

## ROC7: checks after the 2026.6.2 upgrade

- [ ] Setup tab orientation check (nose down, roll right, yaw right).
- [ ] Receiver tab (LiPo in, props off, do not arm): sticks and AUX1-AUX6 move the right channels.
- [ ] Motors tab, props off: motor order and direction unchanged (props out).
- [ ] GPS gets satellites outdoors; ESC telemetry shows RPM / temperature.
- [ ] First hover in Angle mode before flying the custom tune hard (D values were remapped to the new D-max scheme).

## STELLAR25: other open checks

- [ ] GPS gets satellites outdoors (BZ-121 on UART1).
- [ ] Hear the low-battery alert once (temporarily set L1 to 4.4 on the radio, then back to 3.5).
