---
title: Frontplate Design - v9
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-frontplate-design-v9
tags:
  - frontplate
  - enclosure
  - 3d-printing
  - openscad
  - retro-nfc-music-player
  - backlighting
---

# Retro NFC Music Player — Frontplate Design — v9

Supersedes [[Frontplate Design - v8|Retro NFC Music Player - Frontplate Design - v8]]. Corrects v8's approach after a key clarification: **there is no separate diffuser panel.** This is a single-part print with a manual filament swap partway through the thickness — black PETG for the front, clear PETG for the rest, fuzzy skin on the rear face for diffusion. Same 200 × 175 × 4mm plate; one watertight solid. Adds `backlight_test_coupon.scad`.

## Why v8's through-cuts were wrong for this technique

v8 assumed a two-part assembly (opaque black front plate + separate clear diffuser panel behind it) and made the control labels full through-cuts on that basis — reasonable for that design, but it doesn't fit a single dual-colour part. An open hole has no plastic in it at all, so there's nothing there for fuzzy skin to act on, and light passes straight through with hard edges right at the icon — the opposite of "fuzzy skin as diffuser."

**v9's fix:** each label is now a **recess**, not a through-cut, cut only through the black skin. The clear structural layer beneath stays fully intact and continuous with the rest of the plate. Light diffuses through that continuous clear layer (scattered by the fuzzy skin on the rear face) and exits only where the black skin has been removed — crisp icon silhouette, still one solid piece, no second panel or mounting geometry to design. This also retroactively resolves the diffuser-panel open question from v8: there's nothing left to design, because the "diffuser" is just the clear portion of this same part.

## The one parameter that matters most

```
label_depth = 1.5;  // ⚠️ MUST equal your slicer's black→clear swap height, from the front
```

This has to match the exact layer height (measured from the front, print face-down) where you do the manual filament swap. Too shallow and a thin black film remains over the icons, blocking light; too deep just eats into the clear layer harmlessly (both are clear PETG either way) but wastes margin. 1.5mm is a reasonable midpoint of the "1–2mm" you described — change it here if your actual swap height differs, and change it in the coupon file to match.

## `backlight_test_coupon.scad`

90 × 40 × 4mm, three glyphs (skip, play, plus — deliberately different stroke geometries: thin bar, solid triangle, cross) so you can check whether shape affects how well light comes through. This is the one thing no render can answer, so it's worth printing before the full plate:

- Slice and swap filament exactly as planned for the real print
- Enable fuzzy skin on the **rear (top, LED-facing) surface only** — not the front, or the visible icon face loses crispness
- Hold it to a warm-white LED from behind and check three things: does the surrounding black field actually block light (black PETG opacity at 1–2mm varies a lot by brand/pigment — this is a real risk, not a formality), does the icon read as crisp and evenly lit rather than an overdiffused blob, and is there any visible hot-spotting from individual LEDs

If `label_depth` needs adjusting after this test, change it in both files and reprint the coupon before touching the full plate.

## Also carried over from v8 (unchanged)

Power label stays removed — the power button is itself an illuminated switch, so a separate glyph next to it is redundant. Nothing new needed here; already correct.

## Idea, not implemented

The NFC ring is currently a 0.6mm decorative engrave — shallower than the black layer, so it won't backlight as-is. A glowing "tap here" ring using the same technique could look great, but it changes the tap target's tactile/visual design in a way that wasn't asked for, so it's flagged here rather than done. Say the word if you want it.
