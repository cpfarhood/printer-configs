# pinky

ZeroG Mercury One conversion of an Ender 5 Plus. Sibling of [thebrain](thebrain.md) and built from the same shared files.

**Everything here is read from the config files. The machine has not been inspected live.** Treat the thebrain notes as the best guide to how it behaves, and verify before relying on any of it.

## Config

- Machine file: `printer-pinky.cfg`
- Shared with thebrain: `printer-common.cfg`, `printer-mercury_one_plus.cfg`, and the macro files. The host's own `printer.cfg` has not been read, so the toolhead and stepper files it includes are unverified.
- The `[mcu]` comment says BTT SKR Mini E3 V3 on the Pi UART at 500000 baud, with a CAN toolhead board (`head0`).

## Differences from thebrain (as of 2026-11-22)

| Setting | pinky | thebrain |
|---|---|---|
| `dockable_probe` dock / approach X | 209 | 210.5 |
| `z_calibration switch_offset` | 0.470 | 0.525 |
| `startupinit` | True | False |

Everything else in the two machine files is identical apart from the CAN UUID.

## Changes that reached pinky through shared files on 2026-11-22

These were made after an incident on thebrain and have **not been tested on pinky**:

- `stepper_z position_min` -15 → -2.
- `bed_mesh mesh_max` 377,350 → 335,315. The new bounds were proven on thebrain's bed. Whether pinky's plate sits in the same place relative to its coordinates is unknown.
- `PAUSE`, `RESUME` and `CANCEL_PRINT` now come from Mainsail, with the LED and power-off hooks enabled in `_CLIENT_VARIABLE`.
- While a print is paused, the idle timeout turns off only the hotend, then gives up after 4 more hours.
- Hotend `max_temp` 310 → 285, if pinky uses `toolhead-afterburner.cfg`.

## Still true on pinky

- `startupinit: True`: every Klipper start heats to 184/50°C, homes, and sweeps the bed at about 19 mm. Anything taller left on the bed gets hit.
- `[z_calibration] end_gcode` calls `DOCK_PROBE`, which is not a command in the installed `dockable_probe` module on thebrain. It prints "Unknown command" and does nothing.

## To check when someone is at the machine

- Where the probe pin runs off the plate on the right and back edges, before widening the mesh.
- Which Klipper fork and version the host runs. Option and command names depend on it.
- Belt resonance on each diagonal, gantry tilt convergence, and Z calibration repeatability, as measured on thebrain.
