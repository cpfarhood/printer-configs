# thebrain

An Ender 5 Plus converted to a ZeroG Mercury One (CoreXY), then customised further. Moved from Danger Klipper to Kalico on 2026-11-22. Last inspected live on 2026-11-25.

**Status, 2026-11-24:** in service. New belts and adjusted, lubricated rails went on that day after a Y-axis fault on 2026-11-23, the shapers were re-saved for the new belts, and the baseline cube printed with the same time and filament as on 2026-11-22. See "Incidents".

**Host reinstalled, 2026-11-25:** the Pi's boot drive failed that evening and the host was rebuilt on a blank MainsailOS image. Everything that lived only on the old drive is gone. All three boards connect and the saved values are back in place, but nothing has homed, probed or printed on the new host yet. See "Rebuilding the host".

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
| Z endstop | Nozzle-touch pin beside the bed at X388 Y81, used by `z_calibration` |
| X endstop | On the toolhead board, read over CAN |
| Frame sensor | Thermistor on the mainboard's TH0 port, feeding `z_thermal_adjust` |
| Accelerometer | ADXL345 on the toolhead board |
| Lighting | 25-LED neopixel strip (`main`) plus a white LED channel |
| Camera | Logitech C930e at the front, looking back across the gantry. It sees the bed only when the bed is within roughly 60 mm of the nozzle. |
| Host | Raspberry Pi 4, booting from a USB flash drive, wired Ethernet. Reinstalled 2026-11-25. |
| Screen | 800×480 touchscreen on the Pi's display connector, running KlipperScreen with `screen-mercury_one_plus.cfg` |
| Power | Moonraker switches the printer's supply through a GPIO relay. The Pi stays on. |

### Belts

- Left rear motor = `stepper_x` = **lower** belt.
- Right rear motor = `stepper_y` = **upper** belt.

Confirmed 2026-11-22 by camera: a move along +X+Y turns only `stepper_x`, and the lower rear belt span and the left motor were what moved. A move along -X+Y moved the upper span and the right motor.

## Software

- Host OS: MainsailOS on 64-bit Debian 13 with Python 3.13, since the reinstall on 2026-11-25. Before that it was 32-bit Raspbian 11 with Python 3.9.
- Klipper fork: Kalico `v2026.11.00` since 2026-11-22, on the host and on all three MCUs. The checkout sits on the monthly release tag and Moonraker's Klipper updater is on the `stable` channel, so it follows monthly tags, not every commit. Before that it ran Danger Klipper `v0.12.0-526` (December 2024) on a repository name that no longer received updates.
- `dockable_probe` and `z_calibration` are bundled with Kalico. `led_effect` is a separate plugin, linked into `klippy/plugins/` (Kalico refuses to start if the same module is in both `extras` and `plugins`).
- Option and command names must be checked against Kalico's docs, not mainline Klipper's.
- Resonance tools: on the old 32-bit install they needed the system package `libopenblas0`, which was missing from November 2024 until 2026-11-22. The 64-bit install does not need it; numpy loads in Klipper's Python environment as installed. `TEST_RESONANCES` has not been run on the new host. matplotlib is not installed, so `calibrate_shaper.py` cannot draw graphs.

## Config

The host's `printer.cfg` includes, in order:

1. `printer-common.cfg`
2. `printer-mercury_one_plus.cfg`
3. `toolhead-afterburner.cfg`
4. `printer-thebrain.cfg`
5. `steppers-omc_17hm19-2004s.cfg`
6. `mainsail.cfg`
7. `macros-common.cfg`, `macro-test_speed.cfg`, `macro-heat_soak.cfg`, `macro-wipe_line.cfg`, `macros-lazy_homing.cfg`, `macros-printer_power.cfg`

`macro-adaptive_bed_mesh.cfg` is **not** included. The mesh comes from Klipper's own `BED_MESH_CALIBRATE ADAPTIVE=1`. Until 2026-11-25 `PRINT_START` also called that file's `COMPUTE_MESH_PARAMETERS`, and `[z_calibration]` ran a non-existent `DOCK_PROBE` as its `end_gcode`; each printed "Unknown command" at every print start and did nothing else. Both lines are removed and the fix is loaded on the host. `[z_calibration]` has no `end_gcode` on purpose: `PRINT_START` needs the probe to stay attached for the mesh that follows `CALIBRATE_Z`, and detaches it afterwards.

