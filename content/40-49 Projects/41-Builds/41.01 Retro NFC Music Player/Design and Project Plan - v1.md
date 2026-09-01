---
title: Retro NFC Music Player - Design and Project Plan - v1
type: note
permalink: tinkering/retro-nfc-music-player-design-and-project-plan-1
---

# Retro NFC Music Player — Design & Project Plan

A self-contained, retro-styled music player: real analog VU meters, a now-playing display, physical transport buttons, built-in speakers, and NFC cards that start albums. Local files first; NAS streaming as a future enhancement.

## 1. System Architecture

```ascii
                        ┌─────────────────────────────────────┐
                        │            Raspberry Pi 4             │
                        │                                       │
 NFC card ──► PN532 ────┤ I2C          MPD (player core)        │
                        │               │        │              │
 Buttons ──► GPIO ──────┤ gpiozero      │     ALSA chain       │
                        │               │    (dmix +            │
 Display ◄── ST7789/    ├ SPI0 ◄── now- │     peppyalsa)       │
             SSD1322    │        playing│        │             │
                        │        service│        ├──► I2S ────┼──► HiFiBerry Amp2 ──► Speakers L/R
 VU meters ◄── RC ◄─────┤ PWM0/1 ◄── vu-service ◄┘ (FIFO)     │
              filter    │                                       │
                        └─────────────────────────────────────┘
 Music: microSD/USB now ──► later: NFS/SMB mount of NAS (same MPD library, no rearchitecture)
```

Everything talks to **MPD** as the single source of truth: the NFC service tells MPD what to play, the buttons send MPD commands, the display service asks MPD what's playing, and the VU service taps the ALSA output stream. This means the NAS enhancement is purely a storage change — mount the NAS share into MPD's music directory and rescan.

## 2. Hardware (Bill of Materials)

