# TeamFDM reply — thread #6534 "BTT SFS 2.0 with Leviathan mainboard"

> À poster en réponse au thread (compte user). Ton : retour d'expérience, pas pub.
> https://www.teamfdm.com/forums/topic/6534-btt-sfs-20-with-leviathan-mainboard/

---

I ran into the same kind of trouble with the SFS 2.0 (dead dedicated header,
rails not quite what the label says, hunting for free pins…) and ended up going
a different route entirely: I gave the sensor **its own MCU**.

A Waveshare **RP2040-Zero** (~$3) flashed as a standard **Klipper USB MCU** reads
both SFS signals directly:

```ini
[mcu sfs]
serial: /dev/serial/by-id/usb-Klipper_rp2040_XXXX-if00

[filament_motion_sensor sfs_motion]
switch_pin: ^sfs:gpio0
detection_length: 2.88
extruder: extruder

[filament_switch_sensor sfs_switch]
switch_pin: ^sfs:gpio1
```

- **Zero mainboard pins**, zero mainboard rails — one USB cable to the Pi.
- The SFS is powered from the pico's clean **3.3 V** (it's officially rated
  3.3–5 V per the BTT manual, p.5 — the "5V" in BTT's diagrams is just because
  their example boards are 5V-tolerant STM32s). ⚠️ If you copy this setup, do
  power it at 3.3 V: the RP2040 is *not* 5V-tolerant.
- Both signals are open-collector, so the `^` internal pull-ups do the rest.
- Works the same on any printer/board — nothing Leviathan-specific to figure out.

I designed a one-piece printed receptacle that holds the SFS + pico together and
mounts on a 2020 extrusion, and wrote up the whole thing (wiring, flashing with
Katapult, config):

- GitHub (docs + config + STL): https://github.com/Nitrooxyde/BTT-SFS-2.0-RP2040-Zero-Klipper
- Printables: https://www.printables.com/model/1781579-btt-sfs-20-rp2040-zero-receptacle

Full disclosure: my own project, free/open (MIT + CC-BY). Hope it helps someone
who lands here with the same wiring headache.
