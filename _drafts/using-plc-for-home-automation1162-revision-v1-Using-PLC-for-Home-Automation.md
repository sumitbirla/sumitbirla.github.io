---
id: 1260
title: Using PLC for Home Automation
date: 2020-01-20T11:01:48-05:00
author: Sumit Birla
layout: revision
guid: https://sumitbirla.me/2020/01/1162-revision-v1/
permalink: /2020/01/1162-revision-v1/
---
**Note:** _this post is not meant to teach you what a PLC is. If you are unfamiliar with it, I suggest you make a quick read of the following before returning to this page: <https://www.amci.com/industrial-automation-resources/plc-automation-tutorials/what-plc/>_

Automating routine tasks around the house has been a goal of mine for as long as I can remember. Examples of such tasks are:

  * Turn on outdoor lights at dusk and turn them off at dawn
  * Turn off HVAC when no one is at home
  * Turn off closet lights automatically 30 minutes after being turned on
  * Turn off the sprinkler system when it has rained

In addition to controlling &#8220;things&#8221; I also want to be able read sensors in realtime. Example of such sensors are: temperature, humidity, electricity consumption, water-flow, doors/windows status etc.

Ideally all &#8220;smart devices&#8221; have a local human interface and are also controllable over a network (WiFi, Ethernet, Zigbee, Z-Wave, RS485 etc.) Network control allows one to build higher-level dashboards with big touchscreens and also interface with voice assistants such as Alexa and Google Home. Couple of popular software options for building interfaces are Node Red and HomeAssistant.

The problem with deploying &#8220;smart devices&#8221; has always been related to reliability (remember X10), proprietary nature of the products and just plain poor implementation. There is no way I am going to install a proprietary App to control a light bulb! There are a few checkboxes that need to be ticked for a &#8220;smart home&#8221; device to be incorporated:<figure class="wp-block-table">

<table class="">
  <tr>
    <td>
      NON_SMART_MODE
    </td>
    
    <td>
      It should work as a standard &#8220;non-smart&#8221; device also.
    </td>
  </tr>
  
  <tr>
    <td>
      NETWORKED
    </td>
    
    <td>
      Should be controllable over a data network, but not depend on it. I.e. all logic necessary to operate must exist on the device itself. Device should be unaffected by network failures.
    </td>
  </tr>
  
  <tr>
    <td>
      RELIABILITY
    </td>
    
    <td>
      Will the device still function in 10 years not requiring an upgrade or reboot?
    </td>
  </tr>
  
  <tr>
    <td>
      MAINTAINABILITY
    </td>
    
    <td>
      Can someone other than me troubleshoot in case a device stops working? Are industry-standard protocols and techniques used.
    </td>
  </tr>
  
  <tr>
    <td>
      COST
    </td>
    
    <td>
      Should not cost an exorbitant amount, after all this is probably a hobby project for you.
    </td>
  </tr>
</table></figure> 

As far as hardware goes, the popular choices are Arduino, ESP8266-based boards and Raspberry Pi. In fact such a large portion of the Home Automation enthusiast community uses one of these platforms, that it will be difficult for you to find sample code or help if you use anything else. On message-boards, you are immediately shot down as the mere suggestion of using a PLC saying that it is overkill. <figure class="wp-block-gallery aligncenter columns-3 is-cropped">

<ul class="blocks-gallery-grid">
  <li class="blocks-gallery-item">
    <figure><img src="https://sumitbirla.me/wp-content/uploads/2020/01/Arduino_Uno_-_R3.jpg" alt="" data-id="1171" class="wp-image-1171" /></figure>
  </li>
  <li class="blocks-gallery-item">
    <figure><img src="https://sumitbirla.me/wp-content/uploads/2020/01/D1-Mini-Board-800x800.jpg" alt="" data-id="1179" data-link="https://sumitbirla.me/?attachment_id=1179" class="wp-image-1179" /></figure>
  </li>
  <li class="blocks-gallery-item">
    <figure><img src="https://sumitbirla.me/wp-content/uploads/2020/01/800px-Raspberry_Pi_3_B_39906369025.png" alt="" data-id="1180" data-link="https://sumitbirla.me/?attachment_id=1180" class="wp-image-1180" /></figure>
  </li>
