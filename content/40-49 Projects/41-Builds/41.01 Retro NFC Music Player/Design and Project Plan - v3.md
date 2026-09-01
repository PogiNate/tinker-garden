---
title: Retro NFC Music Player - Design and Project Plan - v3
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-design-and-project-plan-v3
tags:
- design-doc
- retro-nfc-music-player
- gpio-map
- software-stack
---

# Retro NFC Music Player — Design & Project Plan — v3

Supersedes [[Design and Project Plan - v2|Retro NFC Music Player - Design and Project Plan - v2]]. **The change that defines v3: the MAX7219 LED matrix spectrum analyzer is dropped.** It was visually too large for the cabinet, and removing it simplifies three layers at once — hardware (no 5V logic domain, no level shifters), software (no cava, no FIFO tap, no spectrum agent), and the faceplate (see [[Frontplate Design - v3|Retro NFC Music Player - Frontplate Design - v3]]).

Hardware baseline: Pi 5 (2GB) + Active Cooler, Waveshare WM8960 Audio HAT + its 44.5×99.4mm speakers, SSD1309 2.42" OLED (SPI), HiLetgo PN532 (I2C), 6 momentary buttons. Pin-by-pin wiring in [[Wiring Reference|Retro NFC Music Player - Wiring Reference]].

**Now surplus:** MAX7219 32×8 matrix, 2× Adafruit Pixel Shifter. Keep for a future build; nothing else in the design depends on them.

## 1. System Architecture

```ascii
                     ┌──────────────────────────────────────────────┐
                     │              Raspberry Pi 5 (2GB)              │
                     │                                                │
 NFC card ─► PN532 ──┤ I2C1 (0x24)      MPD (player core)            │
                     │                   │              │             │
 Buttons ─► GPIO ────┤ gpiozero/lgpio    │              ▼             │
 (5 discrete pins)   │                   │           ALSA             │
                     │                   │              │             │
 OLED ◄── SPI0/CE0 ──┤ display-agent ◄───┘              ▼             │
 (SSD1309, track)    │  (mpd idle events)            WM8960 I2S       │
                     │                                  │             │
 Laptop ◄── WiFi ────┤ Flask card-admin                 ▼             │
                     │ (pair cards, mixtapes)   Speakers L/R (~1W)    │
                     └──────────────────────────────────────────────┘
 Music: microSD/USB now ─► later: NFS/SMB mount of NAS (same MPD library)
```

Philosophy unchanged: **MPD is the single source of truth.** The NFC agent tells MPD what to play, buttons send MPD commands, the OLED asks MPD what's playing. The NAS enhancement stays a pure storage change.

**Pi 5 note that touches everything:** legacy `RPi.GPIO` does not work on the Pi 5. All GPIO code uses **gpiozero on the lgpio backend** (default on current Pi OS). `spidev` and `luma.oled` both work on Pi 5.

## 2. GPIO / Bus Allocation (v3)

The WM8960 HAT claims I2S (GPIO 18–21), I2C control (GPIO 2/3, addr **0x1a**), and an onboard button on **GPIO 17** (reserved). Everything else passes through the stacking header.

| Function | GPIO | Phys pin | Notes |
|---|---|---|---|
| I2S audio (WM8960) | 18, 19, 20, 21 | 12, 35, 38, 40 | Reserved — do not touch |
| I2C1 SDA / SCL | 2 / 3 | 3 / 5 | Shared: WM8960 codec @ 0x1a + PN532 @ 0x24 — no collision. Verify with `i2cdetect -y 1` |
| WM8960 onboard button | 17 | 11 | Reserved by HAT |
| SPI0 MOSI / SCLK | 10 / 11 | 19 / 23 | OLED only — sole SPI device now |
| OLED chip-select (CE0) | 8 | 24 | SSD1309 CS |
| OLED D/C | 25 | 22 | Data/command |
| OLED RST | 24 | 18 | Reset |
| Play/pause | 5 | 29 | Button → GND, internal pull-up, active-low |
| Prev | 6 | 31 | ″ |
| Next | 16 | 36 | ″ |
| Vol up | 22 | 15 | ″ (vertical pair, faceplate right) |
| Vol down | 23 | 16 | ″ |
| Power button | — (J2 pads) | — | Soldered to the Pi 5's J2 pads — clean shutdown *and* wake |
| Power button LED | — (3V3) | 1 or 17 | LED + ~330Ω to 3V3 |
| **Free** | 4, **7**, 9, 12, 13, 26, 27 | — | GPIO 7 (CE1) freed by dropping the matrix |

### Electrical notes

- **Single voltage domain:** everything is 3.3V logic now. No level shifting anywhere in the build.
- **All buttons:** one leg to GPIO, other to GND; `gpiozero.Button(pin, pull_up=True, bounce_time=0.05)`.
- **PN532 jumpers → I2C mode** before wiring.
- **Perma-Proto is the power hub only** — one 3V3 rail + one GND rail; all signal wires point-to-point. Layout in the Wiring Reference, View 3.
- Stack order: Pi 5 → Active Cooler → tall stacking header (≥16mm) + standoffs → WM8960.

## 3. Software Stack (v3)

