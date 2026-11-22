# slappysquirrel

Voron V0 / BoxZero, CoreXY.

**Everything here is read from the config files, last edited 2024-01-14. The machine has not been inspected live.**

## Hardware, per config

| Part | What the config says |
|---|---|
| Mainboard | Header says BTT SKR Mini E3 V3, but every pin is an RP2040 `gpioNN` name and the serial ID is `usb-Klipper_rp2040_…`. That matches a BTT SKR Pico. Unverified. |
| Host link | USB serial |
| Toolhead | Mini Afterburner (50:10 gearing), 0.4 mm nozzle |
| Probe | Klicky-style dockable probe driven by the vendored macros in `probe-slappysquirrel.cfg`, also used as the Z endstop |
| X/Y homing | Physical endstops, 100 mm/s |
| Drivers | TMC2209, 1.0 A on X/Y, 0.5 A on Z, 0.6 A on the extruder |
| Extras | One neopixel under the bed, bed screw tilt adjust |

## Limits and geometry

| Setting | Value |
|---|---|
| Travel | X 0…118, Y 0…115, Z -15…120 |
| Speed / accel | 600 mm/s, 12000 mm/s² |
| Bed mesh | 30,5 → 115,115 |
| Hotend `max_temp` | 270°C (Generic 3950) |
| Bed `max_temp` | 120°C |

## Config files

- `printer-slappysquirrel.cfg`: frame and machine in one file.
- `toolhead-slappysquirrel.cfg`, `probe-slappysquirrel.cfg`
- Shared macro files. The host's `printer.cfg` has not been read, so the exact include list is unverified.

## Known gaps

- **`Z position_min` is -15**, the same setting that let a missed probe drive thebrain's bed into its toolhead on 2026-11-22. `mesh_max` 115,115 is also at the edge of a 120 mm bed. Review both before running a full mesh.
- It has no `_Printer_Variables` or `_CLIENT_VARIABLE` section, so the shared macros run on their built-in defaults.
- Its startup block does not save `ztiltadjust` or `wipeline`. `PRINT_START` treats a missing value as off.
- Since 2026-11-22 the shared macros no longer override `PAUSE`, `RESUME` and `CANCEL_PRINT`, so this printer gets Mainsail's defaults. The idle timeout's paused branch needs a reasonably current `mainsail.cfg` on the host. Unverified here.

## To check when someone is at the machine

- Which board is really fitted, and fix the header.
- Klipper fork and version, and the `mainsail.cfg` version.
- Where the probe actually lands at the mesh corners.
