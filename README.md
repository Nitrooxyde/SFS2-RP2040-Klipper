# SFS 2.0 + RP2040‑Zero — Post‑Y Filament Sensor for Klipper

![Klipper](https://img.shields.io/badge/Klipper-USB%20MCU-orange)
![MCU](https://img.shields.io/badge/MCU-RP2040--Zero-green)
![Sensor](https://img.shields.io/badge/sensor-BTT%20SFS%202.0-blue)
![Status](https://img.shields.io/badge/status-functional-brightgreen)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

A **plug‑and‑play, single‑USB post‑extruder filament sensor** for Voron / Klipper
setups: a **BigTreeTech Smart Filament Sensor (SFS) 2.0** read by a
**Waveshare RP2040‑Zero** flashed as a standard **Klipper USB MCU**, housed in a
**custom one‑piece printed receptacle**.

The SFS exposes two signals — a **runout switch** (filament present / absent) and a
**motion encoder** (~1 pulse / 2.88 mm of filament travel). The RP2040‑Zero turns
both into native Klipper objects over a single USB cable:

- `[filament_switch_sensor sfs_switch]` → presence on the common trunk
- `[filament_motion_sensor sfs_motion]` → clog / slip / no‑movement detection

No extra wiring to the main board, no free GPIO required on the toolhead MCU —
the sensor is its own little USB microcontroller. This is the **post‑Y sensor**
of a larger multi‑lane / endless‑spool architecture (see *The bigger picture*).

---

## Why this exists

The mechanical drive (Orbiter) and the buffer (Mellow LLL Plus) can tell you a lot,
but neither can reliably answer *“is filament actually moving through the hotend
feed path, right now?”* after the Y‑junction. A dedicated **motion encoder placed
post‑Y, close to the extruder** is the honest discriminator between **“it advances”**
and **“it slips”** — precisely the signal needed to:

- pause on a real runout / clog instead of air‑printing,
- confirm which lane is actually feeding during an endless‑spool swap,
- validate load / unload sequences (the motion signal moves even when the switch
  sensor is `ENABLE=0`, verified in Klipper source).

Doing it on its own RP2040‑Zero over USB keeps the whole thing **full‑USB and
plug‑and‑play**, consistent with the rest of the rig (Eddy, LLL Plus, EBB — all USB
or CAN MCUs).

---

## What's in this repo

| Path | Contents |
|------|----------|
| [`docs/01-hardware.md`](docs/01-hardware.md) | SFS 2.0 + RP2040‑Zero specs, verified dimensions, BOM |
| [`docs/02-wiring.md`](docs/02-wiring.md) | SFS → Pico wiring, **the 3V3 rule**, pin map |
| [`docs/03-firmware-flashing.md`](docs/03-firmware-flashing.md) | Building & flashing Klipper on the RP2040‑Zero (Katapult) |
| [`docs/04-klipper-integration.md`](docs/04-klipper-integration.md) | `printer.cfg` blocks, safe bring‑up, usage & macros |
| [`docs/05-cad-bracket.md`](docs/05-cad-bracket.md) | The printed receptacle: features, print settings, source |
| [`klipper/sfs.cfg`](klipper/sfs.cfg) | Ready‑to‑`[include]` Klipper config snippet |
| [`cad/REC_body_v1.stl`](cad/) | Print‑ready one‑piece receptacle (SFS 2.0 + RP2040‑Zero) |
| [`cad/SFS20_Support_V2.FCStd`](cad/) | FreeCAD source of the receptacle |
| [`firmware/`](firmware/) | Notes on the stock‑Klipper RP2040 build (no custom C) |

---

## Quick start (TL;DR)

1. **Print** `cad/REC_body_v1.stl` (ABS/ASA, 4 walls, 0.2 mm — see
   [docs/05](docs/05-cad-bracket.md)).
2. **Flash** the RP2040‑Zero with **stock Klipper, RP2040, USB, Katapult bootloader**
   ([docs/03](docs/03-firmware-flashing.md)).
3. **Wire** SFS → Pico: **VCC → 3V3 (never 5V)**, GND → GND, motion → **GP0**,
   switch → **GP1** ([docs/02](docs/02-wiring.md)).
4. **Add** the config below to `printer.cfg` (or `[include klipper/sfs.cfg]`),
   fix the serial to *your* board, `FIRMWARE_RESTART`.

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

## The bigger picture — multi‑lane without a selector

This sensor is one brick of a larger goal: **multicolor / multi‑material on a single
direct‑drive toolhead without a selector MMU** — no ERCF, no BoxTurtle, no AMS.
One **Mellow LLL Plus** buffer per lane, a passive splitter, Klipper orchestrating
roles, and — after the Y‑junction — **this SFS** as the single source of truth for
*“filament is moving”*.

In that architecture the SFS enables the **endless‑spool 2‑sensor handshake**:
the **LLL entrance sensor** arms a successor lane on runout, and the **post‑Y SFS
motion** confirms the queue has passed the Y and executes / validates the swap.

Related work in the rig:
- **Mellow FLY LLL Buffer Plus** — canonical Klipper MCU firmware (autonomous
  auto‑feed + full host control).
- Passive splitter, per‑lane roles, `[filament_switch_sensor lll_entrance]`, `O2_sensor`.

---

## Status

**Functional.** The receptacle prints and fits; the RP2040‑Zero runs stock Klipper
over USB (Katapult in place, no BOOT button needed for reflash); both sensor objects
come up in Klipper with the SFS powered at 3V3. The bracket can still be *complemented*
(pico Y‑retention, optional flush lip) but is a working part.

See [`CHANGELOG.md`](CHANGELOG.md) for the full build history.

---

## License

MIT — see [`LICENSE`](LICENSE). The BTT SFS 2.0 CAD model referenced in the docs is
BigTreeTech's own, distributed from their
[smart‑filament‑detection‑module](https://github.com/bigtreetech/smart-filament-detection-module)
repository under their terms.
