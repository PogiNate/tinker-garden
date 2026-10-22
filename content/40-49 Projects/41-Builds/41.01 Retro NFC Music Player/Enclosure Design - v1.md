---
title: Enclosure Design - v1
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/enclosure-design-v1
tags:
- retro-player
- enclosure
- openscad
created_by: Nathanial Dickson
updated_by: Nathanial Dickson
---

# Retro NFC Music Player — Enclosure Design — v1

First pass at the cabinet body behind [[Frontplate Design - v10]]. This replaces the "Cabinet body still to design" line in Phase 6 of [[Retro NFC Music Player - Design and Project Plan - v3]]. Nothing is modelled yet. This note records the architecture decisions and lists what must be measured before any OpenSCAD work starts.

## Architecture: chassis + sleeve

- **Chassis** = bottom plate + faceplate + every electronic part. It is rigid on its own, so with the shell off everything stays powered and testable.
- **Sleeve** = top, both sides and back printed as **one piece**. It slides on from the rear and carries **no electronics and no cables**.

## Decision: the shell wraps the faceplate edges (front rebate)

The faceplate is black for only the front 1.0mm and clear PETG behind that. Its perimeter is a ~3mm clear band, and with LEDs inside (and fuzzy skin on the rear spreading the light), that band will glow. A shell that only butts against the rear face would leave a lit outline around the face.

- The faceplate sits 2mm behind the cabinet front. The shell's front 6.1mm (2mm recess + 4mm faceplate + 0.1mm) has an inner opening of the faceplate outline + clearance per side (0.15–0.3, picked by coupon). This hides the top and side edges and follows the faceplate's 6mm corner radius.
- Behind the rebate the wall steps inward, forming a shoulder that seats against the faceplate's rear perimeter.
- The visible result is a thin, even frame around the face. Target **2mm**.
- The faceplate's bottom edge is hidden by the bottom plate's groove. The groove's front lip continues the 2mm frame along the bottom.
- With the shell on, the rebate clamps the faceplate on three sides, which makes the assembled box rigid.

## Decision: faceplate retention = groove + bonded brackets
## Decision: faceplate retention = groove + brackets (bolted to the corner bosses, optional bond)

Chosen over a glue-free clamp under the speaker tabs and over a v11 faceplate reprint. Reason: v10 is printed and fully wired, so reprinting means rewiring. A groove alone isn't enough; a 3mm slot lets a 175mm-tall plate rock several degrees.

**Bracket zone (Oct 2026):** "Nate Made" illuminated text now sits directly below the three transport buttons; it was added in the slicer, so it is not in the SCAD. The only room for brackets is on each side, between the bottom corner bosses and the speaker corners.

**Revised from "bonded only":** v10 already has four M3 corner bosses (8mm OD, 6mm tall, centred 8mm in from each corner). Each bracket's upright bolts to the bottom corner boss with an M3 screw, so it is mechanical first. An optional bond pad reaches forward to the faceplate rear above the boss and can be epoxied as well.

**Groove (in the bottom plate):** full width, 3mm deep, **4.1mm wide (coupon-verified)**. Its front wall is the 2mm lip.

**Bracket (left; the right one is mirrored), as in the coupon SCAD:**
- x 3.5–21. This clears the shell's inner wall at 1.5 and the volume buttons' counterbores, which start at 22.
- Top at y 48. This clears the speaker locating brackets, which start at 52.75.
- 5mm upright against the boss end face, with an M3 clearance hole on the boss axis.
- **Head rebate (added Oct 9 2026, after assembly):** the boss axis is only 4mm above the foot's top surface, so the M3 head rode up on the foot, tilted the screw and pushed the faceplate forward. Both brackets now have a Ø6.5 × 3.5mm pocket cut into the foot behind the upright, so the head seats flat. ~1.75mm of foot remains under the pocket. Params `brk_head_d` / `brk_head_h` (sized for socket cap 5.5 × 3.0).
- 30mm foot with an 8mm front-to-back slot to an M3 insert in the plate.
- One gusset on the inboard edge.
- Bond pad covers y 18–46 with a 0.4mm epoxy gap over the fuzzy skin and a 45° underside so it prints without supports.
- Right side has more room: the power button is at x 169, and the bracket lands at x 179–196.5 when mirrored.

