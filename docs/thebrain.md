# thebrain

An Ender 5 Plus converted to a ZeroG Mercury One (CoreXY), then customised further. Last inspected live on 2026-11-22.

## Hardware

Checked on the machine unless marked otherwise. Several labels in the config files disagree with this table; the table is right.

| Part | What is fitted |
|---|---|
| Mainboard | BTT SKR Mini E3 V3 (STM32G0B1), on the Pi's UART at 500000 baud. The "BTT Octopus 1.1" header in `printer-thebrain.cfg` is wrong. |
| Toolhead board | Mellow Fly SHT42 (STM32F072) on CAN at 1 Mbit, flashed through CanBoot |
| CAN interface | USB adapter (gs_usb / candleLight) on the Pi, not the mainboard |
| X/Y/Z motors | OMC 17HM19-2004S, 0.9°, 32 microsteps. X/Y at 1.35 A, both Z at 1.25 A, on the board's TMC2209s in spreadCycle |
| Z drive | Two independent belted Z motors, 60:20 pulleys plus a 2:1 pulley block (6:1 overall) |
| Extruder | 50:17 gearing, 0.9° motor at 0.75 A, `rotation_distance` 22.2662775 |
| Hotend | 0.6 mm nozzle, configured as a Generic 3950 thermistor (thermistor type not confirmed) |
| Probe | Klicky-style dockable probe on Klipper's `dockable_probe` module, docked at the back of the gantry |
| Z endstop | Nozzle-touch pin beside the bed at X388 Y85, used by `z_calibration` |
| X endstop | On the toolhead board, read over CAN |
| Frame sensor | Thermistor on the mainboard's TH0 port, feeding `z_thermal_adjust` |
| Accelerometer | ADXL345 on the toolhead board |
| Lighting | 25-LED neopixel strip (`main`) plus a white LED channel |
| Camera | Logitech C930e at the front, looking back across the gantry. It sees the bed only when the bed is within roughly 60 mm of the nozzle. |
| Host | Raspberry Pi 4, booting from a USB flash drive, wired Ethernet |
| Power | Moonraker switches the printer's supply through a GPIO relay. The Pi stays on. |

### Belts

- Left rear motor = `stepper_x` = **lower** belt.
- Right rear motor = `stepper_y` = **upper** belt.

Confirmed 2026-11-22 by camera: a move along +X+Y turns only `stepper_x`, and the lower rear belt span and the left motor were what moved. A move along -X+Y moved the upper span and the right motor.

## Software

- Klipper fork: Danger Klipper (since renamed Kalico) at `v0.12.0-526`, December 2024. The checkout still points at the old repository name, so the update manager reports "up to date" and never offers updates. MCU firmware is `v0.12.0-514` on all three MCUs.
- `dockable_probe` and `z_calibration` are bundled with that fork. `led_effect` is a separate plugin.
- Option and command names must be checked against Danger Klipper / Kalico, not mainline Klipper docs.
- Resonance tools need the system package `libopenblas0`. It was missing from November 2024 until 2026-11-22, so `TEST_RESONANCES` and `SHAPER_CALIBRATE` failed with a numpy import error during that time. `calibrate_shaper.py` still fails because matplotlib is not installed in Klipper's Python environment.

## Config

The host's `printer.cfg` includes, in order:

1. `printer-common.cfg`
2. `printer-mercury_one_plus.cfg`
3. `toolhead-afterburner.cfg`
4. `printer-thebrain.cfg`
5. `steppers-omc_17hm19-2004s.cfg`
6. `mainsail.cfg`
7. `macros-common.cfg`, `macro-test_speed.cfg`, `macro-heat_soak.cfg`, `macro-wipe_line.cfg`, `macros-lazy_homing.cfg`, `macros-printer_power.cfg`

`macro-adaptive_bed_mesh.cfg` is **not** included. `PRINT_START` still calls its `COMPUTE_MESH_PARAMETERS`, which prints "Unknown command" and is skipped. The mesh itself comes from Klipper's own `BED_MESH_CALIBRATE ADAPTIVE=1`.

### Values that live only on the host

These sit in the `SAVE_CONFIG` block of the host's `printer.cfg` and are not in any file in this repo. Copied 2026-11-22:

