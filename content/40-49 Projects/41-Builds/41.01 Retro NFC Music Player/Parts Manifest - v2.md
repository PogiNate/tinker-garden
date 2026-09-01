---
title: Retro NFC Music Player - Parts Manifest - v2
type: note
permalink: personal/tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/parts-manifest-v2
tags:
  - parts-manifest
  - retro-nfc-music-player
  - bom
---

# Retro NFC Music Player — Parts Manifest v2

Supersedes [[Parts Manifest - v1|Retro NFC Music Player - Parts Manifest - v1]]. This reflects parts actually purchased, replacing the HiFiBerry Amp2 + VU meter concept with a WM8960 codec HAT + OLED now-playing display + MAX7219 LED matrix spectrum analyzer. See [[Design and Project Plan - v1|Design & Project Plan]] (architecture doc is due for its own v2 — see notes at bottom).

## What changed from v1

| Dropped | Kept | New |
|---|---|---|
| HiFiBerry Amp2, Dayton Audio drivers, Mean Well PSU, 2× TN-73 VU meters + trim pots, SSD1322 display, rotary encoder | PN532 (swapped Adafruit #364 → HiLetgo), NFC cards, Perma-Proto, panel buttons | Waveshare WM8960 Audio HAT, HiLetgo SSD1309 2.42" OLED, LonelyBinary MAX7219 32x8 matrix, Pi 5 (2GB) in place of Pi 4 |

The VU meters are gone in favor of an LED matrix spectrum analyzer — a different vibe (more "graphic equalizer" than "analog needle") but same peppyalsa-style tap into the audio stream will likely drive it. Volume, previously a rotary encoder, is now two of the panel momentary buttons (vol up/down).

## Already purchased

| Part | Qty | Link | Est. | Notes |
|---|---:|---|---:|---|
| Raspberry Pi 5, 2GB RAM | 1 | user-supplied | — | See cooling section below — needs the Active Cooler, which drives the mounting-hardware item below. |
| Waveshare WM8960 Audio HAT | 1 | [Amazon B098R7TTM4](https://www.amazon.com/dp/B098R7TTM4) | $25 | Stereo CODEC, **I2S** for audio (GPIO 18/19/20/21) + **I2C** for control (GPIO 2/3, shares the bus fine with the PN532). Has pass-through headers, so GPIO access for everything downstream isn't the issue — physical clearance above the Active Cooler is (see mounting hardware, below). Has an onboard configurable button on GPIO17. Drives speakers directly at only ~1W/channel (8Ω BTL) — check your listing's box contents; some ship with just one test speaker, and you'll want a matched stereo pair for L/R. |
| HiLetgo 2.42" SSD1309 128×64 OLED | 1 | [Amazon B0CFF19Z5G](https://www.amazon.com/dp/B0CFF19Z5G) | $14 | **Confirm you ordered the SPI (7-pin) variant, not I2C (4-pin)** — this listing sells both and the interface is picked at purchase, not board-configurable. SPI keeps the display off the shared I2C bus and lands on the now-free SPI0 (CE0). |
| HiLetgo PN532 NFC Module | 1 | [Amazon B01I1J17LC](https://www.amazon.com/dp/B01I1J17LC) | $13 | Same part as v1's role, different vendor (was Adafruit #364, now HiLetgo). Selectable I2C/SPI/HSU via onboard jumpers — set to **I2C** to match the plan, shares GPIO 2/3 with the WM8960 at a different address. |
| LonelyBinary MAX7219 32×8 LED Matrix (3-pack: red/green/blue) | 1 pack | [Amazon B0GTQFCF2V](https://www.amazon.com/dp/B0GTQFCF2V) | $20 | You're using the **green** module for the spectrum analyzer; the red and blue ones are spares/future project fodder. 5-pin SPI (VCC, GND, DIN, CS, CLK) — lands on SPI0 CE1, sharing MOSI/SCLK with the OLED via separate chip-selects. Note: MAX7219 is a 5V-logic part; Pi GPIO is 3.3V. Most people run it fine unbuffered at 3.3V, but if you get flaky columns, that's the first thing to suspect — a cheap 74HCT125 buffer or level shifter fixes it. |
| 12mm Momentary push button, LED power symbol | 1 (pack qty TBD) | [Amazon B0FTVC76PX](https://www.amazon.com/dp/B0FTVC76PX) | $8 | Power/standby button. The LED is a separate circuit from the switch — wire it through its own current-limiting resistor off 3.3V/5V, not through a GPIO, unless you want software control over it (doable, just needs a transistor since GPIO can't reliably sink an LED's current long-term). |
| Other momentary buttons (assorted pack) | 1 pack | [Amazon B07F24Y1TB](https://www.amazon.com/dp/B07F24Y1TB) | $8 | Play/pause, prev, next, **plus volume up/down** (decided — replaces v1's rotary encoder). That's 5 buttons off this pack for transport+volume, plus the dedicated power button above. |

## Adafruit

| Part | Qty | Link | Est. | Notes |
|---|---:|---|---:|---|
| Perma-Proto Quarter-size breadboard PCB | 1 | [Adafruit #589](https://www.adafruit.com/product/589) | $5 | For button wiring, MAX7219 breakout if needed, and the power-button LED resistor. |

## Not in this order (still needed)

| Part | Qty | Link | Est. | Notes |
|---|---:|---|---:|---|
| Tall GPIO stacking header + matching standoffs (≥16mm) | 1 kit | Search "Raspberry Pi 5 HAT stacking header active cooler" — 52Pi, Pimoroni, GeeekPi, and Uctronics all sell kits built for exactly this combo (it's the same kit Raspberry Pi bundles with the M.2 HAT+ for the same clearance problem) | $10 | **The fix for the WM8960/Active Cooler clearance conflict.** The Active Cooler's heatsink/blower sits ~16–19mm above the Pi 5 board (roughly level with the USB-A ports); a standard ~8.5mm HAT header doesn't clear that. Get a header + standoff set rated 16mm or taller (20mm gives more breathing room) so the WM8960 sits level and properly supported above the fan rather than floating on header-pin tension alone. Mount order: Pi 5 → Active Cooler → tall header + standoffs → WM8960. |
| Raspberry Pi 5 Active Cooler (official) | 1 | Raspberry Pi / resellers | $5 | See cooling section below. |
| Pi 5 power supply | 1 | Raspberry Pi / resellers | $12 | Official 27W USB-C PD supply — running the Active Cooler's blower alongside the HAT + peripherals makes this worth doing properly rather than reusing a phone charger. |

Also still needed: stereo speakers (if the WM8960 listing only includes one), speaker wire, microSD card, enclosure materials, solder, hookup wire.

**Estimated total for the purchased-parts order: ~$93.** Add roughly **$27** for the Active Cooler + mounting kit + power supply once sourced.

---

## Does the Pi 5 need active cooling in a cabinet?

Short answer: **yes, budget for it** — it's not strictly mandatory, but a sealed retro cabinet is exactly the case where skipping it will bite you.

- Raspberry Pi's own guidance shifted with the Pi 5: unlike earlier boards, they now recommend active or passive cooling in general, not just for sustained heavy loads.
- The Pi 5's BCM2712 will thermal-throttle within minutes under sustained load with no cooling at all, and a closed wooden/MDF cabinet with no airflow is worse than open-bench conditions — heat has nowhere to go.
- Your actual CPU load here is modest (MPD + a few lightweight Python agents polling I2C/SPI), so you likely won't hit sustained 100% CPU — but the enclosure, not the workload, is the reason to still cool it. Trapped heat plus low airflow is the scenario that turns "probably fine" into "throttling or shortened lifespan."
- **Recommendation:** the official Raspberry Pi 5 Active Cooler (heatsink + PWM blower fan, clips onto the mounting holes, plugs into the dedicated 4-pin fan header, firmware ramps it only when needed). It's inaudible at idle and only ramps under load — fine for a media player that mostly idles at "playing music," not compiling code.
- If total silence matters more to you than a few dollars, third-party quieter 30mm fans (e.g. Noctua NF-A4x10-based coolers) exist, but the official cooler is the well-supported default and the one this manifest's mounting hardware is sized around.
- Practical cabinet note: however you cool it, make sure the cabinet itself has *some* passive ventilation (vents on the back/bottom) — an active cooler moving air inside a fully sealed box just recirculates hot air. Also leave the fan's intake side unobstructed when you're stacking the WM8960 above it — don't box it in on all sides with standoffs and wiring.

## Next: wiring + software

Flagging for the next pass, since the BOM changes ripple into both:

- **GPIO/bus map v2** — I2C (2/3) carries both the WM8960 codec config and the PN532; SPI0 carries the OLED (CE0) and MAX7219 matrix (CE1); the 6 buttons (power, play/pause, prev, next, vol up, vol down) land on remaining free GPIO. Worth drawing this out properly like the v1 doc's table before wiring anything.
- **Software for the spectrum analyzer** — same peppyalsa-taps-the-ALSA-stream idea as v1's VU meters, just mapped to `luma.led_matrix` (which supports MAX7219 over spidev) instead of PWM duty cycle. FFT/band-splitting logic is new work v1 didn't need.
- **Volume control decided** — buttons instead of the rotary encoder; software needs discrete-step `setvol` calls instead of a continuous encoder read.
- MPD as the core player is still the right call — nothing about this hardware swap changes that.

Happy to draft a full v2 Design & Project Plan (GPIO map, software stack, phased build plan) now that the volume-control question is settled — let me know.
