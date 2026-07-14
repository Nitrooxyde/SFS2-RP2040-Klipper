# firmware/

**There is no custom firmware in this project.** The RP2040‑Zero runs **stock
Klipper** compiled for the RP2040 as a **USB MCU**, with the **Katapult** bootloader
so reflashes need no BOOT button.

This folder exists only to hold the build recipe and (optionally) a captured
`.config` / binaries if you want to keep them versioned.

## Build recipe (menuconfig)

```
Micro-controller Architecture ....... Raspberry Pi RP2040
Bootloader offset ................... 16 KiB bootloader   (when using Katapult)
Communication interface ............. USB
```

```bash
cd ~/klipper
make menuconfig      # set the three options above
make clean && make   # -> out/klipper.uf2 / out/klipper.bin
```

Full flashing + Katapult + serial‑identification steps are in
[`../docs/03-firmware-flashing.md`](../docs/03-firmware-flashing.md).

## Reference board

- Serial: `usb-Klipper_rp2040_5303284738FB5E1C-if00`
- Katapult installed → future updates, no button, **two steps**: `make`, then
  `flashtool.py -d <klipper-serial> -r` (reboots the MCU into Katapult), then
  `flashtool.py -d <katapult-serial>` to flash — see
  [`../docs/03-firmware-flashing.md`](../docs/03-firmware-flashing.md).

> If you want a fully reproducible build, drop the Klipper `out/klipper.config`
> here and note the Klipper commit you built from.