**Bonding procedure (if the pad is used):**
1. Seat the faceplate in the groove.
2. Bolt the bracket to the boss and loosely to the plate.
3. Scuff the faces and apply slow-cure 2-part epoxy. CA is weak on PETG.
4. Cure, then tighten.
## Raspberry Pi mounting
- Orientation: Pi long axis parallel to the faceplate, **GPIO header facing the faceplate** (short harness runs). The USB-C/micro-HDMI edge then faces the rear.
- Hole pattern: 58 × 49mm, M2.5 (verify with calipers).
- Current stack: Pi → Active Cooler → tall header → WM8960, **~55mm tall** including header pins. All four Pi holes have screws from below. Two screw up into the standoffs that hold the HAT above the heatsink; the other two only keep the Pi level.
- **Pi sled (decided):** all four screws are replaced by standoffs fixed to the sled.
  - Four identical **M2.5 male-female standoffs, ~6mm**, with the male end up through the Pi.
  - At the two HAT posts, the male end threads into the existing HAT standoff, taking the place of the old screw.
  - At the two levelling posts, an M2.5 nut goes on top of the Pi.
  - Each standoff is held to the sled by an M2.5 screw from **underneath**, in a counterbore so the head sits below the sled's underside. This is done on the bench.
  - The sled screws to the bottom plate with 4× M3 into heat-set inserts, at corners outside the Pi footprint, from the top.
  - To service, undo four screws and lift the whole stack out intact.
- Stack top ends up ~65–70mm above the plate, well inside the ~165mm interior height.
## Shell retention, power, airflow
- **Rear ports, parts ordered (Oct 2026). Both are standard D-type (Neutrik D-series footprint) modules, so the two cutouts match:**
  - **USB-C:** PENGLIN USB 3.1 Type-C D-type panel coupler, **female-to-female** (Amazon B0B4VWZD7X). It needs a short internal **male-to-male e-marked 5A USB-C cable** to the Pi; a right-angle plug at the Pi end is ideal.
  - **3.5mm:** MEIRIYFA D-type panel-mount audio cable, 3.5mm TRS with a short cable (Amazon B0DGSZKYL5). The listing comes in male-plug and female cable-end styles; the **male-plug** style is needed to plug into the WM8960 jack.
  - D-series cutout to verify with calipers on arrival: **Ø24mm hole, 2× M3 holes at 19 × 24mm**, flange ~26 × 31mm.
  - Software check for later: whether plugging into the 3.5mm jack mutes the speakers depends on jack-detect support in the stock WM8960 driver. If it doesn't, switching is a mixer/control job.
- **Rear upstand** on the bottom plate, **flush with the shell's outer back face**, becomes the visible lower rear panel carrying both D-type modules front-mounted. That needs roughly 35mm height and ~70mm width. The shell's back wall ends just above the upstand and clears it while sliding on.
- Ledges or grooves along the bottom plate's side edges keep the shell walls from bowing outward. The shell fixing at the rear is to be detailed in the full plate design.
- Neither cable passes through the shell, so the shell lifts straight off.
- **Ventilation:** slots in the bottom plate (and Pi sled) under the Pi plus feet, with exhaust slots in the top or back (chimney flow). The WM8960 sits over the Active Cooler's intake, so give the slots real open area.
## Open measurements (before any OpenSCAD)
## Depth

No fixed target. Both a cube and a shallower-than-tall box are acceptable. Note that the faceplate is 200 W × 175 H, not square, so a "cube" would be ~200 deep and still wider than tall.

Minimum internal depth, front to back:

| Zone | Approx. |
|---|---|
| Faceplate | 4mm |
| Faceplate-mounted parts + harness (speakers 20.7mm deep; button bodies and terminals deeper) | ~40mm |
| Pi (56mm side front-to-back) | 56mm |
| USB-C / 3.5mm pigtail bend room + upstand | ~25–30mm |
| **Total** | **~125–130mm** |

Proportion options (outer depth): **~150mm** gives a classic tabletop-radio shape, a shorter shell print and a little slack; **~200mm** gives a near-cube with lots of empty space and a longer print. **Decided: 150mm outer depth.** It leaves ~20mm of slack over the minimum. Make depth a single parameter in the SCAD.

**Printer:** 256mm³ build volume. The shell prints standing on its back panel, so its footprint is ~204 × ~177mm and its print height = depth. Either option fits.

## Open measurements (before any OpenSCAD)

- [x] **Actual printed faceplate W × H:** 200.02 × 175.03mm by calipers, so nominal is used.
- [x] **Bracket zone:** between the bottom corner bosses and the speaker corners, on each side.
- [ ] M3 heat-set inserts into the two bottom corner bosses: not done yet (Oct 2026), planned.
- [ ] Deepest faceplate-mounted part (probably the power button with its terminals), to firm up the ~40mm zone above.
- [ ] What currently holds the bottom of each stack post (screws? nuts?), to pick the sled standoffs.
- [ ] Panel-mount USB-C and 3.5mm parts chosen, for their cutout sizes.
- [x] Printer build volume: 256mm³.
- [x] Pi stack height: ~55mm, no bottom standoffs.
- [x] Rear ports: USB-C + 3.5mm audio only.
## First print: corner coupon
## First print: fit coupon (`enclosure_coupon_v1.scad`)

