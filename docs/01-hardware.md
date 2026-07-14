# 01 — Hardware & specifications

## Bill of materials

| Qty | Part | Notes |
|-----|------|-------|
| 1 | **BigTreeTech Smart Filament Sensor (SFS) 2.0** | Owned. Runout switch + motion encoder in one body. |
| 1 | **Waveshare RP2040‑Zero** | The USB MCU. 18 × 23.5 mm. **Not 5 V‑tolerant.** |
| 1 | Printed receptacle | `cad/REC_body_v1.stl` — houses both parts as one piece. |
| 2 | **PC4‑M6** pneumatic fittings | Supplied with the SFS; one at each filament port. |
| 1 | **PTFE tube ID2 / OD4** | **Mandatory** guide into the SFS bore (see below). |
| 2 | M3 screws | Fix the receptacle to the SFS mounting face (self‑tapping into the SFS). |
| 2 | M3 screws + T‑nuts | Fix the receptacle's tab to a 2020 extrusion. |
| 1 | USB‑C cable | RP2040‑Zero → Raspberry Pi. |
| — | Dupont / JST‑to‑bare wiring | SFS's two 3‑pin leads → RP2040‑Zero pads. |

> The BTT SFS 2.0 ships with 3× M3×6 screws, 2× PC4‑M6, and a PTFE ID2/OD4/L30 tube.

---

## BTT SFS 2.0 — verified specifications

Cross‑checked three ways: live measurement of the official STEP in FreeCAD ==
BTT STEP == BTT manual (`SFS V2.0 …_20231123.pdf`).

### Mechanical
- **Body:** X 53.5 · Y 30.3 · Z 27.3 mm (BTT official: 53.1 × 30.3 × 27.3). ~36 g.
- **Mounting face:** the large 53 × 27 face (normal +Y, area ≈ 1054 mm²).
- **3× M3 self‑tapping holes** in that face (pilot Ø2.7, head counterbore ≈ Ø5,
  blind into the plastic — **no inserts**; BTT supplies 3× M3×6):

  | Hole | X (mm) | Z (mm) |
  |------|--------|--------|
  | H1 | −61.7 | +6.7 |
  | H2 | −37.0 | +6.7 |
  | H3 | −37.0 | −5.7 |

  Spacing (BTT manual): **24.7 mm × 12.4 mm**, L‑shaped pattern.

- **Filament path = X axis** (inline pass‑through, bidirectional, 1.75 mm).
  M6 ports at **both ends** (accept the 2× PC4‑M6). The two ports are **diagonally
  offset** (the filament routes around the encoder): Port A ≈ (Y 22.0, Z +6.3),
  Port B ≈ (Y 36.9, Z −6.3). Internal bore: Ø1.75 guide → Ø3.4 groove → Ø4.6/5.0
  PTFE seat.

### The PTFE guide is not optional (empirically confirmed)
On **push** (loading), any un‑guided gap between the fitting and the internal bore
becomes a **buckling point** — the filament escapes at the junction instead of
advancing. Adding the **PTFE ID2/OD4 tube all the way into the internal bore** (not
just seated in the PC4‑M6) unblocked pushing in real use. Invisible on **pull**
(unloading), so easy to under‑estimate at design time — **fit it on both ports.**

### Electrical — **read this before wiring**
- **Rated voltage: 3.3 V – 5 V ONLY.** The manual (p.5 *Specifications*; p.25
  *Caution*) is explicit: *“any higher voltage will cause damage.”*
  → **The SFS works officially at 3.3 V** — the "5V" label on BTT's wiring diagrams
  is because their host boards are STM32 (5 V rail + 5 V‑tolerant FT pins), **not**
  a power‑supply requirement.
- **Detection length:** 2.88 mm (theoretical) per encoder pulse.
- **Cable:** 100 cm Y‑cable — 1× **4‑pin** at the sensor → 2× **3‑pin** to the board.
- **Wire colours / signals:**

  | Colour | Signal | Notes |
  |--------|--------|-------|
  | **RED** | VCC | **Feed 3.3 V for the RP2040** (see wiring). |
  | **BLACK** | GND | |
  | **BLUE** | Switch / runout | Active‑low, **pull‑up required**. |
  | **GREEN** | Motion / encoder | Toggles ~1 pulse / 2.88 mm, **pull‑up required**. |

  Both outputs behave as **switch‑to‑ground / open‑drain** → an **MCU pull‑up is
  mandatory** (the RP2040's internal pull‑ups do the job, hence the `^` prefix in
  the Klipper config).

- **On‑board LEDs** (on the label face): red = empty; blue blinking = extrusion OK;
  blue solid / off = no movement. Detection is internal (no external optical window).

> **Not documented / confirm visually if you rebuild the cable:** the exact JST
> series of the 4‑pin (XH 2.54 vs PH 2.0) and the exact cable‑exit face/orientation.

### Official CAD / manual
BigTreeTech repo: [`bigtreetech/smart-filament-detection-module`](https://github.com/bigtreetech/smart-filament-detection-module)
- `/V2.0/CAD/SFS V2.0 Model.STEP` (the model used to design the receptacle)
- `/V2.0/CAD/SFS V2.0 Bracket for Hurakan.STEP` (BTT's own reference bracket)
- `/V2.0/STL`, `/V2.0/Manual/… _20231123.pdf`

The STEP model was kept locally at `Voron/SFS_V2_0/SFS V2.0 Model.STEP` (≈25 MB, not
bundled here to keep the repo lean).

---

## Waveshare RP2040‑Zero — specifications

- PCB **18 × 23.5 × ~1.6 mm**, **USB‑C** on a short edge.
- RP2040 + 2 MB flash, WS2812 (NeoPixel) LED + power LED, **BOOT/RESET** button.
- 9 castellated pads per side + through‑pads.
- **Logic 3.3 V. NOT 5 V‑tolerant** (RP2040 absolute max ≈ 3.8 V on any pin).
  This is *the* reason the SFS must be powered at 3.3 V here.

> The RP2040‑Zero STEP found online (`crides/kleeb`) has **11 mm male header pins**
> (a low‑profile keyboard variant) — those are a modelling artifact; the real Zero is
> ~1.6 mm bare. They were ignored when designing the receptacle.

### Third RP2040 on the rig — serial hygiene
This Pico is the **third** RP2040 Klipper MCU on the printer (after the **BTT Eddy**
and the **Fysetc PIS**). All three share VID:PID `1d50:614e` (Manufacturer "Klipper",
Product "rp2040"). **Always identify each board by its unique serial** before editing
a `[mcu …]` block — see [docs/03](docs/03-firmware-flashing.md).

Known serials on the reference rig:
- Octopus Pro (STM32): `usb-Klipper_stm32f446xx_300025001350324E31333220-if00`
- BTT Eddy (RP2040 #1): `usb-Klipper_rp2040_504450593844A21C-if00`
- Fysetc PIS (RP2040 #2): `usb-Klipper_rp2040_E663B03597314E27-if00`
- **This SFS Pico (RP2040 #3): `usb-Klipper_rp2040_5303284738FB5E1C-if00`**
