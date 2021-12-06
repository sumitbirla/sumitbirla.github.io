---
id: 506
title: Home Controller
date: 2012-01-16T20:46:54-05:00
author: Sumit Birla
layout: page
guid: http://sumitbirla.com/?page_id=506
---
## Home Controller

Ever since I purchased [my Dell server](http://sumit.tampahost.net/2005/02/new-server-setup/ "New server: Dell PowerEdge SC420") for experimentation with Linux, I have been adding functionality to it. This has yielded some magnificent results. This server has proven to be a great platform for performing numerous tasks around the house. Some of these functions are listed below:

  1. Gateway to the Internet
  2. Wireless Access Point
  3. Network Switch
  4. File Server
  5. Print Server
  6. Telephony Exchange
  7. Media Server
  8. Environmental Data Collection

One of my concerns was how reliable this system would be. It is in the critical path for telephone and Internet services. But as it turns out, this server has never crashed. It has been running for years. I have even added an Uninterrupted Power Supply in case the power goes out. The battery is able to keep the server running for upto 30 minutes. This covers most instances of outages which only last a few seconds in my area. If the power is out for longer, then cell phones act as a backup. Scientists have proven that one can easily survive without Internet for a few hours. Taking all this into consideration, I have been pretty happy with the set up. The final piece of the puzzle is the hooking up of environmental sensors around the house using [Structured Wiring](http://sumit-old.tampahost.net/projects/wiring.php).

<div>
  <img class="aligncenter" src="http://sumit-old.tampahost.net/projects/images/europa_large.gif" alt="diagram of home server" width="520" height="301" />
</div>

Operating System
:   The server is currently running [Fedora 8](http://fedoraproject.org/). This is only because I am familiar with the way things are organized in Red Hat systems. This could very well be a different distribution of Linux. I installed the bare minimum set of packages. There is no Graphical User Interface. Everything has to be controlled via the console. This serves two purposes &#8211; you get to learn how things really work, and second.. the system does not require much memory. The system runs just fine with 256MB of RAM. The swap space is hardly ever touched.

Internet Gateway
:   Setting up a NAT gateway on linux is exceedingly simple. All functionality is already available in the standard Linux kernel. Iptables is a tool for setting up rules for dealing with packets passing through various network interfaces in the system. I added the following to /etc/rc.d/rc.local script:</p> 
    
    <pre># Enable NAT
echo 1 &gt; /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE</pre>
    
    Note that eth0 is the Internet facing interface on the server. Settings ip_forward to 1 instructs the kernel to enable IP routing. Otherwise the packets would never leave the local subnet.

Wireless Access Point
:   The PCI 802.11g card installed in the system is supported by the [ath5k](http://madwifi.org/wiki/About/ath5k) driver available in the [madwifi](http://madwifi.org/) package. I had to download, compile and install this package. It was a problem-free process. This interface is given an IP address of 192.168.2.1 (the wired network uses the 192.168.1.x subnet).</p> 
    
    <pre>wlan0     IEEE 802.11g  ESSID:"europa"  Nickname:"xxxx.xxxx.xxxx"
Mode:Master  Frequency:2.412 GHz  Access Point: 00:13:46:97:73:4D
Bit Rate:0 kb/s   Tx-Power:18 dBm   Sensitivity=1/1
Retry:off   RTS thr:off   Fragment thr:off
Encryption key:1234-5678-90   Security mode:restricted
Power Management:off
Link Quality=35/70  Signal level=-61 dBm  Noise level=-96 dBm
Rx invalid nwid:22629  Rx invalid crypt:0  Rx invalid frag:0
Tx excessive retries:0  Invalid misc:0   Missed beacon:0</pre>

Network Switch
:   A 4-port switch is embedded in the system as a PCI card. The model is the hard to find [HP NC150T](http://h20000.www2.hp.com/bizsupport/TechSupport/Home.jsp?lang=en&cc=us&prodTypeId=329290&prodSeriesId=428161). I found one on eBay and installed it to reduce the number of wires in my server closet. The power is supplied via the PCI bus, so a separate power supply is not necessary. The question arises what happens when the server is down. Well, the server is hardly ever down. If it is down, all network devices would be useless anyway since they wouldn&#8217;t have access to Internet and the Media Center would not have a file server to pick up files from.

File server
:   The server has a 750GB hard drive. Any person with an account on the server can backup files using SSH, rsync or samba

Print Server
:   I have a HP P1006 LaserJet printer. I had to download and install [foo2xqx](http://foo2xqx.rkkda.com/) driver. Cups acts as the print server. It has a web interface for setting up printers and managing print jobs. I am able to print from both Linux and Windows computers. This printer is cheap ($99), compact and LaserJet!

Telephony Server
:   I subscribe to a VoIP service called [ViaTalk](http://www.viatalk.com/) for my land-line. They are similar to Vonage but allow you to use your own adapter/device.[Asterisk](http://www.asterisk.org/) is a versatile open-source software which allows you to register with VoIP servers and control telephones, voice mail, extensions and a lot more. It is a full-fledged telephony system in use commercially. And best of all, it works brilliantly on Linux using very few resources. This is installed and running on the server. It manages all calls in and out of the house. There are three analog phones in the house which are all routed via the server to ViaTalk. Analog cards are controlled via [Digium TDM400P](http://www.digiumcards.com/tdm400p.html) PCI card installed in the server. This eliminates the need for IP phones which are expensive, difficult to set up and require power supply. Analog phones can be purchased for $10.Installed analog card:</p> 
    
    <pre>[root@europa ~]# cat /proc/zaptel/*
Span 1: WCTDM/0 "Wildcard TDM400P REV E/F Board 1" (MASTER) 

	1 WCTDM/0/0 FXOKS (In use)
	2 WCTDM/0/1 FXOKS (In use)
	3 WCTDM/0/2 FXOKS (In use)
	4 WCTDM/0/3 FXOKS</pre>
    
    Relevant sections of /etc/asterisk/zapata.conf
    
    <pre>; Zap/1 BEGIN
signalling=fxo_ks
context=sip-phones
mailbox=3001
callerid="Rosana's Study" &lt;3001&gt;
group = 1
channel =&gt; 1
callgroup=1
pickupgroup=1
; Zap/1 END

; Zap/2 BEGIN
signalling=fxo_ks
context=sip-phones
mailbox=3002
callerid="Sumit's Study" &lt;3002&gt;
usecallerid =&gt; yes
group = 1
channel =&gt; 2
callgroup=1
pickupgroup=1
; Zap/2 END

; Zap/3 BEGIN
signalling=fxo_ks
context=sip-phones
mailbox=3003
callerid="Living Room" &lt;3003&gt;
usecallerid =&gt; yes
group = 1
channel =&gt; 3
callgroup=1
pickupgroup=1
; Zap/3 END</pre>
    
    Relevant sections of /etc/asterisk/sip.conf, phone number x&#8217;ed out
    
    <pre>register =&gt; 1xxxxxxxxxx:password@richmond-1a.vtnoc.net/1xxxxxxxxxx

[viatalk]
type=peer
fromuser=1xxxxxxxxxx
username=1xxxxxxxxxx
authuser=1xxxxxxxxxx
secret=password
host=richmond-1a.vtnoc.net
fromdomain=richmond-1a.vtnoc.net
nat=no
canreinvite=yes
insecure=very
qualify=4000
dtmfmode=inband
dtmf=inband</pre>
    
    With this setup, I am able to use the phones as an intercom and call from one room from another. I can also have two simultaneous outgoing calls. When there is an incoming call, all three phones ring at the same time. And of course, I can monitor and control the whole system remotely by logging in via SSH.

Media Server
:   I have a [TViX M-6500A](http://www.tvix.co.kr/Eng/products/HDM6500A.aspx) media server which can play all sorts of video and audio files. It also can show photos from shared network folders. This device is connected to my TV in the living room. It is also on the home network, so it is able to pick up shared music, movies and photos from an open Samba share on the server.
:   [![media center photo 1](http://sumit-old.tampahost.net/projects/images/mediacenter1_sm.jpg)](http://sumit-old.tampahost.net/photo/display.php?img=/projects/images/mediacenter1.jpg) [![media center photo 2](http://sumit-old.tampahost.net/projects/images/mediacenter2_sm.jpg)](http://sumit-old.tampahost.net/photo/display.php?img=/projects/images/mediacenter2.jpg) [![media center photo 3](http://sumit-old.tampahost.net/projects/images/mediacenter3_sm.jpg)](http://sumit-old.tampahost.net/photo/display.php?img=/projects/images/mediacenter3.jpg)

Environmental Data Collection
:   Dallas Semiconductors builds cheap sensors which use the 1-wire protocol to talk with a controller. I use a serial controller on the server which is connected to the sensors throughout the house using [cat5 cabling](http://sumit-old.tampahost.net/projects/wiring.php). OWFS software is used for reading in the values from sensors. Currently, I only have a temperature and humidity sensor. Eventually, I plan to have 3 temperature sensors, 1 humidity sensor, 1 barometric pressure and a soil moisture sensor. More details will be posted once I have completed this.[Link to initial data](http://sumit-old.tampahost.net/projects/ha-data.php).

Uninterrupted Power Supply
:   A small UPS provides power protection and backup to the server. The model is [Back-UPS ES 500](http://www.apc.com/resource/include/techspec_index.cfm?base_sku=BE500U). It has enough battery to keep the server running for 30 minutes. It also has a USB interface which can be used by a computer for monitoring the status of the UPS. A daemon process on the server called [apcupsd](http://www.apcupsd.org/) continually checks the status. If it notices that the power is out and there is less than 3 minutes of battery remaining, it starts shutting down the server. The BIOS is set to automatically restart the machine when power is restored.</p> 
    
    <pre>[root@europa ~]# apcaccess
APC      : 001,037,0984
DATE     : Thu Aug 07 21:41:19 EDT 2008
HOSTNAME : europa.tampahost.net
RELEASE  : 3.14.3
VERSION  : 3.14.3 (20 January 2008) redhat
UPSNAME  : europa.tampahost.net
CABLE    : USB Cable
MODEL    : Back-UPS ES 500
UPSMODE  : Stand Alone
STARTTIME: Tue Jul 29 10:19:37 EDT 2008
STATUS   : ONLINE
LINEV    : 119.0 Volts
LOADPCT  :  23.0 Percent Load Capacity
BCHARGE  : 100.0 Percent
TIMELEFT :  24.6 Minutes
MBATTCHG : 5 Percent
MINTIMEL : 3 Minutes
MAXTIME  : 0 Seconds
SENSE    : High
LOTRANS  : 088.0 Volts
HITRANS  : 139.0 Volts
ALARMDEL : Always
BATTV    : 13.5 Volts
LASTXFER : Low line voltage
NUMXFERS : 41
XONBATT  : Thu Aug 07 07:56:02 EDT 2008
TONBATT  : 0 seconds
CUMONBATT: 90 seconds
XOFFBATT : Thu Aug 07 07:56:04 EDT 2008
STATFLAG : 0x07000008 Status Flag
MANDATE  : 2005-09-05
SERIALNO : BB0537004943
BATTDATE : 2000-00-00
NOMINV   : 120 Volts
NOMBATTV :  12.0 Volts
FIRMWARE : 824.B1.D USB FW:B1
APCMODEL : Back-UPS ES 500
END APC  : Thu Aug 07 21:41:40 EDT 2008</pre>

What more can this little server do? If you have ideas, please leave a comment below.