| # | Part | Purpose | Notes | Est. price |
|---|------|---------|-------|-----------|
| 1 | Raspberry Pi 4 (2GB) | Brain | Pi 5 also works ([HiFiBerry sells a Pi 5 + Amp2 bundle](https://www.hifiberry.com/shop/bundles/hifiberry-amp2-bundle-4/)); Zero 2 W is too tight on GPIO/headroom for this feature set | ~$45 |
| 2 | [HiFiBerry Amp2](https://www.hifiberry.com/shop/boards/amp2/) | DAC + 60W Class-D amp HAT | Powers the Pi from its own 12–24V input — one supply for everything. Drives passive speakers directly | ~$50 |
| 3 | 12V–24V DC supply, ≥5A (e.g. 19V laptop brick) | Power | Sized for amp peaks + Pi + peripherals | ~$20 |
| 4 | 2× full-range speaker drivers, 4–8Ω, 10–20W (e.g. Dayton Audio 3–4") | Built-in sound | Full-range avoids crossover design | ~$30 |
| 5 | PN532 NFC module (I2C mode) | Card reading | Preferred over RC522 — better range, reads more tag types; well supported ([Phoniebox docs](https://github.com/MiczFlor/RPi-Jukebox-RFID/tree/develop/components/rfid-reader/PN532)) | ~$8 |
| 6 | NTAG215 / MIFARE Classic 1K cards (13.56 MHz) | Album cards | Printable card stock available — print album art on them | ~$15/50 |
| 7 | 2× analog moving-coil VU meter modules (with backlight) | The retro soul | Common "TR-35" / TN-73 style panel meters; 500µA–1mA movement | ~$20 |
| 8 | **SSD1322 256×64 OLED, amber (SPI)** | Now-playing display | Decided: mono amber for maximum vintage. Supported by luma.oled and [mpd_oled](https://github.com/antiprism/mpd_oled) | ~$25 |
| 9 | 3× momentary buttons, panel-mount, chunky/retro style | Play/pause, prev, next | Get quality clicky ones — they're the main tactile interface | ~$8 |
| 9a | Rotary encoder (KY-040 or a quality Bourns PEC11 + retro knob) | Volume | Endless rotation, digital volume via ALSA/MPD; built-in push switch = mute. A vintage-style aluminum or bakelite knob sells the look | ~$5 |
| 10 | Passives: 2× 1kΩ trim pots, 2× series resistors (~10kΩ, calibrate), 2× RC filter (1kΩ + 10µF), hookup wire, protoboard | VU driver + wiring | Values calibrated per meter in Phase 3 | ~$10 |
| 11 | Enclosure: plywood/MDF box or 3D-printed, speaker grille cloth, walnut veneer or vinyl wrap | The look | Front panel: 2 meters up top, display center, buttons below, NFC pad marked on top surface | ~$30 |

**Total: roughly $260.**

## 3. GPIO / Bus Allocation

The Amp2 claims I2S (GPIO 18–21), GPIO 4 (mute), and I2C (GPIO 2/3) for its own config ([HiFiBerry GPIO usage](https://www.hifiberry.com/docs/hardware/gpio-usage-of-hifiberry-boards/)). Everything else is planned around that:

| Function | Pins | Notes |
|----------|------|-------|
| I2S audio (Amp2) | GPIO 18, 19, 20, 21 | Reserved — do not touch |
| Amp2 mute | GPIO 4 | Reserved |
| I2C bus (Amp2 config + PN532) | GPIO 2, 3 | Shared bus — PN532 (addr 0x24) doesn't collide with the Amp2's TAS5756 (0x4d) |
| VU meter L / R | GPIO 12 (PWM0), GPIO 13 (PWM1) | The two hardware PWM channels that *don't* overlap I2S — this is why the meters get jitter-free needles |
| SPI0 display | GPIO 10 (MOSI), 11 (SCLK), 8 (CE0) + GPIO 25 (DC), 27 (RST) | |
| Buttons | GPIO 5 (play/pause), 6 (prev), 16 (next), 26 (spare/shutdown) | Internal pull-ups, active-low |
| Volume encoder | GPIO 17 (A), 22 (B), 23 (push = mute) | `gpiozero.RotaryEncoder`; internal pull-ups |

Since the Amp2 covers the GPIO header, use a **stacking header or GPIO splitter/breakout** to reach the free pins (the Amp2 passes all pins through).

## 4. Software Stack

| Layer | Choice | Why |
|-------|--------|-----|
| OS | Raspberry Pi OS Lite (64-bit) | Headless, minimal |
| Player core | [MPD](https://www.musicpd.org/) | Battle-tested, gapless playback, client library in every language, and NAS support later = just a mounted share |
| ALSA config | `dmix` + [peppyalsa](https://www.diyaudio.com/community/threads/peppyalsa-alsa-plugin-for-vu-meters-and-spectrum-analyzers.328688/) plugin | peppyalsa taps the output stream and writes live volume levels to a FIFO — this feeds the VU meters without any audio-path hacks |
| NFC service | **Custom Python** (`nfc-agent`) | ~150 lines: poll PN532 via I2C (Adafruit `adafruit-circuitpython-pn532`), read UID, look up UID→album in a small JSON/SQLite map, issue `clear/add/play` via `python-mpd2` |
| VU service | **Custom Python** (`vu-agent`) | Read peppyalsa FIFO → smooth with attack/decay ballistics (real VU = ~300ms integration) → set PWM duty on GPIO 12/13 via `rpi-hardware-pwm` |
| Display service | **Custom Python** (`display-agent`) | `python-mpd2` idle events → render artist/title/elapsed (+ album art if ST7789) with `luma.oled`/`luma.lcd`. ([mpd_oled](https://github.com/antiprism/mpd_oled) exists as a ready-made alternative) |
| Buttons | **Custom Python** (`buttons-agent`, or fold into nfc-agent) | `gpiozero` Button → MPD pause/previous/next |
| Card admin | Tiny Flask web page | "Tap a card, pick an album" pairing UI, reachable from your laptop |
| Service mgmt | systemd units for each agent | Auto-start, restart-on-failure, journald logs |

**Alternative considered:** [Phoniebox / RPi-Jukebox-RFID](https://github.com/MiczFlor/RPi-Jukebox-RFID) does NFC→playback out of the box and is worth reading for reference, but it brings its own web app and assumptions; your VU + display + button requirements mean custom glue code anyway, and MPD-centric custom services stay simpler.

### Software to be written (the actual dev work)

1. `nfc-agent.py` — card polling, UID→album map, MPD control, debounce. Re-tap semantics (decided): tapping the currently-playing card toggles pause — the card doubles as a play/pause control.
2. `vu-agent.py` — FIFO reader, VU ballistics (attack ~10ms, release ~300ms), log-scale mapping to PWM duty, calibration constants per meter.
3. `display-agent.py` — MPD idle loop, text layout + scrolling for long titles, idle-state screen (clock? logo?).
4. `buttons-agent.py` — debounced GPIO → MPD commands; long-press next/prev = seek (optional); rotary encoder → `setvol` ±3 per detent (against MPD's software mixer), push = mute toggle. Show a transient volume bar on the display via a small IPC hook to display-agent.
5. `card-admin` Flask app — list mappings, "pair mode" endpoint.
6. systemd unit files + install script.

### Future NAS enhancement (design hooks now, build later)

Mount the NAS via NFS or SMB in `/etc/fstab` (with `x-systemd.automount,noauto` so boot doesn't hang if the NAS is off), symlink or bind-mount it into `/var/lib/mpd/music/nas/`, then `mpc update`. Card mappings just point at `nas/...` paths. Optionally later: [Music Assistant](https://www.music-assistant.io/) or Mopidy if you ever want multi-room or streaming services — both can coexist with or replace MPD behind the same agents since the agents only speak the MPD protocol (Mopidy implements it too).

## 5. Project Plan

### Phase 0 — Order parts, prep (weekend 0)
Order everything in the BOM. While waiting: flash Pi OS Lite, enable SSH/I2C/SPI via `raspi-config`, get comfortable with `mpc` (MPD's CLI client) on the bare Pi.

### Phase 1 — Core audio (weekend 1)
1. Mount Amp2 on the Pi, connect 19V supply to the Amp2 (the Amp2 powers the Pi — do **not** also plug in USB-C).
2. Wire the two speakers to the Amp2 screw terminals.
3. In `config.txt`: comment out `dtparam=audio=on`, add `dtoverlay=hifiberry-dacplus-std` (current kernels; `hifiberry-dacplus` on older ones — see [HiFiBerry's config guide](https://www.hifiberry.com/docs/software/configuring-linux-3-18-x/)).
4. Install MPD, point it at a test music folder on the SD card, verify playback with `mpc`.
5. **Milestone: music plays through the speakers.**

### Phase 2 — NFC (weekend 2)
1. Wire PN532 (set its DIP switches to I2C mode) to GPIO 2/3 + 3.3V + GND. Verify with `i2cdetect -y 1` (expect 0x24).
2. Write and test `nfc-agent.py` interactively: tap card → print UID.
3. Add the UID→album JSON map and MPD control; write the pairing Flask app.
4. **Milestone: tap a card, an album starts.**

### Phase 3 — VU meters (weekend 3)
1. Bench-test each meter: find full-scale current with a bench supply/multimeter and pick series resistor + trim pot so 3.3V PWM @ 100% ≈ full deflection.
2. Wire GPIO 12/13 → RC filter (1kΩ + 10µF) → trim pot → meter.
3. Build the ALSA chain: `snd-aloop`-free approach — MPD → dmix → peppyalsa → hw. Confirm FIFO emits levels during playback.
4. Write `vu-agent.py`; tune ballistics until the needles *dance* rather than twitch.
5. **Milestone: needles move with the music.**

### Phase 4 — Display + buttons (weekend 4)
1. Wire display on SPI0, buttons on GPIO 5/6/16/26, rotary encoder on GPIO 17/22/23, with ground returns.
2. Write `display-agent.py` and `buttons-agent.py` (incl. encoder volume + mute).
3. Convert everything to systemd services; reboot test — the whole system must come up unattended.
4. **Milestone: fully functional on a breadboard/bench.**

### Phase 5 — Enclosure & final assembly (weekends 5–6)
1. Design front panel layout (meters top, display center, transport buttons in a row below with the volume knob to their right, NFC zone marked on top of the cabinet — the PN532 reads through ~2–3cm of wood/plastic, so it mounts *inside* under a printed marker).
2. Build the box; cut speaker openings, panel holes; line with acoustic damping if you're fussy.
3. Assembly order: speakers first → front panel components (meters, display, buttons) → PN532 taped under the top panel → Pi+Amp2 on standoffs at the rear → wiring loom → power inlet.
4. Smoke test at low volume, then calibrate VU trim pots against a reference track.
5. **Milestone: it looks like 1974 and works like 2026.**

### Phase 6 — Future: NAS streaming
fstab automount of the NAS share → bind into MPD library → rescan. No agent changes.

## 6. Risks & Open Questions

- **I2C sharing:** HiFiBerry cautions novices about adding I2C slaves alongside their boards; addresses don't collide here, but if flakiness appears, the PN532 can move to UART or SPI1 as a fallback.
- **PWM + audio:** hardware PWM channels on GPIO 12/13 are on a different PWM block usage than I2S audio here, but verify no audio-driver claims on PWM after enabling the HiFiBerry overlay (quick `dtoverlay` check in Phase 3).
- **Meter drive:** cheap VU modules vary wildly in coil resistance/sensitivity — hence the per-meter calibration step, don't skip it.
- **Encoder volume path:** MPD's `setvol` needs a working mixer. With the Amp2, either use its hardware volume control (`Digital` ALSA mixer on the TAS5756) or set `mixer_type "software"` in `mpd.conf`. Prefer the hardware mixer — it keeps full DAC resolution; verify the mixer name with `amixer` in Phase 1.

Decided: display = SSD1322 mono amber; re-tapping the current card = pause toggle; volume = rotary encoder knob (not buttons).

## Sources

- [HiFiBerry Amp2](https://www.hifiberry.com/shop/boards/amp2/) · [Amp2 datasheet](https://www.hifiberry.com/docs/data-sheets/datasheet-amp2/) · [GPIO usage of HiFiBerry boards](https://www.hifiberry.com/docs/hardware/gpio-usage-of-hifiberry-boards/) · [Pi 5 Amp2 bundle](https://www.hifiberry.com/shop/bundles/hifiberry-amp2-bundle-4/)
- [Phoniebox / RPi-Jukebox-RFID](https://github.com/MiczFlor/RPi-Jukebox-RFID) · [PN532 reader docs](https://github.com/MiczFlor/RPi-Jukebox-RFID/tree/develop/components/rfid-reader/PN532)
- [Audio VU Meters & Raspberry Pi (Hackaday.io)](https://hackaday.io/project/26951-audio-vu-meters-raspberry-pi) · [Driving a VU meter with a DAC](https://menno.io/posts/raspberry_pi_vu_meter/) · [rpi-vumonitor-python](https://github.com/mrwunderbar666/rpi-vumonitor-python)
- [peppyalsa ALSA plugin](https://www.diyaudio.com/community/threads/peppyalsa-alsa-plugin-for-vu-meters-and-spectrum-analyzers.328688/) · [PeppyMeter](https://forums.raspberrypi.com/viewtopic.php?t=147474)
- [mpd_oled](https://github.com/antiprism/mpd_oled) · [Music Assistant](https://www.music-assistant.io/faq/stream-to/)
- [Raspberry Pi model comparison 2026](https://raspberry.tips/en/raspberrypi-einsteiger/raspberry-pi-models-comparison-2026-pi-5-pi-4-zero) · [Best Pi music player guide (SunFounder)](https://www.sunfounder.com/blogs/news/best-raspberry-pi-music-player-guide-setup-software-and-streaming-options)