### Values that live only on the host

These sit in the `SAVE_CONFIG` block of the host's `printer.cfg` and are not in any file in this repo. Copied 2026-11-22. When the host was rebuilt on 2026-11-25 this table was the only surviving copy, and the block was typed back in from it. Update the table whenever one of these is re-saved.

| Section | Values |
|---|---|
| `[extruder]` PID | kp 19.970, ki 1.022, kd 97.602 |
| `[heater_bed]` PID | kp 56.711, ki 0.863, kd 931.480 |
| `[input_shaper]` | X mzv 66.0 Hz, Y mzv 46.0 Hz (59.2 and 43.2 Hz until 2026-11-24, on the old belts) |
| `[stepper_z]` | `position_endstop` 7.846 |
| `[dockable_probe]` | `z_offset` 8.110 |
| `[bed_mesh default]` | A stale 3×3 adaptive mesh over 155–214 mm, not used by prints. Not carried over in the rebuild. |

## Coordinates

| Thing | Position |
|---|---|
| Travel | X 0…395, Y 0…390, Z -2…385 |
| Y endstop | Triggers at 375. Y can travel on to 390 to reach the probe dock. |
| Probe dock | X210.5 Y384, approach from Y360, detach by sliding to X235 |
| Probe offset from nozzle | X -1.5, Y +19.75 |
| Z endstop pin | Nozzle at X388 Y81; probe switch body at X383 Y64. Both were moved forward on 2026-11-25 after the owner watched where they landed on the pin: the nozzle from Y85 in two 2 mm steps, the switch from Y65 by 1 mm. |
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
| Retraction | 1.5 mm at 60 mm/s, 0.3 mm Z lift | Done by the slicer. The unused `[firmware_retraction]` section was removed on 2026-11-25, so `G10`/`G11` no longer exist on this printer. |
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

These figures are for the old belts. The new belts' figures are under "2026-11-23 and 24: Y fault, new belts".

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

The test file `audit_airprint.gcode` was in the host's gcode folder until the drive failed on 2026-11-25. It shows in the print history as two cancelled jobs.

## Incidents

### 2026-11-25: boot drive failed, host reinstalled

Between prints that evening Moonraker answered one request and then stopped answering, while the Pi still answered ping. The owner found the filesystem on the boot drive broken and reinstalled the host from a blank MainsailOS image. Nothing was printing and the printer was powered off at the time.

Lost with the drive: the host's `printer.cfg`, `moonraker.conf`, `variables.cfg` and camera config, the Kalico checkout and its Python environment, the `led_effect` plugin, KlipperScreen, the firmware build configs and `~/flashmcu.sh`, the CanBoot flashing scripts, the pre-Kalico backup, the print history, and every uploaded gcode file.

Not lost: the firmware on the three boards, everything in this repo, and the saved calibration values, because they had been copied into the table under "Config".

The rebuild is described under "Rebuilding the host".

### 2026-11-23 and 24: Y fault, new belts

**What happened.** On 2026-11-23 the owner tightened the lower belt to even out the two belts. The next sweeps showed Y no longer following moves smaller than about ±0.1 mm, and its 42 Hz resonance gone; X was unaffected. Loosening and re-tightening both belts changed nothing. By hand the owner found a tight-loose pulse about once per motor turn and the gantry 0.5–1 mm out of square at the back. The printer came back on 2026-11-24 with new belts and adjusted, lubricated rails.

Share of each commanded Y move that reached the toolhead at bed centre, at the configured `accel_per_hz: 50` (1.0 = follows). The first two columns came from an older script, so compare them loosely.

| Size of the move | 2026-11-22 | 2026-11-23, fault | 2026-11-24, new belts |
|---|---|---|---|
| ±0.115 mm (11 Hz) | 1.15 | 0.86 | 1.40 |
| ±0.060 mm (21 Hz) | 0.78 | 0.30 | 0.78 |
| ±0.042 mm (30 Hz) | 1.20 | 0.10 | 0.72 |
| ±0.036 mm (35 Hz) | not recorded | not recorded | 0.20 |
| ±0.032 mm (40 Hz) | 4.90 (the resonance) | 0.10 | 0.20 |

