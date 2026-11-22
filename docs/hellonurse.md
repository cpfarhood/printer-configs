# hellonurse

RatRig V-Minion, cartesian.

**Everything here is read from the config files, last edited 2024-02-14. The machine has not been inspected live.**

## Hardware, per config

| Part | What the config says |
|---|---|
| Mainboard | Header says BTT SKR Mini E3 V3, but every pin is an RP2040 `gpioNN` name and the commented serial ID is `usb-Klipper_rp2040_…`. That matches a BTT SKR Pico. Unverified. |
| Host link | Pi UART (`/dev/ttyAMA0`) |
| Toolhead | EVA3 "Squirrelbrain" mod, Sherpa Mini extruder, CHC V6 hotend, 0.4 mm nozzle |
| Probe | BLTouch, also used as the Z endstop |
| X/Y homing | Sensorless (TMC2209 virtual endstops), 40 mm/s |
| Drivers | TMC2209, 0.6 A on X/Y/Z, 0.85 A on the extruder |
| Extras | Filament switch sensor, filament unload button, 14-LED neopixel strip with LED effects |

## Limits and geometry

| Setting | Value |
|---|---|
| Travel | X -4…180, Y -6…185, Z -5…180 |
| Speed / accel | 200 mm/s, 2000 mm/s² |
| Bed mesh | 35,5 → 175,160 |
| Hotend `max_temp` | 275°C (Generic 3950) |
| Bed `max_temp` | 130°C |

## Config files

- `printer-hellonurse.cfg`: frame and machine in one file, including its own `_Printer_Variables`.
- `toolhead-hellonurse.cfg`
- Shared macro files. The host's `printer.cfg` has not been read, so the exact include list is unverified.

## Behaviour inherited from shared files

- `startupinit: True` and `warmonboot: 1`: every Klipper start heats to 185/50°C, homes, and sweeps the bed.
- It has no `_CLIENT_VARIABLE` section. Since 2026-11-22 the shared macros no longer override `PAUSE`, `RESUME` and `CANCEL_PRINT`, so this printer gets Mainsail's defaults with no LED changes on pause or cancel.
- The shared idle timeout's paused branch sets Mainsail's `idle_state` variable on `RESUME`. That needs a reasonably current `mainsail.cfg` on the host. Unverified here.
- `Z position_min` is -5 with the probe as Z endstop. Worth reviewing in the light of the thebrain incident.

## To check when someone is at the machine

- Which board is really fitted, and fix the header.
- Klipper fork and version, and the `mainsail.cfg` version.
- That pause, resume and cancel behave after the 2026-11-22 macro change.