| Layer | Choice | Why |
|---|---|---|
| OS | Raspberry Pi OS Lite (64-bit, Bookworm+) | Headless; lgpio/gpiozero current |
| Player core | MPD | Gapless, `python-mpd2`, NAS = mounted share later |
| Audio out | `dtoverlay=wm8960-soundcard` → ALSA | MPD `audio_output type "alsa"`. Verify mixer name with `amixer`, set `mixer_type` accordingly |
| NFC service | `nfc-agent.py` | Poll PN532 over I2C (`adafruit-circuitpython-pn532`), UID → card map → drive MPD via `python-mpd2`. Re-tapping the currently-playing card toggles pause |
| Display service | `display-agent.py` | `python-mpd2` idle events → `luma.oled` (ssd1309) on SPI0/CE0. Artist/title/elapsed, scrolling long titles, transient volume bar, idle clock |
| Buttons | `buttons-agent.py` | gpiozero → MPD: play/pause, prev, next; vol ±5 per press with hold-to-repeat (`hold_time=0.4`, `hold_repeat=True`) |
| Card admin | Flask app (`card-admin`) | See §4 |
| Card map store | SQLite (`cards.db`) | Rows-with-types keeps mixtapes clean |
| Service mgmt | systemd units per agent | Auto-start, restart-on-failure, journald |

**Removed in v3:** MPD FIFO output, cava, `spectrum-agent.py`, `luma.led_matrix`. The FIFO tap and its risks (parallel-output stalls, cava latency tuning) are gone with them.

**The OLED now carries the whole visual load.** With no matrix, `display-agent.py` deserves more design attention than v2 gave it — it's the only dynamic element on the face. Worth considering: larger typography now that nothing competes with it, a compact progress bar, album art as 1-bit dither on track change, or a slim level meter along one edge if some visual motion is still wanted.

### 3a. Card map schema

```sql
CREATE TABLE cards (
  uid TEXT PRIMARY KEY,       -- NFC UID, hex string
  type TEXT NOT NULL,         -- 'album' | 'playlist'
  target TEXT NOT NULL,       -- album: MPD path e.g. 'Artist/Album'
                              -- playlist: MPD stored-playlist name
  label TEXT,                 -- human-readable, shown in admin UI
  created_at TEXT DEFAULT (datetime('now'))
);
```

`nfc-agent` on tap: `album` → `clear; add <path>; play`. `playlist` → `clear; load <name>; play`. Unknown UID → log it and show a prompt on the OLED (was: flash "?" on the matrix); the admin UI picks unknowns up for pairing.

## 4. Flask card-admin app

Flask + `python-mpd2` on the Pi, reachable at `http://<pi>:5000` from the laptop. No auth beyond the LAN.

**Core (registration):**
- **Pair mode:** page shows "tap a card…" → nfc-agent (in pair mode via Unix socket) reports the UID instead of playing → UI shows the UID plus an MPD library browser (artists → albums via `list`/`lsinfo`) → pick album, name the card, save.
- **Card list:** all registered cards with label/type/target; re-pair and delete.
- **Unknown-UID inbox:** cards tapped but unregistered show up ready to pair.

**Stretch (mixtape cards):**
- A card can point at an **MPD stored playlist** instead of an album.
- Mixtape builder: search/browse the library, order tracks, save via `playlistadd`/`save` — the playlist lives in MPD, editable from any MPD client.
- Later: per-card shuffle flag (`random 1` before play).

Keeping playlists inside MPD means the card map stays a thin pointer layer.

## 5. Project Plan (v3 phases)

### Phase 0 — Bench prep
Flash Pi OS Lite; enable SSH, I2C, SPI. Fit Active Cooler, tall stacking header + standoffs, WM8960. Boot, confirm cooling, `i2cdetect` shows 0x1a.

### Phase 1 — Core audio
WM8960 overlay in `config.txt`, verify ALSA device, install MPD, point at test music, `mpc play` → **music from the speakers.** Confirm mixer name for `setvol`; find the distortion ceiling now (~1W/channel).

### Phase 2 — Display
Wire the OLED (CE0 + DC/RST). Test via luma examples, then write `display-agent.py` → **track info on the OLED.** This is now the main UI surface, so budget real time here.

### Phase 3 — NFC + buttons
PN532 on the shared I2C bus, verify 0x24. Buttons wired to the Perma-Proto, power button soldered to J2. Write `nfc-agent.py` and `buttons-agent.py` → **tap a card, an album plays; full physical control.**

### Phase 4 — Flask card-admin
Registration flow + card list + unknown-UID inbox, then the mixtape stretch goal → **new cards paired from the couch.**

### Phase 5 — Integration hardening
systemd units for all agents; unattended reboot test; clean-shutdown test via power button; volume-bar IPC between buttons-agent and display-agent.

### Phase 6 — Enclosure
Frontplate is designed and parametric ([[Frontplate Design - v3|Retro NFC Music Player - Frontplate Design - v3]], 220×185). Remaining: measure ear-hole spacing and module hole spacings, print, dry-fit, then trim the harness to measured lengths + 20mm slack. Cabinet body (sides/back/base) still to design — needs Pi-stack mounting, ventilation for the Active Cooler, and rear port access.

## 6. Risks & Open Questions

- **I2C bus sharing** (WM8960 ctrl + PN532): addresses don't collide; if PN532 polling ever glitches audio config, it can fall back to SPI1 or UART.
- **Pair-mode IPC** between Flask and nfc-agent — leaning Unix socket; decide in Phase 4.
- **WM8960 output power** (~1W/ch BTL) — fine for a desktop cabinet; a small I2S amp or line-out to powered speakers is the escape hatch.
- **OLED as sole display** — if the face feels static in practice, that's a `display-agent` design problem to solve in software, not a reason to reinstate the matrix.
- **Speaker ear-hole spacing** still estimated; it sets the faceplate width, so measure before printing.

*Resolved and closed by v3: MAX7219 3.3V logic margin, cava latency/feel, MPD FIFO parallel-output stalls — all moot with the matrix gone.*