</ul></figure> 

Granted that Arduino has been a huge step up in terms of rapid prototyping and led to really fascinating projects being built by people with little electrical engineering background. A quick Google search will show you basically anything you want to build, someone has already done it and posted their schematic and code online. But my question always comes back to its longevity. Past the fun-build stage, will I enjoy maintaining it? If I am not around, will anyone be able to make heads or tails of what I have built? Does it actually simplify my everyday routine in someway, or just add another thing that needs to be maintained?

<hr class="wp-block-separator" />

## Why PLC?

Well, let&#8217;s see how the various devices fare when it comes to the above-listed criteria:<figure class="wp-block-table">

<table class="">
  <tr>
    <td>
    </td>
    
    <td>
      Arduino
    </td>
    
    <td>
      ESP8266
    </td>
    
    <td>
      Raspberry PI
    </td>
    
    <td>
      PLC
    </td>
  </tr>
  
  <tr>
    <td>
      NETWORKED
    </td>
    
    <td>
      ❌
    </td>
    
    <td>
      ✅
    </td>
    
    <td>
      ✅
    </td>
    
    <td>
      ✅
    </td>
  </tr>
  
  <tr>
    <td>
      RELIABILITY
    </td>
    
    <td>
      ❌
    </td>
    
    <td>
      ❌
    </td>
    
    <td>
      ❌
    </td>
    
    <td>
      ✅
    </td>
  </tr>
  
  <tr>
    <td>
      MAINTAINABILITY
    </td>
    
    <td>
      ❌
    </td>
    
    <td>
      ❌
    </td>
    
    <td>
      ❌
    </td>
    
    <td>
      ✅
    </td>
  </tr>
  
  <tr>
    <td>
      COST
    </td>
    
    <td>
      ✅
    </td>
    
    <td>
      ✅
    </td>
    
    <td>
      ✅
    </td>
    
    <td>
      ❌
    </td>
  </tr>
</table></figure> 

I think the table above probably has caused some disagreements in your mind. Especially ones related to &#8220;Reliability&#8221; and &#8220;Maintainability&#8221;. Let me give you simple example: you have a networked esp8266 switch that turns ON/OFF your pool pump motor. You have soldered a bunch of wires to relays and one day it does not turn on. What do you do about it? Will someone other than you be able to resolve this issue? Is it a software problem or hardware?

I have been running Node-Red at home which does a whole bunch of functions. Everything from sending signals to turn ON/OFF lights to reading sensors. Some of the sensors are 433MHz sensors decoded using RTL dongle, some are Z-Wave and some are WiFi. Every now and then I am forced to update my Linux server because I need the update to be able to run some software. Guess what, WiFi AP may not come back online, certain Node-Red flows may fail until I address the underlying issue. The point is a seemingly benign operation on a computer may have unexpected effect with rest of the system. That is why you will never see a Windows or Linux directly controlling safety-critical industrial equipment.



<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img src="https://sumitbirla.me/wp-content/uploads/2020/01/6ed1052-1cc08-0ba0-nfs.jpg" alt="" class="wp-image-1174" width="340" height="340" /></figure>
</div>

Let&#8217;s look at an example PLC that I have chosen to use for some of my projects:

#### Siemens LOGO! 12/24 RCE 

  * Logic module with display
  * Power supply: 12/24 VDC
  * 8 DI (Digital Input) 4 of which can be used as AI (Analog Inputs)
  * 4 DO (Digital Output) Relay
  * Real Time Clock (RTC)
  * Memory: 400 blocks
  * modularly expandable
  * Ethernet
  * Integrated web server

