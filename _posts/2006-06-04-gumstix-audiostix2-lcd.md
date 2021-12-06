---
id: 567
title: Gumstix/Audiostix2 interface to LCD
date: 2006-06-04T12:00:16-04:00
author: Sumit Birla
layout: post
guid: http://sumitbirla.com/?p=567
permalink: /2006/06/gumstix-audiostix2-lcd/
image: /wp-content/uploads/2006/06/Gpstix.jpg
categories:
  - Linux
---
<a href="http://sumit.tampahost.net/2006/06/gumstix-audiostix2-lcd/gpstix/" rel="attachment wp-att-571"><img class="alignright size-medium wp-image-571" title="Gpstix" src="http://sumit.tampahost.net/wp-content/uploads/2006/06/Gpstix-168x300.jpg" alt="" width="168" height="300" srcset="https://sumitbirla.me/wp-content/uploads/2006/06/Gpstix-168x300.jpg 168w, https://sumitbirla.me/wp-content/uploads/2006/06/Gpstix.jpg 488w" sizes="(max-width: 168px) 100vw, 168px" /></a>[Gumstix](http://gumstix.com/) is a very neat, hobbyist-friendly embedded computer which runs Linux. It is about the size of a stick of gum (hence the name) and has a strong community support system. Various breakout and add-on boards are available making the platform versatile. I had intended to use Gumstix in my [CarPC project](http://sumit.tampahost.net/projects/projectsindash-pc/ "Indash PC"), but I ended up spending most of my time getting the LCD to interface directly with PXA LCD driver. I have listed my findings below for the benefit of others.

### Audiostix2 to LQ050Q5DR01 interface

The following table shows the connections between the Audiostix2 board and the LCD. Since the Gumstix outputs a 16-bit color (RGB565) and the LCD expects 18-bit color, we simply ground the least significant bits (LSB) of red and blue. Note that the BIAS signal is not needed for this LCD and has been left open.

<div>
  <pre>GPSstix        Sharp LQ050Q5DR01      My understanding
--------------------------------------------------------------
LCLK            PIN_3   Hsync         signals the end of line
PCLK            PIN_13  CLK           pixel clock
FCLK            PIN_28  Vsync         signals the end of a frame
LDD15           PIN_29  R5            RED_4 - Most significant bit
LDD14           PIN_27  R4            RED_3
LDD13           PIN_25  R3            RED_2
LDD12           PIN_21  R2            RED_1
LDD11           PIN_19  R1            RED_0
LDD10           PIN_8   G5            GREEN_5 - Most significant bit
LDD9            PIN_6   G4            GREEN_4
LDD8            PIN_4   G3            GREEN_3
LDD7            PIN_37  G2            GREEN_2
LDD6            PIN_35  G1            GREEN_1
LDD5            PIN_33  G0            GREEN_0
LDD4            PIN_24  B5            BLUE_4 - Most significant bit
LDD3            PIN_22  B4            BLUE_3
LDD2            PIN_20  B3            BLUE_2
LDD1            PIN_16  B2            BLUE_1
LDD0            PIN_14  B1            BLUE_0
BIAS            -                     Not used
-               PIN_1   GND
-               PIN_2   VCC           +3.3V
-               PIN_5   T0            Leave open (thermistor output)
-               PIN_7   T1            Leave open (thermistor output)
-               PIN_9   HVR           GND (Horiz. of vertical scan direction)
-               PIN_10  GND
-               PIN_11  GND
-               PIN_12  B0            GND (LSB of blue is unused)
-               PIN_15  GND
-               PIN_17  R0            GND (LSB of red is unused)
-               PIN_18  GND
-               PIN_23  GND
-               PIN_26  GND
-               PIN_30  TEST          Leave open
-               PIN_31  GND
-               PIN_32  TEST          Leave open
-               PIN_34  TEST          Leave open
-               PIN_36  TEST          Leave open
-               PIN_38  ENAB          GND (signal to settle horiz.)
-               PIN_39  VCC           +3.3V
-               PIN_40  GND</pre>
</div>

### PXA Register Setup for LQ050Q5DR01

<div>
  <pre>LCD Controller Control Register 0 (7-23)
LCCR0                    0x003008f9  00000000 00110000 00001000 11111001
LCCR0_ENB                         1  LCD controller enable
LCCR0_CMS                         0  LCD monochrome operation enable
LCCR0_SDS                         0  LCD dual panel display enable
LCCR0_LDM                         1  LCD disable done IRQ disable
LCCR0_SFM                         1  LCD start of frame IRQ disable
LCCR0_IUM                         1  LCD fifo underrun error IRQ disable
LCCR0_EFM                         1  LCD end of frame IRQ disable
LCCR0_PAS                         1  LCD active display enable
LCCR0_DPD                         0  LCD send 8 pixel on L_DD[7:0] at each clock
LCCR0_DIS                         0  LCD controller disable
LCCR0_QDM                         1  LCD quick disable IRQ disable
LCCR0_PDD                         0  LCD palette DMA request delay
LCCR0_BM                          1  LCD branch start IRQ disable
LCCR0_OUM                         1  LCD fifo underrun IRQ disable

LCD Controller Control Register 1 (7-26)
LCCR1                    0x46ff2d3f  01000110 11111111 00101101 00111111
LCCR1_PPL                       319  LCD pixels per line (+1)
LCCR1_HSW                        11  LCD horizontal sync pulse width (+1)
LCCR1_ELW                       255  LCD end of line pixel clock wait count (+1)
LCCR1_BLW                        70  LCD beginning of line pixel clock wait count (+1)

LCD Controller Control Register 2 (7-28)
LCCR2                    0x0500fcef  00000101 00000000 11111100 11101111
LCCR2_LPP                       239  LCD lines per panel (+1)
LCCR2_VSW                        63  LCD vertical sync pulse width (+1)
LCCR2_EFW                         0  LCD end of frame line clock wait count (+1)
LCCR2_BFW                         5  LCD beginning of frame line clock wait count (+1)

LCD Controller Control Register 3 (7-31)
LCCR3                    0x04300007  00000100 00110000 00000000 00000111
LCCR3_PCD                         7  LCD pixel clock divisor (+1)
LCCR3_ACB                         0  LCD AC bias pin frequency (+1)
LCCR3_API                         0  LCD AC bias pin transitions per interrupt
LCCR3_VSP                         1  LCD L_FCLK vertical sync polarity active low
LCCR3_HSP                         1  LCD L_LCLK horizontal sync polarity active low
LCCR3_PCP                         0  LCD data sampled on falling edge of L_PCLK
LCCR3_OEP                         0  LCD L_BIAS output enable active low
LCCR3_BPP                        16  LCD bits per pixel
LCCR3_DPC                         0  LCD double pixel clock rate at L_PCLK</pre>
</div>

### Sample Framebuffer Code

The C code below can be used to check the dimensions & depth of your framebuffer device. Additionally, it prints a couple of tests patterns to screen. If you happen to have a 16-bit display and have downloaded the following raw image, then you will also see a picture being printed to screen.

<pre><a href="http://sumit-old.tampahost.net/images/taj.raw">/images/taj.raw</a></pre>

<div>
  <img class="aligncenter" src="http://sumit-old.tampahost.net/writings/images/Gumstix_lcd.jpg" alt="Gumstix connected to LCD" width="500" height="333" />
</div>

<div>
  <pre>#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;
#include &lt;string.h&gt;
#include &lt;fcntl.h&gt;
#include &lt;linux/fb.h&gt;
#include &lt;sys/mman.h&gt;

#define RGB565(r,g,b) (((r &gt;&gt; 3) & 31) &lt;&lt; 11) | (((g &gt;&gt; 2) & 63) &lt;&lt; 5) | ((b &gt;&gt; 3) & 31)

/* structures for retrieving framebuffer information */
struct fb_var_screeninfo vinfo;
struct fb_fix_screeninfo finfo;

/* size of video memory in bytes */
long int screensize;

/* pointer to memory mapped framebuffer */
char *fbp;

/*
 * Function to set a pixel in framebuffer with the specified
 * (r,g,b). Works only with 32bpp and 16bpp modes.
 */
void set_pixel(int x, int y, unsigned char r, unsigned char g, unsigned char b)
{
    long int location;
    location = (x+vinfo.xoffset) * (vinfo.bits_per_pixel/8) +
                       (y+vinfo.yoffset) * finfo.line_length;

    if ( vinfo.bits_per_pixel == 32 ) {
        *(fbp + location) = b;
        *(fbp + location + 1) = g;
        *(fbp + location + 2) = r;
        *(fbp + location + 3) = 0;
    } else  { //assume 16bpp
        *((unsigned short int*)(fbp + location)) = RGB565(r,g,b);
    }
}

/*
 * Program entry point. Opens /dev/fb0 and prints two patterns
 * to it.
 */
int main()
{
    int fd, fbfd, x, y;
    unsigned char r, g, b;

    /* Open the file for reading and writing */
    fbfd = open("/dev/fb0", O_RDWR);
    if (fbfd == -1) {
        perror("/dev/fb0");
        exit(1);
    }

    /* Get fixed screen information */
    if (ioctl(fbfd, FBIOGET_FSCREENINFO, &finfo)) {
        perror("FBIOGET_FSCREENINFO");
        exit(2);
    }

    /*  Get variable screen information */
    if (ioctl(fbfd, FBIOGET_VSCREENINFO, &vinfo)) {
        perror("FBIOGET_VSCREENINFO");;
        exit(3);
    }

    printf("%dx%d, %dbpp\n", vinfo.xres, vinfo.yres, vinfo.bits_per_pixel );

    /* Figure out the size of the video memory in bytes */
    screensize = vinfo.xres * vinfo.yres * vinfo.bits_per_pixel / 8;

    /* Map the device to memory */
    fbp = (char *)mmap(0, screensize, PROT_READ | PROT_WRITE, MAP_SHARED,
                       fbfd, 0);
    if ((int)fbp == -1) {
        perror("mmap()");
        exit(4);
    }

    printf("Turning screen red...\n");
    for ( y = 20; y &lt; vinfo.yres-20; y++ )
        for ( x = 20; x &lt; vinfo.xres-20; x++ )
            set_pixel(x, y, 255, 0, 0);

    sleep(2);
    printf("Turning screen green...\n");
    for ( y = 20; y &lt; vinfo.yres-20; y++ )
        for ( x = 20; x &lt; vinfo.xres-20; x++ )
            set_pixel(x, y, 0, 255, 0);

    sleep(2);
    printf("Turning screen blue...\n");
    for ( y = 20; y &lt; vinfo.yres-20; y++ )
        for ( x = 20; x &lt; vinfo.xres-20; x++ )
            set_pixel(x, y, 0, 0, 255);

    /* PATTERN #1: something funky */
    sleep(2);
    printf("Printing pattern #1...\n");

    for ( y = 0; y &lt; vinfo.yres; y++ )
        for ( x = 0; x &lt; vinfo.xres; x++ )
            set_pixel(x, y, 31-(y-100)/16, (x-100)/6, 10);

    sleep(2);

    /* Blank the screen */
    memset(fbp, '\0', screensize);

    /* PATTERN #2: vertical and horizontal lines at 1/3 and 2/3 */
    printf("Printing pattern #2...\n");
      for ( y = 0; y &lt; vinfo.yres; y++ ) {
        set_pixel(2*vinfo.xres/3, y, 255, 255, 255);
        set_pixel(vinfo.xres/3, y, 255, 255, 255);
      }

    for ( x = 0; x &lt; vinfo.xres; x++ ) {
        set_pixel(x, 2*vinfo.yres/3, 255, 255, 255);
        set_pixel(x, vinfo.yres/3, 255, 255, 255);
    }

    sleep(2);

    printf("Printing pictures...\n");
    fd = open("taj.raw", O_RDONLY);

    if (fd == -1)
        perror("taj.raw");
    else
    {
        for ( y = 0; y &lt; 213; y++ )
            for ( x = 0; x &lt; 320; x++ )
            {
                read(fd, &r, 1);
                read(fd, &g, 1);
                read(fd, &b, 1);
                set_pixel(x, y, r, g, b);
            }

        close(fd);
    }

    printf("Done\n");
    munmap(fbp, screensize);
    close(fbfd);
    return 0;
}</pre>
</div>