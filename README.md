# betaflight-profile

Betaflight configuration backups for my quads, one folder per craft.

| Craft | Board (Betaflight target) | Firmware | Config |
|---|---|---|---|
| [STELLAR25](stellar25/) | JHEMCU GHF405AIO-HD (`JHEF405PRO`) | Betaflight 2026.6.2 | [`stellar25/diff-all.txt`](stellar25/diff-all.txt) |

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
