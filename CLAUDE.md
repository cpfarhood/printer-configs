# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Klipper firmware configuration fragments (`.cfg`) for a fleet of 3D printers. There is no build, lint, or test tooling. Changes can only be checked by loading them on the printer host (Klipper `RESTART` / `FIRMWARE_RESTART`, errors show in Mainsail/Fluidd or `klippy.log`). Most commits are small calibration tweaks to a single printer file, such as `switch_offset` or dock positions.

## Deployment

Each printer host (a Pi running Klipper + Moonraker) has this repo cloned at `~/printer_data/config/printer-configs`. Moonraker's `update_manager` entry `farhoodliquor-printer-configs` is meant to pull it from GitHub (`cpfarhood/printer-configs`) and restart Klipper, but the entry is commented out in the Mercury machines' shared Moonraker include (seen on pinky; thebrain was pulled by hand). So a change reaches a printer only after it's pushed to `master` and pulled on the host with `git pull`, followed by a Klipper restart. A host can be far behind: check its checkout before assuming it has a change. Moonraker settings live in a sibling repo, `cpfarhood/printer-moonraker-configs`.

Moonraker's `[power printer]` device has `klipper` as a bound service. When the idle power-off macro fires, Klipper is stopped and the Moonraker API reports "Klippy Host not connected". That is expected, not a fault. On printers with `startupinit: True`, every Klipper start runs `PRINTER_STARTUP`, which heats, homes and sweeps the toolhead across the bed. That includes the restart after a config pull.

## Printer notes

`docs/<printer>.md` records what the config can't: the hardware actually fitted (several config headers are wrong), measurements, incidents and open items. Read the printer's file before changing its config, and update it when you learn or change something. `docs/mercury-one-motion.md` covers what thebrain and pinky share: their Y axis ignores very small moves, which is normal for them, and the file says how to measure it. The repo is public, so keep hostnames, addresses, usernames and credentials out of every file.

Probe moves are the main way to damage these machines. A probe that never triggers keeps driving Z to `position_min`. Before any probing, check that each point is one the printer's normal prints already use; `docs/thebrain.md` describes an incident and the watchdog approach.

## How the files compose

No file here has `[include]` directives. Each printer host has its own `printer.cfg` (not in this repo) that includes a stack of these fragments. For example, thebrain includes `printer-common` → `printer-mercury_one_plus` → `toolhead-afterburner` → `printer-thebrain` → `steppers-omc_17hm19-2004s`, then `mainsail.cfg` and the macro files. Klipper merges sections across includes, and the **last definition of a key wins**, so a section can appear in several files and each file sets only some of its keys (for example `[extruder]`, `[dockable_probe]`, `[stepper_x]`). Include order matters.

Layers, by filename prefix:

- `printer-common.cfg`: shared on every host (save_variables, virtual_sdcard, idle_timeout → power-off check, host MCU).
- `printer-<frame>.cfg`, for example `printer-mercury_one_plus.cfg`: the frame/kinematics base (steppers, bed, z_tilt, bed_mesh, fans, neopixel).
- `printer-<machine>.cfg`, for example `thebrain`, `pinky`: per-machine overrides stacked on a frame file. `thebrain` and `pinky` are both Mercury One Plus machines and are near-identical, differing mainly in calibration values and CAN UUIDs. `hellonurse` (RatRig V-Minion) and `slappysquirrel` (Voron V0) are self-contained frame+machine files.
- `toolhead-*.cfg`, `probe-*.cfg`, `steppers-*.cfg`: hardware modules that fill in or override pins and currents for `[extruder]`, probes, and stepper motors. `retired-*` files are no longer in use.
- `macros-*.cfg` / `macro-*.cfg`: G-code macros shared across printers. `macros-common.cfg` holds `PRINT_START`, `PRINT_END`, and filament load/unload. `PAUSE`, `RESUME` and `CANCEL_PRINT` come from Mainsail's `mainsail.cfg` on the host; `macros-common.cfg` only supplies `_PAUSE_HOOK`, `_RESUME_HOOK` and `_CANCEL_HOOK`, which a printer opts into through `variable_user_*_macro` in its `_CLIENT_VARIABLE`.
- `plugin-*.cfg`: the config section for an optional Klipper plugin. Include one only on a host where that plugin is installed, or Klipper will not start.
- `screen-*.cfg`: KlipperScreen config, not Klipper.

## Two per-printer variable mechanisms (keep in sync)

The shared macros read printer-specific settings from two separate places:

1. **`save_variables`** (`printer.save_variables.variables`, aliased `svv`/`SVV`). A `[delayed_gcode INITIALIZATION]` block in the frame-level file writes these with `SAVE_VARIABLE` at boot and then calls `PRINTER_STARTUP`. `PRINT_START` uses them to decide on probe docking, `Z_TILT_ADJUST`, `CALIBRATE_Z`, bed mesh type, and wipe line.
2. **`[gcode_macro _Printer_Variables]`**, defined in the machine-level file. `PRINTER_STARTUP`, `PRINT_START` temperature defaults, and all LED-effect logic (`main_panel`/`bed_panel`) read from it.

Both define overlapping flags (`etemp`, `btemp`, `dockableprobe`, `ztiltadjust`, `autozcalibrate`, `bedmeshonprint`, `bedmeshtype`, `wipeline`). The value `PRINT_START` acts on comes from the save_variables block, not from `_Printer_Variables`, so a feature toggle must be changed in the right file. The LED flags are named `main_led`/`bed_led` in save_variables but `main_panel`/`bed_panel` in `_Printer_Variables`, and only the latter are read.

`_CLIENT_VARIABLE` in machine files configures Mainsail's built-in client macros (park, retract).

## Macro dependencies

The shared macros assume these are present on the host: Moonraker power device named `printer` (`_POWER_OFF_PRINTER`), the `led_effect` plugin with effects `printer_init`/`idle`/`active`/`complete`/`pause`/`cancel`/`error` defined per machine (when `main_panel` is enabled), and on Mercury machines the `z_calibration` plugin and `[dockable_probe]` (`ATTACH_PROBE`/`DETACH_PROBE`). `macro-adaptive_bed_mesh.cfg` is a vendored third-party macro (Frix_x v4). `probe-slappysquirrel.cfg` is vendored Klicky probe macros.
