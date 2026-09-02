---
title: Ultimate Player Kit Mini
type: note
permalink: tinkering/40-49-projects/41-builds/41.02-player-aid-boxes/player-aid
tags:
  - 3d-printing
  - dungeons-and-dragons
---


![[Ultimate Player Kit Mini Open.jpeg]]![[Ultimate Player Kit Mini Closed.jpeg]]
A fun box for dice, stats, a character mini, spell cards, etc. 

The original design was created by "About30Cows". The files are downloadable on [Makerworld](https://makerworld.com/en/models/3038763-ultimate-player-kit-mini-version#profileId-3416856). So far I've made four of these for people and they are well received. 

## Filament Notes


![[Pasted image 20260821150708.png|400]]

This project uses three filaments: 
1. a **base** color 
2. an **accent** color
3. **(optionally)** a hidden filament that will not be seen when it's put together. The only reason to use a different filament for this is because it's purely utilitarian, so a cheaper boring filament is perfectly acceptable.
To figure costs I'm going to use the filaments used in the images above. 

| Purpose    | Amount  | $ per kg | $ used |
| ---------- | ------- | -------- | ------ |
| Base Color | 343.75g | $21.99   | $7.55  |
| Accent     | 38.19g  | $13.99   | $0.53  |
| Internal   | 52.41g  | $13.99   | $0.73  |

This gives a rough total cost of \$8.81 for filament used. 

Using a less fancy-sparkly base filament will obviously reduce the price, but we are still under \$10 for filament. 

## 3D Printing Notes

Overall the project takes about 22 hours to print, meaning the printer itself is working for 22 hours. The `.3mf` file for this project splits it into 12 plates:
1. Door Left
2. Door Right
3. Tray
4. *Stats Plate*
5. *Skills Plate*
6. *Number Wheels*[^1]
7. *Symbol Wheels*
8. Wheel Rods
9. Hinges
10. *Corner Embellishments*
11. Lock Mechanism
12. *Lock Embellishments*

The plates in _italics_ are plates that use both the base and accent colors. 

> [!warning] Use Glue on the Hotbed!
> The doors and tray *generally* do fine but I had one print where one corner peeled up a little. It didn't ruin the print but a simple pass or two with a glue stick makes the print come out much more flat. The number and symbol wheels _need_ glue to keep the very fine first few layers down. 

### Number Wheels
This is the part that gave me the most trouble in terms of failed prints/spaghetti errors. Glue on the hotbed helped tremendously. If you are still getting failures adjust your z-offset as well. And of course re-level your bed if you are _still_ having trouble. 

The original `.3mf` file from the creator requires you to print the number wheels tray three times. I have modified the file in my repo to print more wheels per tray, and reduced it to two. 

Theoretically you could fit all the number wheels you need (68) on one tray, but given that this is also the print with the greatest risk of failure it seems wiser to keep it split across at least two. 

### Slicer Settings

I've made a few adjustments to the original settings:

**First**: A way to make the whole project easier to deal with: on any plate that has multiples of the same item, delete all but one, then create the right number of _instances_ of that one. This way when you change the base item it changes all the instances as well. If they are each individual items you have to do this manually.

**Second**: **Turn off Brims!!!** Especially on the doors. The brims wreck havoc on the locking mechanism tracks and I have never had adhesion problems with the doors that weren't adequately solved by using glue on the hotbed. 


## Assembly Notes
Assembly is remarkably straightforward. The most tedious part is threading wheels onto wheel rods. To save you some time here are the wheel rod "loadouts" you need. `S` means "symbol wheel", `0` means "number wheel"

- **6 rods**: `S00 S00 S00` (Everything on the "skills" side)
- **3 rods**: `S0 S0 S0 S0` (Modifiers and saves for STR/DEX/CON/INT/WIS/CHA)
- **1 rod**: `000 000 000` (HP)
- **1 rod**: `00 S00 00` (AC/Initiative/Proficiency)
- **1 rod**: `0 0 00` (Death saves and hit dice type)

## Felt Lining

the dice rolling area benefits from being lined with felt to reduce the noise of rolling dice. The dimensions of that area are:

- Length: 150mm
- Width: 110mm
- Depth: 21mm

So you need 5 pieces of felt:
- (1) 150 x 110
- (2) 110 x 21
- (2) 150 x 21
Adhesive-backed felt is generally a dollar per 9in x 12 in piece (228.6mm x 304.8mm), so the cost is not prohibitive. 

[^1]: You will need to print this one three times
    
