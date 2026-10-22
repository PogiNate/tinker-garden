---
title: Retro NFC Music Player - Design and Project Plan - v3
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-design-and-project-plan-v3
tags:
- design-doc
- retro-nfc-music-player
- gpio-map
- software-stack
updated_by: Nathanial Dickson
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


## Build status

**2026-09-26 — hardware layer proven.** Faceplate wired to the Pi. Verified: all five transport/volume buttons; `i2cdetect -y 1` shows `1a` (WM8960) and `24` (PN532); power button on J2 performs a full clean shutdown and restart. Backlight and power LEDs live on the 3V3 rail. Not yet exercised: OLED (SPI — will never appear in `i2cdetect`) and PN532 card reads. Next: add user to `i2c,spi,gpio` groups, set up a `--system-site-packages` venv with `luma.oled` + `adafruit-circuitpython-pn532` + `rpi-lgpio` (Pi 5 needs the lgpio shim in place of `RPi.GPIO` for luma), then run the OLED and PN532 smoke tests from the Wiring Reference's first-power-up checklist. Phases 0–1 and the hardware half of Phase 3 are effectively complete; Phase 2 (display-agent) is next.


**Tooling decision (2026-09-26): all Python via `uv`.** Code lives in a uv project (`retro-player/`) rather than a loose venv. Dependencies: `gpiozero rpi-lgpio luma.oled adafruit-circuitpython-pn532 adafruit-blinka python-mpd2` (Flask added at Phase 4). Pi 5 trap: `luma.core` and `adafruit-blinka` may pull in `RPi.GPIO`, which collides with `rpi-lgpio` (both provide the `RPi` package) — check `uv tree`, and if present exclude it with `[tool.uv] override-dependencies = ["RPi.GPIO ; sys_platform == 'never'"]`. systemd units (Phase 5) should call `.venv/bin/python` directly rather than `uv run`.


**2026-09-26 (later) — PN532 reads verified; environment fixes.** `uv run rfid_test.py` returns clean UIDs from two tap cards (DIP switch + I2C wiring confirmed). Fixes found along the way:
- **Python pinned to 3.13** (`uv python pin 3.13`). The initial build failed looking for `Python.h` while compiling `lgpio`; pinning 3.13 resolved it. Keep this pin on any rebuild.
- **`spidev` must be added explicitly** (`uv add spidev`). `luma.core` treats it as optional, so `luma.oled` alone gives `ModuleNotFoundError: No module named 'spidev'` at `spi()`.
- Current dependency set: `gpiozero rpi-lgpio luma.oled spidev adafruit-circuitpython-pn532 adafruit-blinka python-mpd2`.


**2026-09-26 — all faceplate hardware verified.** OLED smoke test passes over SPI0/CE0 (after adding `spidev`). With the earlier button, power/wake, `i2cdetect`, and PN532 card-read checks, every faceplate component is now proven end to end. **Hardware phase complete.** Remaining work is software only: MPD + WM8960 audio (Phase 1 playback), `display-agent` (Phase 2), `nfc-agent` + `buttons-agent` (Phase 3 software), Flask card-admin (Phase 4), systemd hardening (Phase 5).


**Speakers — electrical (2026-09-26):** Waveshare kit speakers are **8Ω, 5W**, wired directly to the WM8960 HAT's speaker outputs. The WM8960 delivers ~1W/ch into 8Ω, so the speakers have ~5× headroom and the **amp, not the speakers, sets the loudness limit**. Phase 1 tasks: find the clipping ceiling by ear, then have `buttons-agent` clamp volume to it (and scale the OLED volume bar to that ceiling). Point MPD's `mixer_control` at the WM8960 **speaker** control (confirm name via `amixer -c wm8960soundcard`), not the headphone control. Outputs are **BTL**: never ground either speaker lead, never share leads between channels, never parallel the speakers.