- **Y follows again down to about ±0.04 mm.** Below that it does not. This is normal for these machines and is explained in [mercury-one-motion.md](mercury-one-motion.md). A second session the same day repeated point for point.
- **What fixed it is not known in detail.** Belts, rail adjustment and lubrication were all changed together.
- **Resonances on the new belts:** Y one peak at 45.2 Hz (41.7 Hz on the old belts), measured with `ACCEL_PER_HZ=100`. X a main peak at 65.4 Hz with a small one at 56.1 Hz.
- **The belts are balanced.** Diagonal sweeps at `ACCEL_PER_HZ=120`: `AXIS=1,1` peaks at 45.3, 54.7 and 67.2 Hz; `AXIS=1,-1` at 45.2, 54.6 and 65.5 Hz. On the old belts the two read 54.0 and 63.3 Hz.
- **Shapers saved:** X mzv 66.0 Hz and Y mzv 46.0 Hz, each leaving 0.0% residual vibration on both of the day's sweeps. The old values still fitted (59.2 Hz left 1.1–1.5%, 43.2 Hz left 0.0%), so the gain is less smoothing. Previous file on the host: `printer-20261124_231109.cfg`.
- **Fast-move test, no filament:** 3 loops at 150 mm/s and 2500 mm/s², then 5 loops at 300 mm/s and 5000 mm/s², re-homing after each stage. Home moved by 0.07 mm of belt at most (one tooth is 2 mm): no skipped teeth.
- **First start sequence after the rebuild:** Z homed on the pin, the tilt adjust converged in two or three passes (first correction 1.8 mm), and Z calibration and the adaptive mesh ran, on three print starts.

**Baseline cube, 2026-11-24.** `cube_kalico.gcode`, the file from 2026-11-22: completed in 1554 s of printing with 1937 mm of filament, the same as on 2026-11-22. Owner's verdict while it printed: looks good. Main board 42°C and toolhead board 55°C at the end; no communication errors.

**First real part, same evening.** A 200 mm PETG spool adapter sent by the owner (230/75°C, 27 layers) completed in 1647 s of printing against the slicer's 1622 s estimate, with 2310 mm of filament. The start sequence levelled the bed in three passes, calibrated Z (offset 0.482 mm) and probed a 5×5 adaptive mesh over 85–285 mm on both axes. No errors; main board 42°C.

Two earlier attempts that evening were cancelled by the owner because the extruder was not gripping the filament. The cause was the grub screw on the extruder's drive gear, which the owner re-tightened. Spare extruder parts are on hand.

Raw sweeps are on the host in `~/resonance-2026-11-24/` and `~/resonance-2026-11-24b/`.

### 2026-11-22: false emergency stop during testing

A test watchdog read Kalico's momentary all-zero live position as the bed rising and sent an emergency stop. Nothing was moving toward the nozzle. Klipper shut down at the start of a probe command; a firmware restart and re-home recovered it. The watchdog rule was corrected the same evening.

### 2026-11-22: bed driven into the toolhead by a missed probe

A full-bed `BED_MESH_CALIBRATE` was run as a flatness check. The seventh point put the probe at X377 Y20, off the plate. With no trigger and `position_min` at -15, the bed kept rising to Z -15. It knocked the probe off its magnets and the silicone sock off the hotend. Both were refitted by hand and later checks were normal.

Normal prints had never hit this because the adaptive mesh only probes around the part. A print using the right-hand 40 mm of the bed could have.

Fixed the same day: `position_min` -15 → -2 and `mesh_max` 377,350 → 335,315.

## Kalico migration

Done on 2026-11-22. The sections below record the preparation, what was done, and what the checks afterwards showed.

### What the dry run showed

Kalico `v2026.11.00` was set up in a separate clone and Python environment on the host, beside the live install.

| Check | Result |
|---|---|
| Python packages on the host's Python 3.9 | Install in about 30 s from prebuilt wheels. Kalico needs 3.9 or newer, so this host is at the minimum. |
| Firmware for all three MCUs, from the saved build configs | Builds with the existing toolchain. Each image uses under a third of flash. |
| This printer's `printer.cfg`, loaded offline | Loads without errors. The offline log showed no warnings, but the live printer reports one: `off_below` in `[fan]` is deprecated in favour of `min_power`. That was changed to `min_power` on 2026-11-25. |
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

A full copy of the live install was taken on 2026-11-22 into `~/backups/pre-kalico-2026-11-22/` on the host: the `klipper` tree, `klippy-env`, the config folder, the service files, the build configs, and firmware images built from the installed source for the mainboard and toolhead board. It sat on the same drive as the install, so it protected against a bad migration, not a dead drive. **It was lost when that drive failed on 2026-11-25.** There is no longer a Danger Klipper install or firmware image to roll back to; the steps below are kept as a record.

