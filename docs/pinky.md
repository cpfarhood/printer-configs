# pinky

ZeroG Mercury One conversion of an Ender 5 Plus. Sibling of [thebrain](thebrain.md) and built from the same shared files.

**Status, 2026-11-23:** powered on for the first time since March 2025, probe replaced, and moved to Kalico the same day. Homing, probing, Z calibration and a no-extrusion print all pass. It has not printed with filament yet. An intermittent homing timeout stopped one print start before the move; see "Open items".

## Host and software

| Part | What is there |
|---|---|
| Host | Raspberry Pi 4 Model B rev 1.4, 4 GB, Raspbian 11 with a 64-bit kernel and 32-bit userland, Python 3.9.2, booting from a USB drive. Wired and wireless interfaces are both up. |
| Klipper | Kalico `v2026.11.00` since 2026-11-23, checked out at the release tag. Before that, Danger Klipper `v0.12.0-526-g3fcbc0b4` (December 2024), the same commit thebrain ran before its own move. |
| Boards | Main board BTT SKR Mini E3 V3 (STM32G0B1) on the Pi's serial port at 500000 baud. Toolhead board Mellow Fly SHT42 (STM32F072) on CAN at 1 Mbit, with a Katapult (CanBoot) bootloader. Both confirmed by the saved firmware build configs and by flashing. All three MCUs run `v2026.11.00`. |
| Moonraker | `v0.9.3-62` |
| Plugins | `led_effect` (October 2024 checkout) linked into `klippy/plugins`. `z_calibration` and `dockable_probe` are part of Kalico. |
| Mainsail macros | `mainsail.cfg` dated 2024-01-17. It has the `user_pause_macro` / `user_cancel_macro` hooks and the `idle_state` variable the shared macros rely on. |
| Other services | KlipperScreen, crowsnest, sonar, Obico, Mobileraker companion |
| Power | Same arrangement as thebrain: Moonraker switches the printer through a GPIO relay, off at boot, with `klipper` as a bound service. |

Moonraker took about four and a half minutes after boot to start answering. Until then Mainsail loads but shows a gateway error.

### This checkout is behind

The config repo on the host is at `c91d0d1` (2025-02-26), 23 commits behind `master`. **None of the 2026-11-22 changes are on the machine yet.** Neither `moonraker.conf` nor its shared include has an `update_manager` entry for this repo, so it only updates by a manual `git pull`.

Until it is pulled, pinky still has:

- `stepper_z position_min: -15`, the setting that let a missed probe drive thebrain's bed into its toolhead.
- `bed_mesh mesh_max: 377,350`.
- The old `PAUSE`, `RESUME` and `CANCEL_PRINT` overrides in `macros-common.cfg`, and the old idle timeout that abandons a paused print.
- Hotend `max_temp` 310.

After a pull it gets the fixed versions of all of these. They were proven on thebrain's bed and macros, not on pinky's.

## Config

`printer.cfg` on the host includes, in order:

- `printer-common.cfg`, `printer-mercury_one_plus.cfg`, `toolhead-afterburner.cfg`, `printer-pinky.cfg`, `steppers-ldo_42sth48-2504ac.cfg`
- `mainsail.cfg`
- `macros-common.cfg`, `macro-test_speed.cfg`, `macro-heat_soak.cfg`, `macro-wipe_line.cfg`, `macros-lazy_homing.cfg`, `macros-printer_power.cfg`

The stepper file is the main hardware difference from thebrain:

| | pinky | thebrain |
|---|---|---|
| X/Y/Z motors | 1.8° (200 steps per turn), 2.5 A rated. The file's comments name Fabreek Honey Badger 42HS48-25044A on X/Y and LDO 42STH48-2504AC on Z. | 0.9° (400 steps per turn), 2.0 A rated |
| Microsteps | 64 | 32 |
| Resolution | 320 microsteps per mm | 320 microsteps per mm |
| Run current | 1.35 A on all four, about half the motors' rating (the TMC2209 limit) | 1.35 A on X/Y, 1.25 A on Z |

