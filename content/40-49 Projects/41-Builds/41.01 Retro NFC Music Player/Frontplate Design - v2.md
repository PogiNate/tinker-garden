---
title: Retro NFC Music Player - Frontplate Design - v2
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-frontplate-design-v2
tags:
- frontplate
- enclosure
- 3d-printing
- openscad
- retro-nfc-music-player
---

# Retro NFC Music Player — Frontplate Design — v2

Supersedes [[Frontplate Design - v1|Retro NFC Music Player - Frontplate Design - v1]]. Phase 6 of [[Design and Project Plan - v2|Retro NFC Music Player - Design and Project Plan - v2]], pulled forward so panel components can be placed and the harness trimmed to measured lengths. Modeled parametrically in **OpenSCAD** (`frontplate.scad`); printer envelope 256×256×256mm.

## Why v2 exists

Speakers measured with calipers: **44.5 × 99.4mm, screw ears on all four corners** — much larger than v1's 30×75 placeholder. At real size they no longer fit beside the matrix (131mm pocket + 2×~54mm speaker zones ≈ 240mm, leaving nothing for margins), and the v1 bottom-right NFC zone landed underneath the right speaker.

Layout restructured from "speakers flank everything" to **three horizontal bands**, which separates the speakers from the matrix in Y so they no longer compete for width:

| Band | Contents |
|---|---|
| Top | Spectrum matrix, full width |
| Middle | Speakers left + right, flanking a center stack: OLED above, NFC tap zone below |
| Bottom | Power button + 5 transport buttons |

The NFC zone moving to dead center is an upgrade, not a compromise: it's a natural tap target, it's ~35mm clear of both speaker bodies (**speaker magnets degrade NFC read range**, so distance matters), and it's well away from the button row so you can't mash a button while tapping.

## Plate

| Property | Value |
|---|---|
| Overall | **240 × 200 × 4mm**, 6mm corner radius (was 240×170) |
| Print orientation | Face-down (smooth front off the bed), brim recommended |
| Fasteners | Heat-set inserts: M2.5 (matrix retainer, OLED, PN532), M3 (speakers, cabinet corners) |
| NFC pocket | Rear pocket thinned to **2mm** wall behind tap zone |

## Component placement (X, Y from BOTTOM-LEFT of front face)

| Feature | Center | Opening / spec |
|---|---|---|
| Spectrum matrix window | 120, 170 | 127 × 31 (slightly **under** the 128×32 active area so the plate lip retains the module); 131×35 rear pocket, 1.5mm deep |
| Matrix retainer bosses | 120 ± 71, 170 | 2× M2.5 — printed retainer bar spans the module's short ends (its own holes fall inside the window) |
| OLED window | 120, 128 | 56 × 29, 4× M2.5 bosses at 52 × 32 |
| NFC tap zone | 120, 78 | Ø45 engraved ring front; Ø52 × 2mm rear pocket; 4× M2.5 bosses at 36 × 34 |
| Speaker L / R | 32, 96 / 208, 96 | Ø-slat grille 40 × 90 (inset from the 44.5×99.4 body); 4× M3 ear bosses, 5mm standoff |
| Power button | 36, 28 | Ø12.4 (12mm panel-mount + LED ring) |
| Transport row (5×) | 80 / 100 / 120 / 140 / 160, 28 | Ø12.4, 20mm pitch — prev · play/pause · next · vol− · vol+ |

Verified clearances in the rendered solid: speaker top edge → matrix pocket 6.8mm · speaker bottom → button row 12mm · NFC pocket → nearest speaker ear 35mm · OLED → NFC 9.5mm · matrix bosses clear the speakers entirely (different Y band). Model renders as **one watertight solid**, bbox 240 × 200 × 10mm.

## Still to measure (calipers before final print)

- [ ] Speaker **ear screw-hole spacing** (X × Y) — currently estimated 52.5 × 107.4
- [ ] Matrix module footprint (sets `matrix_pocket`, est. 131 × 35)
- [ ] OLED board hole spacing + **active-area offset from board edge** (window may need re-centering)
- [ ] PN532 board hole spacing (est. 36 × 34)
- [ ] Cabinet body wall thickness at mating edge (sets corner-boss positions)

## Print settings

PLA or PETG, 0.2mm layers, 4–5 perimeters, 15–20% infill. Chamfered windows need no supports; the NFC pocket ceiling bridges 52mm — add support there only if your printer bridges poorly. Brim against warp on the 240mm span. Heat-set inserts: M2.5 → 3.4mm bores, M3 → 4.0mm bores.

The 5mm speaker standoff leaves a small air gap behind the grille; a thin foam gasket ring closes it if you want tighter coupling.

Wiring tie-in: dry-fit all components to the printed plate, position the Pi stack + [[Wiring Reference|Perma-Proto]] behind it, then cut harness to measured length + 20mm service slack.
