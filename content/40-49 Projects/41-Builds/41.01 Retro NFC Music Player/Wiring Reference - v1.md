---
title: Retro NFC Music Player - Wiring Reference - v1
type: bernch-note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-wiring-reference
tags:
  - wiring
  - gpio-map
  - retro-nfc-music-player
  - reference
  - perma-proto
updated_by: Nathanial Dickson
status: Superseded
---

# Retro NFC Music Player — Wiring Reference

> **Superseded by [[Wiring Reference - v2|Retro NFC Music Player - Wiring Reference - v2]] (2026-10-04).** The PN532 moved off I2C1 to its own bus on GPIO 26/27. Kept for history.

Bench-side companion to [[Design and Project Plan - v3|Retro NFC Music Player - Design and Project Plan - v3]] §2. Three views: by signal, by physical pin, and the Perma-Proto board layout. All header connections are made to the WM8960 HAT's pass-through header at the top of the stack — pin numbering is identical to the Pi's own header. Pin 1 is the corner with the square pad, nearest the microSD end.

> **Revised for the matrix-free design.** The MAX7219 32×8 matrix and both Adafruit Pixel Shifters are **out**. This frees GPIO 7 (CE1), removes the 5V logic domain entirely, and leaves SPI0 with a single device (the OLED on CE0). Everything on the board is now 3.3V logic.

> **Reviewed 2026-09-13 before first faceplate hookup.** No GPIO assignments have changed since the matrix was dropped. Added: a faceplate-position → pin map (View 0), the OLED "SDA" gotcha, PN532 DIP positions, J2 wiring clarification, the backlight LEDs (simple 3V3 LEDs on the proto board — no GPIO), and a first-power-up checklist.

## View 0 — Buttons by faceplate position (use this at the bench)

Facing the plate ([[Retro NFC Music Player - Frontplate Design - v9]]):

| Position | Button | GPIO | Phys pin |
|---|---|---|---|
| Far left | Power — switch | — | **across the two J2 pads** (Pi underside) |
| Far left | Power — LED | — | 3V3 → 330Ω → anode; cathode → GND rail |
| Center trio, left | Prev | 6 | 31 |
| Center trio, middle | Play/pause | 5 | 29 |
| Center trio, right | Next | 16 | 36 |
| Right column, top | Vol up | 22 | 15 |
| Right column, bottom | Vol down | 23 | 16 |

Every GPIO button's second leg → proto board GND rail. No external resistors; internal pull-ups via gpiozero.

**J2:** connect the switch across both J2 pads rather than J2-to-a-header-GND. One of the pads is ground anyway, but using both removes any chance of choosing the wrong one. The button's four pins per [[Component Specifications]]: Ground · LED 3V3 (via 330Ω) · Ground · J2 pad.

## Board-label → Pi pin (the "SDA" gotcha)

Two boards both have a pin labelled **SDA**, and they mean different things:

| Board | Label | What it actually is | Pi pin |
|---|---|---|---|
| **SSD1309 OLED** (SPI) | GND | ground | GND rail |
| | VCC | 3.3V | 3V3 rail |
| | **SCK** | SPI clock | GPIO 11 (pin 23) |
| | **SDA** | **SPI data (MOSI) — NOT I2C** | GPIO 10 (pin 19) |
| | RES | reset | GPIO 24 (pin 18) |
| | DC | data/command | GPIO 25 (pin 22) |
| | CS | chip select | GPIO 8 (pin 24) |
| **PN532** (I2C, 4-pin group) | GND | ground | GND rail |
| | VCC | 3.3V | 3V3 rail |
| | **SDA** — printed **TXD** on the back face | **real I2C data** | GPIO 2 (pin 3) |
| | **SCL** — printed **RXD** on the back face | I2C clock | GPIO 3 (pin 5) |

**PN532 DIP switches for I2C: switch 1 = ON, switch 2 = OFF** (silkscreen table: HSU 0 0 / I2C 1 0 / SPI 0 1). Set before powering. Use the 4-pin group, not the 8-pin SPI group.