- The owner confirmed the 200-step motors on 2026-11-23. Because the motors differ, thebrain's resonance figures are not a baseline for pinky; measure pinky on its own.
- The file has a second `[tmc2209 stepper_y]` header that sets `driver_SGTHRS: 110`. Y homes on a physical switch, so it does nothing.
- The extruder is the same on both: 0.9°, 32 microsteps.

### Values that live only on the host

| Setting | Value |
|---|---|
| `[extruder]` PID | kp 18.115, ki 0.693, kd 118.312 |
| `[heater_bed]` PID | kp 50.732, ki 0.505, kd 1274.653 |
| `[input_shaper]` | X mzv 63.2 Hz, Y mzv 42.0 Hz (104.4 Hz until 2026-11-24) |
| `[stepper_z] position_endstop` | 4.796 |
| `[dockable_probe] z_offset` | 7.260 |
| Saved `default` bed mesh | 7×7 over 20…330, range about 0.30 mm, centre high |

The old Y value was not a real resonance. See "Resonance, 2026-11-23": at the configured test strength Y does not follow the small moves a sweep uses, so that fit had nothing to work with. Measured with a stronger shake, Y resonates at 40.9 Hz. The saved value was changed to mzv 42.0 Hz on 2026-11-24 by editing the saved block in the host's `printer.cfg` with Klipper stopped (a `SAVE_CONFIG` only writes values a calibration command has queued); Klipper loaded it on the next start. The previous file is kept beside it as `printer-20261124_*.cfg`. No print has run with the new value yet.

### Differences from thebrain in the machine file

| Setting | pinky | thebrain |
|---|---|---|
| `dockable_probe` dock / approach X | 209 | 210.5 |
| `z_calibration switch_offset` | 0.470 | 0.525 |
| `startupinit` | True | False |
| `lowerbedoncancel`, `STARTUP_IDLE` block | absent | present |
| LED colours | purple | red |

## History

- 567 jobs, about 961 print hours and 7.2 km of filament.
- Last completed print: 2025-02-27 (PETG at 225°C, 106 minutes).
- The last three attempts, on 2025-02-28 (twice) and 2025-03-07, were cancelled before printing anything.
- The final session's log ends with `Probe triggered prior to movement` twice during the probing at the start of a print, right after a normal reading at 35,165. On a Klicky-style probe that usually means the probe was not on the toolhead or its circuit was open. The printer has not run since, so **assume the probe or its dock needs attention**.
- The same logs show the neopixel update errors and the dead `COMPUTE_MESH_PARAMETERS` and `DOCK_PROBE` calls seen on thebrain.

## First power-on, 2026-11-23

Done with the repo pulled to `master` and the `startupinit: False` change checked out, bed confirmed clear by the owner.

| Check | Result |
|---|---|
| Klipper start | Ready 7 s after the relay switched on. Nothing heated or moved. Only warning: `off_below` is deprecated. |
| Firmware | Main board and toolhead board both on `v0.12.0-526-g3fcbc0b4` |
| Temperatures | Nozzle and bed both read 19–20°C, matching the frame sensor |
| Endstops | X, Y and Z open at rest. X and Y use physical switches. |
| Homing | `G28` completed normally and parked at 388,85 Z20 |
| Probe pick-up | `ATTACH_PROBE` reported the probe attached |
| Probing at bed centre | Two readings, 7.511 and 7.309 mm, then `Probe triggered prior to movement` |

After the error the probe pin read triggered (open circuit) continuously with the bed 40 mm away, and Klipper reported the probe as docked although the camera shows the dock empty. The 2025-03-07 log has the same pattern: one normal reading, then the same error twice.

What that points to, not yet confirmed by looking at the part:

- The two readings are 0.2 mm apart, against 0.002–0.006 mm on thebrain's probe. The probe moved on its mount between touches.
- The circuit then stayed open. On this probe the magnets are also the electrical contacts, so either the probe unseated enough to lose contact, or the switch stuck pressed.
- Things to look at: magnets loose or dirty, on the probe and on the toolhead mount; the switch plunger sticking; a broken wire at the toolhead.

