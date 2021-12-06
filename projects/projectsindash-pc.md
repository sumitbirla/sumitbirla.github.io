---
id: 513
title: Indash PC
date: 2012-01-16T20:55:51-05:00
author: Sumit Birla
layout: page
guid: http://sumitbirla.com/?page_id=513
---
This web page describes my efforts for constructing a highly integrated indash computer system for my car. The basic idea behind this is to have a low-power, solid-state computer in the double-DIN stereo slot with an LCD display that can play _mp3s, radio,_ show _GPS maps_ and perform _OBD Diagnostics_ with minimal user intervention.

<center>
  <img src="http://sumit-old.tampahost.net/projects/indashpc/carpc_concept.gif" alt="" /><br /> <em>Concept Diagram</em>
</center>

## 

## The Hardware

### 

### The Computer

PCM-5820 is a low-power, pentium level, single board computer from [Advantech](http://www.advantech.com/). It is adequate for running a light version of linux and playing mp3s. It has built-in video, ethernet, serial ports, parallel port, USB and sound. The system runs on a single 5V supply and consumes about 8 watts typically. I attach a CompactFlash to the IDE connector for storage. This ensures that the system is solid-state (no hard disk).

<center>
  <img src="http://sumit-old.tampahost.net/projects/indashpc/PCM-5820.jpg" alt="" /><br /> <em>Advantech&#8217;s PCM-5820</em>
</center>&nbsp;

* * *

### Power Supply

The power supply is based on National Semiconductors linear regulator &#8211; LM1084. Input voltage is in the range 6.4V to 20V and output, a steady 5V (or so it is hoped).

<center>
  <img src="http://sumit-old.tampahost.net/projects/indashpc/12v_regulator.gif" alt="" />
</center>The diagram above is for a 12V design, but the 5V version is identical, just a different chip.

* * *

### Audio Amplifier

The output from the computer board is pretty low power. In order to power car speakers, I need to amplify the audio using the 10W + 10W amplifier shown below. It is available for purchase from [Quasar Electronics](http://www.quasarelectronics.com/3088.htm).

<center>
  <a href="http://sumit-old.tampahost.net/projects/indashpc/q3088.jpg"><img src="http://sumit-old.tampahost.net/projects/indashpc/q3088.jpg" alt="" width="325" height="145" border="0" /></a>
</center>

* * *

<img src="http://sumit-old.tampahost.net/projects/indashpc/usbradio.gif" alt="" align="right" /> 

### Radio

The radio used for this project is an off-the-shelf USB radio from D-Link. The USB link is used for controlling the radio frequency. The audio out from the radio is connected to the audio-in on the computer board. Simple enough? Linux kernel comes with a driver to control this radio (lucky me!)

* * *

### Keypad

The parallel port will be used to monitor the state of five navigation keys. There is a simple parallel port programming library called[parapin](http://www.circlemud.org/~jelson/software/parapin/) for doing this. Hardware interface diagram coming soon&#8230;

* * *

### LCD

I am still searching for an appropriate 320&#215;240 TFT LCD to interface with Advantech&#8217;s PCM-5820. Another hurdle is setting Linux FrameBuffer to that resolution.

## The Software

There are two different components that I will develop for this project.

  1. **GPS Mapping and Routing** &#8211; called [TMRS](http://sumit-old.tampahost.net/software/tmrs.php)
  2. **OBD2 Interface** &#8211; software for talking with [BR3](http://www.obddiagnostics.com/) and fetching OBDII codes.

Apart from the above two, the main user interface for the system will be designed with the use of [DirectFB](http://www.directfb.org/) library. X-Windows is simply too bloated for an embedded system and introduces too many dependencies.

## Updates

**July 18, 2004**

It has been a long time since I posted an update. But the work never stopped. [TMRS](http://sumit-old.tampahost.net/software/tmrs.php) has reached a level of maturity where it can be used in a navigation application. I have also build a sensor device to measure temperature, humidity and acceleration which communicates with the main board using serial connection. Also in works are designs for display and chassis for the DIN sized computer.

**November 23, 2003**

Added [pictures](http://sumit-old.tampahost.net/photo/browse.php?dir=indashpc) section to this site. Programmed photo browsing function (screenshot in pictures section). Bought a PC/104 Vehicle Power supply module which neatly docks with my Single Board Computer.

**October 19, 2003**

TMRS appears to be working well with a server interface. I have written a quick and dirty [program](http://sumit-old.tampahost.net/projects/indashpc/viewer.png) in C# to fetch maps over the network.

**August 26, 2003**

The project has once again taken a new direction. I have created an Open Source project called Tiger Mapping and Routing Server (TMRS). It is designed to be POSIX compliant and should run on many platforms. Its job is to run in the background and perform one of the following two tasks when requested by a higher level application:

  1. Given start and destination addresses, compute the best path (driving directions). This is mostly done.
  2. Given longitude/latitude, draw a map centered around that coordinate at a specified scale and return a JPEG or PNG image. I am currently coding this part.

**January 06, 2003**

A lot of progress has been made since the last entry. I have purchased a [Deluo](http://www.deluo.com/) USB GPS sensor. I have designed the GUI and created modules to play radio and mp3s. I will post screenshots as soon as I have figured out how to capture from framebuffer.

**December 24, 2002**

Purchased a [Hauppage](http://www.hauppage.com/) WinTV/FM Radio card for my system. Found software to control the radio tuner and the volume. TV works fine in X-windows but not in framebuffer mode. Still working on this. I have abandoned the idea of using [Gtk](http://www.gtk.org/) or [Qt](http://www.trolltech.no/) since the use of tiny widgets is not appropriate for car use. The interface needs to be super simple so the user does not spend any time trying to figure out which button to click. Also created [mockups](http://sumit-old.tampahost.net/projects/indashpc/mockups.html) of the screen.

**December 15, 2002**

Started looking into the software that will be used for this project. Tried out [DirectFB](http://www.directfb.org/). It is a very low level library that lets you draw directly to the video framebuffer. Very cool, but in order to take advantage of code already written for mp3 player, GPS etc, I will have to look into using Gtk. I want to avoid using X-server, if possible. Good news is that Gtk has a port for framebuffer. The bad news is that I can&#8217;t seem to get it to work! Stay tuned&#8230;

**December 10, 2002**

Purchased [KILL-A-WATT](http://www.p3international.com/products/special/P4400/P4400-CE.html) meter from [Radio Shack](http://www.radioshack.com/) today for $40. Turns out that my development system consumes a mere **23 watts**while playing mp3s! My Pentium 4 windows machine, on the other hand, consumes anywhere between **75 watts** and **115 watts**.

**December 06, 2002**

Managed to boot a [minimal linux system](http://sumit-old.tampahost.net/projects/indashpc/linembed.php) from CompactFlash.

**November 28, 2002**

The construction of the [development workstation](http://sumit-old.tampahost.net/projects/indashpc/devsystem.php) is complete.