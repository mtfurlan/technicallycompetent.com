---
title: Temperature controlled peltier box
date: 2024-10-01
redirectFrom: drafts/temp-box
---

I have a drug that needs to be "room temperature", or 20-25C, but the rooms in
my house are more like 15-30C, and I sometimes go outside for a few weeks.

So, peltier box that can heat and cool as required.

Parts:
* amazon special STC-1000 thermostat with heat/cool relay outputs
* constant current driver
* peltier pads
* h bridge relay I found
* a pair of diodes, the paper said 1N4002

![Electrical schematic](/assets/pages/temp-box/schematic.jpg "such schematic")

Looks like I can fit all the stuff inside the STC-1000 enclosure, lots of space
![starting controller top](/assets/pages/temp-box/start-top.jpg)
![starting controller bottom](/assets/pages/temp-box/start-bottom.jpg)

The not populated stuff is almost certainly because one of the versions
supports 240VAC not 12VDC, but I have a fair bit less components than the three
[Big Clive has](https://youtu.be/T4umSkJjXwY?si=-hwj4bqc9eLnjrpS)...

Seems to work though, so I made everything fit

It did require sanding down the constant current/constant voltage driver a mm or
so

![final controller tip](/assets/pages/temp-box/final-top.jpg)
I reused the heat/cool relay screw terminals for my fan output by just cutting
the relay traces and drilling more holes in the board to fit my wires
![final controller bottom](/assets/pages/temp-box/final-bottom.jpg)

Next steps: make a box and see how badly it keeps temperature

Would maybe be nice to have a button to enable the display, for future battery
reasons, but if I actually want to care about low power the extra relays and
STC-1000 is the wrong approach anyway