### Probe replaced, same day

The owner swapped the probe. Two attempts in the console history around the swap failed (one `Probe triggered prior to movement` after a single reading, one `Probe attach failed!`); whether those were before or after the new probe went in is not recorded. Everything after that passed:

| Check | Result |
|---|---|
| `Z_TILT_ADJUST` (run by the owner) | Converged in four passes: range 0.874 → 0.135 → 0.021 → 0.009 mm. The bed had drifted 0.87 mm out of level across the two Z motors while idle. |
| Pick-up and docking | 4 of 4 clean, by the reported probe state |
| Repeatability at bed centre, 3 × 10 samples | Range 0.018–0.019 mm, standard deviation 0.005–0.007 mm. Medians 7.234, 7.230, 7.219. |
| `CALIBRATE_Z`, cold | probe 7.256, switch 12.481, nozzle 5.358 → offset -0.336 mm |

- The repeatability is good enough for the configured 0.033 mm sample tolerance, but about three times looser than thebrain's probe at its bed centre.
- In February 2025, hot and with the old probe, `CALIBRATE_Z` gave offsets of -0.20 to -0.29 mm (probe 7.33–7.41, switch 12.51–12.55, nozzle 5.36–5.42). The new figure is about 0.1 mm lower. Some of that is a cold bed and nozzle; some may be the new switch. `switch_offset` (0.470) was tuned for the old probe, so **check the first layer before trusting it**.
- For a watchdog during `CALIBRATE_Z`, the touches happen at about Z 5.4 over the nozzle pin, Z 12.5 at the switch position and Z 7.3 on the bed.

The camera snapshot address on this host is `/webcam/snapshot`.

### Air print, same day

The no-extrusion test file from thebrain (`audit_airprint.gcode`, 190/50°C, a square at Z20 around the bed centre), run with the repo at `master`.

| Step | Result |
|---|---|
| First attempt | Cancelled itself within a second: `Error during homing x: Communication timeout during homing`. The cancel then powered the cold printer off 30 s later, as designed. |
| Second attempt, `PRINT_START` | Completed in about 7 minutes from cold |
| Tilt adjust inside it | Four passes: 0.820 → 0.122 → 0.020 → 0.003 mm |
| `CALIBRATE_Z`, hot | probe 7.316, switch 12.459, nozzle 5.316 → offset -0.297 mm |
| Adaptive mesh | 3×3 around the object, completed |
| Pause | Parked at X390 Y385, 15 mm up |
| Resume | Went back and carried on |
| Cancel | Both heater targets to 0 at once, parked at X390 Y385 |
| Power-off after cancel | Relay cut 395 s later, when the nozzle fell below 50°C |

- No errors came from the pause, resume or cancel hooks. The LED effects themselves could not be checked; the strip is out of the camera's view.
- The bed was 0.82 mm out of level again at the start, after being levelled to 0.009 mm half an hour earlier. It loses its level every time the Z motors are released. The tilt adjust at each print start corrects it.
- About 2.5 of the 7 minutes were spent after the mesh, waiting for the nozzle to settle: it rose to 191.2°C when the part fan switched off and `M109` waits for it to steady.
- `PRINT_START` ends with an 11 mm extrude after a 10 mm retract, so the "no-extrusion" test does push about 1 mm of filament if any is loaded.
- Not run on pinky: a pause held past the idle timeout, the 4-hour give-up, and an unused printer powering itself off.

### Homing timeout: what the logs show

`Communication timeout during homing` has stopped four print starts in the logs kept on the host: one on 2025-02-27 and two on 2025-02-28 while probing, and one on 2026-11-23 while homing X. X's endstop and the probe are on the toolhead board and the motors are on the main board, so during those moves the host has to pass a keep-alive between the two boards within `multi_mcu_trsync_timeout`, 0.025 s by default. Y and Z home on switches wired to the main board and do not depend on it.

Per-session link counters from the Klipper logs:

