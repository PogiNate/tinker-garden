---
title: Retro NFC Music Player - Frontplate Design - v1
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-frontplate-design-v1
tags:
- frontplate
- enclosure
- 3d-printing
- openscad
- retro-nfc-music-player
---

# Retro NFC Music Player — Frontplate Design — v1

Phase 6 of [[Design and Project Plan - v2|Retro NFC Music Player - Design and Project Plan - v2]], pulled forward so panel components can be placed and the harness trimmed to measured lengths. Wide boombox landscape; all interfaces on the front face. Modeled in **OpenSCAD** (parametric — `frontplate.scad`); printer envelope 256×256×256mm.

## Plate

| Property | Value |
|---|---|
| Overall | **240 × 170 × 4mm**, 6mm corner radius |
| Print orientation | Face-down (smooth front off the bed), brim recommended |
| Fasteners | Heat-set inserts: M2.5 (matrix, OLED, PN532 bosses), M3 (cabinet-assembly corner bosses) |
| NFC pocket | Rear pocket thinned to **2mm** wall behind tap zone — PN532 reads through it |

## Component placement (X from left, Y from top, to feature center)

| Feature | Center (X, Y) | Opening / spec |
|---|---|---|
| Spectrum matrix window | 120, 30 | 132 × 36 (128×32 active + 2mm reveal), front edge chamfered |
| OLED window | 120, 77 | 58 × 30, front edge chamfered |
| Speaker grille L | 27, 67.5 | ~30 × 75 slat grille — **pending speaker measurement** |
| Speaker grille R | 213, 67.5 | ″ |
| Power button | 36, 135 | Ø12.4 hole (12mm panel-mount + LED ring) |
| Transport row (5×) | 80 / 100 / 120 / 140 / 160, 135 | Ø12.4 holes, 20mm pitch — order: prev · play/pause · next · vol− · vol+ |
| NFC tap zone | 204, 135 | Ø45 engraved ring on front; Ø52 × 2mm-deep pocket on rear |

Clearances verified: matrix PCB (128 wide) clears both speaker bodies; NFC ring clears right grille by ~12mm and transport row by ~20mm; 12mm buttons need ~15mm rear depth; matrix module is deepest front component (~15mm w/ headers).

## Open measurements (calipers before final STL)

- [ ] Speaker body: length × width × depth, and mounting (screw ears vs bare box → bosses vs retaining clips)
- [ ] Matrix module mounting-hole spacing (X × Y) and hole Ø
- [ ] OLED board mounting-hole spacing + active-area offset from board edge
- [ ] PN532 board mounting-hole spacing
- [ ] Cabinet body wall thickness at mating edge (sets corner-boss positions)

## Print settings guidance

PLA or PETG, 0.2mm layers, 4–5 perimeters (plate is mostly walls), 15–20% infill, no supports needed if windows are chamfered ≤45°. The 240mm span will want a brim or raft against warp; PETG preferred if the cabinet lives near speaker heat, though at 1W/channel that's negligible.

Wiring tie-in: dry-fit all components to this plate, position the Pi stack + [[Wiring Reference|Perma-Proto]] behind it, then cut harness to measured length + 20mm service slack.
