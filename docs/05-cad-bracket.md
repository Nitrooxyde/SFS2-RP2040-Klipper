# 05 — The printed receptacle

A **one‑piece receptacle** that holds the **SFS 2.0** and the **RP2040‑Zero** together
and mounts to a **2020 extrusion** — so the whole post‑Y sensor is a single printed
part with one USB cable leaving it.

- **Print file:** [`cad/REC_body_v1.stl`](../cad/REC_body_v1.stl)
  — binary STL, **63 536 facets**, watertight (FreeCAD `MeshPart` export, linear
  deviation 0.01 / angular 0.05; mesh vs BREP volume within ~0.01 %).
- **Source:** [`cad/SFS20_Support_V2.FCStd`](../cad/SFS20_Support_V2.FCStd)
  — object `REC_body_v1`.
- Approx. bounding box: **X ≈ 79 mm** (incl. the 2020 tab) · **Y ≈ 47 mm** ·
  **Z ≈ 25 mm**.

> Design note: this is the clean parametric reconstruction. An earlier
> patch‑on‑patch version (`SUP_bracket_v1`) was scrapped and the part was rebuilt
> from scratch, face by face, against the real SFS/RP2040 geometry.

## Features

- **RP2040‑Zero cavity** at the −X end, USB‑C exposed through an opening cut to the
  **real connector contour** (obround; the tight axis is the ~3.4 mm connector
  height — see note below).
- **2× M6 filament ports** matching the SFS bore, for the PC4‑M6 fittings.
  → Run the **PTFE ID2/OD4** guide all the way into the SFS bore on both
  (see [docs/01](01-hardware.md)).
- **2× M3 mounting tunnels** through the SFS mount face — Ø3.4 clearance shaft with a
  **flat Ø6 counterbore** for M3×12 button‑head screws (button head = flat‑bottom
  counterbore, never a conical countersink), ~6 mm thread engagement into the SFS.
- **2× BOOT/RESET access holes**, **ovalised 3.0 × 5.0 mm**, aligned to the Pico's
  KEY1/KEY2 buttons.
- **4× PCB standoffs**, trimmed to seat exactly on the board's top face
  (**Y = 56.60 mm + 0.10 mm clearance**) — no bottoming‑out on the PCB.
- **Reinforced outer wall ≈ 3.45 mm** for rigidity.
- **2020 mounting tab** on the −X face: extended +20 mm in X, **2× M3 through‑holes**
  for T‑nut mounting to the extrusion.
- Front mouth aligned to a single plane with a small **R0.3 fillet** on the front
  contour edges.

## Suggested print settings

This is a functional structural bracket, so print it strong:

| Setting | Value |
|---------|-------|
| Material | **ABS or ASA** (matches the toolhead environment; PETG works too) |
| Layer height | 0.2 mm |
| Walls / perimeters | **4** (wall count is the real strength lever, not layer height) |
| Infill | 30–40 % |
| Supports | Only where the USB‑C / port / button openings overhang, depending on
  orientation |

> Orientation is not prescriptive — pick the face that puts the mounting tab and the
> filament ports on clean surfaces and minimises support inside the pico cavity.

## Known‑open / can be complemented

The part is **functional** but was left with a few optional refinements never forced:

- **Pico Y‑retention** — the cavity is open; the RP2040 is held by the wall + USB‑C,
  add a clip/lid if you want positive retention.
- **−Y mouth flush lip** — a ~0.01 mm residual skin (1/20 of a layer, not sliced) was
  left because OCC refused a perfectly coincident cut; reconstruct the −Y face as a
  separate parametric frame if you want a geometric flush.
- Minor plane‑alignment nits (floor/ceiling ±10.728 vs ±10.7) — cosmetic.

## ⚠️ Print the up‑to‑date STL, from the right place
When rebuilding/exporting: the FreeCAD export on the headless server writes into the
**snap sandbox** (`~/snap/freecad/2337/…`), *not* the real home, and writing to the
home root is blocked by AppArmor. Export to `~/Desktop/…` (SCP‑accessible) and
**always re‑read the file on disk** (facet count / size / date) to prove the content
before printing — do not trust the "export succeeded" message alone. (A stale STL once
got printed for 1h40 this way.)