| Link | Resent bytes | Invalid bytes |
|---|---|---|
| Toolhead board (CAN) | 0 in every session | 0 in every session |
| Main board (serial from the Pi), 2026-11-23, 22 minutes of homing and probing with heaters off | 9, all at connect | 0 |
| Main board, 2026-11-23, 9 minutes with heaters on | 941 | 279 |
| Main board, 2025-02-27, 4-hour print | 118,057 | 2,263 |

- **The CAN side is clean.** The interface counters on the Pi also show no bus errors.
- **The main board's serial link is not.** Every logged session with the heaters on shows corrupted data, and the one session without them shows none, so it looks like electrical noise from heater switching reaching the wiring between the Pi and the board. Klipper resends, so prints survive it, but a resend waits 0.025 s, the same as the homing timeout.
- **The link errors do not explain every timeout.** In three of the four, including the one on 2026-11-23, neither counter moved in the second the timeout happened. Something else also delays the keep-alive.
- **The LED updates are a candidate.** `Neopixel update did not succeed` is logged in bursts of up to seven a second, and each line means eight failed attempts in a row on the main board. With an animated effect running the main board reports being busy about 11% of the time, against 0.5% with a static one. Not tested: whether the timeouts go away with the effects off.

Change made: `multi_mcu_trsync_timeout: 0.05` in `printer-pinky.cfg`. That is long enough to ride out one resend or one late message. The cost is that if a board really does stop answering during a homing or probing move, the motors run for up to 0.05 s instead of 0.025 s before stopping: 0.25 mm at the 5 mm/s probing speed, 6 mm at the 120 mm/s X homing speed.

## Resonance, 2026-11-23

Measured on Kalico with the toolhead accelerometer at bed centre: sweeps along X, along Y, and along each diagonal (one belt at a time). The accelerometer's axes are not mapped in the config: the chip's X is vertical, its Y is the toolhead's X and its Z is the toolhead's Y, the same as on thebrain.

| Sweep | Peaks | Best fit |
|---|---|---|
| X | 57.7 and 63.7 Hz | mzv 61.0 Hz, no residual vibration |
| Y, at the configured `accel_per_hz: 50` | 18, 62, 93 and 103 Hz, none dominant | Nothing fits: every shaper leaves 15–37% |
| Y, at `ACCEL_PER_HZ=100` | 40.9 Hz | mzv 42.0 Hz, no residual vibration |
| Diagonal, one belt | 57.6 and 63.6 Hz | mzv 60.0 Hz |
| Diagonal, other belt | 57.6 and 63.6 Hz | mzv 61.0 Hz |

- **X is healthy.** The saved value (mzv 63.2 Hz) is within 2 Hz of the new fit and was left alone.
- **The diagonals say nothing about the belts.** Both show the same two peaks, but those are X's peaks (57.7 and 63.7 Hz): at this test strength the Y part of a diagonal move does not follow, so each diagonal sweep only shows X. Belt balance on pinky is unmeasured. Repeat the diagonals with `ACCEL_PER_HZ=100` to measure it.
- **Y does not follow small moves.** The table below compares the acceleration measured at the drive frequency with the acceleration commanded, at the moment the sweep passes each frequency.

| Drive | Commanded stroke | X follows | Y follows |
|---|---|---|---|
| 8 Hz | ±0.16 mm | 1.08 | 1.12 |
| 11 Hz | ±0.12 mm | 1.20 | 1.41 |
| 18 Hz | ±0.07 mm | 0.96 | 0.83 |
| 20 Hz | ±0.06 mm | 1.01 | 0.47 |
| 22 Hz | ±0.06 mm | 1.02 | 0.23 |
| 30 Hz | ±0.04 mm | 1.04 | 0.10 |
| 40 Hz | ±0.03 mm | 1.28 | 0.09 |
| 60 Hz | ±0.02 mm | 3.82 (resonance) | 0.32 |

