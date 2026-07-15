# TeamFDM reply — thread #6534 "BTT SFS 2.0 with Leviathan mainboard"

> À poster en réponse au thread (compte user). Court : un apport tech sur leur
> problème + le repo en une ligne, comme alternative.
> https://www.teamfdm.com/forums/topic/6534-btt-sfs-20-with-leviathan-mainboard/

---

Small data point for future readers on the voltage side: the SFS 2.0 is
officially rated **3.3–5 V** (BTT manual, p.5 — the "5V" in their wiring
diagrams is just because the example boards are 5V-tolerant STM32s). Mine runs
fine on a clean, regulated 3.3 V — so if it doesn't work from a board's 3.3 V
pin, it's likely that rail sagging under load rather than the sensor needing 5 V.

Alternative approach if someone prefers to skip board pins entirely: I run mine
from a $3 RP2040-Zero flashed as a Klipper USB MCU (sensor powered by the pico's
3V3, one USB cable to the Pi). Write-up + printed mount here if it can help:
https://github.com/Nitrooxyde/BTT-SFS-2.0-RP2040-Zero-Klipper
