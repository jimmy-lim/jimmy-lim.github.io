---
layout: post
title: 20210412 Throwback of My First Arduino-Powered Electronic Project
---

March is Victoria's birthmoon. Was low on budget thus a DIY gift idea come into my mind (A simple music box, featuring Arduino Nano + photoresistor + buzzer)

The setup is simple, sample codes are available [here](https://www.instructables.com/How-to-use-a-photoresistor-or-photocell-Arduino-Tu/) and [there](https://github.com/robsoncouto/arduino-songs). Everything could have done in an hour *if nothing wrong*. 2 Arduino Nano were blown (5V shorted to GND) during USB powered, referring to [this post]({{ site.baseurl }}/Reparing-Arduino-Nano/) for fixing guide. Luckily I got 3 Nano in total, and god blessed the last Nano survived to stay permanently in the DIY music box as Year 2021 Victoria's birthday gift. 

The Nano was powered by 3 AA batteries (packed into battery compartment chopped off from of faulty bubblegun), Everything is put together in a cardboard magnetic flap lid box, and hot-glued fixed in place. A on/off switch was added too.

![image1]({{ site.baseurl }}/images/arduino-music-box-1.jpeg)
![image2]({{ site.baseurl }}/images/arduino-music-box-2.jpeg)

The main loop is simple, ready photoresistor value, if the value is low (box is closed), then it's ready to play some music the next time the box open.
```
int value = 0;          // Store value from photoresistor (0-1023)
bool bReady = false;

void loop(){
  value = analogRead(pResistor);
  Serial.println(value);
  if (value > 200){
    if(bReady) {
  		play();
  		bReady = false;
  	}
  }
  else {
  	bReady = true;
  }
  delay(1000); //Small delay
}
```

![image1]({{ site.baseurl }}/images/arduino-music-box-1.jpeg)
![image2]({{ site.baseurl }}/images/arduino-music-box-2.jpeg)

**Tips/Tricks**
- This is a very basic project (in case you are electronic background). Well, I am from software background, thus took awhile to figured out everything.
- The BOM costs less than NZD10 I guess. But for your partner who appreciate your hardwork, she'll be very happy because this is 1-of-a-kind DIY music box that you made just for her.