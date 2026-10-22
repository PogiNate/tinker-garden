---
title: Retro NFC Music Player - Wiring Reference - v2
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/retro-nfc-music-player-wiring-reference-v2
tags:
  - wiring
  - gpio-map
  - retro-nfc-music-player
  - reference
  - perma-proto
created_by: Nathanial Dickson
updated_by: Nathanial Dickson
status: Implemented
publish: false
---

# Retro NFC Music Player — Wiring Reference — v2

Supersedes [[Retro NFC Music Player - Wiring Reference]]. Bench-side companion to [[Design and Project Plan - v3|Retro NFC Music Player - Design and Project Plan - v3]] §2. It has three views: by signal, by physical pin, and the Perma-Proto board layout. All header connections go to the WM8960 HAT's pass-through header at the top of the stack. Its pin numbering is identical to the Pi's own header. Pin 1 is the corner with the square pad, nearest the microSD end.

> **What changed in v2 (2026-10-04): the PN532 moved off I2C1 onto its own bus.** SDA → **GPIO 26 (pin 37)**, SCL → **GPIO 27 (pin 13)**, using a bit-banged `i2c-gpio` bus (`/dev/i2c-3`). Its power is unchanged. **Why:** the WM8960 is write-only over I2C. Sharing I2C1 with the PN532 made the codec's register writes fail (`SDA stuck at low` / `lost arbitration` / `-121`) whenever the class-D amp switched on or off. The driver then believed the codec was playing while it was actually muted or powered down, giving silent or buzzing playback until a reboot. After the move there were 0 I2C errors across 8 `speaker-test` runs and MPD playback, including while the PN532 was being polled. Full write-up in the v3 build log. **Never put the PN532 back on I2C1.** Everything else is unchanged from v1.

Required in `/boot/firmware/config.txt` under `[all]`:

```
dtparam=i2s=on
dtoverlay=wm8960-soundcard
dtoverlay=i2c-gpio,bus=3,i2c_gpio_sda=26,i2c_gpio_scl=27
```

The PN532 needs no extra pull-ups: its board has its own, which go to 3V3 because it is powered from the 3V3 rail. In code, open the bus with `adafruit_extended_bus.ExtendedI2C(3)` rather than `busio.I2C(board.SCL, board.SDA)`.

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

Every GPIO button's second leg goes to the proto board GND rail. No external resistors are needed; the internal pull-ups are enabled via gpiozero.

**J2:** connect the switch across both J2 pads rather than from J2 to a header GND. One of the pads is ground anyway, but using both removes any chance of choosing the wrong one. The button's four pins, per [[Component Specifications]]: Ground · LED 3V3 (via 330Ω) · Ground · J2 pad.

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
| | **SDA**, printed **TXD** on the back face | **real I2C data** | **GPIO 26 (pin 37)** — own bus, I2C3 |
| | **SCL**, printed **RXD** on the back face | I2C clock | **GPIO 27 (pin 13)** — own bus, I2C3 |

**PN532 DIP switches for I2C: switch 1 = ON, switch 2 = OFF** (silkscreen table: HSU 0 0 / I2C 1 0 / SPI 0 1). Set them before powering on. Use the 4-pin group, not the 8-pin SPI group.

**The 4-pin header is shared between HSU (serial) and I2C.** The I2C labels (SDA/SCL) are printed on one face of the board and the HSU labels (TXD/RXD) on the other. They are the same physical pins, in the same order: TXD = SDA, RXD = SCL. The factory default is HSU (both switches OFF), so the switch really does have to be moved; nothing appears on the bus until it is. If `i2cdetect -y 3` still shows no `24` after wiring, swap the two data wires. Reversed SDA/SCL is harmless and is the most common cause.

## View 1 — By signal

| Peripheral | Connection | Pins (BCM) |
|---|---|---|
| WM8960 Audio HAT | Stacked on tall header (no hand wiring) | I2S: 18, 19, 20, 21 · I2C1: 2, 3 (addr 0x1a, **codec only**) · onboard button: GPIO 17 (reserved) |
| PN532 NFC | **I2C3 (`i2c-gpio`), addr 0x24** | SDA → **GPIO 26**, SCL → **GPIO 27**, 3V3, GND |
| SSD1309 OLED | SPI0, CE0 — **only SPI device** | MOSI → 10, SCLK → 11, CS → 8, DC → 25, RST → 24, 3V3, GND |
| Play/pause · prev · next | Discrete GPIO, active-low | 5 · 6 · 16 — each to GND, internal pull-ups via gpiozero |
| Vol up / vol down | Discrete GPIO, active-low | 22 · 23 — vertical pair on the faceplate's right side |
| Power button switch | **J2 pads** (board underside) | Clean shutdown + wake, same as the onboard PWR button |
| Power button LED | 3V3 | Via ~330Ω series resistor (always on while powered) |
| **Backlight LEDs** (behind the transport + volume label cutouts) | **3V3 rail + GND rail on the Perma-Proto** | Plain LEDs, one series resistor each, no GPIO. On whenever the Pi is on |

