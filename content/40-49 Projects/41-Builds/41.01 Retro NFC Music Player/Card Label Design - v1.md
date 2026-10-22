---
title: Card Label Design - v1
type: design
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/card-label-design-v1
tags:
- nfc
- cards
- labels
- print
- design
created_by: Nathanial Dickson
updated_by: Nathanial Dickson
---

# Retro NFC Music Player — Card Label Design — v1

Label designs for the NFC tag cards used by [[41.01 Retro NFC Music Player]]. Decided 2026-10-13.

Interactive template page (upload covers, live preview, print guides): https://claude.ai/artifact/ECgm1maRmuQJNHtLkz5m9L

## Print stock
- Label sheet template: `R011_SI-LABEL-LS-0302.docx` — US Letter, 10 labels per sheet, 2 across × 5 down
- Each label is 3 × 2 in, **landscape only**
- Gutters: ⅛ in between labels, both directions
- Column left edges: 1³⁄₁₆ in and 4⁵⁄₁₆ in from the page edge
- Row top edges: ¼, 2⅜, 4½, 6⅝, 8¾ in
- Bleed: 1/16 in max per side (neighbouring bleeds just meet in the gutter)
- Safe zone: keep text ⅛ in inside the cut
- Artwork at 300 dpi: 900 × 600 px trimmed, 975 × 675 px with bleed

## Album cards — decided
- **Layout:** landscape blur fill, title on top
  - Background: the cover, enlarged and blurred, tinted with a colour sampled from the cover
  - Album title across the top, single line, ellipsis if too long
  - Sharp square cover below the title, on the left (~1.44 in square)
  - Right column: artist (caps), short rule, "TAP TO PLAY" + NFC glyph at the bottom
- **Fonts:** "Hi-fi" pairing
  - Title: Righteous
  - Artist: Rubik 500, uppercase, letter-spaced

## Mixtape cards — decided
- **Layout:** classic cassette label — dark/coloured shell, cream label, ruled writing strip across the top with an "A" side box, three thin stripes under the window, "C60 · TYPE I · NORMAL BIAS" small print
- **Colorways kept:**
  - Sunset: black shell `#1c1b1d`, label `#f4f0e5`, stripes `#e7a23a` / `#d6562a` / `#9a2c4a`
  - Ocean: navy shell `#1d2836`, label `#f6f6f2`, stripes `#5fa8d3` / `#2a9d8f` / `#1d3557`
  - Avocado: brown shell `#3a2b20`, label `#efe6cf`, stripes `#d9a521` / `#c8662b` / `#7a8b3a`
- **Handwriting font:** "Neat pen" — Caveat 600
- Strip can be left blank to hand-write after printing

## Options considered and set aside
- Album: portrait blur fill (doesn't fit the landscape-only label sheet), landscape blur fill with side text, literal cassette, stylized cassette
- Album fonts: Poster (Archivo Black), Marquee (Bebas Neue), Liner notes (Fraunces), Tape deck (IBM Plex Mono)
- Mixtape: clear shell, coloured shell; colorways Chrome, Sherbet, Midnight
- Handwriting: Permanent Marker, Gochi Hand, Reenie Beanie, Rock Salt

## Next steps
## Label builder (2026-10-13)
- Decided: build the label sheet tool into card-admin on the Pi, not as a claude.ai artifact. An artifact page can't reach MPD on the LAN; card-admin already has library search and cover lookup.
- `/labels` page: ten slots laid out like the sheet (row by row, left then right). Each slot is empty, album (library search, editable title/artist for the label) or mixtape (colorway, C-60/C-90, name or blank). Picking an existing mixtape's name sets its tape length.
- "Done · build PDF": 300 dpi letter PDF, 1/16 in bleed, empty slots left blank for part-used sheets. Optional cut lines for a plain-paper alignment test. Print at 100% / actual size.
- Renderer: `retro_player.labels` (Pillow), shared by previews and PDF. Fonts bundled: Righteous, Rubik, Caveat, JetBrains Mono (small print), all OFL.
- PR: https://github.com/PogiNate/retro-player/pull/1 (branch `card-labels`)

## Next steps
- Merge the PR, deploy to the Pi, and check render time with real covers
- Test print with cut lines against a label sheet to confirm alignment