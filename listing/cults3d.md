# Fiche Cults3D — prête à coller

> Fichiers à uploader : `cad/SFS_Support_2020.stl`
> Images : `docs/img/receptacle-pico-cavity.jpg`, `docs/img/receptacle-usbc-opening.jpg`
> **+ recommandé : 2–3 photos réelles** (pièce imprimée, module monté sur le 2020, câblage) — les photos réelles font ×3 sur les téléchargements.
> Catégorie : **Outil → Accessoires pour imprimante 3D**
> Prix : Gratuit (suggestion) · Licence : **« CC - Attribution »** dans leur menu (= CC BY, le « 4.0 » n'est pas affiché ; cohérent avec le repo MIT). Alternative si tu veux interdire la vente par d'autres : « CC - Attribution - Pas d'utilisation commerciale ».

---

## Titre (FR)

Capteur de filament USB autonome pour Klipper — support BTT SFS 2.0 + RP2040-Zero

## Titre (EN)

Standalone USB Filament Sensor for Klipper — BTT SFS 2.0 + RP2040-Zero receptacle

---

## Description (FR)

**Un module capteur de filament complet, autonome, en une seule pièce imprimée — un câble USB et c'est branché.**

Ce support monobloc loge un **BigTreeTech Smart Filament Sensor 2.0** et un **Waveshare RP2040-Zero** flashé en MCU Klipper USB. Le pico lit les deux signaux du SFS (présence + mouvement) et les expose à Klipper comme objets natifs :

- `filament_switch_sensor` → présence du filament (runout)
- `filament_motion_sensor` → mouvement réel (bourrage, glissement, sous-extrusion)

**Pourquoi un MCU dédié ?**
- **Zéro pin nécessaire** sur votre carte principale ou toolhead
- **Un seul câble USB** — pas de fils de signaux à tirer à travers l'imprimante
- **Plug-and-play** : déplaçable d'une machine à l'autre
- Électriquement autonome : le SFS est alimenté en 3,3 V par le pico

**Caractéristiques du support :**
- Fixation sur **profilé 2020** (patte avec 2 trous M3 pour écrous T)
- Se visse sur la face de montage du SFS (2× M3×12 tête bombée, lamages intégrés)
- Cavité pour le RP2040-Zero avec ouverture USB-C au contour du connecteur
- Accès aux boutons BOOT/RESET (trous oblongs 3×5 mm)
- 2 ports filament M6 pour raccords PC4-M6

**Assemblage :** le pico se colle à la **colle chaude aux 4 coins** dans sa cavité (simple, ferme, démontable). Passez les guides PTFE ID2/OD4 jusqu'au fond des ports du SFS (indispensable en poussée).

**⚠️ Câblage : alimentez le SFS en 3,3 V uniquement** (le RP2040 n'est pas tolérant au 5 V). Câblage complet, flash Klipper/Katapult et config prête à l'emploi sur le GitHub :
👉 **github.com/Nitrooxyde/BTT-SFS-2.0-RP2040-Zero-Klipper**

**Impression :** ABS/ASA (ou PETG), 0,2 mm, 4 périmètres, 30–40 % de remplissage. Supports uniquement sur les ouvertures selon l'orientation. STL vérifié étanche (59 488 facettes). Encombrement : 69 × 48 × 25 mm.

**BOM :** SFS 2.0 · RP2040-Zero · 2× M3×12 tête bombée · 2× M3 + écrous T · 2× PC4-M6 (fournis avec le SFS) · PTFE ID2/OD4 · câble USB-C · colle chaude.

Testé en réel sur une Voron 2.4.

---

## Description (EN)

**A complete, self-contained filament sensor module in a single printed part — one USB cable and you're wired.**

This one-piece receptacle houses a **BigTreeTech Smart Filament Sensor 2.0** and a **Waveshare RP2040-Zero** flashed as a Klipper USB MCU. The pico reads both SFS signals (presence + motion) and exposes them to Klipper as native objects:

- `filament_switch_sensor` → filament presence (runout)
- `filament_motion_sensor` → actual movement (clog, slip, under-extrusion)

**Why a dedicated MCU?**
- **Zero pins needed** on your main or toolhead board
- **One USB cable** — no signal wires across the printer
- **Plug-and-play**: move it between machines
- Electrically self-contained: the SFS runs on 3.3 V from the pico

**Receptacle features:**
- Mounts on a **2020 extrusion** (tab with 2× M3 holes for T-nuts)
- Screws onto the SFS mounting face (2× M3×12 button-head, counterbores included)
- RP2040-Zero cavity with a USB-C opening cut to the connector contour
- BOOT/RESET button access (3×5 mm oval holes)
- 2× M6 filament ports for PC4-M6 fittings

**Assembly:** hot-glue the pico at its **four corners** in the cavity (simple, firm, serviceable). Run the PTFE ID2/OD4 guides all the way into the SFS bore (mandatory when pushing filament).

**⚠️ Wiring: power the SFS at 3.3 V only** (the RP2040 is not 5 V-tolerant). Full wiring, Klipper/Katapult flashing and a ready-to-include config on GitHub:
👉 **github.com/Nitrooxyde/BTT-SFS-2.0-RP2040-Zero-Klipper**

**Printing:** ABS/ASA (or PETG), 0.2 mm, 4 walls, 30–40 % infill. Supports only on the openings depending on orientation. STL verified watertight (59,488 facets). Footprint: 69 × 48 × 25 mm.

**BOM:** SFS 2.0 · RP2040-Zero · 2× M3×12 button-head · 2× M3 + T-nuts · 2× PC4-M6 (supplied with the SFS) · PTFE ID2/OD4 · USB-C cable · hot glue.

Tested for real on a Voron 2.4.

---

## Tags

`klipper` `voron` `filament sensor` `runout sensor` `btt sfs` `rp2040` `smart filament sensor` `2020 extrusion` `3d printer accessory` `filament runout`
