# Voron 2.4 R2, 300 mm: build plan and parts list

**Status, 2026-11-25:** an idea under discussion, nothing ordered or dismantled. The plan is to build one Voron 2.4 R2 with a 300 mm bed from the parts of pinky and thebrain, buying what does not carry over.

The quantities come from Voron's own configurator for the 300 mm specification (data read from vorondesign.com on 2026-11-25). The "have" column comes from [thebrain.md](thebrain.md) and [pinky.md](pinky.md). Anything marked **check** has not been confirmed on the machines.

## Size

| | mm |
|---|---|
| Build volume | 300 × 300 × 280 |
| Frame outside | 460 × 460 × 480 |

The Mercury machines print 370 × 370 today, so this is a smaller bed.

## Decisions still open

- **Toolhead.** Keep the Afterburner and its extruder (they are Voron parts and fit the X carriage), or print a Stealthburner.
- **Probe.** Keep the Klicky dockable probe, or change to Voron Tap.
- **Toolhead wiring.** A CAN umbilical with the Fly SHT42 board removes the X and Y cable chains and most of the high-flex wire in the list below. Assumed here.
- **Display.** KlipperScreen on an existing touchscreen in place of the Mini 12864.
- **A/B motors.** Pinky's 1.8° LDO pair on the TMC2240s, with thebrain's four 0.9° motors on Z. Assumed here.

## What carries over

| Voron part | Qty | From | Notes |
|---|---|---|---|
| Controller with 7+ stepper outputs | 1 | BTT Octopus V1.1 (STM32F446) | Eight driver sockets. With the extruder on the toolhead board it drives A, B and four Z. |
| TMC2209 drivers | 7 | On the Octopus already | Two TMC2240s are on order for A and B. |
| Raspberry Pi | 1 | Either printer's Pi 4 | One left over. |
| NEMA17 motors, A and B | 2 | pinky: LDO 42STH48-2504AC, 1.8° | |
| NEMA17 motors, Z | 4 | thebrain: OMC 17HM19-2004S, 0.9° | thebrain has four of these (X, Y, Z, Z1). Pinky's two Z motors are spares; their model is not recorded. |
| Extruder and its motor | 1 | Either Afterburner toolhead | 50:17 geared. The drive-gear grub screw worked loose on thebrain on 2026-11-24. |
| Hotend, 24 V | 1 | Either toolhead | 0.6 mm high-flow nozzle on both. Hotend model and thermistor type are not confirmed. |
| Part fan 5015, hotend fan 4010 | 1 + 1 | Either toolhead | |
| Toolhead board | 1 | Mellow Fly SHT42 (CAN) | Carries the extruder driver, the accelerometer, the X endstop and the probe input. |
| CAN interface | 1 | USB CAN adapter | The Octopus can also act as the bridge. |
| Probe | 1 | Klicky dockable probe | Replaces the inductive probe and its BAT85 diode. |
| Z endstop | 1 | Nozzle-touch pin | Same arrangement Voron uses. |
| X and Y endstop switches | 2 | Existing switches | **Check** they are the D2F type the printed parts hold. |
| 24 V power supply | 1 | Either printer's supply | Voron specifies 200 W, because the bed is mains heated. **Check** the model and rating. |
| 5 V supply for the Pi | 1 | Existing | **Check** how the Pis are powered today. |
| Power relay | 1 | Existing GPIO relay | Optional on a Voron. |
| Display | 1 | Existing touchscreen | KlipperScreen. |
| Camera, neopixels | | Existing | Not in the Voron list. |
| GT2 20-tooth pulleys, 6 mm, 5 mm bore | 3 | Motor pulleys from both machines | **Check** count and bore. |
| GT2 6 mm belt, 2 × 1800 mm | 2 | thebrain's belts, new on 2026-11-24 | **Check** length; the Mercury's loops should be longer than 1800 mm. |
| F695 bearings | 20 | Idler stacks on both machines | **Check** type and count. |
| C13 mains cord | 1 | Existing | |

## What has to be bought

### Frame

| Part | Qty |
|---|---|
| 2020 extrusion, 420 mm, tapped ends | 10 (four X, four Y, two bed supports) |
| 2020 extrusion, 480 mm, drilled for blind joints | 4 (verticals) |
| 2020 extrusion, 400 mm | 2 (gantry Y) |
| 2020 extrusion, 380 mm | 1 (gantry X) |
| 2020 extrusion, 290 mm | 1 (cross brace) |
| DIN rail, 35 mm wide, 415 mm | 2 |
| Rubber feet | 4 |

The Ender 5 Plus frames do not supply these. Voron advises against V-slot extrusion because of how the MGN9 rails sit on it.

### Linear rails

| Part | Qty | Notes |
|---|---|---|
| MGN9H, 350 mm | 6 | Two for Y, four for Z |
| MGN12H, 350 mm | 1 | X. The Mercury machines have six MGN12H rails between them; one could be used if it is 350 mm long. **Check** lengths. |

### Motion and Z drive

