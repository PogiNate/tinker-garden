---
title: Retro NFC Music Player - Frontplate Design - v3
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-frontplate-design-v3
tags:
- frontplate
- enclosure
- 3d-printing
- openscad
- retro-nfc-music-player
---

# Retro NFC Music Player — Frontplate Design — v3

Supersedes [[Frontplate Design - v2|Retro NFC Music Player - Frontplate Design - v2]]. Compact tabletop-radio faceplate, **220 × 185 × 4mm**. Parametric OpenSCAD source (`frontplate.scad`). Printer envelope 256³.

## Changes from v2

| Change | Rationale |
|---|---|
| **LED matrix dropped** | Too large for the overall design. Removes the top band entirely and cascades into the wiring and software (see below) |
| Plate 240×200 → **220×185** | Narrower and shorter now that the matrix isn't setting minimum width |
| OLED given a **recessed front bezel** (88 × 46, 1.2mm deep) | With the matrix gone the OLED is the centerpiece; a raw 56×29 hole read as sparse and undersized. The bezel gives it visual weight like a tuner window |
| Speaker slat grilles → **plain openings + rear mesh rebate** | Printed slats replaced by real speaker mesh: 46 × 96mm rebate 1.5mm deep on the back; mesh drops in and the speaker bolts over it, clamped by the ear bosses — no glue |
| Volume buttons → **vertical pair on the right** | Balances the power button on the left; transport row is now just 3 buttons (prev · play/pause · next) centered |

**200mm width was tried and rejected:** each speaker's ear bosses claim ~60.5mm of width (52.5mm hole spacing + 8mm boss OD), so two speakers plus a usable center column need ~215mm minimum. 220 leaves ~7.75mm edge margin.

## Component placement (X, Y from BOTTOM-LEFT of front face)

| Feature | Center | Spec |
|---|---|---|
| OLED window | 110, 134 | 56 × 29 through-window, 1mm front chamfer |
| OLED bezel | 110, 134 | 88 × 46 front recess, 1.2mm deep, r3 |
| OLED seating pocket | 110, 134 | 64 × 37 rear recess, 1.5mm deep |
| OLED bosses | ±26, ±16 | 4× M2.5 |
| NFC tap zone | 110, 78 | Ø45 engraved ring front; Ø52 × 2mm rear pocket; 4× M2.5 bosses at 36 × 34 |
| Speaker L / R | 38, 108 / 182, 108 | 40 × 90 opening (r3); 46 × 96 × 1.5mm rear mesh rebate; 4× M3 ear bosses, 5mm standoff |
| Power button | 35, 24 | Ø12.4 |
| Transport row | 90 / 110 / 130, 24 | Ø12.4, 20mm pitch — prev · play/pause · next |
| Volume up / down | 185, 34 / 185, 14 | Ø12.4, stacked vertically, 20mm pitch |

Model verified as **one watertight solid**, bbox 220 × 185 × 10mm.

## Still to measure

- [ ] Speaker **ear screw-hole spacing** — estimated 52.5 × 107.4; this drives plate width, so confirm before printing
- [ ] OLED board hole spacing + **active-area offset from board edge** (may shift the window within the bezel)
- [ ] PN532 board hole spacing (est. 36 × 34)
- [ ] Cabinet body wall thickness at mating edge

## Print / assembly

PLA or PETG, 0.2mm layers, 4–5 perimeters, brim. Face-down orientation. NFC pocket ceiling bridges 52mm — support there only if your printer bridges poorly. Heat-set inserts: M2.5 → 3.4mm bores, M3 → 4.0mm bores. Speaker mesh cut to ~46 × 96mm.

Wiring tie-in: dry-fit all components, position the Pi stack + [[Wiring Reference|Perma-Proto]] behind the plate, then cut the harness to measured length + 20mm service slack.
