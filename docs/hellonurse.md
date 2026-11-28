# hellonurse

RatRig V-Minion, cartesian.

**Everything here is read from the config files, last edited 2024-02-14. The machine has not been inspected live.** As of 2026-11-28 it has its three axis motors but no host computer and no wiring; the owner plans to bring it back on a Raspberry Pi Zero 2.

## Hardware, per config

| Part | What the config says |
|---|---|
| Mainboard | Header says BTT SKR Mini E3 V3, but every pin is an RP2040 `gpioNN` name and the commented serial ID is `usb-Klipper_rp2040_…`. Every step, direction, enable, driver UART, heater, thermistor, fan and LED pin matches BigTreeTech's SKR Pico V1.0 pin diagram, so the wiring below assumes that board. Not confirmed by eye. |
| Host link | Pi UART (`/dev/ttyAMA0`, 250000 baud) |
| Toolhead | EVA3 "Squirrelbrain" mod, Sherpa Mini extruder, CHC V6 hotend, 0.4 mm nozzle |
| Probe | BLTouch, also used as the Z endstop |
| X/Y homing | Sensorless (TMC2209 virtual endstops), 40 mm/s |
| Motors | X, Y and Z: OMC 17HM15-0904S (0.9°) per the config comments. Z is belted: 16-tooth pulley driving an 80-tooth one, 8 mm of travel per motor turn. |
| Drivers | TMC2209, 0.6 A on X/Y/Z, 0.85 A on the extruder. BigTreeTech's manual recommends a fan on the drivers above 0.8 A. |
| Extras | Filament switch sensor, filament unload button, 14-LED neopixel strip with LED effects |

## Wiring, per config

Where each cable goes on an SKR Pico V1.0 for the config as it stands. Connector names are the ones printed on the board and on BigTreeTech's pin diagram.

| Device | Connector | Pin in the config |
|---|---|---|
| Power supply | POWER (12/24V, GND) | |
| X motor | X | `gpio11` step, `gpio10` dir, `gpio12` enable |
| Y motor | Y | `gpio6`, `gpio5`, `gpio7` |
| Z motor | Z1 or Z2 (one driver feeds both) | `gpio19`, `gpio28`, `gpio2` |
| Extruder motor | E | `gpio14`, `gpio13`, `gpio15` |
| Hotend heater | HE | `gpio23` |
| Hotend thermistor | TH0 | `gpio27` |
| Bed heater | HB | `gpio21` |
| Bed thermistor | THB | `gpio26` |
| Hotend (heatbreak) fan | FAN1 | `gpio17` |
| Part cooling fan | FAN2 | `gpio18` |
| Electronics fan | FAN3 | `gpio20` |
| BLTouch, 3-wire plug (5 V, ground, control) | SERVOS | `gpio29` |
| BLTouch, 2-wire plug (ground, signal) | Z-STOP, on its signal and ground pins | `gpio25` |
| LED strip, 14 LEDs | RGB (GND, signal, 5V) | `gpio24` |
| Filament switch | Raspberry Pi header, GPIO6 (pin 31) | `rpi:gpio6` |
| Filament unload button | Raspberry Pi header, GPIO5 (pin 29) | `rpi:gpio5` |

- **X and Y home without switches.** The config reads the drivers' stall signal on `gpio4` and `gpio3`, which are the X-STOP and Y-STOP pins. That needs the jumpers fitted on X-DIAG and Y-DIAG and nothing plugged into X-STOP or Y-STOP. No jumper on Z-DIAG: Z-STOP carries the BLTouch signal.
- **The BLTouch signal is on Z-STOP, not on the board's PROBE connector** (`gpio22`). Either works, but the config has to match the plug.
- **Fans run at the supply voltage.** FAN1–3 switch 12/24V, so the fans must match the power supply. Which supply this machine has is not recorded.
- **The filament switch and unload button are wired to the Pi, not the board.** They need the Pi's host MCU service installed (`[mcu rpi]` in `printer-common.cfg`), and their own power and ground from the Pi header. Which sensor it is and what voltage it wants is not recorded; the config's comments match the Orbiter filament sensor's sample config.

### Raspberry Pi link

The config talks to the board over the Pi's serial pins, not USB. The Pico has a 5-pin connector for this on the edge opposite the power terminals, with pins 5V, 5V, GND, IO1, IO0. BigTreeTech's cable runs straight to the first five pins of the outer row of the Pi header, and the Pi is powered through it:

| Pico pin | Pi header pin |
|---|---|
| 5V | 2 (5V) |
| 5V | 4 (5V) |
| GND | 6 (GND) |
| IO1 (board receive) | 8 (GPIO14, Pi transmit) |
| IO0 (board transmit) | 10 (GPIO15, Pi receive) |

A Pi Zero 2 has the same header, though it is often sold without the pins soldered in. Do not plug a separate power supply into the Pi while it is fed from the board. How much current the board's 5 V supply can give is not stated in the documents read.

On the Pi, per BigTreeTech's Klipper notes: remove `console=serial0,115200` from `cmdline.txt`, add `dtoverlay=pi3-miniuart-bt` to `config.txt` (this moves Bluetooth off the good serial port; newer Raspberry Pi OS releases call the overlay `miniuart-bt`, so check the name when setting up), and use `serial: /dev/ttyAMA0` with `restart_method: command`, which the config already has. The board's firmware must be built for "UART0 on GPIO0/GPIO1" with the flash chip set to "GENERIC_03H with CLKDIV 4". It is flashed by fitting the Boot jumper, pressing Reset and copying `klipper.uf2` to the drive that appears over USB; `make flash` does not work on this board.

Sources, read 2026-11-28: BigTreeTech's SKR Pico repository (`Klipper/README.md`, `Klipper/Images/pinout.png`, `Klipper/Images/wiring_uart.png`, the V1.0 instruction manual) and Klipper's `generic-bigtreetech-skr-pico-v1.0.cfg`.

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
- Before the first power-on with a new host: `startupinit` and `warmonboot` make every Klipper start heat, home and sweep the bed. Turn them off for bring-up.
- The firmware on the board dates from 2024 or earlier and will need re-flashing to match a fresh Klipper install.
- Supply voltage, and that the fans match it.
- Klipper fork and version, and the `mainsail.cfg` version.
- That pause, resume and cancel behave after the 2026-11-22 macro change.
