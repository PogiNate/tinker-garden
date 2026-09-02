---
title: Frontplate Design - v10
type: note
tags:
- 3d-printing
- frontplate
- enclosure
- openscad
- retro-nfc-music-player
- design-doc
permalink: tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/frontplate-design-v10
---

# Retro NFC Music Player — Frontplate Design — v10

Supersedes [[Frontplate Design - v9]]. 

The following changes were made by hand in the [[OpenSCAD]] file:

- Repositioned the power and volume buttons so that the power button is on the left and volume buttons on the right
- Corrected the play symbol, which was facing backwards 🤦
- Removed the two support pillars from the NFC assembly, since they were colliding with the DIP switches on the NFC component.

The following change was made in the `.3mf` file after generating an `.stl` file from OpenSCAD:

- Added decorative text below the transport buttons. This text is also a cut-through to the transparent PETG and will also have LED light behind it.

## Slicer Notes

I set `label_depth` to `1.1mm`. In the slicer I kept the front 1.0mm black, the rest transparent. 

Some settings to help the print:

- Turn on **interface shells** to create a second layer of opacity between the black and transparent PETG
- Increase **bottom shell layers** to **5** since this is in fact the user-facing surface of the component.
- change **sparse infill pattern** to _adaptive cubic_. It's less obvious looking when light shines through it than rectilinear.
- Turn brim **off** or set to **painted** but really just **use glue** and never use brims.
- Turn **Fuzzy Skin** on and set to **painted**.
    - Paint the full back surface with a fill tool.
- Set fuzzy skin noise type if you want. I set it to **perlin** because why not?

## Other Notes

This seems to be the one to go with. I am going to start assembly using this frontplate.