# TODO

## STELLAR25: chirp autotune (in progress, set up 2026-11-07)

Firmware has the Chirp build option and the test settings are on the quad (see
[stellar25/README.md](stellar25/README.md#autotune-chirp)). Blackbox flash is erased.

- [ ] Setup tab orientation check: nose down, roll right, yaw right all move the 3D model the same way.
- [ ] Fly the chirp test: Angle mode hover, SA down / wait for `CHIRP IS FINISHED` / SA up, three times (roll, pitch, yaw).
- [ ] Blackbox tab: *Save flash to file* (`.bbl`).
- [ ] Autotune tab: *Import* the `.bbl`, review the recommended gains before *Apply*.
- [ ] Remove the test settings and save:
      `set debug_mode = NONE`, `set blackbox_high_resolution = OFF`, `aux 6 0 0 900 900 0 0`, `aux 7 0 0 900 900 0 0`.
- [ ] Erase the blackbox flash, `diff all`, commit the new tune to `stellar25/diff-all.txt`.

## STELLAR25: other open checks

- [ ] GPS gets satellites outdoors (BZ-121 on UART1).
- [ ] Hear the low-battery alert once (temporarily set L1 to 4.4 on the radio, then back to 3.5).
