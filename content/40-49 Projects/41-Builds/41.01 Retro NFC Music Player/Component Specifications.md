---
title: Component Specifications
type: note
permalink: tinkering/40-49-projects/41-builds/41.01-retro-nfc-music-player/component-specifications
---

Actual dimensions of each component, measured with digital calipers.

## HiLetgo PN532 NFC Module
![[PN532 Photo.jpeg]]
### Physical Dimensions
- Width: 42.6mm
- Height: 40.6mm
- Depth: 3.9mm
    - Deepest point, which is the DIP switches on the front surface. 
- Screw holes are 6.0mm from the edge, measured with calipers resting on the inner edge of the screw hole and outer edge of the board.
### Interface
The module has two sets of pins: 
1. 8-pin grouping:
    1. SCK
    2. MISO
    3. MOSI
    4. SS
    5. VCC
    6. GND
    7. IRQ
    8. RSTO
2. 4-pin grouping:
    1. GND
    2. VCC
    3. SDA
    4. SCL

The DIP switches determine mode. The silk screen reads:

| MODE | Switch 1 | Switch 2 |
| :--: | :------: | :------: |
| HSU  |    0     |    0     |
| I2C  |    1     |    0     |
| SPI  |    0     |    1     |
## HiLetgo 2.42" OLED
![[OLED Back.jpeg]]
![[OLED Front.jpeg]]
### Physical Dimensions
- Overall
    - Width: 71mm
    - Height: 43.3mm
    - Depth: 6mm (exclusive of pins)
- Display
    - Width: 61.8mm
    - Height: 39.7mm

### Interface
SPI Interface. Pins in order:
- GND
- VCC
- SCK
- SDA
- RES
- DC
- CS

## Waveshare Speakers
![[Speaker.jpeg]]The actual Audio HAT is mounted on the Raspberry Pi, and doesn't need measurements for the faceplate. The speakers do, however.

### Physical Dimensions
- Overall:
    - Width: 44.5mm
    - Height: 99.5mm
    - Depth: 20.7mm
- Screw holes:
    - Vertical depth: 9.8mm
    - center-to-center width: 35.4mm
    - center-to-center height: 91.1mm
- Wires: 4.8mm from front edge

## Power Button
![[Power Button.jpeg]]
### Physical Dimensions
- Diameter: 11.5mm
- Threading depth: 10.4mm

### Interface
Four pins: 
 - Ground
 - LED 3v3, with a 330Ω resistor.
 - Ground
 - J2 Pad
This is a Normally Open (NO) button.

## Other Momentary Buttons
![[Other Buttons.jpeg]]
### Physical Dimensions:
- Diameter: 11.5mm
- threading depth: 5mm
    - NOTE: need to allow for a 1mm washer as well.

### Interface

These are Normally Open (NO) Buttons. one pin to ground, the other to the appropriate pin on the Raspberry Pi.