| Section | Values |
|---|---|
| `[extruder]` PID | kp 19.970, ki 1.022, kd 97.602 |
| `[heater_bed]` PID | kp 56.711, ki 0.863, kd 931.480 |
| `[input_shaper]` | X mzv 59.2 Hz, Y mzv 43.2 Hz |
| `[stepper_z]` | `position_endstop` 7.846 |
| `[dockable_probe]` | `z_offset` 8.110 |
| `[bed_mesh default]` | A stale 3×3 adaptive mesh over 155–214 mm. Not used by prints. |

## Coordinates

| Thing | Position |
|---|---|
| Travel | X 0…395, Y 0…390, Z -2…385 |
| Y endstop | Triggers at 375. Y can travel on to 390 to reach the probe dock. |
| Probe dock | X210.5 Y384, approach from Y360, detach by sliding to X235 |
| Probe offset from nozzle | X -1.5, Y +19.75 |
| Z endstop pin | Nozzle at X388 Y85; probe switch body at X383 Y65 |
| Pause / cancel park | X390 Y385, 15 mm above the print (Mainsail default) |
| Bed mesh | 0,20 → 335,315, 7×7 |
| Proven probe area | Probe positions X0…335, Y20…315 |

**The probe misses the plate at X377 Y20.** The plate's real right and back edges have not been measured. Do not widen `mesh_max` without watching where the probe pin lands.

Z positive moves the bed down, away from the nozzle.

## How it behaves

- **Power on:** Moonraker turns the relay on and starts Klipper. Queueing a job also powers it on. Since 2026-11-22 (`startupinit: False`) nothing heats or moves at startup. A thebrain-only block, `STARTUP_IDLE`, sets the idle LED effect and runs a one-second dwell to start the idle timer. It does nothing if a queued job is already printing.
- **Print start** (`PRINT_START`): heat bed and nozzle (nozzle 10°C short), home, attach probe, level the gantry, heat nozzle fully, `CALIBRATE_Z`, adaptive bed mesh, dock probe, prime 11 mm. Takes about 3.5 minutes from cold at 190/50°C.
- **Print end** (`PRINT_END`): heaters off, bed lowered to 95% of Z travel, park, motors off, power-off check scheduled.
- **Pause:** Mainsail's `PAUSE` retracts 1 mm, lifts 15 mm and parks at X390 Y385. `_PAUSE_HOOK` switches the LEDs to the pause effect.
- **Paused and idle:** after 10 idle minutes only the hotend turns off. The bed stays hot and the motors stay on. If the print is still paused 4 hours after that, it is cancelled and the printer shuts down.
- **Resume:** Mainsail's `RESUME` reheats the hotend to its previous target if it was turned off, un-retracts, and continues.
- **Cancel:** Mainsail's `CANCEL_PRINT` retracts, parks 15 mm above the print, and turns the heaters and part fan off. `_CANCEL_HOOK` sets the cancel LED effect, lowers the bed to 95% of Z travel (because thebrain sets `lowerbedoncancel: 1`), and schedules the power-off check, so the relay cuts once the nozzle is under 50°C.
- **Idle:** after 10 idle minutes the motors and heaters turn off, the part fan runs, and the relay cuts once nozzle and bed are both under 50°C. A printer that is powered on and never used powers itself off after about 10.5 minutes.
- **Motors off:** the bed sags when the Z motors release. The gantry tilt measured 1.3–1.6 mm out after each power-off, which is why every print starts with `Z_TILT_ADJUST`.

## Slicer

SuperSlicer 2.7.61, printer profile "Mercury One Plus", print profile "Normal @Mercury One Plus", filament "PLA (Overture) @Mercury One Plus". Read from a config bundle on 2026-11-22.

| Setting | Value | Note |
|---|---|---|
| Bed shape | 370 × 370, origin at the front left | The probe misses the plate at X377, so the usable right edge is unconfirmed |
| Max print height | 350 | |
| Start code | `SET_PRINT_STATS_INFO`, then `print_start EXTRUDER= BED= CHAMBER= SIZE=` | `SIZE` and `CHAMBER` are not used by `PRINT_START` |
| End code | `print_end` | |
| Heat waits | The slicer adds `M109` and `M190` after `print_start` | Redundant; `PRINT_START` has already heated |
| Layer height / nozzle | 0.2 mm / 0.6 mm | |
| Speeds | Default 120 mm/s, perimeters 60%, external 50%, first layer 30%, travel 160 | |
| Acceleration | Profile asks for 3000; output is 1500, and 750 on the first layer | Capped by the printer profile's 1500 "extruding" machine limit |
| Retraction | 1.5 mm at 60 mm/s, 0.3 mm Z lift | Slicer retraction, not `G10`/`G11` |
| Pressure advance | 0.02 in the filament profile, but switched off | Nothing is sent, so the printer runs at 0 |
| Filament | 190°C, bed 65°C, flow 0.96, fan 100% from layer 2 | |
| Object labels | OctoPrint-style comments | Moonraker turns these into `EXCLUDE_OBJECT_DEFINE` on upload. A file that reaches the host any other way gets a full-bed mesh instead of an adaptive one. |

