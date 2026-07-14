# 04 — Klipper integration

## Config block

Add the following to `printer.cfg`, or drop [`klipper/sfs.cfg`](../klipper/sfs.cfg)
next to it and `[include sfs.cfg]`.

```ini
[mcu sfs]
serial: /dev/serial/by-id/usb-Klipper_rp2040_5303284738FB5E1C-if00   # <-- set to YOUR board

[filament_motion_sensor sfs_motion]
switch_pin: ^sfs:gpio0        # GREEN — motion encoder (open-collector → pull-up ^)
detection_length: 2.88        # mm; SFS theoretical pulse pitch. Tune at the bench.
extruder: extruder
pause_on_runout: False        # bring-up: observe, don't auto-pause yet

[filament_switch_sensor sfs_switch]
switch_pin: ^sfs:gpio1        # BLUE — runout switch (open-collector → pull-up ^)
pause_on_runout: False
```

Points to get right:
- **The `^` (internal pull‑up) is mandatory** on both pins — the SFS outputs are
  switch‑to‑ground. Drop it and the inputs float.
- **`sfs:` prefix** = the alias of the `[mcu sfs]` block. `gpio0`/`gpio1` are the
  RP2040 GP0/GP1 pads (see [docs/02](02-wiring.md)).
- **`detection_length`** starts at the theoretical 2.88 mm. Raise it if you get false
  runout triggers (a longer window tolerates more slack/jitter before flagging
  no‑movement); it is a Klipper *debounce distance*, not a physical measurement.

## How Klipper reads these

- `[filament_switch_sensor sfs_switch]` — simple presence: `filament_detected`
  true/false. Fires runout logic when it goes empty.
- `[filament_motion_sensor sfs_motion]` — **not an instantaneous measurement**. On
  each encoder pulse it sets a target = *current extruder position + `detection_length`*,
  then every **0.25 s** checks whether the extruder passed that target **without** a new
  pulse. If it did → no‑movement → runout event. So the signal is a **debounced proxy**:
  confirmation arrives within the next few mm of extrusion, not at the exact instant.
  **Mount the sensor close to the extruder** when you can: the less slack between the
  encoder wheel and the drive gear, the tighter `detection_length` can be and the
  earlier a real no‑movement condition is flagged.

> The motion sensor's `filament_detected` state updates **even when the sensor is
> `ENABLE=0`** (in Klipper's source, `RunoutHelper.note_filament_present()` records
> the state *before* the `sensor_enabled` gate — only runout/insert **event
> processing** is disabled). That is what makes `sfs_motion` a usable
> *“it slips vs it advances”* probe during manual feed jogs.
>
> **Two limits to know** (per Klipper's implementation):
> - The **no‑movement check is forward‑only**: it flags when the extruder position
>   passes *last pulse position + `detection_length`*. During a **retract/unload**
>   the position moves backwards and never passes the target, so `filament_detected`
>   will not go False from lack of wheel movement — for unload verification, watch the
>   **pulse‑driven transitions** (each encoder pulse re‑asserts presence) instead.
> - Moves must go **through the extruder** (`G1 E…` while the printer is in the
>   *Printing* idle state). `FORCE_MOVE` neither triggers the check nor updates the
>   tracked extruder position.

## Safe bring‑up, then arm

1. Deploy with **`pause_on_runout: False`** on both. Nothing auto‑pauses.
2. Optionally add a `RESPOND` echo to *see* transitions in Mainsail without side
   effects while you confirm wiring and polarity:

   ```ini
   [filament_switch_sensor sfs_switch]
   switch_pin: ^sfs:gpio1
   pause_on_runout: False
   runout_gcode:  RESPOND MSG="SFS switch: RUNOUT"
   insert_gcode:  RESPOND MSG="SFS switch: filament present"
   ```

3. Watch the objects live:

   ```
   QUERY_FILAMENT_SENSOR SENSOR=sfs_switch
   QUERY_FILAMENT_SENSOR SENSOR=sfs_motion
   SET_FILAMENT_SENSOR SENSOR=sfs_motion ENABLE=1
   ```

   Or in the Mainsail console / status panel.
4. When wiring, polarity and `detection_length` are validated, **arm** it by setting
   `pause_on_runout: True` — or point `runout_gcode` at your own macros instead of a
   raw pause.

## Deploy checklist (Voron / Pi hygiene)

- **Edit `printer.cfg` via Mainsail or `sed`/`cat` on the Pi — not SCP‑overwrite.**
- **Back up first**, then edit.
- **Confirm `print_stats.state == 'standby'`** before `FIRMWARE_RESTART`.
- Never restart the machine mid‑print to add the sensor.

## What to build on top

The two objects are ordinary Klipper sensors — use them for whatever your setup needs:

- **runout → pause** — the classic use (`pause_on_runout: True` once validated),
- **clog / jam detection** — extruder commanded to move, encoder reports no movement,
- **slip / under‑extrusion detection** — commanded‑vs‑wheel differential,
- **feed verification in custom macros** — `sfs_motion`'s state keeps updating even
  when the sensor is `ENABLE=0`, so any automation (load/unload sequences,
  multi‑material feeds, …) can query *“is filament actually moving?”* without arming
  runout side effects. Mind the two limits above: no‑movement detection is
  **forward‑only** (during retracts, watch pulse‑driven transitions instead), and
  only extruder‑driven moves count.

> **Note on metering:** the SFS does **not** count meters. Klipper accumulates nothing
> from it (the wheel is ~1 pulse / 2.88 mm and slips), so real consumption still comes
> from `print_stats.filament_used` (or a spool manager). The SFS's unique value is the
> *commanded‑vs‑wheel differential* → detect slip / under‑extrusion.
