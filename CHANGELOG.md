# Changelog / build history

Chronology of how the SFS 2.0 + RP2040‑Zero USB filament sensor came together
(reference build: Voron 2.4, Raspberry Pi host).

## 2026‑07‑02 — Decision & scope
- Chose to read the **BTT SFS 2.0** with a dedicated **RP2040‑Zero flashed as a
  Klipper USB MCU** — a self‑contained, plug‑and‑play sensor module instead of wiring
  the two signals back to the main board. Two signals →
  `[filament_motion_sensor]` (clog/slip) + `[filament_switch_sensor]` (presence).
- Imported the official **SFS V2.0 Model.STEP** and the RP2040‑Zero model in FreeCAD.
  Cross‑verified SFS dimensions three ways (live measurement == STEP == manual).

## 2026‑07‑02 → 07‑03 — Receptacle CAD
- First bracket attempt (boolean patch‑on‑patch) was **rejected** and the part
  **rebuilt from scratch parametrically**, face by face, as `REC_body_v1` in
  `SFS20_Support_V2.FCStd`.
- Built up: pico cavity, USB‑C opening (real connector contour), 2× M6 filament ports,
  2× M3 mounting tunnels (Ø6 flat counterbore for M3×12 button‑head), 2× BOOT/RESET
  access holes, 4× PCB standoffs, mouth alignment + R0.3 front fillet.

## 2026‑07‑03 — STL exports
- `REC_body_v2.stl` (64 160 facets) exported.
- Later superseded by the re‑exported `REC_body_v1.stl` (see 07‑04).

## 2026‑07‑04 — Firmware, wiring decision, refinements
- **SFS spec verified from the official manual (PDF):** rated **3.3–5 V**, detection
  length **2.88 mm**. Conclusion: power the SFS at **3.3 V** for the (non‑5 V‑tolerant)
  RP2040.
- **Pin mapping decided:** motion → **GP0**, switch → **GP1**, pull‑ups on both.
- **RP2040‑Zero flashed** stock Klipper USB + Katapult →
  serial `usb-Klipper_rp2040_5303284738FB5E1C-if00`. Re‑tested OK.
- Receptacle refined: M3 tunnels' counterbores corrected, outer wall reinforced to
  ~3.45 mm, BOOT/RESET holes ovalised 3.0 × 5.0, PCB standoffs re‑trimmed to seat on
  the board face (Y 56.60 + 0.10 clearance), **2020 mounting tab** added.
- `printer.cfg` bring‑up block prepared (safe mode, `pause_on_runout: False`).
- **`REC_body_v1.stl` re‑exported** (63 536 facets, watertight) — the current print
  file.
- USB‑C opening widened by 0.3 mm (commit `elargir_usbc_+0.3`); later noted the
  *tight* axis is the ~3.4 mm connector‑height axis, not the one widened.

## 2026‑07‑05 — Deployed
- **Sensor module deployed on the reference printer at VCC 3.3 V, both sensor objects
  enabled in Klipper.**
- `sfs_motion` confirmed as the *"advances vs slips"* discriminator during manual
  load/unload jogs (its motion state updates even when the sensor is `ENABLE=0`).
- Confirmed empirically: **PTFE ID2/OD4 guide into the SFS bore is mandatory** on push
  (without it the filament buckles at the junction).

## 2026‑07‑15 — This repo
- Consolidated hardware, wiring, firmware, Klipper integration and CAD into a
  standalone, documented repository. Bracket bundled as print‑ready STL + FreeCAD
  source; Klipper config as a ready‑to‑include snippet.
- Documentation reframed: the module is a **generic standalone USB filament sensor** —
  application‑specific context from the reference printer (multi‑material feed
  experiments) removed from the docs; what you build on the two signals is up to
  your config.

---

### Notes on state
- The receptacle is **functional**; the FreeCAD source (`SFS20_Support_V2.FCStd`) and
  the print‑ready STL are both in `cad/`.
- The config to use is [`klipper/sfs.cfg`](klipper/sfs.cfg) — adapt the serial to your
  board.