Parametric OpenSCAD in the same X/Y frame as `frontplate_v10.scad`. Z = 0 is the cabinet front. Pieces, all from one build plate (`part = "all"`):

- **shell_top ×3:** top-front corner with the rebate and shoulder, at 0.15 / 0.2 / 0.3mm clearance per side. Count the notches on the outer face near the back edge (1 = tightest). Printed front-face-up, which is how the full shell will print.
- **shell_bottom:** bottom-front edge of a side wall, filling around the faceplate's rounded bottom corner.
- **plate:** front-left corner of the bottom plate, with the groove, lip, bracket insert and the start of the side rib.
- **bracket:** left bracket, printed foot-down.

`part = "assembly"` shows them in place against a ghost faceplate. An interference check came back clean: the bracket touches the boss end face and the plate top, with no overlaps.

Key coupon parameters:
- Faceplate recessed **2mm** behind the cabinet front.
- **2mm** visible frame.
- **1.5mm** shoulder overlap, so the shell wall is 3.7mm thick behind the rebate.
- Plate **6mm** thick.
- Side ribs start behind the brackets (z ≥ 45).

**Test sequence:**
1. Seat the plate coupon on the real faceplate's bottom-left corner and check the groove fit.
2. Bolt the bracket to the corner boss and check alignment with the plate insert.
3. Press each shell_top coupon onto a top corner and pick the clearance.
4. Check that shell_bottom seats on the plate and hides the corner.

**Coupon results (Oct 2026), all passed. These values are now locked for the full parts:**
- Groove: 4.3mm was loose; **`groove_w = 4.1`** fits perfectly.
- Bracket: slot lines up with the plate insert perfectly.
- Shell rebate: **0.2mm per side** (the 2-notch piece) is the best fit.
- Shell bottom: hides the faceplate's rounded bottom corner perfectly.

## Bottom plate v1 (`bottom_plate_v1.scad`)

Full plate, Pi sled and both brackets in one parametric file (`part` = plate / sled / bracket_l / bracket_r / assembly). The same coordinate frame as the faceplate and coupon. An interference check came back clean: plate vs faceplate, brackets, sled and the Pi-stack envelope.

- **Plate:** 204.4 × 150 × 6mm, matching the shell's outer width.
  - Verified 4.1mm groove with 2mm lip, full width.
  - M3 inserts for both brackets (x 12 and 188, z 29.5).
  - Side ribs inside the shell walls from z 45 to the upstand.
  - Intake vent slots under the Pi.
  - Shallow pockets underneath for stick-on rubber feet. Use ≥6mm-tall feet so the vents can breathe.
  - Prints bottom-down at ~55mm tall, which fits the 256mm printer.
- **Rear upstand** = the visible lower back panel.
  - 40mm tall, 3.7mm thick (same as the shell wall), flush with the shell's outer back, spanning between the shell's inner side walls.
  - Thickened band at the top plus a 3mm **tongue** that rises inside the shell's back wall. The tongue hides the seam and keeps the wall from bowing in.
  - Gussets on the inside face.
- **D-type cutouts** at x 130 (USB-C) and x 162 (audio), centred 20mm above the plate top.
  - Ø24 teardrop hole (tip hidden under the flange), M3 through-holes at 19 × 24 diagonal, nuts on the inside.
  - Upstand 3.7mm thick, so M3×10 screws work.
  - `dmod_diag` flips which diagonal the screws use. **Verify the cutout on arrival.**
- **Shell fixing:** 2× M3 from the rear through the shell's back wall into heat-set inserts in pillars at x 15 and 185, y ≈ 47.2. **The shell model needs matching holes.**
- **USB-C loop:** the coupler is offset ~60mm sideways from the Pi's USB-C (x ≈ 69) so the 50cm cable can loop. There's a cable-tie anchor at x 100, z 125.
- **Pi 5 + sled:**
  - The Pi is centred left to right (x 57.5–142.5) with the GPIO edge at z 52 and the port edge at z 108.
  - Sled is 4mm thick, with counterbored M2.5 standoff screws from below and vents that match the plate.
  - It fixes to the plate with 4× M3 on ears outside the Pi footprint.
  - Pi underside sits 10mm above the plate top; stack top ~68mm.
