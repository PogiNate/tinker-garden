---
title: Software Handoff - v1
type: guide
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/software-handoff-v1
tags:
- retro-nfc-player
- handoff
- claude-code
- software
created_by: Nathanial Dickson
updated_by: Nathanial Dickson
---

# Retro NFC Music Player — Software Handoff — v1

Handoff brief for **Claude Code** picking up the software side of the build. Hardware is finished and verified; everything left is software. Read this first, then the docs it points to.

## 0. Ground rules for working on this project

- **Source of truth is the Basic Memory vault**, project `tinkering`. Always pass `project="tinkering"` on every basic-memory call — omitting it can silently resolve to the wrong project. If `tinkering` isn't in `list_memory_projects`, stop and tell Nate.
- Project folder: `40-49 Projects/41-Builds/41.01 Retro NFC Music Player/`. Start from the index note **41.01 Retro NFC Music Player** — it links every current and superseded doc.
- **Current design doc:** [[Design and Project Plan - v3|Retro NFC Music Player - Design and Project Plan - v3]] (architecture, GPIO map, software stack, phases, build log). **Pin-level detail:** [[Wiring Reference - v1|Retro NFC Music Player - Wiring Reference]].
- Vault follows **Johnny Decimal**. New project docs go inside the 41.01 folder; never write to the vault root or an ad hoc folder. Vault rules live in `00-09 Administration/04-AI Instructions/04.01 Basic-Memory.md`.
- **Doc conventions:** docs are versioned (`- v1`, `- v2`…); when superseding, write a new version and keep the old one. Keep the 41.01 index note current. Record corrections and resolved decisions explicitly so they aren't revisited.
- Basic Memory tip: `write_note` with `overwrite: true` is more reliable than chained `edit_note` calls when content has tables or complex structure; `edit_note` is fine for small single-spot edits.

## 1. ⚠️ The existing README is stale

`retro-player/README.md` in the vault **predates v3** and describes a dead design (HiFiBerry Amp2, peppyalsa, VU meters on PWM, `cards.json`, `pip` venvs). Do **not** follow its setup steps. Treat any `vu_agent.py` / peppyalsa / `asound.conf` meter-tap material as obsolete. `nfc_agent.py` from that era may be a useful starting reference, but it uses a JSON card map — v3 uses SQLite. ~~Rewriting the README to match v3 is part of this job.~~ **Resolved 2026-10-04:** the vault README has been deleted; the v3 README is `README.md` in the git repo.

## 2. Where things stand (as of 2026-09-26)

**Verified working on the Pi:**
- All five transport/volume buttons (gpiozero, internal pull-ups)
- `i2cdetect -y 1` shows `1a` (WM8960) and `24` (PN532)
- PN532 card reads — `uv run rfid_test.py` returns clean UIDs from two cards
- SSD1309 OLED smoke test over SPI0/CE0
- Power button on J2 — clean shutdown and wake (hardware-level, no software needed)

**Not done yet:** MPD + WM8960 audio playback, and every agent / service in §5.

## 3. Hardware the code talks to