The rollback was:

1. Stop the `klipper` service.
2. Replace `~/klipper` and `~/klippy-env` with the copies from the backup.
3. Flash the mainboard from the restored tree: `./scripts/flash-sdcard.sh -f <backup>/firmware/danger-3fcbc0b4-btt-skr-mini-e3-v3.bin -b 500000 /dev/ttyAMA0 btt-skr-mini-e3-v3`.
4. Flash the toolhead board: `python3 ~/CanBoot/scripts/flash_can.py -i can0 -u <uuid from printer-thebrain.cfg> -f <backup>/firmware/danger-3fcbc0b4-mellow-fly-sht42.bin`.
5. Rebuild and flash the host MCU: `make clean KCONFIG_CONFIG=config.rpi && make flash KCONFIG_CONFIG=config.rpi`.
6. Start the `klipper` service.

### What was done

1. `~/klipper` repointed at the Kalico repository and checked out at tag `v2026.11.00`. The old `master` branch is still in the checkout.
2. Python packages updated from Kalico's requirements file.
3. `led_effect` moved from `klippy/extras/` to `klippy/plugins/`.
4. Firmware built from the saved build configs and flashed: host MCU with `scripts/flash-linux.sh`, toolhead board over CAN with `flash_can.py`, mainboard with `scripts/flash-sdcard.sh`. All three verified first time.
5. `[update_manager klipper]` with `channel: stable` added to the host's `moonraker.conf`.

### Checks afterwards

| Check | Kalico | Danger Klipper, same day |
|---|---|---|
| Start | Ready, all three MCUs on `v2026.11.00` | |
| Probe repeatability, bed centre, 20 samples | Spread 0.016 mm, σ 0.003 mm | 0.015 mm, σ 0.003 mm |
| Gantry tilt | Converges in 3–4 passes | 4–5 passes |
| Z calibration, three runs, cold nozzle | 0.202, 0.204, 0.194 mm. Nozzle touch varied 0.006 mm. | Hot nozzle: 0.210, 0.221, 0.234 mm. Nozzle touch varied 0.018 mm. |
| Z homing, nozzle on the endstop after each of 12 homes | Within 0.05 mm | Six homes within 0.019 mm, measured a different way |
| X/Y homing | About 0.01 mm | About 0.01 mm |
| `PRINT_START`, pause, resume, cancel (air print) | All work. Bed lowers on cancel. | Same |
| Baseline cube | 1554 s of printing, 1937 mm of filament. Owner's verdict: looks good. | 1546 s, 1937 mm. Owner's verdict: a very good print, with some surface noise left to tune out. |
| Timing or communication errors during the cube | None | None |
| Power-off after a finished print | Relay cut about 5.5 minutes after the cube ended, once the bed fell below 50°C | Same behaviour |
| Neopixel update errors | Still present: about 8,000 in 46 minutes | 11,278 in 38 minutes |
| Unused printer powers itself off | Relay cut 632 s after power-on with nothing touched | About 630 s |
| Paused past the idle timeout | Hotend target to 0 at 10 minutes; bed held at 50°C, motors and relay on. Resume reheated from 157°C to 190°C and carried on. | Same |
| Full-bed mesh, five samples per point | All 49 points, 0.511 mm range, four tolerance retries | 0.579 mm range with two samples per point |
| Resonance, X | mzv 58.6 Hz; peaks at 62.1 and 52.8 Hz | mzv 58.2 Hz; peaks at 63.4 and 52.5 Hz |
| Resonance, Y | mzv 43.2 Hz; peak at 41.9 Hz | mzv 42.8 Hz; peak at 41.7 Hz |
| Belts alone | Lower about 54 Hz, upper about 64 Hz | Lower 54.0 Hz, upper 63.3 Hz |

Every check from the Danger Klipper baseline has now been repeated on Kalico. The saved input shaper (X mzv 59.2 Hz, Y mzv 43.2 Hz) still fits and was left unchanged. Still untested on either version: the 4-hour give-up on a paused print, and a queued job powering the printer on.

### Things to know from the day