- **Internal USB-C cable (suggested):** 50cm e-marked 5A, one right-angle end: StarTech rugged right-angle 50cm, or UNITEK C14123BK 0.5m (dual right-angle version: C14147A).

**Still to place:** the Perma-Proto power hub (location/mounting not yet decided).

## Shell v1 (`shell_v1.scad`), Oct 10 2026

Faceplate + bottom plate assembly complete and good (Oct 10 2026). Shell modelled against `bottom_plate_v1.scad` as built; `part = "assembly"` needs both files in the same folder (`use <bottom_plate_v1.scad>`).

- **Outer:** 204.4 W × 174.2 H (above the plate top) × 150 D. Top corners follow the faceplate's 6mm radius plus the frame (8.2mm outer); bottom corners are square where the walls stand on the plate.
- **Walls:** 3.7mm (2mm frame + 0.2 clearance + 1.5 shoulder) on the top, sides and back.
- **Front rebate:** coupon-verified 0.2mm/side, 6.1mm deep, shoulder 0.1mm behind the faceplate rear. Same profile as the passing coupons.
- **Back wall:** bottom edge at y 43.2, 0.2mm above the upstand. The upstand's tongue rises just inside it.
- **Fixing:** 2× M3 through plain 3.4mm holes at x 15 / 185, y 47.2, into the upstand pillar inserts. **Use button- or socket-head screws, not countersunk:** the holes are only 4mm above the wall's bottom edge, so a countersink would break out.
- **Exhaust:** 11 slots in the top, 3mm wide, running front-to-back over the Pi (x 62–138, z 62–122). 8 louvre slots high on the back wall (x 45–155, y 110–160). Both toggle with `top_vents` / `back_grille`.
- **Verified in CAD:** no interference with the plate (walls rest on its top), faceplate (all four corner bosses), brackets, sled or Pi-stack envelope. A slide-on sweep from fully off to seated is clear the whole way.
- **Print:** `part = "shell"` puts the back panel on the bed with the front up. No supports: the rebate only thins the wall going up, top-panel vents print as vertical slots, and back grille slots are in the first layers. Footprint 204.4 × 174.2, height 150; fits the 256mm printer. ~372 cm³ of model volume, so it's the biggest print of the build. The back panel is the bed face, so use elephant-foot compensation if the slicer has it.

## Shell v2 (`shell_v2.scad`), Oct 12 2026: front rim + optional hood

Requested: (1) a rim that holds the faceplate like the plate's groove; (2) an experimental angled hood, 10–20mm forward at the top, tapering to just the lip at the bottom. v1 is kept for history.

- **Rim:** 2mm lip in front of the faceplate's top and side edges, overlapping it by **3mm** (`rim_w`) to match the bottom plate's lip. The faceplate is now in a groove on all four sides: lip-to-shoulder gap 4.1mm, same as the verified plate groove.
- **Install direction changed:** with a rim the shell can't slide on from the rear, because the rim would have to pass through the faceplate (confirmed in CAD). It now **lowers on from above**: the faceplate's sides run down the side grooves and its top edge seats in the top groove, then the 2 rear screws go in as before.
  - The groove's bottom corners are squared off so the shell clears the faceplate's straight sides on the way down. The small gaps at the faceplate's rounded bottom corners hide behind the rim.
  - Lowering sweep verified clear (no hood and 20mm + flare); seated fit clear against the faceplate, plate, brackets, sled and Pi.
- **Hood:** `hood_depth` sets the forward projection at the top (0 = flat front). The front plane runs from z 0 at the plate top to z −hood_depth at the top of the shell. `hood_flare` (mm per side at the top front, ≤4) widens the opening toward the front. Options rendered: 0 / 10 / 15 / 20 / 20+3 flare. **Hood depth not yet chosen.**
- **Print orientation is now top-down** (top panel on the bed): `part = "shell"`.
  - Back-down (v1) would print the rim as a ~3mm flat ledge over air, and the hood as a near-flat top surface with heavy layer terracing.
  - Top-down has no overhangs except the rounded top edges, an 8.2mm fillet against the bed. The hood face prints nearly vertical.
  - Footprint 204.4 × (150 + hood); height 174.2. Volume ~375 cm³ (no hood) to ~413 cm³ (20mm).
- **Back grille is now vertical slots** (x 50–150, y 100–160), so they print as vertical slots instead of 110mm bridges.
- **`part = "corner_coupon"`:** top-front-left corner (rim, groove, shoulder, hood slope), 35mm cube, ~10 cm³. Test the rim fit on the real faceplate before the full print.