- X tracks at 1.0 and then rises into its resonance, as a rigid axis should. Y tracks down to a stroke of about ±0.07 mm and then stops moving: something takes up roughly 0.05 mm before the gantry shifts.
- On the diagonal sweeps the X part of the motion follows and the Y part does not, whichever belt is driving. That rules out one motor, one pulley or one belt, and points at the gantry's travel along Y: the Y rails, or the X beam sitting out of square on its carriages.
- This is the same pattern thebrain showed on 2026-11-23 after its belts were adjusted (0.86, 0.30, 0.10 at 11, 21 and 30 Hz). The saved 104.4 Hz Y value dates from before March 2025 and is what a sweep with this fault produces, so pinky has probably been like this for a long time, through much of its 567 jobs.
- What it costs in prints: Y detail finer than about 0.05 mm is lost.

### What takes up the motion

0.06 mm is 19 microsteps, about 0.3 of one full step of these 200-step motors (0.2 mm per full step). Between full steps a stepper holds position like a spring, so a load that resists can make the rotor lag the commanded position. Two further sets of Y sweeps tested that.

Motor current lowered on both X/Y motors with `SET_TMC_CURRENT`, then restored:

| Current | Stroke at which Y's tracking falls to half | X at ±0.04 mm |
|---|---|---|
| 1.35 A (normal), before and after | ±0.065 mm | 1.04 |
| 1.0 A | ±0.08 mm | |
| 0.7 A | ±0.10 mm | 0.83, no cut-off |

Shake strength changed at normal current (`ACCEL_PER_HZ`, which scales the stroke at every frequency):

| `ACCEL_PER_HZ` | Where Y stops following |
|---|---|
| 25 | 12 Hz, stroke ±0.05 mm |
| 50 (configured) | 20–22 Hz, stroke ±0.06 mm |
| 100 | Nowhere: Y follows through the whole sweep |

- **The cut-off is set by stroke, not frequency.** It sits at ±0.05–0.06 mm whatever frequency that falls on.
- **It grows as the motor current drops.** (Every sweep, X included, shows a drop of about a quarter between 12 and 13 Hz that looks like a feature of the test signal; the 0.7 A figure falls on it, so treat that one as approximate.) So the lost motion is the motors winding up against friction in Y, not slack in a belt or pulley, which would not care about current. X on the same motors keeps following at half the current, so the friction is in the gantry's travel along Y.
- **With the stronger shake Y shows its real resonance:** one clean peak at 40.9 Hz, best fit mzv 42.0 Hz with no residual vibration. That is within 1 Hz of thebrain's Y (41.9 Hz).
- Raising the motor current would shrink the lost motion, but the config notes 1.4 A as these drivers' limit and the main board already reads 87°C. Reducing the friction is the fix.

The raw sweep files are in `~/resonance-2026-11-23/` on the host.

## Open items