Notice the neatly packaged module. It has a built-in display with buttons for basic configuration. The 4 Relay output can be used to switch currents up to 2.0A each. There are 8 general purpose inputs, 2 of which can also take 0-10V analog input. 0-10V sensors are an industrial standard along with 4-20mA. Ethernet is built-in along with a web server for building Human Machine Interface (HMI). So for the price of a single PLC, you are getting quite a bit!

Why not pick a different brand of PLC like Allen-Bradley or Schneider Electric? Well, in my case the decision was made for me. Siemens LOGO! appears to be the only PLC whose programming software runs on Linux and Mac. Based on YouTube videos I have watched, I would say programming on most systems will be very similar, so I don&#8217;t feel like I have invested time in a brand-specific system.

Now, let&#8217;s look at the software. On Arduino, you use a subset of C programming language to control your hardware. You have to manage your own switch debounce, interrupts, logic-level shifting and power for driving delays. If you have any time-based scheduling function involved, you may also need to add an RTC. Also you are responsible for adding your own interface (buttons, displays etc.) How much time and cost do you think will be needed to add the above to a base-arduino? Will you have complete confidence in the functioning of the hardware once you are done with the project? Will you need to create a 3-D printed enclosure to neatly package your project? Again &#8211; calculate how much it will cost.

As a final point of the blog post, let&#8217;s talk about the software. On Arduino or Raspberry Pi, you can pretty much write code any way you like. Five years from now, will you remember what your code is doing? Will someone else be able to access your code (without access to you) and figure out the functionality?

For one of my first PLC projects, I created the program below using Functional Block Diagram (FBD). The more commonly accepted programming paradigm is Ladder Logic which I didn&#8217;t use for no particular reason. By showing you this program, I just want to give an idea how &#8220;high level&#8221; programming on a PLC is. Truly, you just have to worry about the logic. The low level details are taken care of by the CPU and related hardware.<figure class="wp-block-image size-large">

<img src="https://sumitbirla.me/wp-content/uploads/2020/01/PoolPumpSoftware.png" alt="" class="wp-image-1185" /> </figure> 

This program is complete software for the control of my pool pump motor. It should be easy to understand to anyone with Industrial Automation background. Even if you don&#8217;t, it essentially boils down to basic digital logic. It is not hard to figure out what each block does by reading its documentation. 

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img src="https://sumitbirla.me/wp-content/uploads/2020/01/80759179_10156932012767076_649938223874703360_o.jpg" alt="" class="wp-image-1189" /></figure>
</div>

Above program implements the following functions:

  1. Manual START / STOP buttons
  2. Network START / STOP functions
  3. Network SET / READ schedule
  4. Local Display to SET schedule
  5. Local Display to show current status and configuration

As you see you can implement a robust solution with just a little extra cost upfront. It actually may turn out cheaper to use a PLC if you look for deals. For example, I bought mine for $66 on AliExpress. 

If you think about it, there are a lot of appliances and devices around the house whose controlling circuitry are essentially single-purpose &#8220;PLC&#8221; under the hood:

  * Thermostat
  * Sprinkler System Controller
  * Pool Pump Controller
  * Oven / Cooking Stove / Microwave
  * Dishwasher / Clothes Washer / Dryer
  * Coffee Machine
  * Garage Door Opener

What if all of the above were built using a general purpose PLC that you could connect to over the Ethernet and add remote control and diagnostics capabilities. That would be a home-automators dream!

#### Other examples of PLC use

  * Traffic Lights
  * Elevators / Lift
  * Theme park rides
  * Automatic car wash

<p class="has-text-align-left">
  When you are out and about, look for control panel enclosures with emergency stop buttons. You will see how widespread PLC based automation is.
</p>

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img src="https://sumitbirla.me/wp-content/uploads/2020/01/OEM-street-light-electrical-control-panel-box.jpg" alt="" class="wp-image-1258" width="245" height="249" /></figure>
</div>

Let me know your thoughts below.