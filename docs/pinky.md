# pinky

ZeroG Mercury One conversion of an Ender 5 Plus. Sibling of [thebrain](thebrain.md) and built from the same shared files.

**Status, 2026-11-23:** powered on for the first time since March 2025. The probe that ended its last session was replaced the same day and the new one passes its first checks. It has not printed yet. Nothing else about how it behaves has been checked; treat the thebrain notes as the best guide.

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

The stepper file is the main hardware difference from thebrain: 64 microsteps and different motors (its comments name Fabreek Honey Badger 42HS48-25044A and LDO 42STH48-2504AC). Which motors are really fitted is unverified.

### Values that live only on the host

| Setting | Value |
|---|---|
| `[extruder]` PID | kp 18.115, ki 0.693, kd 118.312 |
| `[heater_bed]` PID | kp 50.732, ki 0.505, kd 1274.653 |
| `[input_shaper]` | X mzv 63.2 Hz, Y mzv 104.4 Hz |
| `[stepper_z] position_endstop` | 4.796 |
| `[dockable_probe] z_offset` | 7.260 |
| Saved `default` bed mesh | 7×7 over 20…330, range about 0.30 mm, centre high |

The Y shaper value looks wrong. thebrain, the same frame, resonates at about 42 Hz in Y. A fit of 104 Hz is what a sweep with no clear peak produces, which is also what thebrain's Y sweep looked like on 2026-11-23 when its gantry stopped following small moves. Re-measure before trusting it.

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
| Pick-up and docking | 4 of 4 clean, checked by the probe state and the camera |
| Repeatability at bed centre, 3 × 10 samples | Range 0.018–0.019 mm, standard deviation 0.005–0.007 mm. Medians 7.234, 7.230, 7.219. |
| `CALIBRATE_Z`, cold | probe 7.256, switch 12.481, nozzle 5.358 → offset -0.336 mm |

- The repeatability is good enough for the configured 0.033 mm sample tolerance, but about three times looser than thebrain's probe at its bed centre.
- In February 2025, hot and with the old probe, `CALIBRATE_Z` gave offsets of -0.20 to -0.29 mm (probe 7.33–7.41, switch 12.51–12.55, nozzle 5.36–5.42). The new figure is about 0.1 mm lower. Some of that is a cold bed and nozzle; some may be the new switch. `switch_offset` (0.470) was tuned for the old probe, so **check the first layer before trusting it**.
- For a watchdog during `CALIBRATE_Z`, the touches happen at about Z 5.4 over the nozzle pin, Z 12.5 at the switch position and Z 7.3 on the bed.

The camera snapshot address on this host is `/webcam/snapshot`.

## Before powering on

- `startupinit: True`: as soon as the relay switches on, Klipper starts, heats to 184/50°C, homes, and sweeps the toolhead around the bed starting at about 19 mm. Anything on the bed taller than that gets hit. Clear the bed first, or turn `startupinit` off as was done on thebrain.
- Check by eye that the probe is sitting in its dock.
- Pull the repo first so the Z floor and mesh limits are the fixed ones.

## To check once it is running

- Where the probe pin runs off the plate on the right and back edges, before any full mesh.
- Probe repeatability, gantry tilt convergence and Z calibration repeatability, as measured on thebrain.
- Resonance on X, Y and each belt diagonal, including whether Y follows small moves at all (see thebrain's 2026-11-23 notes).
- Which motors and boards are actually fitted.