- **Kalico's live position can read zero.** `motion_report.live_position` returns `0, 0, 0` for an instant between moves. A watchdog that acted on a single reading sent a false emergency stop. Ignore all-zero readings and require two low readings in a row.
- **`GET_POSITION` is not a homing-repeatability measure.** On Kalico the `stepper:` and `mcu:` Z values carry an offset that changes between homes while the toolhead position stays correct. To check Z homing, run `PROBE_Z_ACCURACY` at the endstop after each home and compare where the nozzle touches.
- **The offline harness's fake probe never triggers**, and Kalico's CAN statistics module crashes on its simulated reply. Both are harness artifacts, not printer faults.

## Rebuilding the host

What a blank MainsailOS image needed before this printer worked again, as done on 2026-11-25. The boards were not reflashed: the host was put on the same Kalico tag their firmware already carried.

1. **Kalico.** Point `~/klipper` at the Kalico repository, check out the tag the boards run (`v2026.11.00` then), and install `scripts/klippy-requirements.txt` into `~/klippy-env`. It installs cleanly on Python 3.13.
2. **Host MCU.** Build with a config that holds only `CONFIG_MACH_LINUX=y` (`make olddefconfig`, then `make`), copy `out/klipper.elf` to `/usr/local/bin/klipper_mcu`, install and enable `scripts/klipper-mcu.service`, and add the login user to the `tty` group.
3. **`led_effect`.** Clone `julianschill/klipper-led_effect` and link `src/led_effect.py` into `~/klipper/klippy/plugins/`. Do not use its install script: it links into `extras`.
4. **Config repos.** Clone this repo and `printer-moonraker-configs` into `~/printer_data/config/`.
5. **`printer.cfg`.** The include list under "Config", in that order, followed by a `SAVE_CONFIG` block holding the values in the table there.
6. **`moonraker.conf`.** Add `[include printer-moonraker-configs/moonraker-mercury_one_plus.cfg]` (power relay, webcam, job queue, KlipperScreen and theme updaters) and `[update_manager klipper]` with `channel: stable`.
7. **Mainsail theme.** Clone `ZeroGDesign/zerog-mainsail-theme` into the home folder and run its `install.sh`, which links `.theme` into the config folder.
8. **Camera.** In `crowsnest.conf` set `mode: camera-streamer` and `resolution: 1920x1080`. The stock file is ustreamer at 640x480, and the shared Moonraker webcam entry expects camera-streamer's WebRTC stream.
9. **KlipperScreen.** Clone it and run `scripts/KlipperScreen-install.sh` with the X11 backend, declining the NetworkManager change. `KlipperScreen.conf` in the config folder holds one line that includes `printer-configs/screen-mercury_one_plus.cfg`.

Things the image already had: the UART the mainboard uses (`enable_uart=1`, Bluetooth overlay disabled, no serial console), the CAN interface at 1 Mbit, and the ARM toolchain. `sudo` asks for the account password on this image.

Checked afterwards, printer powered on, nothing moved or heated:

| Check | Result |
|---|---|
| Klipper state | Ready. Mainboard, toolhead board and host MCU all report Kalico `v2026.11.00`. |
| Saved values | Both PIDs, shaper (mzv 66.0 / 46.0 Hz), Z endstop 7.846 and probe offset 8.110 load as in the table |
| Config warnings | Only the known `off_below` deprecation |
| Temperatures | Nozzle 20°C, bed 22°C, mainboard 29°C: all plausible for a cold printer |
| Start-up variables | `variables.cfg` rewritten by the `INITIALIZATION` block on first start |
| Camera | 1920x1080 snapshot works |
| Moonraker | Starts with no warnings; the power relay switches the printer on and off |

Not yet checked on the new host: homing, probing, `PRINT_START`, a print, resonance tests and power-off after a print. KlipperScreen's log shows it running on the panel at 800x480 and connected to Moonraker, but nobody has looked at or touched the screen yet.

## Open items

Ordered roughly by value. None of these are done.

**Probing**

