---
layout: post
title: 20210401 Repairing Arduino Nano
---

Was preparing a [simple music box]({{ site.baseurl }}/Throwback-First-Arduino-Project/) for Victoria's birthday. It's the combo of Arduino Nano + photoresistor + buzzer. 
As the first electronic project of mine, 2 Arduino Nano were burnt \(5V shorted to GND, powered by USB\) due to loose parts on breadboard.

Referring to [this](https://www.instructables.com/How-to-Fix-Fried-Arduino-NanoUnoMega/), even my Arduinos do not have visible damage, I'd go ahead to replace the diode and successfully revived these burnt Arduino Nano.

![image2]({{ site.baseurl }}/images/arduino-repair-2.jpeg)

**Tips/Tricks**
- No Desoldering Braid ? Heat the solder then quickly/gently knock your Nano on table, most solder will fall off. Repeat for both sides, then try to heat and tick the diode away from 1 side using sharp tool \(toothpick, needle, tweezer\).
- Reminder for non-electronic background, the Diode, by nature, have polarity \(must solder at correct direction\)