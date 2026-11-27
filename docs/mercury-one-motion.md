# Mercury One machines: Y motion

Findings that apply to both [thebrain](thebrain.md) and [pinky](pinky.md), from accelerometer sweeps on 2026-11-22 to 2026-11-24. Each printer's own numbers are in its file.

## Summary

- **Both machines ignore very small Y moves.** At the standard resonance-test strength, thebrain's Y stops following below about ±0.04 mm and pinky's below about ±0.07 mm. X follows every move tested on both, down to ±0.025 mm.
- **Cause:** the motors wind up against friction in Y's travel before the gantry moves. The size of the dead band is the friction divided by how stiffly the drive holds position.
- **It is not a fault at this size.** ±0.04 mm is 13 microsteps, 0.4 of one full step of thebrain's motors, and about a fifteenth of a 0.6 mm extrusion width. thebrain printed a good 20 mm cube at this level on 2026-11-22 and again on 2026-11-24.
- **It is useful as a health check.** thebrain's dead band grew to about ±0.1 mm after a belt adjustment on 2026-11-23 and came back to ±0.04 mm after new belts and rail work. A change in this number means something mechanical changed.
- **Not known:** why Y has so much more friction than X on both machines, and whether pinky's ±0.07 mm shows in a print. pinky has not printed since it was measured.

## What changes the dead band

| Change | Effect on the dead band | Measured on |
|---|---|---|
| Both belts loosened "a lot" | ±0.065 mm → ±0.04 mm | pinky, 2026-11-24 |
| Belts tightened again | Back to about ±0.07 mm | pinky, 2026-11-24 |
| Motor current 1.35 A → 1.0 A → 0.7 A | ±0.065 mm → ±0.08 mm → about ±0.10 mm | pinky, 2026-11-23 |
| Position along Y | Worst mid-travel, smaller at both ends | pinky, 2026-11-24 |
| New belts, rails adjusted and lubricated | About ±0.1 mm → ±0.04 mm | thebrain, 2026-11-24 |
| Shake strength (`ACCEL_PER_HZ` 25, 50, 100) | The cut-off follows the size of the move, not its frequency | pinky, 2026-11-23 |

- thebrain's 0.9° motors follow smaller moves than pinky's 1.8° ones. A 0.9° motor is about twice as stiff as a 1.8° one at equal torque, so this is expected, but the two machines were never compared under controlled conditions.
- Candidates for the extra friction in Y that were never separated: the two Y rails not parallel, the X beam out of square on its carriages, the frame pulling the rails inward, and carriage preload.

## If it ever needs improving

No config change shrinks the dead band. The levers, from the ZeroG documentation and other sources (listed at the end):

- **Rail alignment.** ZeroG's manual (pages 143–147) has the procedure: the left Y rail stays fixed, the right one is loosened, the gantry is run front to back, and the screws are tightened from the centre outward, before the belts go on.
- **Belt tension.** ZeroG's figure is 110–120 Hz on the 150 mm span between the tensioner pin and the toothed-idler pin, with equal belt lengths. The belts are 6 mm GT2. Tighter belts made Y worse on pinky.
- **Frame.** One builder traced bad Y binding to the Z-rod assembly pulling the rails inward; loosening the upper inner extrusion screws helped. One report only.
- **Motors.** 0.9° motors on pinky's X and Y would roughly halve the motors' share of the wind-up.
- **Not motor current.** 1.35 A is at the TMC2209's 1.4 A continuous design guideline. BTT advises active cooling above 1.2 A, and Klipper's stock config for the SKR Mini E3 V3 uses 0.58 A. A board with stronger drivers would be needed to go higher.

## How to measure

All of this runs with X and Y homed only. `SET_KINEMATIC_POSITION Z=20` after `G28 X Y` lets a sweep run without homing Z or moving the bed; `M84` afterwards clears it.

