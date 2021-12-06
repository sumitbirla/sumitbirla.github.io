---
id: 559
title: Zoom 5001 PCI Cable Modem
date: 2007-04-22T12:00:01-04:00
author: Sumit Birla
layout: post
guid: http://sumitbirla.com/?p=559
permalink: /2007/04/zoom-5001-pci-cable-modem/
image: /wp-content/uploads/2007/04/zoom5001.jpg
categories:
  - Linux
  - Networking
---
Zoom 5001 is a DOCSIS compliant internal cable modem which is not in production any more. There hasn&#8217;t been a replacement product. It appears as though people prefer the external modem that the cable company provides. I, however, wanted less clutter under my desk and snagged this card off of eBay. I paid $13 for it, and $10 for shipping.

<a href="http://sumit.tampahost.net/2007/04/zoom-5001-pci-cable-modem/zoom5001/" rel="attachment wp-att-562"><img class="aligncenter size-full wp-image-562" title="Zoom 5001 PCI Cable Modem" src="http://sumit.tampahost.net/wp-content/uploads/2007/04/zoom5001.jpg" alt="" width="400" height="300" srcset="https://sumitbirla.me/wp-content/uploads/2007/04/zoom5001.jpg 400w, https://sumitbirla.me/wp-content/uploads/2007/04/zoom5001-300x225.jpg 300w" sizes="(max-width: 400px) 100vw, 400px" /></a>

&nbsp;

On linux, this card is supported by the tulip driver. It appears like any other ethernet card and is configured similarly. In fact, the installation was quite boring until I started digging further into how cable modems work.

<div>
  <pre>Linux Tulip driver version 1.1.13 (May 11, 2002)
eth1: Conexant LANfinity rev 8 at d0030000, 00:40:36:11:C9:3A, IRQ 21.

[root@europa ~]# ifconfig eth1
eth1      Link encap:Ethernet  HWaddr 00:40:36:11:C9:3A
          inet addr:67.8.X.X  Bcast:255.255.255.255  Mask:255.255.224.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:6334391 errors:0 dropped:0 overruns:0 frame:0
          TX packets:15 errors:72144 dropped:0 overruns:0 carrier:46325
          collisions:0 txqueuelen:1000
          RX bytes:578863979 (552.0 MiB)  TX bytes:2458 (2.4 KiB)
          Interrupt:21</pre>
</div>

A couple of things to note above. The _TX packets_ and _errors_ counts appear to be swapped. I am not sure whether this is a driver issue or something else. Also the _carrier_ count is high, but I do not notice any connectivity problems.

On the box and on the card itself, there are two MAC addresses printed. One is labelled the PC-MAC and the other is C-MAC. This implies that there are two network chips on the card. One which the PC sees and is supported by Tulip driver (eth1 in my case), and one interface which faces the cable network. There must be a bridge between the two interfaces which is transparent to the operating system. This turned out to be true as demonstrated below.

<p style="text-align: center;">
  <img class="aligncenter" src="http://sumit-old.tampahost.net/images/zoom5001_logic.png" alt="Logical diagram of the modem" width="400" height="365" />
</p>

I setup an alias ethernet interface called eth1:0 and gave it the IP address of 192.168.100.2. From my past experience, I had known that cable modems generally have an IP address of 192.168.100.1 which displays some diagnostic information when viewed via a browser. With this new interface, I was able to ping the second ethernet chip on the card. Had I not done this, packets destined for 192.168.100.1 would have been send to the default gateway out on the cable network and never reached anywhere.

<div>
  <pre>[root@europa ~]# ifconfig eth1:0 192.168.100.2 netmask 255.255.255.0
[root@europa ~]# ifconfig eth1:0
eth1:0    Link encap:Ethernet  HWaddr 00:40:36:11:C9:3A
          inet addr:192.168.100.2  Bcast:192.168.100.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          Interrupt:21

[root@europa ~]# ping 192.168.100.1
PING 192.168.100.1 (192.168.100.1) 56(84) bytes of data.
64 bytes from 192.168.100.1: icmp_seq=1 ttl=64 time=2.47 ms
64 bytes from 192.168.100.1: icmp_seq=2 ttl=64 time=0.848 ms
64 bytes from 192.168.100.1: icmp_seq=3 ttl=64 time=0.840 ms

--- 192.168.100.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
rtt min/avg/max/mdev = 0.840/1.386/2.472/0.768 ms</pre>
</div>

The next step was to determine if I can query some information from this cable-network facing interface. For this I used NMAP &#8211;

<div>
  <pre>[root@europa ~]# nmap 192.168.100.1

Starting Nmap 4.03 ( http://www.insecure.org/nmap/ ) at 2007-04-22 10:57 EDT
Interesting ports on 192.168.100.1:
(The 1671 ports scanned but not shown below are in state: closed)
PORT    STATE    SERVICE
80/tcp  open     http
161/tcp filtered snmp
162/tcp filtered snmptrap
MAC Address: 00:40:36:11:C9:37 (Tribe Computer Works)

Nmap finished: 1 IP address (1 host up) scanned in 3.948 seconds</pre>
</div>

As you can see, HTTP and SNMP services are running on this interface. But they don&#8217;t return any data. As it turns out, Roadrunner sets a secret community string when the modem comes online so users cannot access it. Not even read-only access. As for the web interface, it does not return any data. It simply closes connection. This is a mystery to me.

<div>
  <pre>[root@europa ~]# wget http://192.168.100.1
--11:08:41--  http://192.168.100.1/
           =&gt; `index.html'
Connecting to 192.168.100.1:80... connected.
HTTP request sent, awaiting response... No data received.

[root@europa ~]# snmpwalk 192.168.100.1 public
snmpwalk: Timeout (Sub-id not found: iso -&gt;&gt; public)</pre>
  
  <div>
  </div>
</div>