Pi 5 (2GB) + Active Cooler → tall stacking header → Waveshare WM8960 Audio HAT (8Ω/5W speakers on the HAT's speaker outputs).

| Function | Bus / GPIO (BCM) | Notes |
|---|---|---|
| WM8960 audio | I2S 18/19/20/21; I2C ctrl @ `0x1a` | Never touch these pins. GPIO 17 = HAT's onboard button, reserved |
| PN532 NFC | ~~I2C1 (GPIO 2/3)~~ **I2C3 via `i2c-gpio`, SDA=26 / SCL=27** @ `0x24` | **Changed 2026-10-04:** sharing I2C1 corrupted WM8960 writes. Use `ExtendedI2C(3)`. See [[Wiring Reference - v2|Retro NFC Music Player - Wiring Reference - v2]] |
| SSD1309 OLED 2.42" | SPI0, CS=GPIO 8 (CE0), DC=25, RST=24 | Only SPI device |
| Play/pause | GPIO 5 | Active-low, `Button(pin, pull_up=True, bounce_time=0.05)` |
| Prev / Next | GPIO 6 / GPIO 16 | ″ |
| Vol up / Vol down | GPIO 22 / GPIO 23 | ″ |
| Power button | J2 pads | Handled by firmware — no code |
| Backlight + power LEDs | 3V3 rail | Always on, no GPIO |

Free GPIO if ever needed: 4, 7, 9, 12, 13. (26/27 are now the PN532's bus.)

## 4. Environment — decided, with known traps

- **OS:** Raspberry Pi OS Lite 64-bit (Bookworm+), headless. SSH, I2C and SPI enabled.
- **All Python via `uv`.** The code is a uv project, `retro-player/`, on the Pi. No pip, no loose venvs.
- **Python pinned to 3.13** (`uv python pin 3.13`) — without it `lgpio` fails to build (missing `Python.h`). Keep the pin.
- **Current deps:** `gpiozero rpi-lgpio luma.oled spidev adafruit-circuitpython-pn532 adafruit-blinka python-mpd2` (add `flask` in Phase 4).
- **`spidev` must be listed explicitly** — `luma.core` treats it as optional and `luma.oled` fails at `spi()` without it.
- **Pi 5: never use `RPi.GPIO`.** gpiozero runs on the lgpio backend; `rpi-lgpio` provides the `RPi` shim luma needs. If `uv tree` shows the real `RPi.GPIO` being pulled in (by `luma.core` or `adafruit-blinka`), exclude it:
  ```toml
  [tool.uv]
  override-dependencies = ["RPi.GPIO ; sys_platform == 'never'"]
  ```
- User must be in the `i2c`, `spi`, `gpio` groups (and `audio` for anything touching ALSA directly).
- **systemd units call `.venv/bin/python` directly**, not `uv run`.

## 5. Work to do (v3 plan, software only)

Work in order; each phase ends with something observable on the device.

### Phase 1 — Audio (do first)
- `dtoverlay=wm8960-soundcard` in `/boot/firmware/config.txt`; confirm the card appears in `aplay -l`. Verify the overlay/driver situation on the current Pi 5 kernel rather than assuming.
- Install MPD + mpc; MPD `audio_output` → `type "alsa"` on the WM8960 card.
- Mixer: list controls with `amixer -c wm8960soundcard` (verify card name). Point MPD's `mixer_control` at the **speaker** volume control, not headphone.
- **Done when:** `mpc play` produces music from the speakers and `mpc volume` changes it.
- **Find the clipping ceiling by ear with Nate** — the ~1W/ch amp, not the speakers, limits loudness. That ceiling becomes the max volume `buttons-agent` allows and the full-scale of the OLED volume bar.
- **Speaker safety:** outputs are BTL. Never ground a speaker lead, share leads between channels, or parallel the speakers.

### Phase 2 — `display-agent`
- `python-mpd2` idle events → `luma.oled` `ssd1309` on SPI0/CE0.
- Shows artist / title / elapsed, scrolls long titles, transient volume bar, idle clock, and an "unknown card" prompt.
- The OLED is now the **only** dynamic element on the faceplate (the LED matrix was dropped), so give it real design attention: larger type, compact progress bar; optional ideas are 1-bit dithered album art on track change or a slim level meter. Propose, don't over-build.

### Phase 3 — `nfc-agent` + `buttons-agent`
- **nfc-agent:** poll PN532 over I2C (`adafruit-circuitpython-pn532`) → look up UID in SQLite → drive MPD.
  - `album` → `clear; add <path>; play`
  - `playlist` → `clear; load <name>; play`
  - Re-tapping the currently playing card toggles pause.
  - Unknown UID → log it, record it for the admin inbox, and have the OLED show a prompt.
  - Supports a **pair mode** where it reports UIDs instead of playing (used by card-admin).
- **buttons-agent:** gpiozero → MPD: play/pause, prev, next; volume ±5 per press with hold-to-repeat (`hold_time=0.4`, `hold_repeat=True`), clamped to the Phase 1 ceiling.

**Card map schema (`cards.db`):**
```sql
CREATE TABLE cards (
  uid TEXT PRIMARY KEY,       -- NFC UID, hex string
  type TEXT NOT NULL,         -- 'album' | 'playlist'
  target TEXT NOT NULL,       -- album: MPD path 'Artist/Album'; playlist: MPD stored-playlist name
  label TEXT,
  created_at TEXT DEFAULT (datetime('now'))
);
```

### Phase 4 — Flask `card-admin`
- Flask + `python-mpd2`, `http://<pi>:5000`, LAN only, no auth.
- Pair flow: "tap a card…" → nfc-agent in pair mode reports UID → MPD library browser (artists → albums via `list`/`lsinfo`) → name and save.
- Card list with re-pair/delete; unknown-UID inbox.
- **Stretch:** mixtape cards — build an MPD stored playlist (`playlistadd`/`save`) and pair a card to it. Playlists live in MPD, so the card map stays a thin pointer layer. Later: per-card shuffle flag.

### Phase 5 — Hardening
- systemd unit per agent (auto-start, restart-on-failure, journald), ordered after MPD.
- Volume-bar IPC between buttons-agent and display-agent (or have display-agent react to MPD `mixer` idle events — simpler if it works).
- Unattended reboot test; clean-shutdown test via the power button.

## 6. Settled decisions — don't relitigate

- **MPD is the single source of truth.** Agents only talk to MPD; they don't share playback state with each other.
- The MAX7219 spectrum analyzer is **gone**, along with cava, the MPD FIFO output, and the level shifters. Don't reintroduce them; if the face feels static, solve it in `display-agent`.
- uv for everything; Python 3.13; gpiozero/lgpio; SQLite card map with a `type` column.
- Music source: local microSD/USB for now. NAS later is a pure storage change (mount the share as MPD's `music_directory`) — keep paths configurable, but don't build for it now.

## 7. Open questions — raise with Nate, don't guess

- ~~**Pair-mode IPC** between Flask and nfc-agent. Leaning toward a Unix socket; decide in Phase 4.~~ **Resolved 2026-10-06:** MPD client-to-client channel (`pair_mode on|off` / `card_tapped <uid>`). See the v3 build log.
- **Exact mixer control name** and **max volume ceiling** — need on-device testing with Nate listening.
- **Pi specifics not recorded in the vault:** hostname, login user, path to `retro-player/`, music directory location, and whether `retro-player/` is under git (and where the remote is). Confirm before writing config files or systemd units.
- ~~**Repo layout** — one package with entry points per agent vs. separate scripts. Propose one before scaffolding.~~ **Resolved 2026-10-06:** one package, `src/retro_player/`, with an entry point per agent, configured by `config.toml`. See the v3 build log.
- If PN532 polling ever disturbs the codec on the shared I2C bus, the fallback is moving the PN532 to SPI1 or UART — flag it, don't rewire.

## 8. Writing back to the vault

When a phase lands or a decision is made:
- Append a dated entry to the **Build status** section of Design and Project Plan - v3 (or create v4 if the design itself changes, and update the index note).
- ~~Replace the stale `retro-player/README.md` with a v3-accurate setup guide (keep its location).~~ **Resolved 2026-10-04:** the vault README was deleted. The README now lives in the git repo ([PogiNate/retro-player](https://github.com/PogiNate/retro-player)); keep setup docs there, not in the vault.
- Any new doc goes in the 41.01 folder and gets linked from the 41.01 index note.

Part of [[41.01 Retro NFC Music Player]].
