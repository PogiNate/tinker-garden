---
title: Retro NFC Music Player - Design and Project Plan - v2
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-design-and-project-plan-v2
tags:
- design-doc
- retro-nfc-music-player
- gpio-map
- software-stack
---

# Retro NFC Music Player — Design & Project Plan — v2

Supersedes [[Design and Project Plan - v1|Retro NFC Music Player - Design and Project Plan - v1]] (architecture/GPIO/software sections). Hardware baseline is the [[Parts Manifest - v2|Parts Manifest v2]]: Pi 5 (2GB) + Active Cooler, Waveshare WM8960 Audio HAT, SSD1309 2.42" OLED (SPI), MAX7219 32×8 green LED matrix, HiLetgo PN532 (I2C), 6 momentary buttons (power + play/pause + prev + next + vol up/down), plus 2× Adafruit Pixel Shifter (on hand) for the matrix's 5V logic. Detailed pin-by-pin wiring lives in [[Wiring Reference|Retro NFC Music Player - Wiring Reference]].

## 1. System Architecture

```ascii
                     ┌──────────────────────────────────────────────┐
                     │              Raspberry Pi 5 (2GB)              │
                     │                                                │
 NFC card ─► PN532 ──┤ I2C1 (0x24)      MPD (player core)            │
                     │                   │          │                 │
 Buttons ─► GPIO ────┤ gpiozero/lgpio    │      ┌───┴────┐            │
 (5 discrete pins)   │                   │      │ audio  │ fifo       │
                     │                   │      │ (ALSA) │ output     │
 OLED ◄── SPI0/CE0 ──┤ display-agent ◄───┘      │   │    │   │        │
 (SSD1309, track)    │  (mpd idle events)       │   ▼    │   ▼        │
                     │                          │ WM8960 │ cava/FFT   │
 Matrix ◄─ SPI0/CE1 ─┤ spectrum-agent ◄─────────┤  I2S   │ 32 bands   │
 (MAX7219, 32×8,     │                          └───┬────┘            │
  via pixel shifters)│  Flask card-admin (pair       │                │
 Laptop ◄── WiFi ────┤  cards, mixtape playlists)    ▼                │
                     │                        Speakers L/R (~1W BTL)  │
                     └──────────────────────────────────────────────┘
 Music: microSD/USB now ─► later: NFS/SMB mount of NAS (same MPD library)
```

Unchanged philosophy from v1: **MPD is the single source of truth.** The NFC agent tells MPD what to play, buttons send MPD commands, the OLED asks MPD what's playing, and the spectrum analyzer taps MPD's audio output. The NAS enhancement stays a pure storage change.

**Pi 5 note that touches everything:** legacy `RPi.GPIO` does not work on the Pi 5. All GPIO code uses **gpiozero on the lgpio backend** (the default on current Pi OS). `spidev`, `luma.oled`, and `luma.led_matrix` all work on Pi 5.

## 2. GPIO / Bus Allocation (v2)

