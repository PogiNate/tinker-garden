---
title: Retro NFC Music Player - Frontplate Design - v8
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-frontplate-design-v8
tags:
- frontplate
- enclosure
- 3d-printing
- openscad
- retro-nfc-music-player
- backlighting
---

# Retro NFC Music Player — Frontplate Design — v8

Supersedes [[Retro NFC Music Player - Frontplate Design - v7]]. Same 200 × 175 × 4mm plate. Two changes, both driven by a new material/lighting plan: **black PETG front plate, backlit from behind through a separate clear-PETG diffuser panel with a fuzzy-skin finish, lit by warm-white LED strips inside the cabinet.**

## Change 1 — Control labels are now through-cuts, not engraving

A 0.5mm engraved recess in opaque black PETG doesn't backlight — there's no opening for light to reach the diffuser behind it, so it would just stay a dark groove. The five control labels (skip-prev, play, skip-next, plus, minus) are now cut **fully through the 4mm plate**, same technique as the button holes and OLED window.

Checked each glyph for trapped islands before making the cut — a shape like a ring (material fully enclosed by the cut on all sides) would fall out as a loose piece if cut through. All five are simply-connected: the skip icons (triangle + separate bar), the play triangle, and the plus/minus crosses each remove material without leaving anything isolated. Safe as through-holes; verified in the rendered solid.

## Change 2 — Power label removed

The power button is itself an illuminated switch (LED ring, wired per [[Retro NFC Music Player - Wiring Reference]]), so a separate engraved/backlit power glyph next to it would be redundant. The `icon_power()` module stays in `frontplate.scad` (unused) in case it's wanted elsewhere later; the call to place it is removed. Power button position and hole are unchanged.

## Open design question — the diffuser panel itself

This file is ready to print as-is, but the **clear PETG diffuser panel is not yet designed**, and there's a real mechanical constraint worth deciding before modeling it: the space directly behind the front plate is not empty. It's occupied by, at different depths:

| Feature | Protrusion behind the plate |
|---|---|
| Speaker tab posts | 9.8mm (deepest) |
| PN532 support posts | 3mm |
| Corner assembly bosses | 6mm |
| OLED pocket | shallow, 1.4mm recess (doesn't protrude) |
| Button counterbores | 2mm recess (doesn't protrude) |

A single flat diffuser sheet mounted immediately behind the whole plate would collide with the speaker posts in particular. Before modeling it, worth settling:

1. **Scope** — does the diffuser/backlight cover just the five label windows (the actual point of this change), or also other openings like the NFC ring or around the buttons?
2. **Mounting depth** — how far back does the diffuser sit? This is also an LED-strip question (strip thickness + throw distance to diffuse evenly without hot-spotting).
3. **One sheet vs. local windows** — a full sheet needs cutouts to clear every post above; small individual diffuser windows behind just the backlit cutouts avoids that entirely and may be simpler to print and fit.

This is also entangled with **Phase 6's still-undesigned cabinet body** ([[Retro NFC Music Player - Design and Project Plan - v3]] §5) — the diffuser's standoff depth and the LED strip both live in that volume, so some of this may be worth deciding together with the cabinet rather than in isolation.
