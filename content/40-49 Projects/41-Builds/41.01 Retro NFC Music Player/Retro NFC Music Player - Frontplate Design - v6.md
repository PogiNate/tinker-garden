---
title: Retro NFC Music Player - Frontplate Design - v6
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-frontplate-design-v6
tags:
- frontplate
- enclosure
- 3d-printing
- openscad
- retro-nfc-music-player
---

# Retro NFC Music Player — Frontplate Design — v6

Supersedes [[Retro NFC Music Player - Frontplate Design - v5]]. 200 × 175 × 4mm plate (13.8mm deep including speaker posts). Parametric OpenSCAD (`frontplate.scad`), plus a standalone `speaker_test_coupon.scad` for validating one speaker mount before printing the full plate. One watertight solid.

## Change 1 — Speaker tab spacing corrected

Vertical tab spacing is **91.1mm**, not the 70.7 recorded in v5. This dissolves the problem v5 was built to solve:

| | v5 (70.7) | v6 (91.1) |
|---|---|---|
| Tab hole Y | ±35.35 — **inside** the 82mm opening | ±45.55 — **outside** it |
| Gap to opening edge | −5.65mm (collision) | **+1.05mm of solid plate** |
| Gap to body end | 14.4mm | 0.70mm |
| Fix required | 12 × 12mm pads intruding into the opening corners | none |

**v5's intrusion pads are removed entirely.** The four posts now land on solid plate above and below the opening, exactly as expected once the tabs are only 1mm thick at the outer edges. The posts still sink to the mesh-rebate floor so they stay solid where the rebate passes under them; the mesh keeps its four corner notches.

Post OD reduced 8 → 7mm: at the new Y positions the posts sit 0.7mm from the body end, and 7mm keeps clearance to the corner brackets (whose arms were also shortened 14 → 12mm). M3 insert in 7mm post = 1.5mm wall, adequate.

## Change 2 — OLED mounts flush

The glass stands **2.6mm proud of its PCB**, so the board drops into a **1.4mm rear pocket** (71.5 × 43.8) and the glass fills the window level with the front face.

- **Window is now glass-sized**, 57.0 × 29.3 (56.5 × 28.8 + 0.25mm clearance per side) — a through-hole with no retaining lip, since the glass passes through rather than sitting behind it.
- **Bezel recess removed.** With flush glass, a 1.2mm recess would leave the screen standing proud of the bezel floor. Replaced by a shallow **engraved frame groove** (64 × 36, 1.5mm wide, 0.6mm deep) that keeps the visual weight without any depth conflict.
- **Screw posts gone.** Fixings are now pilot holes in the pocket floor.
- **Header relief** added: a 26 × 6mm slot 1mm deeper than the pocket along the board's top edge, for the pin-header solder tails. ⚠️ Verify position and length against the actual board.

### The hard constraint worth knowing

Flush mounting fixes the material under the board at exactly **2.6mm — and no plate thickness changes that**, because the glass height sets it. Every millimetre the plate gains goes into the pocket, not under the board.

That rules out heat-set inserts (M2.5 inserts are ~4mm and would break through the front face). v6 therefore uses **M2.5 thread-forming screws into 2.1mm pilot holes, 2.0mm deep**, leaving 0.6mm of material before the front face. That's ~3 layers at 0.2mm — enough not to show, and plenty of grip for a ~15g display.

*Alternative if the screws ever feel marginal:* the pocket already captures the board on all four sides, so a printed retainer bar pressing it forward — screwed to two full-depth bosses placed outside the 71 × 43.3 board footprint — would give unlimited engagement. Not needed unless the pilot holes disappoint.

## `speaker_test_coupon.scad`

60 × 120 × 4mm, one speaker port, ~19cm³ — a few minutes to print. Standalone (parameters mirrored, not included) so it slices without pulling in the whole plate. Validates:

1. Tab post positions against the real slots
2. Post height vs the tab flange — i.e. the **9.8mm assumption**
3. Where the 39 × 82 opening actually falls on the drivers
4. Mesh fit in the 43 × 90 rebate and whether the speaker clamps it flat
5. M3 inserts seating in 7mm posts

Coupon is 4mm rather than the requested 3mm so it matches the real plate — post height and mesh clamp are measured from the back face either way, but 4mm also shows how the opening reads at true thickness. Set `coupon_t = 3` to save filament if only hole positions matter.

**If the tab flange doesn't stand 9.8mm off the plate, measure the real gap and set `spk_tab_z` in BOTH files before printing the plate.**

## `oled_test_coupon.scad`

95 × 70 × **4mm**, ~17.5cm³. The speaker coupon printed correctly, so this is the companion for the other tricky mount. Standalone (parameters mirrored) so it slices on its own.

⚠️ **`coupon_t` must stay 4mm.** Flushness is plate thickness minus the 2.6mm glass stand-off, so a thinner coupon tests nothing about the thing this coupon exists to test.

Validates:

1. The glass passes through the 57.0 × 29.3 window and sits **flush with the front face**
2. The board seats in the 1.4mm pocket (71.5 × 43.8) without side slop — 11.8 / 13.1mm of coupon margin around it
3. Pilot holes align with the 66.4 × 38.5 mounting holes, and M2.5 thread-forming screws grip in 2.6mm **without breaking through the front face**
4. Header solder tails clear the relief slot — position and length are still a guess, so this is the main unknown
5. How the engraved frame groove actually reads

**Check the front face over the four pilot holes first.** Only 0.6mm (three layers at 0.2mm) stands between the pilot and the visible surface. If it dimples or breaks through, drop `oled_pilot_z` to 1.6 in both files, or switch to the retainer-bar mount described above.

Then seat the board, screw it down, and sight along the front face. Glass proud → `oled_stand` is under-measured; glass low → over-measured. Adjust in both files before printing the plate.

## Remaining verification

- [ ] Speaker tab depth (the 9.8mm assumption) — the coupon settles this
- [ ] OLED header relief position/length
- [ ] OLED lit area vs the 56.5 × 28.8 glass once powered; use `oled_win_off` if it sits off-centre
- [ ] Cabinet body wall thickness at the mating edge, to finalise corner-boss positions