- Probing along the front edge reads low on the first touch and creeps up over the next few. Five samples per point, bed cold, later on 2026-11-22: front-left corner (0,20) spread 0.076 mm with the first sample 0.068 mm low; front-right corner 0.030 mm; mid-front 0.022 mm; mid-left 0.012 mm; centre and all four back positions 0.002–0.006 mm. It looks like the bed giving slightly under the probe at the front. The config rejects a point whose samples differ by more than 0.033 mm and retries, so a full-bed mesh completes but needs retries at the front-left. Adaptive meshes around a part in the middle of the bed are unaffected.
- Earlier the same evening the front-left corner was far worse: spreads of 0.13–0.27 mm and "Probe triggered prior to movement" at (5,20) and (0,25), enough to abort a full mesh at its first point. That cleared after the second test cube was taken off the plate. A plate not sitting flat at that corner after the first cube was removed is the likely cause, but that was not confirmed, and the same mesh had passed that afternoon on Danger Klipper. If full meshes start failing at the front-left again, check the plate seating first.

**First-layer consistency**

- `PRINT_START` heats the nozzle to full print temperature before `CALIBRATE_Z`, and the nozzle touch is the noisiest part of that measurement. With a cold nozzle the touch varied 0.006 mm over three runs, against 0.018 mm hot. Probe at about 150°C, then heat.
- `switch_offset` was hand-tuned about a dozen times between October 2024 and January 2026, in a band from 0.485 to 0.585. Back-to-back calibrations differ by about 0.024 mm, so steps of 0.01–0.02 are inside the noise. On 2026-11-24 three print starts within 40 minutes gave offsets of 0.477, 0.432 and 0.458 mm, a spread of 0.045 mm. On 2026-11-25 it went from 0.525 to 0.515 after the owner judged a first layer on the textured plate about 0.02 mm too close; that was the first print after the nozzle's pin position moved to Y81, and one print is within this scatter, so only half the step was taken. The owner took the other half later the same evening: 0.505. The smooth-plate adjustment in the OrcaSlicer start G-code was reduced by the same amounts (0.03 to 0.01 mm), so the smooth plate's height is unchanged.
- **Moving the touch points on the pin changed what the calibration measures (found 2026-11-26).** The measured `switch − nozzle` distance rose by 0.14 mm when the nozzle and switch positions moved forward on 2026-11-25, while the probe's reading on the bed stayed where it was. `switch_offset` had been tuned by eye against the old reading, so every start after the move put the nozzle about 0.12 mm closer to the bed than before, on a 0.2 mm first layer. Only 0.02 mm was taken off that evening. A full plate with a 255 cm² solid floor came out with a flare about 0.8 mm out and 0.8 mm tall (the three solid bottom layers plus the first sparse one) and was scrapped.

  | Starts | `switch − nozzle` | Probe on bed | `probe − (switch − nozzle)` |
  |---|---|---|---|
  | Six before the move (2026-11-24 and 11-25, `switch_offset` 0.525, first layers judged good) | 7.443–7.488, mean 7.462 | 8.427–8.463 | 0.958–1.008 |
  | Three after the move (2026-11-25 evening) | 7.576–7.620, mean 7.604 | 8.458–8.493 | 0.841–0.916 |

  `switch_offset` 0.385 puts `switch − nozzle + switch_offset` back at the value it had before the move (7.462 + 0.525 = 7.987). Which touch position reads the true distance is not known, and the owner's by-eye estimate at the time was 0.02 mm too close, not 0.12. So 0.385 is a calculated value: confirm it with a measured single-layer print before a real plate. The old 0.485–0.585 band belongs to the old touch positions and no longer applies.
- **Start-to-start scatter.** The last column above should be a constant of the hardware. It spread 0.050 mm over the six earlier starts and 0.075 mm over the three later ones, so the first-layer height differs from one start to the next by several hundredths whatever `switch_offset` is. Which of the three readings wanders has not been measured.
- **Check `switch − nozzle` against the recent mean, not a wide band.** The 0.14 mm jump was logged on the first start after the move and passed a 7.30–7.80 mm gate unnoticed. After any change to the pin positions, the probe or the nozzle, compare the new reading with the previous ones before printing.
- Z homes at 20 mm/s onto the nozzle pin. Slower would likely tighten the 0.019 mm spread.
- Mesh fade is off, so the 0.58 mm dome is carried through the full height of every print. `fade_start: 1`, `fade_end: 10` would stop that.
- `z_tilt` `z_positions` are -30 and 380 (410 mm apart). The 85%-per-pass convergence implies the real pivots are about 470–490 mm apart. Measure them; correct values should converge in two passes.

**Motion**

