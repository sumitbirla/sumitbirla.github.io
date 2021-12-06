---
id: 1058
title: Evolution of the Home Server
date: 2016-11-21T11:12:32-05:00
author: Sumit Birla
layout: post
guid: http://sumitbirla.com/?p=1058
permalink: /2016/11/evolution-of-the-home-server/
image: /wp-content/uploads/2016/11/81hjCxFfW-L._SL1500_.jpg
categories:
  - Home Automation
  - Linux
  - Networking
  - Technology
---
Ever since broadband became available (around 1998), I have always run a general purpose Linux home server. I have used it as a router, a file server and to experiment with programming various things. The earliest servers were full-fledged tower computers, then I moved over to ITX form-factor and finally to an Intel NUC. While the Intel NUC is a very nice, compact and low power device, it only has one ethernet port. So I needed an additional network switch to connect my desktop, TV and printer. As many know, I am allergic to clutter comprising of cable, power supplies, and devices in general. Ideally, a single low power device should be able to do it all.

After a couple of years of running the Intel NUC, I finally found my close-to-ideal computer to act as the following:

1. Network Switch  
2. Router  
3. Wireless Access Point  
4. Network Attached Storage (NAS)  
5. Plex Media Server  
6. TV Tuner Streaming

The little IBOX-N10a has 4 gigabit ethernet ports, two mSATA slots which I can use for SSD and wifi card. The device runs Intel Celeron J1900, Quad Core CPU at 2.00 GHz (turbo 2.4 GHz) and it is fanless with the chassis acting as a big heatsink. I have installed 4GB of RAM and a 1TB SSD from Samsung.

