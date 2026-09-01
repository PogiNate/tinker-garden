---
title: Retro NFC Music Player - Parts Manifest - v1
type: note
permalink: tinkering/retro-nfc-music-player-parts-manifest-v1
---

# Retro NFC Music Player — Parts Manifest

Order links grouped by vendor. Prices are approximate as of July 2026 — confirm at checkout. See [[Design and Project Plan - v1|Design & Project Plan]] for what each part does.

## Amazon

| Part | Qty | Link | Est. |
|------|----:|------|-----:|
| HiFiBerry Amp2 (DAC + 60W amp HAT) | 1 | [Amazon B076DLCRHF](https://www.amazon.com/HiFiBerry-Hifiberry-AMP2-Amp2/dp/B076DLCRHF) — also direct from [hifiberry.com](https://www.hifiberry.com/shop/boards/amp2/) | $55 |
| Raspberry Pi 4 Model B, 2GB | 1 | [Amazon search — Pi 4 2GB](https://www.amazon.com/raspberry-pi-4-2gb/s?k=raspberry+pi+4+2gb) (Adafruit [#4292](https://www.adafruit.com/product/4292) was out of stock; buy wherever it's ~$45 — beware scalper pricing) | $45 |
| Mean Well GST60A18-P1J 18V/60W supply | 1 | [Amazon B071RTLS54](https://www.amazon.com/GST60A18-P1J-3-33A-Reliability-Industrial-Adaptor/dp/B071RTLS54) — HiFiBerry's own recommendation; 18V is the sweet spot for 4–8Ω speakers | $30 |
| Dayton Audio PC83-8 3" full-range driver, 8Ω/30W | 2 | [Amazon B0751B1Z61](https://www.amazon.com/Dayton-Audio-PC83-8-Full-Range-Driver/dp/B0751B1Z61) | $13 ea |
| TN-73 analog VU meters w/ backlight (set of 2) | 1 set | [Amazon B08P63PNMH](https://www.amazon.com/TN-73-Amplifier-Chassis-Internal-Resistance/dp/B08P63PNMH) | $20 |
| SSD1322 3.12" 256×64 OLED, amber/yellow, SPI | 1 | [Amazon B0CKY1Q7Q1](https://www.amazon.com/Display-Module-Graphic-SSD1322-Monochrome/dp/B0CKY1Q7Q1) — confirm "4-wire SPI" jumper setting on arrival | $28 |
| NTAG215 NFC cards, 50-pack, printable CR80 | 1 pack | [Amazon B07PQJNNWS](https://www.amazon.com/Ntag215-Memory-Compatible-Enabled-Phone-50/dp/B07PQJNNWS) | $16 |
| Vintage-style aluminum knob, 6mm D-shaft | 1 | [Amazon search — "aluminum knob 6mm potentiometer vintage"](https://www.amazon.com/s?k=aluminum+knob+6mm+potentiometer+vintage) — pick one that matches your panel | $8 |
| 1/4W metal-film resistor kit + 10µF electrolytic caps | 1 kit | [Amazon search — "metal film resistor kit"](https://www.amazon.com/s?k=metal+film+resistor+kit+1%2F4w) — cheaper than picking single values before VU calibration | $12 |
| Misc: panel-mount DC jack (5.5×2.1mm), speaker grille cloth, 22AWG hookup wire, M2.5 standoffs | — | Amazon, as needed | $25 |

## Adafruit

| Part | Qty | Link | Est. |
|------|----:|------|-----:|
| PN532 NFC/RFID controller breakout v1.6 | 1 | [Adafruit #364](https://www.adafruit.com/product/364) — includes antenna, headers, one test card | $40 |
| Rotary encoder + extras (Bourns PEC11, 24 detents, push switch) | 1 | [Adafruit #377](https://www.adafruit.com/product/377) | $5 |
| 16mm panel-mount momentary pushbutton — black (play/pause) | 1 | [Adafruit #1505](https://www.adafruit.com/product/1505) | $2 |
| 16mm panel-mount momentary pushbutton — red (prev) | 1 | [Adafruit #1445](https://www.adafruit.com/product/1445) | $2 |
| 16mm panel-mount momentary pushbutton — yellow (next) | 1 | [Adafruit #1502](https://www.adafruit.com/product/1502) | $2 |
| GPIO stacking header, extra-long 2×20 | 1 | [Adafruit #2223](https://www.adafruit.com/product/2223) — lifts the Amp2 so free GPIO pins stay reachable | $3 |
| Perma-Proto half-size breadboard PCB | 1–2 | [Adafruit — search "Perma-Proto half"](https://www.adafruit.com/search?q=perma-proto+half) — for the VU driver RC filters and button wiring | $5 ea |

## Digi-Key

| Part | Qty | Link | Est. |
|------|----:|------|-----:|
| Bourns 3386P-1-102LF 1kΩ trimmer (VU calibration) | 2 | [Digi-Key 993042](https://www.digikey.com/en/products/detail/bourns-inc/3386P-1-102LF/993042) | $1.37 ea |
| (Optional) Mean Well GST60A18-P1J if Amazon listing dries up | 1 | [Digi-Key GST60A18-P1J](https://www.digikey.com/en/products/detail/mean-well-usa-inc/GST60A18-P1J/7703714) | $30 |

Digi-Key's $7 ground shipping makes a two-trimmer order silly on its own — either add the power supply here, or grab a trimmer assortment on Amazon instead.

## Not in this order

microSD card (32GB+, you likely have one), enclosure materials (plywood/MDF or 3D-print filament — Phase 5 decision), speaker wire, solder.

**Estimated total: ~$330 shipped.**

## Ordering notes

- **Amp2 availability:** The Pi Hut lists the Amp2 as discontinued; HiFiBerry's own shop and Amazon still carry it. If it disappears, the [Amp4](https://www.hifiberry.com/amps/) is the successor (also 60W, same overlay family) — the design doesn't change.
- **Pi 4 vs Pi 5:** if Pi 4 2GB pricing looks inflated (2026 memory-price situation), a Pi 5 works identically here — HiFiBerry sells a [Pi 5 + Amp2 bundle](https://www.hifiberry.com/shop/bundles/hifiberry-amp2-bundle-4/) that could replace lines 1–2.
- **VU meters:** the TN-73 spec (50Ω internal, ~1mA-class movement) is a nominal figure on these imports — the Phase 3 calibration step with the trimmers exists precisely because units vary.
- **SSD1322 modules** often ship configured for 80xx parallel; there's a resistor jumper on the back to set 4-wire SPI. Check before wiring.
- **NTAG215 note:** the PN532 reads NTAG215 fine (ISO 14443A). If you want inkjet-printable album-art cards, verify the listing says "inkjet printable" — some PVC packs are thermal-printer only.