**The 4-pin header is shared between HSU (serial) and I2C.** The I2C labels (SDA/SCL) are silkscreened on one face of the board and the HSU labels (TXD/RXD) on the other — same physical pins, same order. TXD = SDA, RXD = SCL. Factory default is HSU (both switches OFF), so the switch genuinely has to be moved; nothing appears on the I2C bus until it is. If `i2cdetect` still shows no `24` after wiring, swap the two data wires — reversed SDA/SCL is harmless and is the most common cause.

## View 1 — By signal
| Peripheral | Connection | Pins (BCM) |
|---|---|---|
| WM8960 Audio HAT | Stacked on tall header (no hand wiring) | I2S: 18, 19, 20, 21 · I2C: 2, 3 (addr 0x1a) · onboard button: GPIO 17 (reserved) |
| PN532 NFC | I2C1, addr 0x24 | SDA → GPIO 2, SCL → GPIO 3, 3V3, GND |
| SSD1309 OLED | SPI0, CE0 — **only SPI device** | MOSI → 10, SCLK → 11, CS → 8, DC → 25, RST → 24, 3V3, GND |
| Play/pause · prev · next | Discrete GPIO, active-low | 5 · 6 · 16 — each to GND, internal pull-ups via gpiozero |
| Vol up / vol down | Discrete GPIO, active-low | 22 · 23 — vertical pair on the faceplate's right side |
| Power button switch | **J2 pads** (board underside) | Clean shutdown + wake — mirrors the onboard PWR button |
| Power button LED | 3V3 | Via ~330Ω series resistor (always-on while powered) |
| **Backlight LEDs** (behind the transport + volume label cutouts) | **3V3 rail + GND rail on the Perma-Proto** | Plain LEDs, one series resistor each — no GPIO. On whenever the Pi is on |

Free for future: GPIO 4, **7**, 9 (MISO), 12, 13, 26, 27.


## View 2 — By physical pin (40-pin header, top view)

| Phys | Function | · | Phys | Function |
|---|---|---|---|---|
| 1 | **3V3 rail** → proto rail (OLED, PN532, power LED) | | 2 | 5V — *unused now; matrix is gone* |
| 3 | GPIO 2 · I2C SDA | | 4 | 5V |
| 5 | GPIO 3 · I2C SCL | | 6 | GND |
| 7 | GPIO 4 · free | | 8 | GPIO 14 · free |
| 9 | GND → proto rail | | 10 | GPIO 15 · free |
| 11 | GPIO 17 · WM8960 button (reserved) | | 12 | GPIO 18 · I2S clock (HAT) |
| 13 | GPIO 27 · free | | 14 | GND |
| 15 | GPIO 22 · **vol up** | | 16 | GPIO 23 · **vol down** |
| 17 | 3V3 | | 18 | GPIO 24 · **OLED RST** |
| 19 | GPIO 10 · **SPI MOSI → OLED** | | 20 | GND |
| 21 | GPIO 9 · MISO, free | | 22 | GPIO 25 · **OLED DC** |
| 23 | GPIO 11 · **SPI SCLK → OLED** | | 24 | GPIO 8 · **CE0, OLED CS** |
| 25 | GND | | 26 | GPIO 7 · CE1 — **now free** |
| 27 | ID_SD · reserved | | 28 | ID_SC · reserved |
| 29 | GPIO 5 · **play/pause** | | 30 | GND |
| 31 | GPIO 6 · **prev** | | 32 | GPIO 12 · free |
| 33 | GPIO 13 · free | | 34 | GND |
| 35 | GPIO 19 · I2S LRCLK (HAT) | | 36 | GPIO 16 · **next** |
| 37 | GPIO 26 · free | | 38 | GPIO 20 · I2S ADC (HAT) |
| 39 | GND | | 40 | GPIO 21 · I2S DAC (HAT) |

## View 3 — Perma-Proto quarter-size board layout
The proto board is the **power hub only** — all signal wires (SPI, I2C, DC/RST, button GPIO legs) run point-to-point from the Pi header to their device. Fewer joints, shorter stubs, easier debugging.

With the matrix gone there is **no 5V rail and no split voltage domain** — the whole board is 3.3V. The backlight LEDs live here too and draw from the same 3V3 rail. The second rail pair stays spare; leave it unpopulated, or bridge both + rails together for a lower-impedance 3V3 distribution.

