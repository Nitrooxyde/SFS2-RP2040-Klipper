# BTT SFS 2.0 + RP2040‑Zero — Standalone USB Filament Sensor for Klipper

![Klipper](https://img.shields.io/badge/Klipper-USB%20MCU-orange)
![MCU](https://img.shields.io/badge/MCU-RP2040--Zero-green)
![Sensor](https://img.shields.io/badge/sensor-BTT%20SFS%202.0-blue)
![Status](https://img.shields.io/badge/status-functional-brightgreen)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

A **self‑contained, plug‑and‑play filament sensor module** for any Klipper printer:
a **BigTreeTech Smart Filament Sensor (SFS) 2.0** read by a **Waveshare RP2040‑Zero**
flashed as a standard **Klipper USB MCU**, both housed in a **one‑piece printed
receptacle**. One USB cable in, two native Klipper sensor objects out.

The SFS 2.0 exposes two signals — a **runout switch** (filament present / absent) and
a **motion encoder** (~1 pulse / 2.88 mm of filament travel). The RP2040‑Zero turns
both into first‑class Klipper objects:

- `[filament_switch_sensor sfs_switch]` → filament presence
- `[filament_motion_sensor sfs_motion]` → movement / no‑movement (clog, slip)

---

## Why a dedicated USB MCU for a filament sensor?

The usual way to wire an SFS is two signal lines back to the main board. Giving the
sensor **its own $3 microcontroller on USB** instead has real advantages:

- **Zero pins needed** on your main or toolhead board — useful when they're full,
  or when you don't want to run signal wires across the printer.
- **Plug‑and‑play** — one USB cable. Move the module between machines, or add it to
  a printer that was never wired for it.
- **Electrically self‑contained** — the SFS is powered from the Pico's 3V3 pin;
  no shared rails, no level‑shifting questions.
- **A single printed part** — sensor + MCU + mounting = one module on a 2020
  extrusion.

What you do with the two signals is up to your config — they are ordinary Klipper
sensor objects:

- **pause on runout** (`pause_on_runout: True` when you're ready),
- **clog / jam detection** — extruder commanded to move, encoder says nothing moves,
- **slip / under‑extrusion detection** — commanded‑vs‑wheel differential,
- **feed verification inside your own macros** — the motion state keeps updating
  even when the sensor is disabled (verified in Klipper source), which makes it a
  clean *“is filament actually moving?”* probe for any custom automation
  (multi‑material feeds, load/unload sequences, whatever you build).

---

## What's in this repo

| Path | Contents |
|------|----------|
| [`docs/01-hardware.md`](docs/01-hardware.md) | SFS 2.0 + RP2040‑Zero specs, verified dimensions, BOM |
| [`docs/02-wiring.md`](docs/02-wiring.md) | SFS → Pico wiring, **the 3V3 rule**, pin map |
| [`docs/03-firmware-flashing.md`](docs/03-firmware-flashing.md) | Building & flashing Klipper on the RP2040‑Zero (Katapult) |
| [`docs/04-klipper-integration.md`](docs/04-klipper-integration.md) | `printer.cfg` blocks, safe bring‑up, usage |
| [`docs/05-cad-bracket.md`](docs/05-cad-bracket.md) | The printed receptacle: features, print settings, source |
| [`klipper/sfs.cfg`](klipper/sfs.cfg) | Ready‑to‑`[include]` Klipper config snippet |
| [`cad/REC_body_v1.stl`](cad/REC_body_v1.stl) | Print‑ready one‑piece receptacle (SFS 2.0 + RP2040‑Zero) |
| [`cad/SFS20_Support_V2.FCStd`](cad/SFS20_Support_V2.FCStd) | FreeCAD source of the receptacle |
| [`firmware/`](firmware/) | Notes on the stock‑Klipper RP2040 build (no custom C) |

---

## Quick start (TL;DR)

1. **Print** `cad/REC_body_v1.stl` (ABS/ASA, 4 walls, 0.2 mm — see
   [docs/05](docs/05-cad-bracket.md)).
2. **Flash** the RP2040‑Zero with **stock Klipper, RP2040, USB, Katapult bootloader**
   ([docs/03](docs/03-firmware-flashing.md)).
3. **Wire** SFS → Pico: **VCC → 3V3 (never 5V)**, GND → GND, motion → **GP0**,
   switch → **GP1** ([docs/02](docs/02-wiring.md)).
4. **Add** the config below to `printer.cfg` (or copy `klipper/sfs.cfg` next to it
   and `[include sfs.cfg]`), fix the serial to *your* board, `FIRMWARE_RESTART`.

```ini
[mcu sfs]
serial: /dev/serial/by-id/usb-Klipper_rp2040_XXXXXXXXXXXXXXXX-if00

[filament_motion_sensor sfs_motion]
switch_pin: ^sfs:gpio0        # GREEN wire — motion encoder
detection_length: 2.88        # mm; SFS theoretical pulse pitch — tune at the bench
extruder: extruder
pause_on_runout: False        # bring-up safe: observe first, arm later

[filament_switch_sensor sfs_switch]
switch_pin: ^sfs:gpio1        # BLUE wire — runout switch
pause_on_runout: False
```

> ⚠️ **The two golden rules** (details in the docs):
> 1. **Power the SFS at 3.3 V.** The RP2040 is **not** 5 V‑tolerant (~3.8 V abs max).
>    5 V on the signal lines destroys the Pico.
> 2. **Keep the `^` pull‑ups.** The SFS outputs are open‑collector / switch‑to‑ground;
>    without an internal pull‑up they float.

---

## Placement

Anywhere on the filament path — that's the point of a self‑contained module. One
practical rule: **the closer to the extruder, the less slack** between the encoder
wheel and the drive gear, so the tighter `detection_length` can be and the earlier a
real clog or runout is flagged. And fit the **PTFE ID2/OD4 guide all the way into the
SFS bore on both ports** — without it, pushed filament buckles at the junction
(empirically confirmed; see [docs/01](docs/01-hardware.md)).

---

## Status

**Functional.** Built and running on a Voron 2.4: the receptacle prints and fits, the
RP2040‑Zero runs stock Klipper over USB (Katapult in place — no BOOT button needed for
reflash), and both sensor objects run in Klipper with the SFS powered at 3V3. The
bracket still has a few optional refinements open (pico Y‑retention, flush lip) but
is a working part.

See [`CHANGELOG.md`](CHANGELOG.md) for the build history.

---

## License

MIT — see [`LICENSE`](LICENSE). The BTT SFS 2.0 CAD model referenced in the docs is
BigTreeTech's own, distributed from their
[smart‑filament‑detection‑module](https://github.com/bigtreetech/smart-filament-detection-module)
repository under their terms.