## Measurements, 2026-11-22

Bed at 50°C and nozzle at 184°C unless noted.

| Check | Result |
|---|---|
| Nozzle heat-up | 30 → 184°C in about 75 s |
| Bed heat-up | 26 → 50°C in about 2 minutes |
| Heater stability | Within 0.5°C of target |
| X/Y endstop repeatability | About 0.01 mm over 6 homes at 120 mm/s |
| Z endstop repeatability | 0.019 mm spread over 6 homes |
| Probe repeatability | 20 samples: spread 0.015 mm, σ 0.003 mm. The first sample after a travel move reads about 0.013 mm low. |
| Gantry tilt | Converges to under 0.01 mm in 4–5 passes, each pass removing about 85% of the error |
| Z calibration, 3 runs | Offsets 0.210, 0.221, 0.234 mm. Nozzle touch varied 0.018 mm, switch 0.007 mm, bed probe 0.009 mm. |
| Z calibration, later the same day | 0.271, 0.279, 0.277 and 0.230 mm, each at the start of a print, nozzle at 190°C. Seven runs in one day span 0.07 mm. |
| Bed screws | Corners within 0.13 mm of each other |
| Stepper drivers | No over-temperature, short or open-load flags |
| CAN bus | 1.46 million packets, zero errors |
| Temperatures after 40 minutes of testing | Mainboard MCU 45°C, toolhead board 45°C |
| Pressure advance | 0. Not set in config or slicer start code. |
| Neopixel | 11,278 "Neopixel update did not succeed" errors in 38 minutes. Cause not found. |
| Baseline print | 20 mm Voron cube, "Normal @Mercury One Plus", Overture PLA at 190/65°C: 1546 s of printing, 1937 mm of filament, 3×3 adaptive mesh with a 0.068 mm range. Owner's verdict: a very good print, with some surface noise left to tune out. |

### Bed shape

Full 7×7 mesh over 0,20 → 335,315, bed at about 31°C, 2 samples per point. Range 0.579 mm. Values in mm; the top row is the back of the bed.

```
-0.363 -0.251 -0.193 -0.230 -0.276 -0.327 -0.445
-0.325 -0.200 -0.168 -0.184 -0.209 -0.245 -0.330
-0.304 -0.203 -0.103 -0.051 -0.150 -0.181 -0.296
-0.312 -0.203 -0.115 +0.034 -0.112 -0.164 -0.280
-0.308 -0.186 -0.122 -0.013 -0.122 -0.194 -0.304
-0.366 -0.264 -0.142 -0.111 -0.133 -0.228 -0.364
-0.465 -0.263 -0.155 -0.150 -0.225 -0.349 -0.545
```

The plate is a dome: centre high, corners 0.4–0.5 mm low. A mesh at 50°C over a slightly smaller area gave the same shape with a 0.558 mm range.

### Resonance

| Sweep | Strongest peaks | Fitted |
|---|---|---|
| X | 63.4 and 52.5 Hz, nearly equal | mzv 58.2 Hz (saved: 59.2). Klipper's own pick is ei 66.2 Hz. |
| Y | 41.7 Hz | mzv 42.8 Hz (saved: 43.2). Klipper's own pick is zv 42.6 Hz. |
| Lower belt alone (`AXIS=1,1`) | 54.0 Hz | |
| Upper belt alone (`AXIS=1,-1`) | 63.3 Hz | |

The saved shaper values still fit. The lower belt resonates lower than the upper, which points to it being looser and matches the double peak on X.

## Tests of the 2026-11-22 config changes

Run on the machine the same day, after the changes were pulled. The pause and cancel tests used a no-extrusion air print at 190/50°C.