| Rail | Fed from | Serves |
|---|---|---|
| **+** = 3V3 | Pi pin 1 | OLED VCC, PN532 VCC, power-LED resistor, **backlight LED resistors** |
| **−** = GND | Pi pin 9 | OLED/PN532 GND, all six button ground legs, all LED cathodes (power + backlight) |
| Second rail pair | — | Unused; spare, or jumper to the primary rails |

**Row regions (top → bottom, approximate — adapt to your board):**

| Rows | Region | Detail |
|---|---|---|
| 1–2 | OLED power tap | 3V3 + GND out to SSD1309 |
| 3–4 | PN532 power tap | 3V3 + GND out to NFC module |
| 5–9 | Buttons, one row each | GPIO wire from Pi lands in the row; button wire leaves the same row; second leg → GND rail. Order: play/pause (5), prev (6), next (16), vol+ (22), vol− (23) |
| 10 | Power-button LED | 330Ω from 3V3 rail into the row → LED anode; cathode → GND rail |
| 11+ | Backlight LEDs | One row per LED (or per group): series resistor from 3V3 rail → anode; cathode → GND rail. No GPIO — lit whenever the Pi is up |

**3V3 rail budget:** OLED ~30mA + PN532 up to ~150mA with the RF field on + power LED ~5mA + backlight LEDs at ~10–15mA each. For a handful of LEDs that's comfortably a few hundred mA, well within what the Pi 5 header's 3V3 pin supplies.

**Warm-white LEDs on 3.3V — expect small resistors and some brightness variation.** Warm-white forward voltage is typically 2.8–3.2V, so at 3.3V there's only 0.1–0.5V left across the series resistor. That means small values (roughly 22–68Ω for ~10mA) and noticeable LED-to-LED brightness differences from normal Vf variation. If one glyph is dimmer than its neighbours, that's why, not a wiring fault. Each LED has **its own series resistor** — the right topology, since a shared resistor would let the lowest-Vf LED hog current and dim the rest. So any brightness variation is just Vf spread, and evening it out is a per-LED job: measure the dim one's Vf and drop its resistor a step.


## First power-up checklist

1. **Backlight + power LED** should light the moment 3V3 comes up — no software involved. If the backlight is dark, the proto board's 3V3 or GND rail isn't reaching the Pi (pins 1 and 9).
2. `i2cdetect -y 1` → expect **1a** (WM8960) and **24** (PN532). If 24 is missing: DIP switches (1 ON, 2 OFF), or SDA/SCL swapped.
3. Button test — prints which button was pressed; confirms pull-ups and the View 0 position map:
   ```python
   from gpiozero import Button
   from signal import pause
   pins = {"prev": 6, "play": 5, "next": 16, "vol_up": 22, "vol_dn": 23}
   for name, pin in pins.items():
       b = Button(pin, pull_up=True, bounce_time=0.05)
       b.when_pressed = lambda n=name: print(f"{n} pressed")
   pause()
   ```
4. OLED: luma.oled `ssd1309` demo on SPI0/CE0. Blank screen → check the "SDA" gotcha above first.
5. Power button: short press from a running system should trigger a clean shutdown; press again from halt should wake.

Reminder: Pi 5 needs **gpiozero on lgpio** — legacy `RPi.GPIO` does not work.

## Bench notes

- **Wiring clusters by board region:** I2C pair + PN532 power in the top six pins (3, 5, plus 1/6); OLED's SPI group in the 18–24 run; buttons at 15/16 (volume) and 29/31/36 (transport).
- **Ground strategy:** button grounds terminate on the proto board's GND rail, which returns to Pi pin 9. Remaining header GND pins (6, 14, 20, 25, 30, 34, 39) stay free for debug clips.
- **Buttons need no external resistors:** button → GPIO on one leg, GND rail on the other; `gpiozero.Button(pin, pull_up=True, bounce_time=0.05)`.
- **Faceplate button geography** ([[Retro NFC Music Player - Frontplate Design - v9]]): power far left, transport trio center, volume pair stacked vertically at right. Cut the volume harness as a pair — those two runs are the longest reach from the header.
- **I2S pins (HAT group) never get touched** — the stacking header carries them. Listed for debug probing only.
- Power button: **J2 pads only** (GPIO-fallback dropped).