- **Intermittent `Communication timeout during homing`.** `multi_mcu_trsync_timeout` is raised to 0.05 s in `printer-pinky.cfg` as a mitigation; see "Homing timeout" above. It was pulled and confirmed active (0.05) on 2026-11-23, on Kalico. Not yet shown to work: the fault is rare, so it needs a run of print starts without one. The causes behind it are still open: the main board's serial link corrupts data while the heaters are on, and the LED updates fail in bursts.
- **Main board temperature.** The board's own sensor reads 85–88°C in every session long enough to warm up, in 2025 and now, and reaches it within 15 minutes with only the motors energised. Check that the case fan spins and actually blows across the drivers.
- Neopixel update errors, as on thebrain: about 4,500 in the first 45 minutes of the day. Each one is eight failed attempts by the main board.
- `switch_offset` was tuned for the old probe; check the first layer.
- **Y does not follow small moves.** Below about ±0.06 mm the gantry stays put, because the motors wind up against friction in Y before it moves; see "Resonance, 2026-11-23". Candidates: belts too tight (thebrain's Y showed the same fault straight after one belt was tightened), the Y rails, or the X beam out of square on its carriages. Cheapest test: back both tensioners off by a counted amount and repeat the 40-second Y sweep (`FREQ_START=5 FREQ_END=45`) at the configured strength; if Y follows further down, tension is the cause. Belt tension on pinky has not been measured.
- **The saved Y shaper was wrong and is now corrected.** It was 104.4 Hz, which left Y with effectively no input shaping; the measured resonance is 40.9 Hz and the host now has mzv 42.0 Hz (2026-11-24). Check Y ringing on the first real print. The fit's suggested acceleration limit is about 5200, above the configured `max_accel` of 5000.
- `accel_per_hz: 50` in `[resonance_tester]` is too gentle for Y on this machine. Use `ACCEL_PER_HZ=100` on the command until the config is changed.
- The config also carries thebrain's open macro items: the dead `COMPUTE_MESH_PARAMETERS` and `DOCK_PROBE` calls, and `off_below`.

## Before powering on

- `startupinit: True`: as soon as the relay switches on, Klipper starts, heats to 184/50°C, homes, and sweeps the toolhead around the bed starting at about 19 mm. Anything on the bed taller than that gets hit. Clear the bed first, or turn `startupinit` off as was done on thebrain.
- Check by eye that the probe is sitting in its dock.
- Pull the repo first so the Z floor and mesh limits are the fixed ones.

## To check once it is running

- Where the probe pin runs off the plate on the right and back edges, before any full mesh.
- Probe repeatability, gantry tilt convergence and Z calibration repeatability, as measured on thebrain.
- Which motors and boards are actually fitted.

## Kalico migration, 2026-11-23

Done the same way as [thebrain's](thebrain.md#kalico-migration), a day later. Read that section for the behaviour changes; this one records what was different here.

### Dry run

Kalico `v2026.11.00` in a separate clone and Python environment on the host, beside the live install.

| Check | Result |
|---|---|
| Python packages | Installed in 27 s from prebuilt wheels |
| Firmware for all three MCUs, from the saved build configs | Built with the existing toolchain |
| This printer's `printer.cfg`, loaded offline | Loads once `led_effect` is linked into `klippy/plugins`. It also accepts `[danger_options] multi_mcu_trsync_timeout`. |
| Macros run offline: homing, `TEST_SPEED`, `HEAT_SOAK`, filament load and unload, `WIPE_LINE`, LED and general commands, pause, resume, cancel | All complete |
| `PRINT_START`, a square, pause, resume, `PRINT_END`, simulated with probing switched off | Completes |

### What was done

1. Full copy of the live install taken first (see below).
2. `~/klipper` repointed at the Kalico repository and checked out at tag `v2026.11.00`. The old `master` branch is still in the checkout.
3. Python packages updated from Kalico's requirements file.
4. `led_effect` link moved from `klippy/extras/` to `klippy/plugins/`.
5. Firmware built from the saved build configs and flashed: mainboard with `scripts/flash-sdcard.sh`, toolhead board over CAN with `flash_can.py`, host MCU by copying the built program to `/usr/local/bin/klipper_mcu` and restarting `klipper-mcu`.
6. `[update_manager klipper]` with `channel: stable` added to the host's `moonraker.conf`.

Things that differed from thebrain:

- **`sudo` needs the account password on this host.** It was used once, for the host MCU step. Everything else ran as the normal user, with the Klipper service stopped and started through Moonraker.
- **The mainboard's SD card is unreliable.** The first two flash attempts failed with `FR_NO_FILESYSTEM`, although the card (a 32 GB card, FAT32, holding the February 2025 `FIRMWARE.CUR`) is fine when it answers. A read-only test then showed 60 failed reads out of 208 in one run and none in the next two. No read returned wrong data. The third flash attempt uploaded, verified and flashed normally. Expect to retry; if an upload ever fails part-way, remove `firmware.bin` from the card before the board next restarts, because the bootloader flashes whatever file is there.
- **One file in the old Klipper tree was corrupt on disk**: a block of `lib/stm32h7/include/stm32h747xx.h` (not used by any board here) held different text from what git has. Git did not notice because the file's timestamp was unchanged. It was restored from git. The other 2,531 tracked files matched. This suggests the drive lost a write at some point, probably in an unclean power-off; a filesystem check would be worth doing.
- **Restarting Moonraker switches the relay off**, because the power device starts in the off state.

### Backup and rollback

`~/backups/pre-kalico-2026-11-23/` on the host holds the `klipper` tree, `klippy-env`, the config folder, the service files, the old host MCU program, and Danger Klipper firmware images for the mainboard and toolhead board built from the installed source. It is on the same drive as the install.

To roll back:

1. Stop the `klipper` service.
2. Replace `~/klipper` and `~/klippy-env` with the copies from the backup.
3. Flash the mainboard from the restored tree: `./scripts/flash-sdcard.sh -f <backup>/firmware/danger-3fcbc0b4-btt-skr-mini-e3-v3.bin -b 500000 /dev/ttyAMA0 btt-skr-mini-e3-v3`.
4. Flash the toolhead board: `python3 ~/CanBoot/scripts/flash_can.py -i can0 -u <uuid from printer-pinky.cfg> -f <backup>/firmware/danger-3fcbc0b4-mellow-fly-sht42.bin`.
5. Put the host MCU back: copy `<backup>/firmware/danger-3fcbc0b4-klipper_mcu` to `/usr/local/bin/klipper_mcu` (needs `sudo`) and restart `klipper-mcu`.
6. Remove the `[update_manager klipper]` block from `moonraker.conf` and start the `klipper` service.

### Checks afterwards

| Check | Kalico | Danger Klipper, same day |
|---|---|---|
| Start | Ready, all three MCUs on `v2026.11.00`. Only warning: `off_below` is deprecated. | Ready, same warning |
| Homing | Completes, parks at 388,85 Z20 | Same |
| Probe repeatability, bed centre, 10 samples | Range 0.015–0.019 mm, σ 0.004–0.006 mm. Median 7.223 after tilt adjust. | Range 0.018–0.019 mm, σ 0.005–0.007 mm. Medians 7.219–7.234. |
| Gantry tilt | 3 passes standalone (0.040 → 0.013 → 0.001 mm), 2 inside the print start | 4 passes each time, starting from 0.82–0.87 mm |
| `CALIBRATE_Z`, cold | -0.336 and -0.346 mm (probe 7.259, 7.257; switch 12.510, 12.523; nozzle 5.385, 5.390) | -0.336 mm (probe 7.256, switch 12.481, nozzle 5.358) |
| `CALIBRATE_Z`, hot, inside the print start | -0.294 mm (probe 7.328, switch 12.464, nozzle 5.312) | -0.297 mm (probe 7.316, switch 12.459, nozzle 5.316) |
| Air print: `PRINT_START` | About 6.5 minutes from cold, adaptive 3×3 mesh | About 7 minutes |
| Pause, resume, cancel | Parks at X390 Y385, 15 mm up; resumes; cancel drops both heater targets and parks | Same |
| Power-off after cancel | Relay cut 424 s later | 395 s |
| Timing or communication errors | None. The homing timeout did not occur in one print start and the seven homing and probing runs before it. | One homing timeout in two print starts |
| Main board serial link, with heaters on | 537 bytes resent, 220 invalid in about 10 minutes | 941 resent, 279 invalid in 9 minutes |
| Neopixel update errors | Still present: 3,402 in 16 minutes | About 4,500 in 45 minutes |
| Main board temperature | 84°C after 16 minutes | 85–86°C |

- The bed started this session about 0.56 mm low at the centre but nearly level (0.04 mm across the tilt points), where the two earlier power-ons had found it 0.82–0.87 mm out of level. So what the bed does while the Z motors are off is not the same every time.
- The serial link noise, the LED update failures and the board temperature are unchanged by the migration. They are hardware or wiring matters.
- A false emergency stop happened during testing, and it was the test script's fault: the Z watchdog was run around a `G28` on an unhomed printer, where the reported Z is meaningless. The watchdog now ignores Z until it is homed. A firmware restart cleared it.
- Not yet done on Kalico here: a real print, a pause held past the idle timeout, and an unused printer powering itself off.

