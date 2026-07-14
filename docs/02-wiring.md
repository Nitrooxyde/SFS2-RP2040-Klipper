# 02 — Wiring SFS 2.0 → RP2040‑Zero

## ⚠️ Two rules that protect the hardware

1. **VCC = 3.3 V. Never 5 V.**
   The RP2040 is not 5 V‑tolerant (abs max ≈ 3.8 V). The SFS is rated 3.3–5 V and
   works fine at 3.3 V, so powering it from the Pico's **3V3** pin caps every signal
   at 3.3 V → safe by construction. 5 V on VCC would drive the motion/switch outputs
   to 5 V and **destroy the Pico's GPIO**.
2. **Internal pull‑ups on both signal pins** (`^` in Klipper).
   The SFS outputs are open‑collector / switch‑to‑ground. Without a pull‑up they
   float and read garbage.

---

## Pin map (decided for this build)

| SFS wire | Signal | → RP2040‑Zero pin | Klipper pin |
|----------|--------|-------------------|-------------|
| RED | VCC | **3V3** (⚠ not 5V/VBUS) | — |
| BLACK | GND | **GND** | — |
| GREEN | Motion / encoder | **GP0** | `^sfs:gpio0` |
| BLUE | Switch / runout | **GP1** | `^sfs:gpio1` |

> GP0 / GP1 are the RP2040 UART0 pins, but the Pico runs as a **USB** MCU here, so
> UART0 is free — they are used purely as digital inputs with pull‑up.

```
   BTT SFS 2.0                         RP2040-Zero (USB)
 ┌──────────────┐                    ┌──────────────────┐
 │  motion  ────┼── GREEN ──────────►│ GP0  (^sfs:gpio0)│
 │  switch  ────┼── BLUE  ──────────►│ GP1  (^sfs:gpio1)│
 │  VCC     ────┼── RED   ──────────►│ 3V3   ⚠ NOT 5V    │
 │  GND     ────┼── BLACK ──────────►│ GND              │
 └──────────────┘                    │ USB-C ──► Raspberry Pi
                                      └──────────────────┘
```

The SFS's Y‑cable ends in **two 3‑pin** connectors (motion group and switch group),
each carrying its signal + a VCC + a GND. You only need **one** VCC and **one** GND to
the Pico; tie the two grounds together and the two VCCs together at 3V3.

---

## Bring‑up safety procedure (recommended)

Do this **before** trusting the signals — especially important because the RP2040 is
not 5 V‑tolerant:

1. **Power only** first: RED → 3V3, BLACK → GND. Leave GREEN/BLUE **unplugged**.
2. If you are unsure whether the SFS outputs are truly open‑collector, measure
   **output → GND** with a multimeter while powered:
   - ≈ 0 V / floating = open‑collector → **safe to connect** to the Pico.
   - ≈ VCC (push‑pull) = only safe because VCC is 3.3 V; would be fatal at 5 V.
3. Connect GREEN → GP0, BLUE → GP1.
4. Bring the sensor up in Klipper with `pause_on_runout: False` and just watch the
   state in Mainsail (see [docs/04](docs/04-klipper-integration.md)) before arming
   any auto‑pause.

> **Cross‑check the wire colours against your actual cable** — colour conventions can
> vary between SFS batches. The signal *function* (open‑collector switch to GND, one
> for presence, one for motion) is what matters; map whichever wire is which on your
> unit.
