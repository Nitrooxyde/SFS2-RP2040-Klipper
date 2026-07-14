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
   - Klipper directly (no Katapult): **rebuild with Bootloader offset =
     `No bootloader`**, then copy `out/klipper.uf2` onto `RPI-RP2`. (A 16 KiB‑offset
     build copied to a bare board will not boot — there is nothing at the start of
     flash.)

### Katapult (so you never need BOOT again)

**Step 1 — flash Katapult once (BOOT‑button method).**
Build Katapult for the RP2040 (`~/katapult`, `make menuconfig` → RP2040, communication
**USB**), then put the Pico in RPI‑RP2 mode (hold BOOT, plug USB) and copy
`out/katapult.uf2` onto the `RPI-RP2` drive. The board now enumerates as a Katapult
device: `/dev/serial/by-id/usb-katapult_rp2040_XXXX-if00`.

**Step 2 — build Klipper with the 16 KiB offset** (so it sits after Katapult):
`make menuconfig` → RP2040, **Bootloader offset = 16 KiB**, **USB** → `make`
(produces `out/klipper.bin`, which is what Katapult flashes — not the `.uf2`).

**Step 3 — flash Klipper through Katapult** (flashtool defaults to
`~/klipper/out/klipper.bin` wherever you run it from; use `-f` to point elsewhere):
```bash
python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_rp2040_XXXX-if00
```
The board reboots into Klipper and now shows up as
`usb-Klipper_rp2040_XXXX-if00`.

**Later reflashes — no BOOT button, two steps.** With Klipper already running:

```bash
cd ~/klipper && make

# 1) ask the running Klipper MCU to reboot into Katapult (request only — exits)
python3 ~/katapult/scripts/flashtool.py \
    -d /dev/serial/by-id/usb-Klipper_rp2040_5303284738FB5E1C-if00 -r

# 2) the board re-enumerates as usb-katapult_rp2040_XXXX-if00 — flash through THAT
python3 ~/katapult/scripts/flashtool.py \
    -d /dev/serial/by-id/usb-katapult_rp2040_XXXX-if00
```

> `-r` only *requests* the bootloader and exits; over USB the board comes back under a
> **different serial** (`usb-katapult_…`), so the flash itself is always the second
> command. (One‑shot request+flash exists only for CAN devices, not USB.)

## Identify the board's serial (critical — 3rd RP2040 on the rig)

Three RP2040s share the same VID:PID. **Never** assume which `/dev/serial/by-id/`
entry is the SFS Pico.

```bash
# map every declared (uncommented) serial across all Klipper configs first
grep -rE '^\s*serial:' ~/printer_data/config/
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