| Test | Result |
|---|---|
| Klipper start with the new config | Ready in about 6 s. No errors, no heating, no motion. |
| Full mesh over 0,20 → 335,315 | All 49 points probed, no misses |
| `PRINT_START` | Completed in about 3.5 minutes, adaptive mesh around the object |
| Pause | Parked at X390 Y385, 15 mm up. Pause LED effect on. |
| Paused past the idle timeout | At 10 minutes the hotend target went to 0. Bed stayed at 50°C, motors stayed on, relay stayed on, print stayed paused. |
| Resume | Reheated from 145°C to 190°C on its own in about 75 s, then continued printing. Active LED effect back on. |
| Cancel | Both heater targets to 0 at once, toolhead parked, cancel LED effect on |
| Power-off after cancel | Relay cut 6 minutes later, when the nozzle fell below 50°C |

A second round the same day covered the startup block and the bed-lowering on cancel:

| Test | Result |
|---|---|
| Power on with `STARTUP_IDLE` | Idle LED effect on and idle timer running within 3 s of ready. No heating, no motion, motors off. |
| Powered on and left unused | Idle timeout fired at 10 minutes. Relay cut about 35 s later. |
| Cancel with `lowerbedoncancel: 1` | After parking, the bed lowered from Z35 to Z365.75 in about 14 s. Heaters off, cancel LED effect on. |
| Power-off after that cancel | Relay cut 6 minutes later, when the nozzle fell below 50°C |

Not tested: the 4-hour give-up on a paused print, a queued job powering the printer on, and anything on the other three printers.

The test file `audit_airprint.gcode` is still in the host's gcode folder and can be reused. It shows in the print history as two cancelled jobs.

## Incidents

### 2026-11-22: bed driven into the toolhead by a missed probe

A full-bed `BED_MESH_CALIBRATE` was run as a flatness check. The seventh point put the probe at X377 Y20, off the plate. With no trigger and `position_min` at -15, the bed kept rising to Z -15. It knocked the probe off its magnets and the silicone sock off the hotend. Both were refitted by hand and later checks were normal.

Normal prints had never hit this because the adaptive mesh only probes around the part. A print using the right-hand 40 mm of the bed could have.

Fixed the same day: `position_min` -15 → -2 and `mesh_max` 377,350 → 335,315.

## Kalico migration

Status on 2026-11-22: prepared, not started. The live install is still Danger Klipper.

### What the dry run showed

Kalico `v2026.11.00` was set up in a separate clone and Python environment on the host, beside the live install.

| Check | Result |
|---|---|
| Python packages on the host's Python 3.9 | Install in about 30 s from prebuilt wheels. Kalico needs 3.9 or newer, so this host is at the minimum. |
| Firmware for all three MCUs, from the saved build configs | Builds with the existing toolchain. Each image uses under a third of flash. |
| This printer's `printer.cfg`, loaded offline | Clean. No errors or deprecation warnings. |
| Macros run offline: pause, resume, cancel, filament load and unload, `TEST_SPEED`, `HEAT_SOAK`, `WIPE_LINE`, LED and general commands | All complete |
| The baseline cube gcode, simulated start to finish | Completes. 1608 s simulated against 1601 s on the installed version. |

Probing cannot be checked offline: the simulated probe never triggers. Gantry tilt, Z calibration and meshing have to be proven on the machine, under the watchdog described below.

The offline harness is Klipper's batch mode: `klippy.py printer.cfg -i test.gcode -o out -d mcu.dict -d rpi=rpi.dict -d head0=head0.dict`, run against a copy of the config. Two things in the copy had to change for it to work: the toolhead MCU is given a `serial:` instead of `canbus_uuid:` (Kalico's CAN statistics module crashes on the simulated reply), and the probing flags in `INITIALIZATION` are set to 0. The same harness against the installed Danger Klipper hangs on some sequences after `G28`, so it is not a reliable A/B baseline. The live printer is.

### Behaviour that changes

- Homing no longer backs off after the final endstop touch. The axis stays on the switch.
- `SET_VELOCITY_LIMIT` no longer accepts `ACCEL_TO_DECEL`. `TEST_SPEED` now uses `MINIMUM_CRUISE_RATIO`, which both versions accept.
- `probe`, `homing`, `dockable_probe` and `z_calibration` were all reworked. No option this printer sets was removed, but probing is the main functional risk.
- Resonance tests shake harder, so the shaper should be re-run afterwards.
- `MIGRATE_TO_KALICO` only repoints the git remote and checks out `main`. Python packages, firmware and plugins are separate steps.

### Backup and rollback

A full copy of the live install was taken on 2026-11-22 into `~/backups/pre-kalico-2026-11-22/` on the host: the `klipper` tree, `klippy-env`, the config folder, the service files, the build configs, and firmware images built from the installed source for the mainboard and toolhead board. It sits on the same drive as the install, so it protects against a bad migration, not a dead drive.