**2026-10-04 — code under git; README moved to the repo.** `~/retro-player` on the Pi (hostname `tap-player`, user `nate`) is now a git repo pushed to [PogiNate/retro-player](https://github.com/PogiNate/retro-player) (`main`, over an SSH key set up on the Pi). The first commit holds the uv project, the OLED/PN532 smoke tests, and a v3 `README.md`. **Decision:** the README lives in the repo only. The stale vault `retro-player/README.md` (from the HiFiBerry/VU-meter design) was deleted. Also confirmed on the Pi: `aplay -l` lists `wm8960soundcard`, and `mpd`/`mpc` are installed. MPD configuration is still to do (Phase 1).


**2026-10-04 — Waveshare driver removed; stock WM8960 driver in use.** Waveshare's `install.sh` had been run on 2026-09-26. It installed out-of-tree DKMS codec modules, overwrote the stock `wm8960-soundcard.dtbo`, stashed the stock `snd-soc-wm8960` module, and added a boot service that replaced `/etc/asound.conf` and the ALSA mixer state on every boot. Its `uninstall.sh` uses pre-Pi 5 paths, so the removal was done by hand. The stock module and overlay were restored and verified against the `linux-image-6.18.50+rpt-rpi-2712` package checksums. Backups are in `~/waveshare-removal-backup-2026-10-04/` on the Pi. **Don't run Waveshare's installer again.**
- `config.txt` `[all]`: `dtparam=i2s=on` + `dtoverlay=wm8960-soundcard` only (removed `i2s-mmap`).
- `/etc/asound.conf`: default PCM/ctl → `wm8960soundcard` by name. The card is now **card 2** (HDMI takes 0/1), so always refer to it by name.
- Mixer state saved in `/var/lib/alsa/asound.state` (now a real file, so `alsactl store` persists): Speaker 86%, Playback 100%, L/R Output Mixer PCM on.
- MPD `device`/`mixer_device` typo fixed (`CARD=wm8960soundcard`). MPD output enabled; the library has 1 album from `/srv/music`.
- **Open issue — I2C errors during playback:** with the Waveshare driver, `speaker-test` runs logged `SDA stuck at low` / `lost arbitration` / codec register writes failing with `-121`. Playback was intermittent and sometimes had static. Not yet re-tested on the stock driver. Next steps: re-test, then try `dtparam=i2c_arm_baudrate=50000`. If errors persist, test with the PN532 disconnected from the I2C bus.


**2026-10-04 — Root cause of flaky audio: PN532 on the shared I2C bus.** The WM8960 is **write-only** over I2C, so the driver keeps a cache of what it sent and can't check the chip. With the PN532 on I2C1, starting and stopping playback produced `SDA stuck at low` / `lost arbitration`, and codec writes failed with `-121`. The failures hit the registers for speaker amp enable `0x31`, DAC mute `0x05`, power `0x1a` and clocking `0x04`. The driver's cache then said "on" while the chip stayed off, giving silent runs that persisted until a reboot, and sometimes buzz. Debugfs DAPM showed every widget "On" during silent playback. **Test:** with the PN532 SDA/SCL disconnected, there were 0 I2C errors across 6 `speaker-test` runs plus 15 s of MPD playback, and all voice runs were audible and clean. Errors cluster right when the class-D amp switches, so amp noise coupling into the PN532 wiring is likely involved. **Decision pending:** move the PN532 off I2C1 (own `i2c-gpio` bus, SPI, or UART) versus electrical fixes on the shared bus.
- Also set in MPD: `format "48000:16:2"`. The test album is 192 kHz/24-bit FLAC and the WM8960 tops out at 48 kHz, so MPD's soxr resampler does the conversion rather than `plughw`'s linear one.
- MPD volume maps on a dB scale: `mpc volume 50` → Speaker raw 109 (−12 dB, shown as 86% in amixer). Use the `mpc` number for the ceiling.


**2026-10-04 — Fixed: PN532 moved to its own I2C bus. Audio is reliable.** **Decision (resolves the "pending" item above):** the PN532's SDA → GPIO 26 (pin 37) and SCL → GPIO 27 (pin 13), on a bit-banged bus: `dtoverlay=i2c-gpio,bus=3,i2c_gpio_sda=26,i2c_gpio_scl=27` → `/dev/i2c-3`. Its power is unchanged. I2C1 now carries only the WM8960. Code opens the bus with `adafruit_extended_bus.ExtendedI2C(3)`, and `adafruit-extended-bus` was added to the uv project. **Verified:** `i2cdetect -y 1` shows only `UU` at `1a`, and `-y 3` shows `24`. Both cards read cleanly (`0409307bcf6180`, `04a1447bcf6180`). There were **0 I2C errors** across 4 `speaker-test` runs with the PN532 idle, plus 4 runs and 15 s of MPD playback with the PN532 polling continuously (157 card reads during audio). All runs were audible and clean, with no buzz. **This overrides §2's GPIO table** (I2C1 is no longer shared; 26 and 27 are no longer free) and resolves the §6 "I2C bus sharing" risk. Pin-level detail: [[Wiring Reference - v2|Retro NFC Music Player - Wiring Reference - v2]]. Repo commit `fbccfe9` also documents the working MPD/ALSA setup in the README. **Phase 1 status:** playback works; still to do is finding the volume ceiling by ear.


**2026-10-04 — Phase 1 complete: volume ceiling = 95 (MPD scale).** Tested by ear with "Orange Crush" (*Green*, 192 kHz FLAC resampled to 48 kHz) at `mpc volume` 60 → 70 → 80 → 90 → 100. It was clean at every step, including 100 (Speaker +6 dB, the control's maximum), so the amp never audibly clipped on this material. **Decision:** clamp at **95** for comfort, since 100 was a little loud for the room, not distorted. 95 → Speaker raw 126 (+5 dB). There were 0 I2C errors throughout. Speaker AC/DC boost was left at 4 (not changed).
- **buttons-agent:** clamp `mpc volume` to **0–95**; ±5 per press, so the top step lands exactly on 95.
- **display-agent:** 95 = a full volume bar.
- The clamp lives in software. MPD itself still accepts up to 100 from other clients.
- Next: Phase 2, `display-agent`.


**2026-10-06 — Phase 2 complete: `display-agent` running as a service.** Commit `4799717` on [PogiNate/retro-player](https://github.com/PogiNate/retro-player).
- **Repo layout (resolves the handoff's open question):** one package `src/retro_player/` with an entry point per agent (`retro-display` now; `retro-nfc`, `retro-buttons`, `retro-admin` to come). Shared code: `config.py`, `mpd_link.py` (connect, reconnect with backoff, channel messages), `messages.py`. The smoke tests moved to `tools/` as `retro-rfid-test` / `retro-oled-test`. Tested with pytest (30 tests, no hardware needed).
- **Config:** `config.toml` in the repo root, overridable with `RETRO_PLAYER_CONFIG`. Unknown keys are an error. It holds the volume ceiling 95, display timings, pins, `nfc.i2c_bus = 3` and the cards.db path.
- **Sync between agents:** every agent talks only to MPD and reacts to its idle events (`player`, `mixer`, `message`). Elapsed time is re-read on each player event and counted locally in between. Events MPD doesn't track (unknown card) go over **MPD's client-to-client channel** `retro-player` (`subscribe`/`sendmessage`), with the format `unknown_card <uid>`. This is also the leading option for Phase 4 pair-mode IPC, in place of a Unix socket.
- **Fonts (bundled with licences):** Spleen 8×16 for the title; Press Start 2P for the artist and times (the artist moved from Spleen 6×12, which was too thin on the panel). The splash uses Racing Sans One + Kaushan Script.
- **Screens:**
  - Now playing: artist / title / progress bar / `elapsed ▶ total`. Long lines scroll **one at a time**, title first, with a 2 s hold at 24 px/s.
  - Titles are tidied: edition tags (Remastered/Remaster/Deluxe/Expanded/Anniversary/Bonus Track) are stripped; Live/Acoustic are kept.
  - Pause: ❚❚ plus the elapsed time blinks VCR-style.
  - Volume: a 19-segment meter, 95 = full, shown for 1.5 s.
  - Unknown card: UID prompt for 4 s.
  - MPD down: "waiting for MPD".
- **Decisions (Nate):** **no idle clock**. The screen is **off when stopped**, and turns off after 5 min paused (burn-in protection).
- **Boot:** a ROM-style self-test with real read-only checks: AUDIO (card exists), NFC (PN532 acks on `/dev/i2c-3`), OLED, MPD (album count). Any FAIL holds the screen for 10 s. Then a static splash: **"RETRO" in Bayer-dithered 80s chrome** (bright top, black horizon stripe, lit floor) with a solid rim, and **"Player" in script** below right. Logo option 2 of 6 was chosen after viewing on the OLED. The name is **Retro Player**.
- **Service:** `systemd/retro-display.service` is installed and enabled (After/Wants mpd, Restart=on-failure, runs `.venv/bin/retro-display` as `nate`). SIGTERM exits cleanly so luma blanks the OLED. Stop/restart was verified clean.
- **Next:** Phase 3, `nfc-agent` + `buttons-agent`.


**2026-10-06 — Phase 3 complete: `nfc-agent` + `buttons-agent` running as services.** Commit `9dc847d`. All three agents (`retro-display`, `retro-nfc`, `retro-buttons`) are enabled systemd services. Nate verified every behaviour on the device, and there were 0 I2C errors throughout.
- **Card map** (`cards.py`, `/home/nate/retro-player/cards.db`): the handoff's `cards` schema plus an **`unknown_cards` inbox** (uid, first/last seen, tap count). Registering a card removes it from the inbox. Short-lived connections in WAL mode, so the Flask admin can share the file in Phase 4.
- **`retro-cards` CLI** (stopgap until card-admin): `list`, `inbox`, `add <uid|last> album|playlist <target> [--label]`, `remove`. `add` warns if MPD has no such album dir or stored playlist.
- **nfc-agent:**
  - Polls the PN532 on `/dev/i2c-3` in a worker thread (150 ms per poll). **A tap fires on card arrival**; a card counts as removed after 3 empty polls, so a resting card never retriggers.
  - Re-tapping the playing card toggles pause. **Removing a card does nothing.**
  - Unknown card → inbox + `unknown_card` message. Registered card whose target MPD rejects → **new `card_error <uid>` message**, shown on the OLED as "Can't play card".
  - Keeps its MPD connection alive with a background `idle` loop, so MPD's connection_timeout doesn't drop it between taps.
- **buttons-agent:** gpiozero on lgpio, with a blocking MPD client behind a lock that reconnects and retries once on a stale connection.
  - Play/pause: if stopped with a queue, it plays.
  - **Prev is CD-style:** more than 3 s into a track restarts it; otherwise it goes to the previous track.
  - Next.
  - Volume ±5 with hold-to-repeat every 0.4 s, clamped to **95**. Stepping down from above 95 lands at ≤95.
- **Cards:** `04a1447bcf6180` → album `REM/Green` (label "Green"). `0409307bcf6180` is in the inbox, not yet registered.
- Tests: 53 passing, with MPD faked.
- **Next:** Phase 4, Flask card-admin. Pair mode can use the MPD channel: admin sends `pair_mode on`, and nfc-agent replies `card_tapped <uid>` instead of playing.


**2026-10-06 — Phase 3 addition: auto-stop after 30 s paused (Nate's decision).** The panel has no stop button, so a pause longer than `[playback] pause_stop_after` (default **30 s**; 0 disables) becomes an MPD `stop`. The display already treats stopped as screen off, so this also blanks the OLED. It's implemented in **nfc-agent's** background player watcher: the timer arms on pause, cancels on any other state, and re-checks that playback is still paused before stopping. Verified on the device (paused 15:04:59 → stopped 15:05:29). Display's `paused_off_after` (5 min) stays as a burn-in backstop. After an auto-stop, re-tapping the card restarts its album from track 1. **Confirmed as intended by Nate: like switching on a CD player. Don't change it to resume.** Play/pause resumes at the start of the paused track. Commit after `9dc847d`; 57 tests pass.


**2026-10-06 — Music library moved to the NAS over Tailscale (the planned "NAS later" step).**
- **Tailnet:** Tailscale 1.102.5 installed on the Pi. Its node is **`tap-player` = `100.110.142.57`**; no extra features (SSH/exit node) enabled. NAS: **`disk-station` (Synology DS418play) = `100.97.42.6`**. The connection is direct over the LAN (about 4 ms).
- **Share:** `/volume1/Sync/Plex/Music`, NFSv4.1. DSM NFS rule on shared folder `Sync` for `100.110.142.57`: Read Only, **Map all users to admin**, `sys`, subfolder access allowed.
- **Gotcha, resolved:** mounts over Tailscale were "access denied" because Synology's Tailscale package defaults to **userspace networking**, where NFS sees every tailnet client as `127.0.0.1`. Fixed by switching the NAS to **TUN mode**: `/var/packages/Tailscale/target/bin/tailscale configure-host; synosystemctl restart pkgctl-Tailscale.service`, run once and **added as a root boot-up task in DSM Task Scheduler**, because DSM resets it on reboot. If NAS mounts ever start failing after a DSM update, check this first. A `127.0.0.1` export rule was rejected, since it would let every tailnet device mount the share.
- **Pi side:** `nfs-common`; `/etc/fstab` automount `100.97.42.6:/volume1/Sync/Plex/Music /srv/music nfs4 ro,vers=4.1,soft,timeo=150,retrans=3,noatime,_netdev,nofail,x-systemd.automount,x-systemd.after=tailscaled.service,x-systemd.mount-timeout=60`. It is **read-only**, verified that writes are refused. `nofail` keeps a NAS outage from blocking boot. MPD config is unchanged (`music_directory` is still `/srv/music`). The old local copy was moved to `/srv/music-local` (1.6 GB R.E.M. *Green*). `fstab` backup: `~/waveshare-removal-backup-2026-10-04/fstab.before-nas`.
- **Library:** about 244 GB. First `mpc update` took about 16 min: **779 artists, 604 albums, 11,201 songs**. Formats are mostly FLAC/MP3, plus WAV, M4A, AIFF and OGG. The only scan warnings were 364 malformed ID3 comment frames (harmless). Synology `@eaDir` folders are ignored automatically.
- **Folder layout is mixed** (`Artist/Album/` and `Artist - Album/` at the root). No reorganisation is needed: MPD indexes by tags, and card targets are just the album's folder path. **Phase 4's card-admin should browse by tags** (albumartist → album), not by folder.
- **Card re-pointed:** `04a1447bcf6180` → `REM/Green (25th Anniversary Deluxe Edition)`.


**2026-10-06 — Video files excluded from MPD.** MPD (0.24.4, ffmpeg decoder) was indexing and playing the 36 music-video files Plex keeps beside the albums: 28 `.mp4` (twenty one pilots), 6 `.m4v` (Peter Gabriel *So+*), 1 `.mov`, 1 `.avi`. Fix: Nate created **`/volume1/Sync/Plex/Music/.mpdignore` on the NAS** with `*.mp4 *.MP4 *.m4v *.M4V *.mov *.MOV *.avi *.AVI *.mkv *.MKV *.webm *.wmv *.mpg *.mpeg`. A root `.mpdignore` applies to all subfolders, and the matching is case-sensitive. After `mpc update` (about 2.5 min) **0 videos remain; songs 11,201 → 11,165**. The audio beside the videos is kept (e.g. `Heathens/1/1-Heathens.flac`). `.m4a` audio is unaffected. Plex ignores the dotfile. A local overlay mount was considered as an alternative and rejected as more moving parts than it's worth.


**2026-10-06 — Phase 4 complete: card-admin web app with pairing and remote.** Commit `c7074f5`. It runs as **`retro-admin.service`**, so all four services (display, nfc, buttons, admin) are now enabled. Nate tested pairing and the album picker on his phone.
- **URL: https://tap-player.ermine-minnow.ts.net** (the remote is at `/remote`). The full `…ts.net` name is required, because the certificate doesn't cover the short name.
- **HTTPS (Nate's request):** `sudo tailscale serve --bg --https=443 http://127.0.0.1:5000`. That gives a real **Let's Encrypt** cert (valid to 2027-01-04, auto-renewed by Tailscale; the first request took about 30 s to issue it), and the setting persists across reboots. **Decision:** the app binds to **127.0.0.1 only** (new `[admin]` section: host/port), so it's reachable only over HTTPS on the tailnet. The tailnet is the authentication, so there's still no login. Plain `http://<LAN IP>:5000` no longer works; set `host = "0.0.0.0"` to allow it.
- **Pair-mode IPC (resolves the handoff's open question): the MPD channel, not a Unix socket.**
  1. Admin sends `pair_mode on|off`.
  2. nfc-agent (now subscribed, idling on `player` + `message`) reports the next tap as `card_tapped <uid>` instead of playing. It's one tap per pairing, with a 60 s timeout (`PAIR_TIMEOUT`).
  3. The display shows "PAIR MODE · tap a card" (blinking), then "GOT IT!".

  The web app keeps **one background thread subscribed** to the channel and remembers the last tap, and the page polls `/api/pair/poll?since=`. That way a tap can't be lost between HTTP requests. Leaving the page mid-pairing sends `pair_mode off`.
- **Album index (`admin/library.py`): the unit is the album folder**, since that's what a card targets. It's built from `listallinfo` (11k songs in about 0.2 s) and rebuilt only when `stats.db_update` changes.
  - `Disc N`/`CD N` subfolders fold into their parent.
  - Artist = the most common albumartist→artist tag; "Various Artists" if no majority; folder layout for untagged folders. Title = the majority album tag, else the folder name. Tags are cleaned of BOMs (the library has `﻿` in some tags).
  - Result: **611 albums** (8 `Disc` splits merged; *Star Control* soundtrack no longer split across 3 artists; 11 untagged game OSTs remain "Unknown artist").
- **UI** (phone-first, dark neon theme, the Press Start 2P / Racing Sans One / Kaushan Script fonts served from the package):
  - **Cards:** list with ▶ Play now; edit (rename, re-point, unpair); "missing: <target>" warning.
  - **Inbox:** Pair or dismiss.
  - **Pair:** tap → pick (Search / Artists / Playlists) → name → save. Re-pairing a known card shows what it currently plays.
- **Remote (Nate's request):** cover, title (tidied like the OLED), artist, album, progress (blinks when paused), ⏮ ⏯ ⏭, and volume ± with a 19-segment meter capped at 95. It **reuses `buttons.agent.Controller`**, so its behaviour is identical to the physical buttons, and it polls `/api/status` every second.
  - **Cover art:** MPD `readpicture` (embedded) → `albumart` (`cover.*` only) → `folder.jpg`/`front.jpg` read straight from `/srv/music` (Plex's convention). Resized to ≤600 px JPEG with an LRU cache per folder: *Green*'s 580 KB embedded art becomes 104 KB, 0.2 s the first time and 0.02 s cached.
- Tests: 72 passing.
- **Next: Phase 4b, the mixtape builder** (Nate confirmed he wants it): build MPD stored playlists in card-admin and pair cards to them.


**2026-10-06 — Phase 4b complete: mixtape builder.** Commit `be04f87`; 86 tests pass.
- **Mixtapes = MPD stored playlists** (in `/var/lib/mpd/playlists`, on the Pi, unaffected by the read-only NAS). The **tape type** lives in a new `mixtapes` table in `cards.db` (name, `C60`|`C90`).
- **Tape rule (Nate liked the limitation):** C-60 = 30 min/side, C-90 = 45. Tracks fill **Side A in order**; the first track that doesn't fit starts Side B, since a song can't span the flip. **Saving is blocked if Side B overflows**, which is checked on the server as well as in the browser. The default is C-90.
- **Editor** (card-admin → Mixtapes tab):
  - name; C-60/C-90 toggle; live A/B meters; per-track A/B tags and a "— SIDE B —" marker
  - add songs by search (`search any`, window 0:80) or by album (`find base <path>`, single tracks or "Add all")
  - **drag ☰ to reorder** with vendored **SortableJS 1.15.6** (MIT). Nate chose drag; ▲/▼ buttons are the fallback if it misbehaves.
  - Save writes the playlist with `playlistclear` + `playlistadd`. **Rename** = MPD `rename`, and cards are re-pointed in `cards.db` (their label follows unless customised). **Delete** = `rm`; affected cards then show as "missing".
  - **"⊕ Pair a card"** opens the pair flow with the mixtape preset (`/pair?playlist=`).
- **Tape clunk (Nate's idea):** chosen by ear (candidate #2 of 7) from CC0 Freesound recordings. It's trimmed from ["Tapedeck open and closing" by stecman](https://freesound.org/people/stecman/sounds/376059/) (1.59–2.00 s + tail, peak −6 dBFS, 0.81 s) and bundled as `sounds/tape-close.wav` with `CREDITS.txt`.
  - On a **mixtape card tap**, nfc-agent: `stop` → `clear` → `load` (if the mixtape is missing it shows "Can't play card" and **no clunk**) → sends `mixtape <name>` → plays the WAV with **`aplay` straight to ALSA** (retrying while MPD releases the device) → `play`. The clunk never appears as a track.
  - **Album cards don't clunk.** Config: `[playback] mixtape_sound = true`, `alsa_device = "plughw:CARD=wm8960soundcard"`.
  - The **display** shows a **cassette screen** (the reels animation from splash concept A, with the mixtape name on the label) for 2.5 s.
- **Rejected:** a BudgetPixel "Cassette Insert Click" (CC BY 4.0, AI-generated, a Walkman click rather than a deck door).
- **Channel messages are now shell-quoted** (`shlex`), so arguments may contain spaces. Older single-token messages are unchanged on the wire.
- **Skipped for now (Nate):** per-card shuffle.
- ✅ **Verified on the device (2026-10-06 16:51):** Nate built the first mixtape, **"90's Stylez" (C-60)**, and paired a **new third card `04bd037ccf6180`** to it. The log shows the full chain: unknown card → pair mode on → `pair mode: reporting 04bd…` → tap → `card 04bd… -> playlist 90's Stylez` (clunk via aplay, no warnings) → later auto-stop after 30 s paused. The apostrophe in the name passed through the shell-quoted channel messages correctly.
- **Cards now:** `04a1447bcf6180` → *Green* (album); `0409307bcf6180` → *Blurryface* (album); `04bd037ccf6180` → *90's Stylez* (mixtape).


**2026-10-06 — Album cards show a dithered cover (Nate's idea; "VERY good").** Commit `a8cb17c`; 88 tests pass. Verified on the device with the *Blurryface* card.
- **Choices (from a comparison sheet of 6 real covers × 3 dithers):** **Atkinson** dithering (the classic Mac look: bold shapes, clean whites) won over Floyd–Steinberg (grainy darks) and Bayer (mesh-like on photos). The image is autocontrasted and sharpened first. **Layout: a 64×64 cover on the left; album title on the right** (tidied, so "Green (25th Anniversary Deluxe Edition)" → "Green"; up to 3 lines) **and artist at the bottom** (up to 2 lines). Light, low-contrast covers such as *Green* dither poorly; that's a 1-bit limit.
- **Animation (Nate):** **dial-up load**, with the cover drawing in top to bottom with a bright scanline (0.6 s). It **holds until 3 s**, then **fades out** into the now-playing screen. Because the panel is 1-bit, the "fade" is an **ordered Bayer-8 dissolve** (0.6 s).
- **Plumbing:** after `play`, nfc-agent sends a new `album <path>` message. display-agent (which now keeps a reference to its MPD client) runs `find base <path>`, fetches the first track's cover and dithers it in a worker thread (about 26 ms; 2 ms per frame). Any cover failure just skips the animation. Mixtape cards keep the cassette screen.
- **Refactor:** new `covers.py` holds the cover lookup (embedded `readpicture` → `albumart` → `folder.jpg`/`front.jpg` on disk), shared by the OLED and the web remote. New config `[mpd] music_directory = "/srv/music"` (previously hard-coded in card-admin).
- **Fix:** Spleen has no "…" glyph (it rendered as "o"). Truncation now uses "..", which also fixes the cassette label.
- **Next: Phase 5, hardening** (unattended reboot, power-button shutdown and wake).


**2026-10-06 — Phase 5 (hardening) in progress.**
- **Pre-flight audit:** all services enabled; no failed units; journal is persistent (`/var/log/journal`); NAS automount is `nofail`; `tailscale serve` is saved; EEPROM has `POWER_OFF_ON_HALT=1` and `WAKE_ON_GPIO=0` (J2 button wakes).
- **Recovery tests ✅:** SIGKILL of each agent → systemd restarts it within 2 s (`NRestarts=1`). `systemctl restart mpd` → display and nfc reconnect within about 3 s; the remote keeps working. **Fixed:** python-mpd2 logged a traceback from its idle callback on every MPD restart. `mpd_link` now swallows that one case, since the reconnect already handles it.
- **Decision (Nate): power-on is always stopped, like a CD player.** `/etc/mpd.conf` has `restore_paused "yes"` (backup: `~/waveshare-removal-backup-2026-10-04/mpd.conf.before-restore-paused`), and nfc-agent's first MPD session after start stops a restored pause. The queue is kept. Simulated on the device: playing → MPD restart → paused → nfc start → **stopped** ✅.
- Commit for the above is after `a8cb17c`.
- **Still to do (needs a real reboot, which ends the Claude Code session running on the Pi):**
  1. **Unattended reboot** (`sudo reboot`, nothing playing): check boot time to splash (`systemd-analyze`), all services active, self-test lines, NAS mount on access, HTTPS remote, a card tap works, 0 I2C errors.
  2. **Power button while playing:** check for a clean shutdown in the previous boot's journal (`journalctl -b -1`), wake via button, and that it comes up **stopped**.


**2026-10-06 — ✅ Phase 5 complete: hardened.** Final commit is after `74012c6`.
- **Unattended reboot (17:12):** **14.3 s** to multi-user (1.3 s kernel + 13.0 s userspace; the slowest steps are mpd at 6.3 s and NetworkManager-wait-online at 6.0 s). The display agent started at 14.2 s, the self-test ran 15.2–16.6 s with **all OK (MPD 604 albums)**, then the logo, then the screen went off (stopped). All services active with 0 restarts. NAS automounted over Tailscale; HTTPS remote OK; 0 I2C errors.
- **Found and fixed: logs didn't survive reboots.** Raspberry Pi OS sets `Storage=volatile` (`/usr/lib/systemd/journald.conf.d/40-rpi-volatile-storage.conf`). Added `/etc/systemd/journald.conf.d/90-retro-player-persistent.conf` with `Storage=persistent` and `SystemMaxUse=50M`. (The earlier pre-flight wrongly took the empty `/var/log/journal` folder as proof of persistence.)
- **Power button while playing, first try (17:13:56):** logind powered down; agents stopped in about 1 s; woke via J2; **came up stopped** ✅ (`start-up: playback restored paused, stopping`). **Found and fixed a shutdown ordering bug:** systemd unmounted `/srv/music` (failed: busy) and stopped tailscaled **before MPD**, so MPD stalled for about 27 s ("Decoder is too slow; playing silence") until power-off. Fix: `/etc/systemd/system/mpd.service.d/retro-player.conf` with `After=srv-music.automount srv-music.mount tailscaled.service remote-fs.target`. It's **ordering only**, so a NAS outage still never blocks MPD at boot.
- **Retest (17:17:12):** MPD stopped at +0.55 s → `/srv/music` unmounted cleanly → tailscaled stopped → power-off. **About 3.5 s in total**, with no failed unmount and no stall. Woke **stopped**; everything active; 0 I2C errors.
- Copies of both drop-ins are in the repo under `systemd/system-overrides/`, with install steps in the README.
- **Earlier in Phase 5:** recovery from SIGKILL (all agents back within 2 s), MPD restart (reconnect in about 3 s, traceback noise removed), and "power-on is always stopped" (`restore_paused "yes"` + nfc-agent stops a restored pause).
- **The project plan's software phases 1–5 are all done**, plus extras: NAS over Tailscale, HTTPS remote, mixtapes with C-60/C-90 sides and the tape clunk, dithered album covers. **Remaining from the original plan: Phase 6, the enclosure** (frontplate print and dry-fit, harness trim, cabinet body design).


**2026-10-14 — Enclosure built; printable card labels merged; front-panel button issue open.**
- **Phase 6 (enclosure): complete** (Nate).
- **Card labels (PR #1, Nate):** `/labels` page in card-admin with ten 3×2″ slots matching the R011_SI-LABEL-LS-0302 letter sheet. Each slot is an album (blurred-cover design, Righteous/Rubik) or a mixtape (cassette label in a Sunset/Ocean/Avocado colorway, with a C-60/C-90 field and a handwritten Caveat name). Output is a 300 dpi PDF with 1/16″ bleed and optional cut lines. Reviewed and deployed. **Review fixes** (commit `88bc2d4`):
  1. Album paths are validated against the library index. The path also reached `covers.from_disk`, so `../..` could have read `folder.jpg`/`cover.jpg` from outside `/srv/music`.
  2. The render cache was cut from 32 to 10 labels (about 1.8 MB each).
  3. Blob URLs in `labels.js` are now bounded and revoked.
  4. Admin tests are hermetic: `create_app(mpd_factory=…)`, a refusing MPD stub and an empty music directory.

  102 tests pass.
- **Open issue — buttons after the enclosure build (Nate):** **Play/pause doesn't pause while playing, but does resume when paused. Back seems not to work.** Pins 5/6/16/22/23 all read `hi` with pull-ups when idle (`pinctrl get`), and the agent is running with no errors. Each action is now logged (commit `c5d9c87`: "play/pause: was …", "prev: restart track (was Xs in)" / "previous track", "next"). **Hypotheses:** (a) contact bounce from the enclosure switches registers one push as two presses, so pause→resume looks like a no-op (the debounce is `bounce_time=0.05`); (b) Back works as designed but the CD-style rule (restart the track if more than 3 s in) feels like "nothing happened"; (c) a wiring fault on GPIO 6. **Next:** with Nate at the device, record the button log alongside MPD player events while he presses play/pause and back.


**2026-10-15 — ✅ Resolved: front-panel Play/Pause and Back after the enclosure build.** Commit `afb7aec`; 105 tests pass. Verified on the device by Nate, and the agent log shows one action per push.
- **How it was diagnosed:**
  1. The action log showed Play/Pause presses while playing, and every Back press, **never reaching the agent**.
  2. `pinctrl` sampling showed all pins idle-high during playback, which ruled out the amp pulling them low.
  3. A pin-edge recording showed **the switches do pull the pins low** on every press, so the wiring is OK.
  4. A raw gpiozero probe (agent stopped) found the cause: **the Play/Pause (GPIO 5) and Back (GPIO 6) switches chatter for about 0.4–0.5 s on every push**, flickering every few ms with the longest gap about 63 ms. **Next (GPIO 16) is clean.** With lgpio's debounce (`bounce_time=0.05`, which needs the level stable for 50 ms) they registered **0 of 6** presses.
- **Fix:** the transport buttons use raw edges plus a `ChatterFilter`, which acts on the first press after **150 ms with no edges**; every edge restarts the window. The volume buttons are clean and keep lgpio's debounce and hold-to-repeat. The hypotheses "CD-style Back rule" and "GPIO 6 wiring" were both ruled out.
- **Hardware note:** chatter this long is unusual. The likely cause is caps or plungers in the enclosure not pressing those two switches squarely or fully, or switches damaged during soldering. It's fully handled in software, but if those buttons ever feel scratchy, check the cap alignment.


**2026-10-15 — Track-list back labels and a printer calibration page.** Commits `bc35fd7` and `e3fed73`; 116 tests pass.
- **Track-list backs (Nate's stretch goal):** with **"Track lists"** ticked on `/labels`, the PDF gets **page 2: each slot's back label in the same position**, for the back of the same card.
  - **Album back:** a header band tinted from the cover with a **square thumbnail centred in the band** (Nate's change: header colour on all four sides), then a numbered two-column track list with durations, grouped by disc tag. It uses the largest size where every title fits, down to about 4 pt, then **"+N more"** (Nate chose to keep it).
  - **Mixtape back:** a J-card in the colorway, with handwritten SIDE A / SIDE B tracks on ruled lines and side times against the tape. A name with no saved mixtape gets a blank J-card to fill in by hand.
  - The slot editor shows a back preview.
- **Print-alignment investigation:** the backs printed low in row 1, and row 5's bleed reached into row 4. The layout matches the official **R011_SI-LABEL-LS-0302** template to within 0.006″ (row pitch 153 pt = 2.125″, label 216×144 pt). That pattern meant **the printer was scaling the page**.
  - **Added:** `/labels/calibration.pdf`, with label outlines and bleed zones to hold against a sheet, crosshairs A–D at **7.000″ across / 10.000″ down**, and 1/16″ rulers. **All label PDFs** now carry `/ViewerPreferences /PrintScaling /None`, added as an incremental update so the xref stays valid.
  - **Results:** the **Brother HL-L2460DW (mono laser) prints true.** The **Brother MFC-J1170DW (inkjet) is off by about 1/32″ per inch (about 3%)** on both axes, even at Acrobat "Actual size". **Use the laser printers for labels.** The colour laser that printed the first front sheet aligned correctly. If the inkjet ever has to be used, a per-printer scale correction in `config.toml` would be easy to add.