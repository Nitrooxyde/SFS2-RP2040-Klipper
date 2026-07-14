# Printables listing — ready to paste

> Files to upload: `cad/SFS_Support_2020.stl`
> Images: `docs/img/receptacle-pico-cavity.jpg`, `docs/img/receptacle-usbc-opening.jpg`
> **+ recommended: 2–3 real photos** (printed part, module mounted on the 2020, wiring) — real photos massively boost downloads.
> Category: **3D Printers → Printer Accessories**
> License: pick **"Creative Commons — Attribution"** in the dropdown (= CC BY; the "4.0" version number isn't shown; consistent with the MIT repo). Alternative if you want to forbid others selling it: "Creative Commons — Attribution — NonCommercial".

---

## Title

Standalone USB Filament Sensor for Klipper — BTT SFS 2.0 + RP2040-Zero Receptacle

## Summary (short field)

One-piece receptacle for a BTT SFS 2.0 + RP2040-Zero running Klipper over USB. One cable in → native runout + motion sensors in Klipper. Zero pins needed on your boards. Mounts on 2020 extrusion.

---

## Description

**A complete, self-contained filament sensor module in a single printed part — one USB cable and you're wired.**

This receptacle houses a **BigTreeTech Smart Filament Sensor 2.0** and a **Waveshare RP2040-Zero** flashed as a standard Klipper USB MCU. The pico reads both SFS signals and exposes them to Klipper as native objects:

- `filament_switch_sensor` → filament presence (runout)
- `filament_motion_sensor` → actual movement (clog, slip, under-extrusion detection)

### Why a dedicated MCU?

- **Zero pins needed** on your main or toolhead board
- **One USB cable** — no signal wires across the printer
- **Plug-and-play** — move it between machines, or retrofit a printer never wired for it
- Electrically self-contained: the SFS runs on 3.3 V from the pico

### Receptacle features

- Mounts on a **2020 extrusion** (tab with 2× M3 holes for T-nuts)
- Screws onto the SFS mounting face — 2× M3×12 button-head, flat counterbores included
- RP2040-Zero cavity with a USB-C opening cut to the real connector contour
- BOOT/RESET button access holes (3×5 mm ovals)
- 2× M6 filament ports for the PC4-M6 fittings supplied with the SFS

### Assembly

1. Seat the pico in its cavity (USB-C through the end-wall opening) and **hot-glue it
   at the four corners** — simple, firm, and serviceable.
2. Screw the receptacle to the SFS with 2× M3×12 button-head.
3. Mount on the 2020 with M3 + T-nuts.
4. Run the **PTFE ID2/OD4 guides all the way into the SFS bore** on both ports —
   mandatory when pushing filament, otherwise it buckles at the junction.

### Wiring & Klipper config

**⚠️ Power the SFS at 3.3 V only** — the RP2040 is **not** 5 V-tolerant.
Motion → GP0, switch → GP1, internal pull-ups.

Full wiring guide, Klipper/Katapult flashing steps and a ready-to-include
`sfs.cfg` are on GitHub:
**https://github.com/Nitrooxyde/SFS2-RP2040-Klipper**

### BOM

| Qty | Part |
|-----|------|
| 1 | BigTreeTech SFS 2.0 |
| 1 | Waveshare RP2040-Zero |
| 2 | M3×12 button-head screws |
| 2 | M3 screws + T-nuts (2020 mounting) |
| 2 | PC4-M6 fittings (supplied with the SFS) |
| — | PTFE tube ID2/OD4 |
| 1 | USB-C cable |
| — | Hot glue |

Tested for real on a Voron 2.4. STL verified watertight (59,488 facets).
Footprint: 69 × 48 × 25 mm.

---

## Print settings (structured fields)

- **Printer:** any (designed on a Voron 2.4)
- **Nozzle:** 0.4 mm
- **Layer height:** 0.2 mm
- **Material:** ABS / ASA (PETG works)
- **Perimeters/walls:** 4
- **Infill:** 30–40 %
- **Supports:** only on the USB-C / port / button openings, depending on orientation
- **Brim:** optional

## Tags

klipper, voron, filament-sensor, runout-sensor, btt-sfs, rp2040, smart-filament-sensor, 2020-extrusion, printer-accessory, filament-runout
