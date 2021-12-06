---
id: 1146
title: Low Power Media Center
date: 2019-11-19T15:26:04-05:00
author: Sumit Birla
layout: revision
guid: https://sumitbirla.me/2019/11/1-autosave-v1/
permalink: /2019/11/1-autosave-v1/
---
[<img class="size-full wp-image-315 alignleft" title="Confluence XBMC Theme" src="http://sumit.tampahost.net/wp-content/uploads/2012/01/confluence.jpg" alt="" width="692" height="389" srcset="https://sumitbirla.me/wp-content/uploads/2012/01/confluence.jpg 692w, https://sumitbirla.me/wp-content/uploads/2012/01/confluence-300x168.jpg 300w" sizes="(max-width: 692px) 100vw, 692px" />](http://sumit.tampahost.net/?attachment_id=315)

Until now I have needed two devices for my server needs at home. One to serve as a general purpose router, phone, web and file server ([Dell PowerEdge SC420](http://sumit.tampahost.net/2005/02/dell-sc420/ "New server: Dell PowerEdge SC420")) and another to watch my media on TV ([Popcorn Hour](http://www.popcornhour.com/)). But recently I decided to combine the two and build a single, low power computer that costs less than the resale value of the previous two devices. Save money and save energy, what&#8217;s there to not like. Decribed below is my new set up. So far I have been very pleased with the performance. It consumes only 30W of power, even when playing 1080p HD movies. The rest of the functions don&#8217;t even register as CPU use (maybe 1% or 2%).  
<!--more-->

### &nbsp;

### The Hardware

The key to building this device was to carefully pick the right combination of hardware and software while keeping the cost down. As far as processors go,&nbsp;[ARM processors](http://en.wikipedia.org/wiki/ARM_architecture)&nbsp;have extremely low power consumption and usually incorporate the memory controller and graphics plus additional perpheral connectivity on a single chip. However, the motherboards are usually all geared towards embedded use. I wanted to build a general purpose server with SATA controller and support of a mainstream Linux distribution. Because of this, I chose the[Intel Atom 230](http://en.wikipedia.org/wiki/Intel_atom)&nbsp;processor. It has two cores running at 1.6GHz and doesn&#8217;t require a fan to cool it. Its younger brother &#8211; Intel Atom N330 has 4 cores, but double the thermal dissipation requiring the use of a fan to cool it. I try to minimize the use of moving parts in my projects. So this was out. As it turned out, the 230 has plenty of horsepower to run XMBC; 330 would have been overkill (don&#8217;t believe everything you read on the web; people generally tend to want more, even when it isn&#8217;t needed).

<img class="alignright" src="http://sumit-old.tampahost.net/writings/images/mi-008.jpg" alt="APEX MI-008" /> 

The Atom is fine for running Linux as a router, file server etc., but what about video playback and 3D acceleration necessary for the fancy GUI of XBMC? For this, NVIDIA makes the companion chip called&nbsp;[ION](http://en.wikipedia.org/wiki/Nvidia_Ion). It has numerous HD acceleration features including H.264 decoding and full HD output. And most importantly, the API used to utilize these features&nbsp;[VDPAU](http://en.wikipedia.org/wiki/VDPAU), is fully supported in Linux and XBMC. So I searched for a Mini-ITX motherboard that has Intel Atom N230 CPU and NVIDIA Ion chipset and found the&nbsp;[ZOTAC IONITX-B-E](http://www.zotacusa.com/zotac-ionitx-b-e-atom-n230-1-6ghz-mini-itx-intel-motherboard.html). I added 2 1GB DDR memory sticks to the board to take advantage of the double data rate transfer.

The next step was to find a suitable chassis that would stay quiet and fit under the TV. For this, I found&nbsp;[APEX MI-008](http://www.silentpcreview.com/apex-mi008). It is fairly good looking and very quiet. Luckily, the air intake for the power supply is right above the CPU; so it sucks the rising heat out from the case. I did not need to install any more fans in the system.

The last step was to buy some storage. I took a trip to Best Buy and picked up a&nbsp;[2TB Western Digital Green](http://www.wdc.com/en/products/products.asp?DriveID=576)&nbsp;drive. This drive parks its head every 5 seconds and gets unparked when the Linux ext filesystem does a flush every 8 seconds! This causes excessively high load cycles on the hard drive. To disable the parking, Google wdidle3.exe. This is a DOS tool that WD provides to disable the park timer on the hard driver, increasing its life expectancy in theory.

### The Software

The first order of business after the hardware was assembled was to install Ubuntu 9.10 Server. This is a minimal install with no GUI at all. I have no intention of using the desktop, I control everything from the command line. The only reason the X-Server is required is for XBMC. So I start up a barebones X-Server with absolutely no window manager of any sort and then start up XBMC which occupies 100% of the screen at 1920&#215;1080.

The Linux kernel does not include the VDPAU drivers from NVidia. This needs to be downloaded and installed manually. Thankfully, the installer is pretty good. You can install the drivers just by executing the downloaded binary. It installs the drivers and configures your xorg.conf file. In order to set up screen resolution and other features visually, there is a tool called nvidia-settings which requires X-Windows to be started up. Once it is installed, simply tell XBMC to use VDPAU and everything will run silky smooth.

<pre><a href="http://www.nvidia.com/object/unix.html">http://www.nvidia.com/object/unix.html</a></pre>

There was another problem which apparently is resolved in the next version of the Linux kernel. The coretemp driver in 2.6.31-14-generic-pae is missing information about Intel Atom processors. This driver is used to read temperature off of the Atom processor. I had to download the source code, manually apply the patch, compile and load. Once compiled and loaded, you can use &#8216;sensors&#8217; program from lm-sensors package to read the temperature. You can download my files as well as precompiled version of coretemp for 32-bit kernel 2.6.31. If you have a later kernel, you probably don&#8217;t need this.

<pre><a href="http://sumit-old.tampahost.net/writings/coretemp-patch-2.6.31.tar.gz">coretemp-patch-2.6.31.tar.gz</a></pre>

There doesn&#8217;t appear to be a way to read the chipset temperature yet. But the nvidia-settings tool and BIOS are both able to display the value using some magic that is not available to the open source world.

We haven&#8217;t discussed how the media center is controlled so far. There are various ways to do it &#8211; an infrared remote, RF remote, iPhone app and web interface. I use my iPhone to control it until I can buy a regular remote. It works really well and allows you to control the system from anywhere (I mean anywhere.. as long as you have 3G or wifi).

Here&#8217;s a video from You Tube until I can make my own. If you haven&#8217;t seen XBMC before, this will give you an idea of how cool it is:



### The Specification List

<table id="htpc">
  <tr>
    <th>
      Parameter
    </th>
    
    <th>
      Value
    </th>
  </tr>
  
  <tr>
    <td>
      CPU
    </td>
    
    <td>
      Intel Atom N230 @1.6GHz
    </td>
  </tr>
  
  <tr>
    <td>
      Chipset
    </td>
    
    <td>
      NVidia ION
    </td>
  </tr>
  
  <tr>
    <td>
      Motherboard
    </td>
    
    <td>
      ZOTAC IONITX-B-E
    </td>
  </tr>
  
  <tr>
    <td>
      Memory
    </td>
    
    <td>
      2 x 1GB DDR2
    </td>
  </tr>
  
  <tr>
    <td>
      Storage
    </td>
    
    <td>
      Western Digital Caviar Green 2TB
    </td>
  </tr>
  
  <tr>
    <td>
      Chassis
    </td>
    
    <td>
      APEX MI-008
    </td>
  </tr>
  
  <tr>
    <td>
      Operating System
    </td>
    
    <td>
      Ubuntu 9.10
    </td>
  </tr>
  
  <tr>
    <td>
      Media Center Software
    </td>
    
    <td>
      XBMC 9.11 beta2
    </td>
  </tr>
  
  <tr>
    <td>
      Observed CPU temp
    </td>
    
    <td>
      55°C &#8211; 72°C &nbsp;(spec. max 95°C)
    </td>
  </tr>
  
  <tr>
    <td>
      Observed HD temp
    </td>
    
    <td>
      41°C &#8211; 45°C &nbsp;(spec. max 60°C)
    </td>
  </tr>
  
  <tr>
    <td>
      Power use
    </td>
    
    <td>
      30W &#8211; 35W
    </td>
  </tr>
</table>

**/proc/cpuinfo**

<pre>processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 28
model name      : Intel(R) Atom(TM) CPU  230   @ 1.60GHz
stepping        : 2
cpu MHz         : 1599.940
cache size      : 512 KB
physical id     : 0
siblings        : 2
core id         : 0
cpu cores       : 1
apicid          : 0
initial apicid  : 0
fdiv_bug        : no
hlt_bug         : no
f00f_bug        : no
coma_bug        : no
fpu             : yes
fpu_exception   : yes
cpuid level     : 10
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat
                  clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe nx lm constant_tsc
                  arch_perfmon pebs bts pni dtes64 monitor ds_cpl tm2 ssse3 cx16 xtpr
                  pdcm movbe lahf_lm
bogomips        : 3199.88
clflush size    : 64
power management:</pre>

**lspci**

<pre>00:00.0 Host bridge: nVidia Corporation MCP79 Host Bridge (rev b1)
00:00.1 RAM memory: nVidia Corporation MCP79 Memory Controller (rev b1)
00:03.0 ISA bridge: nVidia Corporation MCP79 LPC Bridge (rev b2)
00:03.1 RAM memory: nVidia Corporation MCP79 Memory Controller (rev b1)
00:03.2 SMBus: nVidia Corporation MCP79 SMBus (rev b1)
00:03.3 RAM memory: nVidia Corporation MCP79 Memory Controller (rev b1)
00:03.5 Co-processor: nVidia Corporation MCP79 Co-processor (rev b1)
00:04.0 USB Controller: nVidia Corporation MCP79 OHCI USB 1.1 Controller (rev b1)
00:04.1 USB Controller: nVidia Corporation MCP79 EHCI USB 2.0 Controller (rev b1)
00:06.0 USB Controller: nVidia Corporation MCP79 OHCI USB 1.1 Controller (rev b1)
00:06.1 USB Controller: nVidia Corporation MCP79 EHCI USB 2.0 Controller (rev b1)
00:08.0 Audio device: nVidia Corporation MCP79 High Definition Audio (rev b1)
00:09.0 PCI bridge: nVidia Corporation MCP79 PCI Bridge (rev b1)
00:0a.0 Ethernet controller: nVidia Corporation MCP79 Ethernet (rev b1)
00:0b.0 IDE interface: nVidia Corporation MCP79 SATA Controller (rev b1)
00:0c.0 PCI bridge: nVidia Corporation MCP79 PCI Express Bridge (rev b1)
00:10.0 PCI bridge: nVidia Corporation MCP79 PCI Express Bridge (rev b1)
00:15.0 PCI bridge: nVidia Corporation MCP79 PCI Express Bridge (rev b1)
00:16.0 PCI bridge: nVidia Corporation MCP79 PCI Express Bridge (rev b1)
00:17.0 PCI bridge: nVidia Corporation MCP79 PCI Express Bridge (rev b1)
00:18.0 PCI bridge: nVidia Corporation MCP79 PCI Express Bridge (rev b1)
03:00.0 VGA compatible controller: nVidia Corporation ION VGA (rev b1)</pre>

**/proc/scsi/scsi**

<pre>Attached devices:
Host: scsi3 Channel: 00 Id: 00 Lun: 00
  Vendor: ATA      Model: WDC WD20EADS-00R Rev: 01.0
  Type:   Direct-Access                    ANSI  SCSI revision: 05</pre>

**sensors**

<pre>coretemp-isa-0000
Adapter: ISA adapter
Core 0:      +65.0°C  (crit = +95.0°C)

coretemp-isa-0001
Adapter: ISA adapter
Core 1:      +65.0°C  (crit = +95.0°C</pre>

**smartctl -a /dev/sda**

<pre>smartctl version 5.38 [i686-pc-linux-gnu] Copyright (C) 2002-8 Bruce Allen
Home page is http://smartmontools.sourceforge.net/

=== START OF INFORMATION SECTION ===
Device Model:     WDC WD20EADS-00R6B0
Serial Number:    WD-WCAVY0222862
Firmware Version: 01.00A01
User Capacity:    2,000,398,934,016 bytes
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   8
ATA Standard is:  Exact ATA specification draft version not indicated
Local Time is:    Sat Dec 19 18:22:23 2009 EST
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

General SMART Values:
Offline data collection status:  (0x84) Offline data collection activity
                                        was suspended by an interrupting command from host.
                                        Auto Offline Data Collection: Enabled.
Self-test execution status:      (   0) The previous self-test routine completed
                                        without error or no self-test has ever
                                        been run.
Total time to complete Offline
data collection:                 (41580) seconds.
Offline data collection
capabilities:                    (0x7b) SMART execute Offline immediate.
                                        Auto Offline data collection on/off support.
                                        Suspend Offline collection upon new
                                        command.
                                        Offline surface scan supported.
                                        Self-test supported.
                                        Conveyance Self-test supported.
                                        Selective Self-test supported.
SMART capabilities:            (0x0003) Saves SMART data before entering
                                        power-saving mode.
                                        Supports SMART auto save timer.
Error logging capability:        (0x01) Error logging supported.
                                        General Purpose Logging supported.
Short self-test routine
recommended polling time:        (   2) minutes.
Extended self-test routine
recommended polling time:        ( 255) minutes.
Conveyance self-test routine
recommended polling time:        (   5) minutes.
SCT capabilities:              (0x303f) SCT Status supported.
                                        SCT Feature Control supported.
                                        SCT Data Table supported.

SMART Attributes Data Structure revision number: 16
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x002f   200   200   051    Pre-fail  Always       -       0
  3 Spin_Up_Time            0x0027   140   140   021    Pre-fail  Always       -       9975
  4 Start_Stop_Count        0x0032   100   100   000    Old_age   Always       -       16
  5 Reallocated_Sector_Ct   0x0033   200   200   140    Pre-fail  Always       -       0
  7 Seek_Error_Rate         0x002e   200   200   000    Old_age   Always       -       0
  9 Power_On_Hours          0x0032   100   100   000    Old_age   Always       -       256
 10 Spin_Retry_Count        0x0032   100   253   000    Old_age   Always       -       0
 11 Calibration_Retry_Count 0x0032   100   253   000    Old_age   Always       -       0
 12 Power_Cycle_Count       0x0032   100   100   000    Old_age   Always       -       14
192 Power-Off_Retract_Count 0x0032   200   200   000    Old_age   Always       -       4
193 Load_Cycle_Count        0x0032   200   200   000    Old_age   Always       -       11
194 Temperature_Celsius     0x0022   109   103   000    Old_age   Always       -       43
196 Reallocated_Event_Count 0x0032   200   200   000    Old_age   Always       -       0
197 Current_Pending_Sector  0x0032   200   200   000    Old_age   Always       -       0
198 Offline_Uncorrectable   0x0030   100   253   000    Old_age   Offline      -       0
199 UDMA_CRC_Error_Count    0x0032   200   200   000    Old_age   Always       -       0
200 Multi_Zone_Error_Rate   0x0008   100   253   000    Old_age   Offline      -       0</pre>