[<img class="aligncenter size-large wp-image-1072" src="http://sumit.tampahost.net/wp-content/uploads/2016/11/819cMXejIvL._SL1500_-1024x568.jpg" alt="819cMXejIvL._SL1500_" width="915" height="508" srcset="https://sumitbirla.me/wp-content/uploads/2016/11/819cMXejIvL._SL1500_-1024x568.jpg 1024w, https://sumitbirla.me/wp-content/uploads/2016/11/819cMXejIvL._SL1500_-300x166.jpg 300w, https://sumitbirla.me/wp-content/uploads/2016/11/819cMXejIvL._SL1500_-900x499.jpg 900w, https://sumitbirla.me/wp-content/uploads/2016/11/819cMXejIvL._SL1500_.jpg 1435w" sizes="(max-width: 915px) 100vw, 915px" />](http://sumit.tampahost.net/wp-content/uploads/2016/11/819cMXejIvL._SL1500_.jpg)

  * [iBOX-N10A](https://www.amazon.com/dp/B01MEGSMRZ) fanless aluminium chassis
  * Celeron J1900, Quad Core Bay Trail 2.0GHz (turbo 2.42GHz), 2MB L2 Cache, 10W TDP
  * Intel 82583V Gigabit Ethernet &#8211; 4 ports
  * Crucial 4GB Single DDR3L 1600 MT/s (PC3-12800) SODIMM 204-pin
  * Samsung 850 EVO &#8211; 1TB &#8211; mSATA Internal SSD
  * IMC Networks 802.11 n/g/b Wireless LAN USB Mini-Card
  * APC Back-UPS Connect BGE90M &#8211; 3 hours backup power

The output of \`lspci\` is shown below. It is fairly simply system with the Celeron J1900 processor providing 4 lanes of PCIe, each connected to a single Intel 82583V Gigabit Network chip. The integrated SATA II controller connects to the mSATA slot while the WIFI card uses the USB signal lines on the second mSATA slot.

<pre class="brush: plain; title: ; notranslate" title="">root@ibox:~# lspci -vt
-[0000:00]-+-00.0  Intel Corporation Atom Processor Z36xxx/Z37xxx Series SoC Transaction Register
+-02.0  Intel Corporation Atom Processor Z36xxx/Z37xxx Series Graphics &amp;amp;amp; Display
+-13.0  Intel Corporation Atom Processor E3800 Series SATA AHCI Controller
+-1a.0  Intel Corporation Atom Processor Z36xxx/Z37xxx Series Trusted Execution Engine
+-1b.0  Intel Corporation Atom Processor Z36xxx/Z37xxx Series High Definition Audio Controller
+-1c.0-[01]----00.0  Intel Corporation 82583V Gigabit Network Connection
+-1c.1-[02]----00.0  Intel Corporation 82583V Gigabit Network Connection
+-1c.2-[03]----00.0  Intel Corporation 82583V Gigabit Network Connection
+-1c.3-[04]----00.0  Intel Corporation 82583V Gigabit Network Connection
+-1d.0  Intel Corporation Atom Processor Z36xxx/Z37xxx Series USB EHCI
+-1f.0  Intel Corporation Atom Processor Z36xxx/Z37xxx Series Power Control Unit
\-1f.3  Intel Corporation Atom Processor E3800 Series SMBus Controller
</pre>

The power consumption when idle is around 8W. When playing a HEVC x265 video via Plex requiring transcoding, the power usage jumps to 15W. The chassis still doesn&#8217;t get too hot. The rest of the applications (dnsmasq, samba, tvheadend) barely use any CPU.

<pre class="brush: plain; title: ; notranslate" title="">root@ibox:~# sensors
coretemp-isa-0000
Adapter: ISA adapter
Core 0:       +38.0°C  (high = +105.0°C, crit = +105.0°C)
Core 1:       +38.0°C  (high = +105.0°C, crit = +105.0°C)
Core 2:       +40.0°C  (high = +105.0°C, crit = +105.0°C)
Core 3:       +40.0°C  (high = +105.0°C, crit = +105.0°C)
</pre>

I am currently running Ubuntu 16.04 LTS Server on this box as the base operating system. dnsmasq acts as DHCP server and caching DNS. hostapd provides the wireless access point functionality. Samba turns the system into a file server while Plex and tvheadend handle the media functions. Plex is able to stream my videos directly to Samsung Smart TV which has a Plex client built in. tvheadend takes the input from the USB TV tuner and makes broadcast channels available over the network much like the hdhomerun devices. With all this software running, the system only uses around 300MB of RAM (out of 4GB). So I am confident that this system to serve me well in the foreseeable future.

### &nbsp;

### Product and installation images

[<img class="alignleft size-medium wp-image-1069" src="http://sumit.tampahost.net/wp-content/uploads/2016/11/81hjCxFfW-L._SL1500_-300x200.jpg" alt="81hjCxFfW+L._SL1500_" width="300" height="200" srcset="https://sumitbirla.me/wp-content/uploads/2016/11/81hjCxFfW-L._SL1500_-300x200.jpg 300w, https://sumitbirla.me/wp-content/uploads/2016/11/81hjCxFfW-L._SL1500_-1024x683.jpg 1024w, https://sumitbirla.me/wp-content/uploads/2016/11/81hjCxFfW-L._SL1500_-900x600.jpg 900w, https://sumitbirla.me/wp-content/uploads/2016/11/81hjCxFfW-L._SL1500_.jpg 1500w" sizes="(max-width: 300px) 100vw, 300px" />](http://sumit.tampahost.net/wp-content/uploads/2016/11/81hjCxFfW-L._SL1500_.jpg)

[<img class="alignleft size-medium wp-image-1076" src="http://sumit.tampahost.net/wp-content/uploads/2016/11/61CHJrmyZ6L._SL1074_-300x210.jpg" alt="61CHJrmyZ6L._SL1074_" width="300" height="210" srcset="https://sumitbirla.me/wp-content/uploads/2016/11/61CHJrmyZ6L._SL1074_-300x210.jpg 300w, https://sumitbirla.me/wp-content/uploads/2016/11/61CHJrmyZ6L._SL1074_-1024x718.jpg 1024w, https://sumitbirla.me/wp-content/uploads/2016/11/61CHJrmyZ6L._SL1074_-900x631.jpg 900w, https://sumitbirla.me/wp-content/uploads/2016/11/61CHJrmyZ6L._SL1074_.jpg 1068w" sizes="(max-width: 300px) 100vw, 300px" />](http://sumit.tampahost.net/wp-content/uploads/2016/11/61CHJrmyZ6L._SL1074_.jpg)

[<img class="alignleft size-medium wp-image-1085" src="http://sumit.tampahost.net/wp-content/uploads/2016/11/IMG_9254-200x300.jpeg" alt="Final Installation" width="200" height="300" srcset="https://sumitbirla.me/wp-content/uploads/2016/11/IMG_9254-200x300.jpeg 200w, https://sumitbirla.me/wp-content/uploads/2016/11/IMG_9254-682x1024.jpeg 682w, https://sumitbirla.me/wp-content/uploads/2016/11/IMG_9254.jpeg 853w" sizes="(max-width: 200px) 100vw, 200px" />](http://sumit.tampahost.net/wp-content/uploads/2016/11/IMG_9254.jpeg)