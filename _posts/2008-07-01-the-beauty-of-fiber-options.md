---
id: 54
title: The beauty of fiber options
date: 2008-07-01T12:00:26-04:00
author: Sumit Birla
layout: post
guid: http://sumitbirla.com/?p=54
permalink: /2008/07/the-beauty-of-fiber-options/
image: /wp-content/uploads/2008/07/fiber-optics.jpg
categories:
  - Networking
  - Technology
---
<div>
  <p>
    After much waiting, Verizon finally decided to lay down fiber optic cables in my neighborhood. Last night, the final connection was made when my FiOS service was activated. I have 10Mbps/2Mbps service. It appears to be working well so far. Ping times are much lower than cable modem. Some pictures of the equipment installed are below.
  </p>
  
  <p>
    I asked the installer to run a cat5e cable to my wiring closet. So now everything is neat and clean inside the house. No modems, no routers, no extra power and ethernet cables. Consequently, the power consumption in my closet has dropped from 94W to 77W. The only gotcha was that the installer is required to install and test with Verizon provided router. After he left, I quickly removed the router and connected the cat5e directly to my server. But the ONT installed outside remembered the router and wouldn&#8217;t let my server have an IP address. So I masqueraded as the router by changing the MAC address on the server. And voila, I was online.
  </p>
  
  <p style="text-align: center;">
    <a href="http://sumit-old.tampahost.net/photo/display.php?img=/images/fios1.jpg"><img src="http://sumit-old.tampahost.net/images/fios1_sm.jpg" alt="ONT installed outside the house" /> </a><a href="http://sumit-old.tampahost.net/photo/display.php?img=/images/fios2.jpg"><img src="http://sumit-old.tampahost.net/images/fios2_sm.jpg" alt="Backup power supply inside the garage" /></a>
  </p>
  
  <pre>[sbirla@europa ~]$ ping google.com
PING google.com (64.233.187.99) 56(84) bytes of data.
64 bytes from jc-in-f99.google.com (64.233.187.99): icmp_seq=1 ttl=244 time=22.9 ms
64 bytes from jc-in-f99.google.com (64.233.187.99): icmp_seq=2 ttl=244 time=22.4 ms
64 bytes from jc-in-f99.google.com (64.233.187.99): icmp_seq=3 ttl=244 time=22.2 ms
64 bytes from jc-in-f99.google.com (64.233.187.99): icmp_seq=4 ttl=244 time=23.0 ms</pre>
</div>