Free for future use: GPIO 4, 7, 9 (MISO), 12, 13. (26 and 27 are now the PN532's bus.)

## View 2 — By physical pin (40-pin header, top view)

| Phys | Function | · | Phys | Function |
|---|---|---|---|---|
| 1 | **3V3 rail** → proto rail (OLED, PN532, power LED) | | 2 | 5V — unused |
| 3 | GPIO 2 · I2C1 SDA (**WM8960 only**) | | 4 | 5V |
| 5 | GPIO 3 · I2C1 SCL (**WM8960 only**) | | 6 | GND |
| 7 | GPIO 4 · free | | 8 | GPIO 14 · free |
| 9 | GND → proto rail | | 10 | GPIO 15 · free |
| 11 | GPIO 17 · WM8960 button (reserved) | | 12 | GPIO 18 · I2S clock (HAT) |
| 13 | GPIO 27 · **PN532 SCL (I2C3)** | | 14 | GND |
| 15 | GPIO 22 · **vol up** | | 16 | GPIO 23 · **vol down** |
| 17 | 3V3 | | 18 | GPIO 24 · **OLED RST** |
| 19 | GPIO 10 · **SPI MOSI → OLED** | | 20 | GND |
| 21 | GPIO 9 · MISO, free | | 22 | GPIO 25 · **OLED DC** |
| 23 | GPIO 11 · **SPI SCLK → OLED** | | 24 | GPIO 8 · **CE0, OLED CS** |
| 25 | GND | | 26 | GPIO 7 · CE1, free |
| 27 | ID_SD · reserved | | 28 | ID_SC · reserved |
| 29 | GPIO 5 · **play/pause** | | 30 | GND |
| 31 | GPIO 6 · **prev** | | 32 | GPIO 12 · free |
| 33 | GPIO 13 · free | | 34 | GND |
| 35 | GPIO 19 · I2S LRCLK (HAT) | | 36 | GPIO 16 · **next** |
| 37 | GPIO 26 · **PN532 SDA (I2C3)** | | 38 | GPIO 20 · I2S ADC (HAT) |
| 39 | GND | | 40 | GPIO 21 · I2S DAC (HAT) |

## View 3 — Perma-Proto quarter-size board layout

Unchanged from v1. The proto board is the **power hub only**: all signal wires (SPI, I2C, DC/RST, button GPIO legs) run point-to-point from the Pi header to their device.

| Rail | Fed from | Serves |
|---|---|---|
| **+** = 3V3 | Pi pin 1 | OLED VCC, PN532 VCC, power-LED resistor, backlight LED resistors |
| **−** = GND | Pi pin 9 | OLED/PN532 GND, all six button ground legs, all LED cathodes (power + backlight) |
| Second rail pair | — | Unused; spare, or jumper to the primary rails |

| Rows | Region | Detail |
|---|---|---|
| 1–2 | OLED power tap | 3V3 + GND out to SSD1309 |
| 3–4 | PN532 power tap | 3V3 + GND out to NFC module |
| 5–9 | Buttons, one row each | The GPIO wire from the Pi lands in the row and the button wire leaves the same row; the button's second leg goes to the GND rail. Order: play/pause (5), prev (6), next (16), vol+ (22), vol− (23) |
| 10 | Power-button LED | 330Ω from 3V3 rail into the row → LED anode; cathode → GND rail |
| 11+ | Backlight LEDs | One row per LED (or per group): series resistor from the 3V3 rail → anode; cathode → GND rail |

The 3V3 rail budget and the warm-white LED resistor notes are unchanged; see v1.

## First power-up checklist

1. **Backlight + power LED** should light as soon as 3V3 comes up, with no software involved. If the backlight is dark, the proto board's 3V3 or GND rail isn't reaching the Pi (pins 1 and 9).
2. `i2cdetect -y 1` → expect **UU** at `1a` (the WM8960, claimed by its driver) and **nothing else**. `i2cdetect -y 3` → expect **24** (PN532). If 24 is missing, check the DIP switches (1 ON, 2 OFF), check for swapped SDA/SCL, or check that the `i2c-gpio` overlay line is in `config.txt`.
3. Button test (same as v1):
   ```python
   from gpiozero import Button
   from signal import pause
   pins = {"prev": 6, "play": 5, "next": 16, "vol_up": 22, "vol_dn": 23}
   for name, pin in pins.items():
       b = Button(pin, pull_up=True, bounce_time=0.05)
       b.when_pressed = lambda n=name: print(f"{n} pressed")
   pause()
   ```
4. OLED: luma.oled `ssd1309` demo on SPI0/CE0. If the screen is blank, check the "SDA" gotcha above first.
5. PN532: `uv run python src/retro_player/rfid_test.py` in the repo prints UIDs as cards are tapped.
6. Power button: a short press on a running system should trigger a clean shutdown; another press from halt should wake it.

Reminder: the Pi 5 needs **gpiozero on lgpio**. Legacy `RPi.GPIO` does not work.

## Bench notes

- **Wiring clusters by board region:** the PN532 data pair is now at pins 13 and 37, on opposite ends of the header, so cut those two runs to their own lengths. The PN532 power stays on the proto rails. The OLED's SPI group is in the 18–24 run. Buttons are at 15/16 (volume) and 29/31/36 (transport).
- **Keep the PN532 data wires away from the speaker leads.** Class-D amp switching noise was the likely trigger for the original I2C failures.
- **Ground strategy:** button grounds terminate on the proto board's GND rail, which returns to Pi pin 9. The remaining header GND pins (6, 14, 20, 25, 30, 34, 39) stay free for debug clips.
- **I2S pins (HAT group) are never touched.** The stacking header carries them; they're listed for debug probing only.
- Power button: **J2 pads only**.

Part of [[41.01 Retro NFC Music Player]].
