# 03 — Flashing Klipper on the RP2040‑Zero

There is **no custom firmware** here. The Pico runs **stock Klipper** as a USB MCU,
with the **Katapult** bootloader so future reflashes need no BOOT button.

## What is flashed on the reference board

- **Klipper**, target **Raspberry Pi RP2040**, communication **USB**.
- **Katapult** bootloader at the start of flash, Klipper offset by 16 KiB.
- Resulting serial: `usb-Klipper_rp2040_5303284738FB5E1C-if00`
  (yours will differ — read it from `/dev/serial/by-id/`).

## Build Klipper for the RP2040‑Zero

On the Pi (or wherever you build):

```bash
cd ~/klipper
make menuconfig
```

Set:
- **Micro-controller Architecture** → `Raspberry Pi RP2040`
- **Bootloader offset** → `16 KiB bootloader` *(if flashing on top of Katapult)*
- **Communication interface** → `USB`

Then:

```bash
make clean
make            # produces out/klipper.uf2 (and out/klipper.bin for Katapult)
```

## First flash (fresh board, UF2 over USB mass‑storage)

1. Hold **BOOT**, plug the USB‑C cable, release BOOT → the Pico mounts as a
   USB drive (`RPI-RP2`).
2. Copy the UF2 to it:
   - Katapult first (recommended): flash `katapult.uf2`, then flash Klipper via
     Katapult (below), **or**
   - Klipper directly: copy `out/klipper.uf2` onto `RPI-RP2`.

### Katapult (so you never need BOOT again)
Build Katapult for RP2040 (USB, 16 KiB), flash `katapult.uf2` once by the BOOT‑button
method above, then flash Klipper through it:

```bash
# find the Katapult device
ls /dev/serial/by-id/
python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_rp2040_XXXX-if00
# or, once Klipper is running, subsequent reflashes:
python3 ~/katapult/scripts/flash_can.py ...   # (USB variant: flashtool.py -d <serial>)
```

After this, every future update = `make` + `flashtool.py -d <serial>` — **no button.**

## Identify the board's serial (critical — 3rd RP2040 on the rig)

Three RP2040s share the same VID:PID. **Never** assume which `/dev/serial/by-id/`
entry is the SFS Pico.

```bash
# map every declared serial across all Klipper configs first
grep -rE 'serial' ~/printer_data/config/ | grep -v '^\s*#'
```

To positively identify the SFS Pico when several RP2040s are plugged:

1. `ls /dev/serial/by-id/` — note the list.
2. Unplug the SFS Pico's USB‑C.
3. `ls /dev/serial/by-id/` again — the entry that **disappeared** is the SFS Pico.
4. Use that exact serial in `[mcu sfs]`.

> **Why this matters:** on this rig, blindly reusing an RP2040 serial once made
> Klipper open the same CDC‑ACM port twice (Eddy + a second block), causing
> `Resource temporarily unavailable` loops and cascading MCU shutdowns that looked
> like a hardware/probe fault. Map serials → devices *first*, edit `printer.cfg`
> *second*.

Known serials on the reference rig are listed in
[docs/01-hardware.md](01-hardware.md#third-rp2040-on-the-rig--serial-hygiene).