To roll back:

1. Stop the `klipper` service.
2. Replace `~/klipper` and `~/klippy-env` with the copies from the backup.
3. Flash the mainboard from the restored tree: `./scripts/flash-sdcard.sh -f <backup>/firmware/danger-3fcbc0b4-btt-skr-mini-e3-v3.bin -b 500000 /dev/ttyAMA0 btt-skr-mini-e3-v3`.
4. Flash the toolhead board: `python3 ~/CanBoot/scripts/flash_can.py -i can0 -u <uuid from printer-thebrain.cfg> -f <backup>/firmware/danger-3fcbc0b4-mellow-fly-sht42.bin`.
5. Rebuild and flash the host MCU: `make clean KCONFIG_CONFIG=config.rpi && make flash KCONFIG_CONFIG=config.rpi`.
6. Start the `klipper` service.

### Plan

1. Move the checkout to the Kalico monthly tag, update the Python environment, reflash all three MCUs, and re-link `led_effect`.
2. Point Moonraker's Klipper updater at the stable channel.
3. Re-run the checks in "Measurements" and "Tests of the 2026-11-22 config changes", probing first and under the watchdog.
4. Print the baseline cube again and compare.

## Open items

Ordered roughly by value. None of these are done.

**First-layer consistency**

- `PRINT_START` heats the nozzle to full print temperature before `CALIBRATE_Z`, and the nozzle touch is the noisiest part of that measurement. Probe at about 150°C, then heat.
- `switch_offset` was hand-tuned about a dozen times between October 2024 and January 2026, in a band from 0.485 to 0.585. Back-to-back calibrations differ by about 0.024 mm, so steps of 0.01–0.02 are inside the noise.
- Z homes at 20 mm/s onto the nozzle pin. Slower would likely tighten the 0.019 mm spread.
- Mesh fade is off, so the 0.58 mm dome is carried through the full height of every print. `fade_start: 1`, `fade_end: 10` would stop that.
- `z_tilt` `z_positions` are -30 and 380 (410 mm apart). The 85%-per-pass convergence implies the real pivots are about 470–490 mm apart. Measure them; correct values should converge in two passes.

**Motion**

- Even out belt tension (lower belt 54 Hz, upper 63 Hz), then re-run the shaper.
- Calibrate pressure advance.
- The slicer profile prints at 750–1500 mm/s² against a machine limit of 5000. The shaper results support about 5000.
- Neopixel update errors. Try a newer `led_effect` and a lower frame rate than 24 fps.

**Macros**

- Remove the dead `COMPUTE_MESH_PARAMETERS` call from `PRINT_START` and `DOCK_PROBE` from `[z_calibration] end_gcode`. Both only print "Unknown command". The probe docks itself after `CALIBRATE_Z`.
- `LOAD_FILAMENT` extrudes 75 mm before waiting for temperature, so it fails below 175°C.
- The frame file rewrites all saved variables at every boot, and `_Printer_Variables` duplicates most of them. One mechanism should go.
- `variable_use_custom_pos.` has a stray period, and `max_extrude_only_distance` is set twice in the toolhead file.

**Host**

- Move to Kalico and reflash the three MCUs. Update Moonraker, `led_effect` and crowsnest.
- The host's `printer.cfg` and `moonraker.conf` are not in version control.
- The Pi runs at about 65°C with the printer off and 47°C with it on, so it depends on a printer-powered fan.
- Unused files on the host: old `klicky-*.cfg` files and dated `printer-*.cfg` backups.

**Labels to correct**

- Board header in `printer-thebrain.cfg`.
- `toolhead-afterburner.cfg` does not mention the Fly SHT42, and its "For Mini Afterburner" comment sits on the full-size 50:17 ratio.

## Working on this printer safely

- Any probing outside the proven area should run under a watchdog that polls `motion_report.live_position` and sends Moonraker's emergency stop if Z drops below about 5 mm during a probe move. Probes normally trigger near Z 8.
- Check that the bed is clear before homing. Raise the bed to about Z45 and the camera can see the plate.
- A no-extrusion "air print" is a cheap way to exercise `PRINT_START`, pause, resume and cancel: define one exclude-object near the bed centre, call `print_start`, then loop a square at Z20.

## History

- 376 jobs, about 690 print hours and 5.7 km of filament as of 2026-11-22.
- Last print before the 2026-11-22 session: 2026-01-09.
- `retired-toolhead-stealthburner.cfg` is this printer's previous toolhead.
