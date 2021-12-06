---
id: 1211
title: My Pool Control Panel
date: 2020-01-17T11:30:23-05:00
author: Sumit Birla
layout: post
guid: https://sumitbirla.me/?p=1211
permalink: /2020/01/my-pool-control-panel/
image: /wp-content/uploads/2020/01/IMG_2331.jpg
categories:
  - Home Automation
  - Swimming Pool
---
This blog post describes my PLC-based pool controller panel.&nbsp; It is a complete replacement of the existing electrical panel.&nbsp; &nbsp;I built this as a learning-experience project in Industrial Automation techniques.&nbsp; &nbsp;I have used a PLC and other DIN-rail mounted components such as contactors and circuit-breakers to achieve the same functionality as the existing panel.&nbsp; Furthermore because I am using a network-connected PLC,&nbsp; I can remotely control and monitor the status of my pool pump.<figure class="wp-block-table aligncenter is-style-stripes">

<table class="">
  <tr>
    <td>
      <img class="wp-image-1231" style="width: 300px;" src="https://sumitbirla.me/wp-content/uploads/2020/01/IMG_2229.jpg" alt="" />
    </td>
    
    <td>
      →
    </td>
    
    <td>
      <img class="wp-image-1232" style="width: 300px;" src="https://sumitbirla.me/wp-content/uploads/2020/01/IMG_2331.jpg" alt="" />
    </td>
  </tr>
  
  <tr>
    <td>
      Existing panel with mechanical timer
    </td>
    
    <td>
    </td>
    
    <td>
      New PLC based control panel
    </td>
  </tr>
</table></figure> 

As you can see above, the new panel looks quite a bit different. But many of the components are the same. For example, the circuit-breakers for the pool pump (16A), heater (50A) and one for an electrical outlet (16A) mirror the old installation. Instead of a mechanical timer, I now use a PLC. The benefit of using a PLC (other than being just plain fun to learn to use) is that I can set a separate pump schedule for each day of the week. Because the PLC has an ethernet connection, I can add the control panel to my larger home automation network and turn pump on/off remotely. At some point in the future, I plan to add sensors to read pH and water temperature since the PLC already has inputs to handle these.

<hr class="wp-block-separator" />

### Bill of Materials

<table class="">
  <tr>
    <th>
      Item
    </th>
    
    <th>
      Qty
    </th>
    
    <th>
      Price
    </th>
    
    <th>
      Source
    </th>
  </tr>
  
  <tr>
    <td>
      Siemens LOGO! 12/24 RCE
    </td>
    
    <td>
      1
    </td>
    
    <td>
      $70
    </td>
    
    <td>
      AliExpress
    </td>
  </tr>
  
  <tr>
    <td>
      Altelix NEMA Enclosure 17x14x6
    </td>
    
    <td>
      1
    </td>
    
    <td>
      $80
    </td>
    
    <td>
      Amazon
    </td>
  </tr>
  
  <tr>
    <td>
      Mean Well DR-15-24 Power Supply, 24 Volt, 0.63 Amp
    </td>
    
    <td>
      1
    </td>
    
    <td>
      $13
    </td>
    
    <td>
      Amazon
    </td>
  </tr>
  
  <tr>
    <td>
      Baomain AC Contactor HC1-63 110V 63A 2 Pole
    </td>
    
    <td>
      1
    </td>
    
    <td>
      $17
    </td>
    
    <td>
      Amazon
    </td>
  </tr>
  
  <tr>
    <td>
      uxcell 1 Pole 50A 230/400V Miniature Circuit Breaker
    </td>
    
    <td>
      1
    </td>
    
    <td>
      $9
    </td>
    
    <td>
      Amazon
    </td>
  </tr>
  
  <tr>
    <td>
      uxcell 1 Pole 16A 230/400V Miniature Circuit Breaker
    </td>
    
    <td>
      2
    </td>
    
    <td>
      $9
    </td>
    
    <td>
      Amazon
    </td>
  </tr>
  
  <tr>
    <td>
      DIN Rail Terminal Blocks, 6-20 AWG, 60 Amp
    </td>
    
    <td>
      1
    </td>
    
    <td>
      $30
    </td>
    
    <td>
      Amazon
    </td>
  </tr>
  
  <tr>
    <td>
      CHINT NP9 push button switch card DIN rail button (red & green)
    </td>
    
    <td>
      2
    </td>
    
    <td>
      $5
    </td>
    
    <td>
      AliExpress
    </td>
  </tr>
  
  <tr>
    <td>
      12 Gauge Silicone wire 10 ft red and 10 ft black
    </td>
    
    <td>
      1
    </td>
    
    <td>
      $11
    </td>
    
    <td>
      Amazon
    </td>
  </tr>
  
  <tr>
    <td>
      Ferrule Crimping Tool Kit
    </td>
    
    <td>
      1
    </td>
    
    <td>
      $26
    </td>
    
    <td>
      Amazon
    </td>
  </tr>
  
  <tr>
    <td>
      DIN Rail 250mm*35mm*7.5mm Steel (10-Pak)
    </td>
    
    <td>
      1
    </td>
    
    <td>
      $14
    </td>
    
    <td>
      eBay
    </td>
  </tr>
</table>

<p class="has-text-align-right">
  <strong>Total (approx.): USD 300</strong>
</p>

### Programming

Although Ladder Logic is the most common way to program in the automation world, I used Function Block Diagram (FBD) just because there seems to be more support online for this method of programming for the Siemens LOGO!<figure class="wp-block-image size-large">

<img src="https://sumitbirla.me/wp-content/uploads/2020/01/PoolPumpSoftware.png" alt="" class="wp-image-1185" /> </figure> 

  * [Download](https://sumitbirla.me/wp-content/uploads/2020/08/Pool-Pump-Control.lsc) the program (open using LOGO! Soft Comfort)
  * <a rel="noreferrer noopener" aria-label="PDF Version (opens in a new tab)" href="https://sumitbirla.me/wp-content/uploads/2020/01/Pool-Pump-Control.pdf" target="_blank"><u>PDF Version</u></a> &#8211; for viewing if you don&#8217;t have LOGO software

<hr class="wp-block-separator" />

#### DEMO:<figure class="wp-block-embed-youtube wp-block-embed is-type-video is-provider-youtube wp-embed-aspect-16-9 wp-has-aspect-ratio">

<div class="wp-block-embed__wrapper">
</div></figure>