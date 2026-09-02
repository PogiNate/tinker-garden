---
title: Frontplate Design - v4
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-frontplate-design-v4
tags:
  - frontplate
  - enclosure
  - 3d-printing
  - openscad
  - retro-nfc-music-player
---

# Retro NFC Music Player — Frontplate Design — v4

Supersedes [[Retro NFC Music Player - Frontplate Design - v3]]. First pass built on real caliper data from [[Component Specifications]] rather than estimates. **200 × 175 × 4mm**, parametric OpenSCAD (`frontplate.scad`). Renders as one watertight solid, ~104cm³.

## Changes from v3

| Change | Driver |
|---|---|
| Plate 220×185 → **200×175** | Speakers mount within their own 44.5mm body footprint, not on wide protruding ears, so the width budget shrank. 200 is now reachable — the target from two passes ago |
| **Rear counterbores on transport + volume buttons** (Ø18 × 2mm) | Measured thread is only **5mm** and needs a 1mm washer. Full 4mm plate would leave 0mm for the nut. Local thickness drops to 2mm, leaving ~2mm of thread for the nut |
| Power button left in full 4mm plate | Its thread is 10.4mm — no relief needed |
| Speaker openings → **39 × 82** | As specified; leaves 2.75mm of plate each side and 8.75mm top/bottom inside the body footprint to clamp the mesh |
| Button hole Ø12.4 → **Ø12.0** | Measured barrel is 11.5mm, not 12 |
| OLED window/bezel resized | Real board is **71 × 43.3 × 6mm**; bezel now 80 × 46 |
| PN532: **2 diagonal screw bosses + 2 plain support pillars** | Board has only two mounting holes, on opposite corners; the empty corners get pillars for stability |
| PN532 mounted **back-to-plate** | DIP switches stand 3.9mm proud on the antenna face. Mounting that face outward would force a 4mm standoff and cost read range; back-to-plate keeps the antenna close *and* leaves the DIP switches reachable from inside the cabinet |

## Component placement (X, Y from BOTTOM-LEFT of front face)

| Feature | Center | Spec |
|---|---|---|
| OLED window | 100, 125 | 57 × 29.5, 1mm front chamfer, `oled_win_off` available to nudge |
| OLED bezel | 100, 125 | 80 × 46 front recess, 1.2mm deep, r3 |
| OLED bosses | ±32.5, ±18.65 | 4× M2.5 — **spacing estimated, see open questions** |
| NFC tap zone | 100, 70 | Ø45 engraved ring; Ø48 × 2mm rear pocket; posts rise from the pocket floor to 3mm proud |
| NFC posts | ±13.9, ±12.9 | 2 bored (M2.5) on one diagonal, 2 plain pillars on the other. Bores stop 1.2mm above the pocket floor so the read window stays solid |
| Speaker L / R | 31, 105 / 169, 105 | 39 × 82 opening (r3); 43 × 90 × 0.6mm rear mesh rebate; L-brackets at all four body corners, 8mm tall |
| Power button | 31, 24 | Ø12.0, **no** counterbore |
| Transport row | 78 / 100 / 122, 24 | Ø12.0 + Ø18 × 2mm rear counterbore, 22mm pitch |
| Volume up / down | 169, 34 / 169, 14 | Ø12.0 + counterbore, stacked, 20mm pitch |

Clearances: OLED bezel → speaker brackets 4.25mm · NFC pocket → bezel 8mm · NFC pocket → transport counterbores 11mm · speaker brackets → button band ~10mm · brackets → plate edge 6.25mm.

## Open questions blocking a final print

1. **Speaker mounting tabs.** "Inset from front edge: 9.8mm" reads as a *depth* measurement (like the "wires 4.8mm from front edge" note), which would put the tabs 9.8mm behind the speaker's front face — they can't bolt flat against a 4mm plate. v4 therefore holds the speakers with corner brackets and defers screw posts. Need: horizontal center-to-center of the two holes on one end, vertical center-to-center between the top and bottom pairs, and whether the tabs stay inside the 44.5mm body width or protrude.
2. **OLED mounting hole spacing** — not captured at all. Currently estimated at 65 × 37.3.
3. **OLED lit area.** 61.8 × 39.7 is almost certainly the *glass*, not the pixels: a 2.42" 128×64 panel has a lit area near 55 × 27.5mm. Need the lit rectangle and its offset from the board edges so the window frames pixels, not unlit border.
4. **PN532 hole diameter and which diagonal** (top-left/bottom-right assumed). Hole centers derived as ~7.4mm in from each edge from the 6.0mm inner-edge measurement.

## Print / assembly

PLA or PETG, 0.2mm layers, 4–5 perimeters, brim, face-down. Heat-set inserts: M2.5 → 3.4mm bores, M3 → 4.0mm bores. Speaker mesh cut 43 × 90mm, dropped into the rear rebate before seating the speaker.

**Test one button first.** The 2mm-plate + 1mm-washer + nut stack on a 5mm thread is the tightest tolerance on the plate; print a small coupon with a single counterbored hole before committing to a full plate.