- **Dead band:** `TEST_RESONANCES AXIS=Y FREQ_START=5 FREQ_END=45 OUTPUT=raw_data NAME=<name>` takes 40 seconds. `track_cmp.py 3 <files>` on the host prints, for each frequency, the measured acceleration divided by the commanded one: 1.0 means the toolhead follows. The size of the commanded move at each frequency is `accel_per_hz ÷ (4π² × frequency)` mm. Use `POINT=x,y,z` to test other places on the bed.
- **Y shaper:** at the configured `accel_per_hz: 50` the Y sweep reaches the dead band before it reaches Y's resonance, and the fit is meaningless (this is how pinky came to have 104.4 Hz saved). Use `ACCEL_PER_HZ=100`. pinky with tight belts needed 150.
- **Belt balance:** the two diagonal sweeps (`AXIS=1,1` and `AXIS=1,-1`) load one belt each. They only show the belts if the shake is strong enough for the Y part to follow: `ACCEL_PER_HZ=120` worked on thebrain. At 50 both diagonals just show X's peaks.
- **Skipped teeth:** `GET_POSITION` after homing gives each motor's step count. Re-home after fast moves and compare. One 2 mm belt tooth is 640 microsteps on both machines.
- **Fitting:** Kalico's `calibrate_shaper.py` needs matplotlib, which is not installed. `shaper_an.py` (peaks and best fits) and `shaper_eval.py` (score named shapers against several sweeps) on each host call the same fitting code.
- **Saving:** `SAVE_CONFIG` only writes values a calibration command has queued. Hand-picked shaper values go into the saved block of the host's `printer.cfg` by editing it, followed by a Klipper restart.

Things that cost time:

- Klipper writes the raw file after the command returns. Wait for the file to stop growing before copying it.
- The accelerometer's axes are not mapped in the config: the chip's X is vertical, its Y is the toolhead's X and its Z is the toolhead's Y.
- Every sweep shows a drop of about a quarter between 12 and 13 Hz. It looks like a feature of the test signal.
- The chip's Y channel returns exactly zero more often than it should, up to 30% of samples in a quiet sweep. Not explained. The toolhead-Y channel barely shows it.
- thebrain's part fan comes on full at the idle timeout and stays on after the next command. Turn it off before a sweep.

## Standard test piece

