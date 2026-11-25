# Printer notes

One file per printer. These hold what the config files can't: what hardware is actually fitted, what has been measured, what has gone wrong, and what is still open.

| Printer | Frame | Notes | Last inspected live |
|---|---|---|---|
| thebrain | ZeroG Mercury One on an Ender 5 Plus | [thebrain.md](thebrain.md) | 2026-11-24 |
| pinky | ZeroG Mercury One on an Ender 5 Plus | [pinky.md](pinky.md) | 2026-11-24 |
| hellonurse | RatRig V-Minion | [hellonurse.md](hellonurse.md) | never (config only) |
| slappysquirrel | Voron V0 / BoxZero | [slappysquirrel.md](slappysquirrel.md) | never (config only) |

[mercury-one-motion.md](mercury-one-motion.md) holds what applies to both Mercury machines: how their Y axis behaves on small moves, how to measure it, and what would change it.

## Keeping these useful

- Date every measurement and say how it was taken. A number without a date goes stale silently.
- Mark anything read from config but not checked on the machine as unverified.
- Values that only live on a printer's host (the `SAVE_CONFIG` block in `printer.cfg`) are copied here so they survive a dead boot drive.
- This repository is public. Hostnames, addresses, usernames, credentials and network details do not belong here.
