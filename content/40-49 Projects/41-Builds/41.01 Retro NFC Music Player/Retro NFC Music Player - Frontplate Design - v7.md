---
title: Retro NFC Music Player - Frontplate Design - v7
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-frontplate-design-v7
tags:
- frontplate
- enclosure
- 3d-printing
- openscad
- retro-nfc-music-player
---

# Retro NFC Music Player — Frontplate Design — v7

Supersedes [[Retro NFC Music Player - Frontplate Design - v6]]. Adds engraved control labels: bold geometric symbols, no text, matching the mid-century-radio direction. Same 200 × 175 × 4mm plate; one watertight solid.

## What's engraved

| Button(s) | Glyph | Depth |
|---|---|---|
| Prev | Skip-back (triangle + bar) | 0.5mm recess |
| Play/pause | Right-pointing triangle | 0.5mm |
| Next | Skip-forward (mirror of prev) | 0.5mm |
| Vol up | Plus | 0.5mm |
| Vol down | Minus | 0.5mm |
| Power | IEC power ring + bar | 0.5mm |

All glyphs are custom 2D polygons (`icon_skip`, `icon_play`, `icon_plus`, `icon_minus`, `icon_power` in `frontplate.scad`) rather than font text — this sidesteps font-availability differences across machines entirely, and let the strokes be drawn deliberately chunky (`label_stroke = 0.42` of the 5mm bounding box) for a bolder, more mid-century feel than a thin technical line would give. If captions are ever added later, a geometric sans (Futura/Century Gothic-alike) would suit the same direction — noted for the future, not needed now.

## Placement — why above vs. beside

Checked clearances before placing anything:

- **Transport row + power:** labels sit centered **above** each button, offset by hole-radius + 3mm gap + half the label size (≈11.5mm total). Tightest case is the center transport button, which still has 8mm of clearance to the NFC pocket edge above it — comfortable.
- **Volume pair:** the two buttons are only 20mm apart (pitch) with 12mm holes, leaving just 8mm between their edges — not enough room for an 11mm label to fit between them without touching either hole. Their labels sit **beside** the buttons instead, offset toward the open plate edge on the right, where there's 31mm of clear space to work with.

## A rendering note, not a design note

While building this, OpenSCAD's preview renderer would not visually display the 0.5mm-deep label recesses at any camera angle or zoom, even with `--render` forcing a full CGAL pass — they looked identical to blank plate. Ray-casting directly into the exported STL confirmed the geometry was correct all along (material removed to exactly the specified depth, at the right coordinates), and building a throwaway variant with the glyphs as raised solids instead of recesses confirmed every shape and position visually. The plate itself verifies as one watertight solid. This appears to be a preview-only quirk with very shallow subtracted features layered among many other cuts — worth knowing about if you ever add more engraving and the preview looks unexpectedly blank; check the STL, not just the preview.

## Open item

The power glyph's ring needed its own proportions — using the same bold stroke width as the bars/triangles made it read as a filled disc rather than a ring. It now uses a dedicated thinner stroke (`ring_stroke = 0.20`) independent of `label_stroke`. Worth a visual check once printed, since 0.5mm-deep engraving at this ring thickness (~1mm) is the fussiest feature to actually resolve well in PLA/PETG — if it doesn't read clearly at print scale, deepen to 0.7–0.8mm or widen the ring stroke slightly.
