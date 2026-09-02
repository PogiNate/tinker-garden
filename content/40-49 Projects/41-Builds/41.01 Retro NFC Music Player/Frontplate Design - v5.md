---
title: Frontplate Design - v5
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-frontplate-design-v5
tags:
  - frontplate
  - enclosure
  - 3d-printing
  - openscad
  - retro-nfc-music-player
---

# Retro NFC Music Player — Frontplate Design — v5

Supersedes [[Frontplate Design - v4|Retro NFC Music Player - Frontplate Design - v4]]. **All component measurements now confirmed** — no estimated dimensions remain. 200 × 175 × 4mm plate (13.8mm deep including speaker posts). Parametric OpenSCAD (`frontplate.scad`); one watertight solid.

## The speaker mounting conflict, and how v5 solves it

The confirmed tab hole spacing is **35.9 × 70.7mm** (centers). The specified opening is 39 × 82. So the holes sit at ±17.95 / ±35.35 while the opening reaches ±19.5 / ±41 — **every tab hole falls inside the opening**, leaving nothing to screw into.

The measurements cross-check cleanly, so this isn't an error: 35.9mm centers imply 7.0mm-wide slots whose inner edges are 28.9mm apart and whose outer edges sit 0.8mm from the body edge — matching the "28.9mm between closest inner edges" and "1mm inset from outer edges" figures exactly.

**Solution:** four solid pads (12 × 12mm, r2, full plate thickness) intrude into each opening's corners, restoring material exactly where the tabs land. Each pad carries an **8mm-Ø post standing 9.8mm proud** of the plate back, since the tab flange sits 9.8mm behind the speaker's front face. M3 heat-set insert in each post, 6mm deep from the top. Behind the mesh the pads are invisible; from the front they read as four small notches in the opening corners.

Alternatives rejected: narrowing the opening to 28.9mm (would occlude the drivers), mounting the speakers to the cabinet body instead (defers the problem and loses the front-face clamp on the mesh).

⚠️ **The one assumption left:** that "screw holes inset from front edge: 9.8mm" is a *depth* measurement, like the "wires 4.8mm from front edge" note — i.e. the tab flange is set back from the front face. If the tabs are actually flush with the front face, set `spk_tab_z = 0` and reprint; everything else holds.

## Confirmed measurements applied

| Component | Measured | Applied |
|---|---|---|
| Speaker body | 44.5 × 99.5 × 20.7 | Opening 39 × 82; brackets at body corners |
| Speaker tabs | 35.9 × 70.7 centers, 7mm slots, 9.8mm behind front face | Pads + 9.8mm M3 posts |
| OLED board | 71 × 43.3 × 6 | 4× M2.5 bosses |
| OLED holes | **66.4 × 38.5** | Applied directly (2.3 / 2.4mm inset from board edges — symmetric, as expected) |
| OLED glass | **56.5 × 28.8** | Window **55.5 × 28** — 0.5mm lip on the sides, 0.4mm top/bottom |
| PN532 board | 42.6 × 40.6, DIP 3.9 proud | Back-to-plate mount, posts 3mm proud |
| PN532 holes | 3mm Ø, **top-left / bottom-right**, 6.0mm to inner edge | Centers 7.5mm in from each edge → **27.6 × 25.6**; M2.5 inserts (right size for a 3mm clearance hole) |
| Buttons | 11.5 Ø; 5mm thread + 1mm washer; power 10.4mm thread | Ø12.0 holes; Ø18 × 2mm rear counterbore on the five short buttons; power in full 4mm plate |

## Remaining verification (cheap, do at dry-fit)

- [ ] **OLED lit area.** Glass is 56.5 × 28.8; on a 2.42" 128×64 SSD1309 the lit rectangle should be near 55 × 27.5, so the 55.5 × 28 window should frame it with a hair to spare. Confirm once powered, and check whether the lit area is vertically centered on the glass — if it sits high or low, set `oled_win_off` rather than resizing.
- [ ] **Speaker tab depth** (the 9.8mm assumption above).
- [ ] Cabinet body wall thickness at the mating edge, to finalize corner-boss positions.

## Print / assembly

PLA or PETG, 0.2mm layers, 4–5 perimeters, brim, face-down. Heat-set inserts: M2.5 → 3.4mm bores, M3 → 4.0mm bores.

Speaker mesh: cut ~43 × 90mm with four small corner notches to clear the pads, drop into the rear rebate, seat the speaker over it inside the corner brackets, then screw through the tab slots into the posts. The slots give a few millimetres of adjustment.

**Print a test coupon first** with one counterbored button hole and one speaker post — those two features carry the tightest tolerances on the plate (2mm plate + 1mm washer + nut on a 5mm thread; and the 9.8mm post height).
