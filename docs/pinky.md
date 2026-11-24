# pinky

ZeroG Mercury One conversion of an Ender 5 Plus. Sibling of [thebrain](thebrain.md) and built from the same shared files.

**Status, 2026-11-23:** powered on for the first time since March 2025. The probe that ended its last session was replaced the same day, and a no-extrusion air print then ran `PRINT_START`, pause, resume, cancel and the power-off correctly. It has not made a real print yet. One of two print starts failed on an intermittent homing timeout; see "Open items".

## Host and software, as found 2026-11-23

| Part | What is there |
|---|---|
| Host | Raspberry Pi 4 Model B rev 1.4, 4 GB, Raspbian 11 (64-bit), Python 3.9.2, booting from a USB drive. Wired and wireless interfaces are both up. |
| Klipper | Danger Klipper `v0.12.0-526-g3fcbc0b4` (December 2024), the same commit thebrain ran before its Kalico move. The remote still points at the old Danger Klipper repository. |
| Toolhead board firmware | Same version, per the March 2025 log. CAN at 1 Mbit. |
| Moonraker | `v0.9.3-62` |
| Plugins | `led_effect` linked into `klippy/extras`. `z_calibration` and `dockable_probe` are present there. |
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
| `[input_shaper]` | X mzv 63.2 Hz, Y mzv 104.4 Hz |
| `[stepper_z] position_endstop` | 4.796 |
| `[dockable_probe] z_offset` | 7.260 |
| Saved `default` bed mesh | 7×7 over 20…330, range about 0.30 mm, centre high |

The Y shaper value is worth re-measuring. thebrain's Y resonates at about 42 Hz, but its motors are different, so that is only a rough guide. A fit as high as 104 Hz is also what a sweep with no clear peak produces, which is what thebrain's Y sweep looked like on 2026-11-23 when its gantry stopped following small moves.

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

## Open items

- **Intermittent `Communication timeout during homing`.** `multi_mcu_trsync_timeout` is raised to 0.05 s in `printer-pinky.cfg` as a mitigation; see "Homing timeout" above. Not yet shown to work: the fault is rare, so it needs a run of print starts without one. The causes behind it are still open: the main board's serial link corrupts data while the heaters are on, and the LED updates fail in bursts.
- **Main board temperature.** The board's own sensor reads 85–88°C in every session long enough to warm up, in 2025 and now, and reaches it within 15 minutes with only the motors energised. Check that the case fan spins and actually blows across the drivers.
- Neopixel update errors, as on thebrain: about 4,500 in the first 45 minutes of the day. Each one is eight failed attempts by the main board.
- `switch_offset` was tuned for the old probe; check the first layer.
- Saved Y input shaper value (104.4 Hz) needs re-measuring.
- The config also carries thebrain's open macro items: the dead `COMPUTE_MESH_PARAMETERS` and `DOCK_PROBE` calls, and `off_below`.

## Before powering on

- `startupinit: True`: as soon as the relay switches on, Klipper starts, heats to 184/50°C, homes, and sweeps the toolhead around the bed starting at about 19 mm. Anything on the bed taller than that gets hit. Clear the bed first, or turn `startupinit` off as was done on thebrain.
- Check by eye that the probe is sitting in its dock.
- Pull the repo first so the Z floor and mesh limits are the fixed ones.

## To check once it is running

- Where the probe pin runs off the plate on the right and back edges, before any full mesh.
- Probe repeatability, gantry tilt convergence and Z calibration repeatability, as measured on thebrain.
- Resonance on X, Y and each belt diagonal, including whether Y follows small moves at all (see thebrain's 2026-11-23 notes).
- Which motors and boards are actually fitted.