The WM8960 HAT claims I2S (GPIO 18–21), I2C control (GPIO 2/3, addr **0x1a**), and has an onboard configurable button on **GPIO 17** (reserved — don't reuse). Everything else passes through the stacking header. Every button and display line below has its own discrete pin — no sharing except the intentional SPI/I2C buses.

| Function | GPIO | Phys pin | Notes |
|---|---|---|---|
| I2S audio (WM8960) | 18, 19, 20, 21 | 12, 35, 38, 40 | Reserved — do not touch |
| I2C1 SDA / SCL | 2 / 3 | 3 / 5 | Shared bus: WM8960 codec @ 0x1a + PN532 @ 0x24 — no collision. Verify both with `i2cdetect -y 1` |
| WM8960 onboard button | 17 | 11 | Reserved by HAT (optional use later) |
| SPI0 MOSI / SCLK | 10 / 11 | 19 / 23 | Shared by both displays (write-only; MISO GPIO 9 left free). Matrix side runs through Pixel Shifter A |
| OLED chip-select (CE0) | 8 | 24 | SSD1309 CS |
| OLED D/C | 25 | 22 | Data/command line |
| OLED RST | 24 | 18 | Reset line |
| Matrix chip-select (CE1) | 7 | 26 | → Pixel Shifter B → MAX7219 CS ("CS" pin on the 5-pin module; DIN→MOSI, CLK→SCLK, both via Shifter A) |
| Play/pause | 5 | 29 | Button → GND, internal pull-up, active-low |
| Prev | 6 | 31 | ″ |
| Next | 16 | 36 | ″ |
| Vol up | 22 | 15 | ″ |
| Vol down | 23 | 16 | ″ |
| Power button (switch) | — (J2 pads) | — | Soldered to the Pi 5's J2 power-button pads — mirrors the onboard PWR button (clean shutdown *and* wake) |
| Power button LED | — (3V3) | 1 or 17 | LED + ~330Ω series resistor to 3V3 → lit whenever powered. If you later want software control, move it to GPIO 12 through an NPN/MOSFET driver |
| Free for future | 4, 9, 12, 13, 26, 27 | — | Spares |

### Electrical notes

- **All buttons:** one leg to GPIO, other leg to GND; `gpiozero.Button(pin, pull_up=True, bounce_time=0.05)`. Ground returns share the proto board's GND rail.
- **MAX7219 logic runs at proper 5V** via 2× [Adafruit Pixel Shifter](https://www.adafruit.com/product/6066) (74HCT2G34, on hand): Shifter A carries DIN + CLK, Shifter B carries CS (second channel spare). Rated 10MHz — fine at luma's default 8MHz SPI; drop to 4MHz if edges look marginal. Cut the debug-NeoPixel trace on the back to keep it dark in the cabinet.
- **OLED + PN532 are 3.3V** — power from 3V3 rail, no level shifting.
- **PN532 jumpers → I2C mode** before wiring anything.
- **Perma-Proto is the power hub only** — 5V rail (matrix + shifters) on one edge, 3V3 rail (OLED, PN532, LED) on the other, grounds returning to Pi pins 39 and 9; all signal wires run point-to-point. Full layout in [[Wiring Reference|Retro NFC Music Player - Wiring Reference]] View 3.
- Stack order (from manifest): Pi 5 → Active Cooler → tall stacking header (≥16mm) + standoffs → WM8960. Keep the cooler's intake unobstructed.

## 3. Software Stack (v2)

| Layer | Choice | Why |
|---|---|---|
| OS | Raspberry Pi OS Lite (64-bit, Bookworm+) | Headless; lgpio/gpiozero current |
| Player core | MPD | Unchanged from v1 — gapless, `python-mpd2`, NAS = mounted share later |
| Audio out | `dtoverlay=wm8960-soundcard` (Waveshare overlay) → ALSA | MPD `audio_output type "alsa"` → WM8960. Verify mixer name with `amixer` and set `mixer_type` accordingly |
| Spectrum tap | **MPD FIFO output → cava (raw mode, 32 bars)** | Second MPD `audio_output type "fifo"` at `/tmp/mpd.fifo` runs in parallel with ALSA. cava reads the FIFO, does the FFT + log band-splitting + smoothing for free, and emits 32 bar values in `raw` output mode to its own FIFO. Alternative if cava disappoints: DIY numpy FFT in spectrum-agent reading `/tmp/mpd.fifo` directly (more control over ballistics, more work). peppyalsa (v1's plan) also still works but cava-on-MPD-FIFO is simpler than inserting an ALSA plugin chain. |
| NFC service | `nfc-agent.py` | Poll PN532 over I2C (`adafruit-circuitpython-pn532`), UID → lookup in card map, drive MPD via `python-mpd2`. Re-tap semantics kept from v1: tapping the currently-playing card toggles pause |
| Display service | `display-agent.py` | `python-mpd2` idle events → `luma.oled` (ssd1309 device) on SPI0/CE0. Artist/title/elapsed, scrolling for long titles, transient volume bar, idle clock |
| Spectrum service | `spectrum-agent.py` | Read cava's raw FIFO → map 32 bands to 32 columns × 8 rows on `luma.led_matrix` (MAX7219, `cascaded=4`, SPI0/CE1). Peak-hold dots optional for the full hi-fi look |
| Buttons | `buttons-agent.py` | gpiozero → MPD: play/pause, prev, next; vol up/down = `setvol ±5` per press, with hold-to-repeat (`hold_time=0.4`, `hold_repeat=True`) |
| Card admin | **Flask app** (`card-admin`) | See §4 — pairing UI + mixtape cards |
| Card map store | SQLite (`cards.db`) | One table; JSON felt fine in v1 but mixtapes make rows-with-types cleaner |
| Service mgmt | systemd units per agent | Auto-start, restart-on-failure, journald |

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

`nfc-agent` behavior on tap: `album` → `clear; add <path>; play`. `playlist` → `clear; load <name>; play`. Unknown UID → flash a "?" pattern on the matrix and log it (the admin UI picks unknowns up for pairing).

## 4. Flask card-admin app

Small Flask + `python-mpd2` app on the Pi, reachable at `http://<pi>:5000` from the laptop. No auth beyond being on the LAN (fine for this; add basic auth later if it ever leaves the house).

**Core (registration):**
- **Pair mode:** page shows "tap a card…" → nfc-agent (in pair mode, coordinated via a small flag file or Unix socket) reports the UID instead of playing → UI shows the UID and a browser of MPD's library (artists → albums via `list`/`lsinfo`) → pick album, name the card, save.
- **Card list:** all registered cards with label/type/target, re-pair and delete actions.
- Unknown-UID inbox: cards tapped on the player but not yet registered appear here ready to pair.

**Stretch (mixtape cards):**
- A card can point at an **MPD stored playlist** instead of an album.
- Mixtape builder page: search/browse the library, add tracks in order, save via MPD's `playlistadd`/`save` — the playlist lives in MPD, so it's also editable from any MPD client.
- Registering a mixtape card = pair mode + pick "playlist" + choose (or create) the playlist.
- Nice-to-have later: shuffle flag per card (`random 1` before play).

Architecture note: keeping playlists inside MPD (stored playlists) rather than in the Flask app's DB means the card map stays a thin pointer layer and MPD remains the single source of truth.

## 5. Project Plan (v2 phases)

### Phase 0 — Bench prep
Flash Pi OS Lite; enable SSH, I2C, SPI. Fit Active Cooler, tall stacking header + standoffs, WM8960. Boot, confirm cooling behavior, `i2cdetect` shows 0x1a.

### Phase 1 — Core audio
WM8960 overlay in `config.txt`, verify ALSA device, install MPD, point at test music, `mpc play` → **music from the speakers.** Confirm mixer name for `setvol`, set volume limits sensibly (~1W/channel — find the distortion ceiling now).

### Phase 2 — Displays
Wire OLED (CE0 + DC/RST) and matrix (CE1, through both Pixel Shifters) per §2. Test patterns via luma examples. Add MPD FIFO output, install cava, confirm raw band output. Write `display-agent.py` and `spectrum-agent.py` → **track info on the OLED, bars dancing on the matrix.**

### Phase 3 — NFC + buttons
PN532 (I2C jumpers set) on the shared bus, verify 0x24. Buttons wired to the Perma-Proto, power button soldered to the J2 pads. Write `nfc-agent.py` and `buttons-agent.py` → **tap a card, an album plays; full physical control.**

### Phase 4 — Flask card-admin
Registration flow + card list + unknown-UID inbox. Then the mixtape stretch goal. → **new cards paired from the couch.**

### Phase 5 — Integration hardening
systemd units for all agents + cava; unattended reboot test; clean-shutdown test via power button; volume-bar IPC between buttons-agent and display-agent.

### Phase 6 — Enclosure & 3D-printed frontplate *(future — own design doc when we get there)*
Inputs to capture during Phases 2–3 for the frontplate model: OLED module outline + active-area offset, matrix module outline (~128×32mm active area), 12mm power button hole, 5 transport/volume button holes + spacing, NFC read zone marker (PN532 reads through a few mm of PLA — mounts behind the plate), speaker openings, cabinet ventilation.

## 6. Risks & Open Questions

- **I2C bus sharing** (WM8960 ctrl + PN532): addresses don't collide; if polling the PN532 ever glitches audio config, PN532 can fall back to SPI1 or UART.
- **cava latency/feel** — if the bars feel laggy or twitchy, tune cava's `framerate`/`integral`/`gravity` before resorting to a DIY FFT.
- **FIFO output + ALSA in parallel** — MPD handles dual outputs fine, but verify the FIFO doesn't stall playback when no reader is attached (cava down); `always_on` and buffer settings may need a look.
- **Pair-mode IPC** between Flask and nfc-agent — flag file vs Unix socket vs tiny local HTTP; decide in Phase 4 (leaning Unix socket).
- **WM8960 output power** (~1W/ch BTL) — fine for a desktop cabinet; if it's not loud enough in practice, a small I2S amp or line-out to powered speakers is the escape hatch, not a redesign.

Decided this pass: spectrum via MPD FIFO + cava; card map in SQLite; playlists live in MPD (mixtape cards point at stored playlists); volume = ±5 per press with hold-to-repeat. **Locked at wiring time:** power button on J2 pads (GPIO-26 fallback dropped); matrix logic level-shifted to true 5V via 2× Adafruit Pixel Shifter (74HCT125 fallback dropped, 3.3V-margin risk eliminated).