| Part | Qty |
|---|---|
| GT2 80-tooth pulley, 5 mm bore, 6 mm wide | 4 |
| GT2 16-tooth pulley, 5 mm bore, 6 mm wide | 4 |
| GT2 20-tooth pulley, 5 mm bore, 9 mm wide | 4 |
| GT2 20-tooth toothed idler, 9 mm wide | 4 |
| GT2 20-tooth toothed idler, 6 mm wide | 2 |
| GT2 closed belt loop, 6 mm wide, 188 mm | 4 |
| GT2 open belt, 9 mm wide, 1100 mm | 4 |
| 625 bearing | 12 |
| 5 × 60 mm shaft, D-cut | 4 |
| 5 × 35 mm shaft | 1 |

### Bed

| Part | Qty | Notes |
|---|---|---|
| Cast aluminium plate (MIC6), 300 × 300 × 8 mm | 1 | |
| Silicone mains heater with thermistor, 250 × 250 mm, 450 W | 1 | The Mercury beds are 24 V and 377 × 370 mm; they do not fit. |
| Solid-state relay (Omron G3A-210B-DC5 or equivalent) and DIN bracket | 1 | |
| Thermal fuse, 150°C | 1 | |
| Magnetic sheet, 300 × 300 mm | 1 | |
| Spring-steel sheet, 300 × 300 mm | 1 | The existing smooth and textured sheets are 377 × 370 mm. |

### Electronics and mains

| Part | Qty | Notes |
|---|---|---|
| Filtered mains inlet with fuse | 1 | 8 A fuse on 120 V mains, 4 A on 230 V |
| Rocker switch, double pole, 16 A | 1 | |
| 60 × 60 × 20 mm fan, 24 V | 3 | Electronics bay and exhaust |
| Wago 221-415 lever nuts | 3 | |
| Crimp terminals and ferrules | set | Spade, fork and ring terminals for the mains and bed wiring |
| Bed wiring connector, 3-pin | 1 pair | |
| 18 AWG and 20 AWG wire | 3 m each | |
| CAN umbilical cable | 1 | In place of the X/Y cable chains and most of the 76 m of high-flex wire in the standard list |
| Z cable chain, 10 × 15 mm, about 500 mm | 1 | Carries the A/B motor and endstop wiring up to the moving gantry; the bed does not move on a 2.4 |

### Enclosure

| Part | Qty |
|---|---|
| Coroplast, 419 × 419 × 4 mm (deck and bottom) | 2 |
| Coroplast, 433 × 453 × 4 mm (back) | 1 |
| Clear acrylic, 433 × 453 × 3 mm (sides) | 2 |
| Clear acrylic, 216.5 × 453 × 3 mm (doors) | 2 |
| Clear acrylic, 433 × 433 × 3 mm (top) | 1 |
| 6 × 3 mm magnets | 8 |
| VHB tape, 1 mm and 3 mm foam tape | 1 each |
| Carbon filter element | 1 |

### Fasteners

Usually bought as a Voron 2.4 kit. Screws and nuts recovered from the donors are not counted.

| Part | Qty | Part | Qty |
|---|---|---|---|
| M5×40 SHCS | 22 | M3×40 SHCS | 30 |
| M5×30 BHCS | 22 | M3×30 SHCS | 25 |
| M5×16 BHCS | 35 | M3×25 SHCS | 6 |
| M5×10 BHCS | 35 | M3×20 SHCS | 22 |
| M5 hex nut | 16 | M3×16 SHCS | 20 |
| M5 post-install T-nut | 68 | M3×12 SHCS | 41 |
| M5 1 mm shim | 42 | M3×8 SHCS | 180 |
| M4×6 BHCS | 7 | M3×6 BHCS | 11 |
| M4 knurled nut | 4 | M3×10 FHCS | 4 |
| M3×50 SHCS | 2 | M3×6 FHCS | 21 |
| M3 hex nut | 6 | M3 post-install T-nut | 96 |
| M3 washer | 4 | M3 hammer-head T-nut | 60 |
| M3 heat-set insert (5 × 4 mm) | 110 | M2×10 self-tapping screw | 18 |

The M5×16, M3×8 and M3 T-nut counts include the extra screws for blind frame joints and seven 350 mm rails.

### Consumables

Thread locker, rail grease (Mobil EP1 or EP2), PTFE tube, cable ties.

### Printed parts

The full Voron 2.4 R2 set in ABS or ASA, printed on the owner's enclosed printer. **Check** that the longest parts for a 300 mm build fit its bed.

## Not needed from the standard list

- Inductive probe and BAT85 diode (Klicky stays).
- Mini 12864 display (KlipperScreen).
- BMG extruder kit and NEMA14 motor (the Afterburner's extruder stays).
- X and Y cable chains.
- Seven separate TMC2209s and the six 17HS19-2004S motors (covered above).

## Before ordering: checks on the donors

1. Length of each MGN12H rail.
2. Model and rating of each 24 V supply, and how each Pi gets its 5 V.
3. Pinky's Z motor model.
4. Hotend model and thermistor type on each toolhead.
5. Count and type of pulleys, toothed idlers and F695 bearings.
6. Length of thebrain's new 6 mm belts.

## What changes in this repo if it is built

A new frame file for the 2.4 (CoreXY with four Z motors and `[quad_gantry_level]` in place of `[z_tilt]`), a pin map for the Octopus, and a notes page for the new printer. The toolhead, probe and macro files can carry over. The OrcaSlicer presets need a 300 mm bed and a mains bed's temperatures.
