---
title: Retro NFC Music Player - Wiring Reference
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-wiring-reference
tags:
- wiring
- gpio-map
- retro-nfc-music-player
- reference
- perma-proto
---

# Retro NFC Music Player — Wiring Reference

Bench-side companion to [[Design and Project Plan - v3|Retro NFC Music Player - Design and Project Plan - v3]] §2. Three views: by signal, by physical pin, and the Perma-Proto board layout. All header connections are made to the WM8960 HAT's pass-through header at the top of the stack — pin numbering is identical to the Pi's own header. Pin 1 is the corner with the square pad, nearest the microSD end.

> **Revised for the matrix-free design.** The MAX7219 32×8 matrix and both Adafruit Pixel Shifters are **out**. This frees GPIO 7 (CE1), removes the 5V logic domain entirely, and leaves SPI0 with a single device (the OLED on CE0). Everything on the board is now 3.3V logic.

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

With the matrix gone there is **no 5V rail and no split voltage domain** — the whole board is 3.3V. That frees the second rail pair; leave it unpopulated as expansion space, or bridge both + rails together for a lower-impedance 3V3 distribution.

| Rail | Fed from | Serves |
|---|---|---|
| **+** = 3V3 | Pi pin 1 | OLED VCC, PN532 VCC, power-LED resistor |
| **−** = GND | Pi pin 9 | OLED/PN532 GND, all six button ground legs, LED cathode |
| Second rail pair | — | Unused; spare, or jumper to the primary rails |

**Row regions (top → bottom, approximate — adapt to your board):**

| Rows | Region | Detail |
|---|---|---|
| 1–2 | OLED power tap | 3V3 + GND out to SSD1309 |
| 3–4 | PN532 power tap | 3V3 + GND out to NFC module |
| 5–9 | Buttons, one row each | GPIO wire from Pi lands in the row; button wire leaves the same row; second leg → GND rail. Order: play/pause (5), prev (6), next (16), vol+ (22), vol− (23) |
| 10 | Power-button LED | 330Ω from 3V3 rail into the row → LED anode; cathode → GND rail |
| 11+ | Spare | Expansion |

## Bench notes

- **Wiring clusters by board region:** I2C pair + PN532 power in the top six pins (3, 5, plus 1/6); OLED's SPI group in the 18–24 run; buttons at 15/16 (volume) and 29/31/36 (transport).
- **Ground strategy:** button grounds terminate on the proto board's GND rail, which returns to Pi pin 9. Remaining header GND pins (6, 14, 20, 25, 30, 34, 39) stay free for debug clips.
- **Buttons need no external resistors:** button → GPIO on one leg, GND rail on the other; `gpiozero.Button(pin, pull_up=True, bounce_time=0.05)`.
- **Faceplate button geography** ([[Frontplate Design - v3|Retro NFC Music Player - Frontplate Design - v3]]): power far left, transport trio center, volume pair stacked vertically at right. Cut the volume harness as a pair — those two runs are the longest reach from the header.
- **I2S pins (HAT group) never get touched** — the stacking header carries them. Listed for debug probing only.
- Power button: **J2 pads only** (GPIO-fallback dropped).