- Y ignores moves under about ±0.04 mm. That is this machine's normal and it prints well; re-run the 40-second Y sweep after any belt or rail work and compare (see [mercury-one-motion.md](mercury-one-motion.md)).
- A pulley or idler ran off-centre on the old belts (tension pulsed once per motor turn). Not re-checked since the new belts.
- The extruder's drive-gear grub screw worked loose on 2026-11-24. Check it if extrusion thins out.
- Calibrate pressure advance.
- The slicer profile prints at 750–1500 mm/s² against a machine limit of 5000. The shaper results support about 5000.
- Neopixel update errors. Try a newer `led_effect` and a lower frame rate than 24 fps.

**Macros**

- `CALIBRATE_Z` run by hand leaves the probe attached; only `PRINT_START` detaches it.
- `LOAD_FILAMENT` extrudes 75 mm before waiting for temperature, so it fails below 175°C.
- `PRINT_START` and `PRINT_END` each retract 10 mm, and Mainsail retracts 5 mm on a cancel. That is a lot for a direct-drive extruder and can pull molten plastic into the cold part of the hotend, more so after cancelled prints in a row. Consider 1–2 mm.
- The frame file rewrites all saved variables at every boot, and `_Printer_Variables` duplicates most of them. One mechanism should go.
- `variable_use_custom_pos.` has a stray period, and `max_extrude_only_distance` is set twice in the toolhead file.

**Host**

- Update Moonraker, `led_effect` and crowsnest. Klipper itself is now current.
- The host's `printer.cfg` and `moonraker.conf` are not in version control.
- **No copy of the host exists.** The drive failure on 2026-11-25 cost a hand rebuild, and the calibration values survived only because they had been copied into this file. Put the host's `printer.cfg` and `moonraker.conf` somewhere that is backed up (the Moonraker one could join `printer-moonraker-configs`), or image the drive once the rebuild has proven itself.
- **The firmware build configs are gone**, along with `~/flashmcu.sh` and the CanBoot scripts. They are needed before the next Kalico update, which has to reflash all three MCUs. What the boards report about their current builds: mainboard STM32G0B1, external crystal, serial on PA3/PA2 (USART2) at 500000 baud; toolhead board STM32F072, external crystal, CAN on PB8/PB9 at 1 Mbit, behind a CanBoot bootloader. Crystal frequencies and bootloader offsets are not reported and must be confirmed before building.
- Moonraker's updater does not track the Kalico checkout on the reinstalled host. It reports the repository as invalid ("Detached HEAD", "Unofficial remote url") and shows version numbers left over from the image's mainline Klipper. Moonraker v0.10.0 rejects `origin` and `primary_branch` in `[update_manager klipper]`. Printing is unaffected; Kalico updates are a manual job here anyway because of the reflash.
- The MainsailOS image dates from May 2026, so Moonraker, Mainsail and crowsnest on the new host are behind again.
- The camera runs on automatic exposure, which meters the dark bed and washes out the toolhead. A manual setting (exposure 156, gain 45, with the part light at 100%) looked better on 2026-11-25 but was applied live only and went with the old drive. It would go on a `v4l2ctl:` line in `crowsnest.conf`.
- The Pi runs at about 65°C with the printer off and 47°C with it on, so it depends on a printer-powered fan.

**Labels to correct**

- Board header in `printer-thebrain.cfg`.
- `toolhead-afterburner.cfg` does not mention the Fly SHT42, and its "For Mini Afterburner" comment sits on the full-size 50:17 ratio.

## Working on this printer safely

- Any probing outside the proven area should run under a watchdog that polls `motion_report.live_position` and sends Moonraker's emergency stop if Z drops below about 5 mm during a probe move. Probes normally trigger near Z 8. The watchdog must ignore all-zero readings and wait for two low readings in a row; see "Things to know from the day".
- `dockable_probe.last_status` reads 0 (unknown) on this machine during probing and printing alike, so a watchdog cannot use it to tell the two apart. Use the file position: it stays on the `PRINT_START` line until probing is over.
- Check that the bed is clear before homing. Raise the bed to about Z45 and the camera can see the plate.
- A no-extrusion "air print" is a cheap way to exercise `PRINT_START`, pause, resume and cancel: define one exclude-object near the bed centre, call `print_start`, then loop a square at Z20.

## History

- 376 jobs, about 690 print hours and 5.7 km of filament as of 2026-11-22. The host's print history was lost with the drive on 2026-11-25 and counts from zero again.
- Last print before the 2026-11-22 session: 2026-01-09.
- `retired-toolhead-stealthburner.cfg` is this printer's previous toolhead.
