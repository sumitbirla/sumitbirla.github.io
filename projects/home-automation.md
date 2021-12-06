---
id: 516
title: Home Automation
date: 2012-01-16T20:59:00-05:00
author: Sumit Birla
layout: page
guid: http://sumitbirla.com/?page_id=516
---
## My Home Automation Setup (completed in 2000)

**Linux PC**:  
Linux makes a good controller for home automation projects. Source code is freely available and the darn thing never crashes &#8211; even if you try really hard! With an apache web server installed, easy to use web interfaces can be created to control things in the house.

**1-wire sensors**:  
From Dallas Semiconductors, these sensors are extremely easy to use. They are tiny and connect to the serial port of any PC. They do not require any external power source (less wires, yay!)

**X-10 Devices**:  
These devices make use of your home&#8217;s electrical wiring to propogate control signals. Various modules can be bought for controlling appliances such as lamps and coffee machines. Attach a motion sensor and you can turn on lights automatically when you walk into a room. X-10 signals can be broadcast from either a remote control or a PC with the appropriate interface.

**Cell Phone**:  
Cell phones with a built in WAP browser can be used in conjunction with Apache web server to receive information and control X-10 devices through the web (ain&#8217;t that cool!)

<center>
  <img src="http://sumit-old.tampahost.net/images/schema.gif" alt="" />
</center>In the diagram above, all components drawn as rectangles are on a solid-state, low power linux computer. The modem is used to log caller ID information. 1-wire sensors provide temperature, humidity and light readings. The Apache web server acts as a central coordinating process while mySQL database saves information fed by the modem and 1-wire sensors.

**References:**

  * [Linux](http://www.linux.org/)
  * [Dallas Semiconductors 1-wire sensors](http://www.iButton.com/)
  * [X-10 Devices](http://www.x10.com/)
  * [Apache web server](http://www.apache.org/)
  * [mySQL Database](http://www.mysql.com/)
  * [Wireless Access Protocol](http://www.wap.com/) (WAP)