From 2026-11-25 the test print for thebrain and pinky is the **Orca cube** (OrcaSlicer's built-in handy model `OrcaCube_v2`, 30 × 30 × 30 mm) at full size, 0.2 mm layers, 0.6 mm nozzle. Use it for before-and-after comparisons of profiles, speeds and calibration.

Earlier baselines in these notes (1546 s and 1554 s of printing, 1937 mm of filament) are for a Voron Design Cube scaled to 20 mm with the old SuperSlicer "Normal" profile. They do not compare with the Orca cube.

### Print profiles

The owner's definition of the profile ladder, set on 2026-11-27. Structure and speed are per profile; temperatures, cooling and the 10 s minimum layer time belong to the filament profile.

- **Standard:** reliable and slow.
- **Strong:** Standard's speeds with a fourth wall, thicker layers and thicker shells.
- **Faster:** predictable quality, noticeably quicker.
- **Fastest:** gives up some surface finish for speed, and one wall.
- **Ultra:** Fastest at 0.3 mm layers.

| | Standard | Strong | Faster | Fastest | Ultra |
|---|---|---|---|---|---|
| Layer height, mm | 0.2 | 0.3 | 0.2 | 0.2 | 0.3 |
| Walls (0.6 mm lines) | 3 | 4 | 3 | 2 | 2 |
| Top / bottom layers | 4 / 3 | 4 / 4 | 4 / 3 | 4 / 3 | 3 / 2 |
| Infill | 10% gyroid | 10% gyroid | 10% gyroid | 10% gyroid | 10% gyroid |
| Outer wall, mm/s | 36 | 36 | 80 | 150 | 150 |
| Inner wall / sparse infill | 72 / 108 | 72 / 108 | 150 / 150 | 190 / 190 | 190 / 190 |
| Solid infill / gap fill | 36 / 36 | 36 / 36 | 120 / 100 | 150 / 150 | 150 / 150 |
| Top surface | 18 | 18 | 50 | 80 | 80 |
| Acceleration, mm/s² | 1500 | 1500 | 5000 | 8000 | 8000 |
| Outer wall / top surface acceleration | 1500 / 1500 | 1500 / 1500 | 3000 / 2000 | 5000 / 3000 | 5000 / 3000 |
| First layer, mm/s | 30 | 30 | 40 | 40 | 40 |

Things to know when reading the table:

- The filament's melt limit caps the real speed. At 24 mm³/s a 0.6 mm line tops out near 200 mm/s on 0.2 mm layers and near 133 mm/s on 0.3 mm layers, so Ultra never reaches the speeds shown.
- The slicer's "The Brain" machine limit is 8000 mm/s²; "Pinky" keeps its own lower limit and clamps these profiles to it.
- Klipper does not clamp `SET_VELOCITY_LIMIT` to the config's `max_accel`. thebrain's config still says 5000, which is what the start sequence and manual moves use; the print file sets its own value per feature.
- On a single small part the 10 s minimum layer time governs and the faster profiles gain little. Judge the steps on a full plate.

Two purpose-built profiles sit beside the ladder. Each is Standard with only the listed settings changed:

- **Vase:** spiral mode, 1 wall, no top layers, no infill, 0.3 mm layers.
- **Voron:** the Voron printed-parts recipe: 4 walls, 5 top and 5 bottom layers, 42% infill, a 5 mm brim and a draft shield.

"RatRig" and a second "Voron" without the machine suffix are leftovers converted from other printers' SuperSlicer profiles. They are tagged for these machines but do not follow the ladder, and are kept for now. Removed on 2026-11-27: "Slow", "-EM Test", "ZeroPanels" and "ZeroPanels-Hinges".

OrcaSlicer 2.5.2 estimates on thebrain with ProtoPasta HTPLA at 200°C. These are slicer estimates, not prints, and nothing has been printed on the new Fastest, Ultra or Strong. `PRINT_START` adds 3 to 7 minutes.

| | Standard | Strong | Faster | Fastest | Ultra |
|---|---|---|---|---|---|
| Orca cube | 52m 43s | 41m 46s | 34m 18s | 30m 56s | 20m 55s |
| Three thin-walled drawers on one plate (75 g) | 5h 15m | 3h 48m | 2h 18m | 1h 35m | 1h 09m |

**First print on Fastest, 2026-11-27 (thebrain, one thin-walled drawer, HTPLA, 44m 36s).** The front wall carries raised dots (1.7 mm wide, 0.3 mm proud) and a raised number. The owner's photos show a faint copy of each dot a few millimetres further along the wall. Read from the print file:

- That wall runs along X, so each dot is a quick Y move of 0.3 mm and back.
- The outer wall ran at 77–91 mm/s, not 150: one small part is held back by the minimum layer time. So the mark appears at Faster's outer wall speed, and what differed from Faster was the acceleration (5000 against 3000 mm/s² on the outer wall, 8000 against 5000 on the inner wall behind it).
- Not known yet: whether the mark is Y ringing that the shaper leaves behind or the extruder pressure catching up after the slow-down at each dot, and at what acceleration it disappears. `Drawer17_outer_accel_bands_200C_PLA.gcode` on thebrain is the same file with the outer wall acceleration stepped by dot row from the bed up: 1500, 2000, 3000, 4000, 5000, 8000 mm/s².
- **That file, printed the same day:** the owner sees some ghosting on every row, the 1500 row included. So the outer wall's acceleration is not what sets it. At every dot the nozzle brakes to about 9 mm/s whatever the acceleration, because `square_corner_velocity` is 5; the inner wall still ran at 8000 mm/s² on every row.
- **Next test, not printed yet:** `Drawer17_pair_XY_rows_200C_PLA.gcode` on thebrain is two drawers on Fastest, the left one as before (dots are Y moves) and the right one turned 90° with its dots facing right (dots are X moves). With two parts the outer wall runs at the full 150 mm/s. One setting changes per dot row from the bed up: (1) Fastest as is, (2) square corner velocity 10, (3) square corner velocity 15, (4) pressure advance smooth time 0.02 s, (5) pressure advance off, (6) every acceleration 1500 mm/s². If the turned drawer is clean where the other is not, the cause is in the Y axis; if both ghost alike, it is the extrusion or the inner wall, and the rows say which.

Before 2026-11-27 Fastest was a 0.3 mm profile, Strong had 0.8 mm lines and 31% infill, and Faster's outer wall was 50 mm/s until late on 2026-11-25. Earlier estimates in the history of this file belong to those versions.

Real prints so far, all the cube with its plug in ProtoPasta HTPLA on thebrain, 2026-11-25:

| Profile as printed | Printing time | Estimate | Owner's measurements |
|---|---|---|---|
| Fastest while it still had 0.2 mm layers, HTPLA flow ratio 1.00 | 39m 01s | 38m 35s | 30.00 mm tall, 30.10 mm wide on both axes; plug very tight |
| Fastest with 0.3 mm layers, HTPLA flow ratio 0.96 | 27m 03s | 26m 33s | 30 × 30 × 30 mm; plug stopped about two-thirds in, then threaded fully after being worked back and forth |
| Standard, HTPLA flow ratio 0.96, pressure advance 0.049, smooth plate with its 0.03 mm first-layer offset | 63m 11s | 62m 42s | About 29.95 mm; plug threads in cleanly; no elephant foot. Owner's verdict: looks good |

The plug is a tight fit by design: its thread crest radius is 7.43 mm and the hole's is 7.53 mm, 0.1 mm of clearance per side, on a thread about 5.6 mm long. The Standard row is the reference result for this piece: pressure advance for this filament was measured at 0.049 with a Klipper tuning tower the same day.

## How these machines compare with other builds

Other Mercury One.1 builds on the Ender 5 Plus frame publish Y resonances of 36–39 Hz and X resonances of 82–86 Hz, with `max_accel` between 6000 and 10000. Here Y is 41–46 Hz and X is 57–66 Hz. Y is normal for the design. X is lower, most likely because the Afterburner toolhead is heavier than the EVA toolheads those builds use. No report of a small-move dead band on Y was found, but few builders would measure for one.

## Sources

- [ZeroG Mercury One.1 manual](https://docs.zerog.one/assets/mercury_one_1_instruction_18-02-2024.pdf), [bill of materials](https://docs.zerog.one/manual/build/mercury_eva/bill_of_material) and [FAQ](https://docs.zerog.one/faq)
- [Build thread with the frame-binding report](https://forum.v1e.com/t/zero-g-mercury-one-1-build/42041?page=4)
- [TMC2209 datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/TMC2209_datasheet_rev1.09.pdf), [BTT TMC2209 manual](https://github.com/bigtreetech/BIGTREETECH-Stepper-Motor-Driver/tree/master/TMC2209/V1.3/manual), [Klipper's stock config for the board](https://github.com/Klipper3d/klipper/blob/master/config/generic-bigtreetech-skr-mini-e3-v3.0.cfg)
- [Klipper TMC drivers documentation](https://www.klipper3d.org/TMC_Drivers.html)
- [Machine Design, "Step angles and accuracy"](https://www.machinedesign.com/archive/step-angles-and-accuracy)
- Community configs for Plus-size builds: [ethomasgt](https://github.com/ethomasgt/Ender-5-Plus-Mercury-One-Klipper-Mainsail), [dborden0](https://github.com/dborden0/merc1_klipper)
