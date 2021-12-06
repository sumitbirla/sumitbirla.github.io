---
id: 521
title: Minimal Linux
date: 2012-01-16T21:01:44-05:00
author: Sumit Birla
layout: page
guid: http://sumitbirla.com/?page_id=521
---
### Minimal Linux System

The idea of this task was to build a light-weight system that would boot from flash memory and run from RAM. This way, the need for a hard drive is eliminated and you are left with a solid-state computer.

Getting this task done took a lot of effort. I briefly describe how I got [my system](http://sumit-old.tampahost.net/projects/11-28-2002.php) going below. Check out the [references](http://sumit-old.tampahost.net/projects/indashpc/linembed.php#references) if you wish to build your own. Before we get started, we need to know which drives are present on which bus.

> <pre>IDE-0 Master: Hard Disk
        IDE-0 Slave : -
        IDE-1 Master: CompactFlash
        IDE-1 Slave : CDROM</pre>

On a linux system, this means that the hard drive is `/dev/hda` and the CompactFlash is `/dev/hdc`. The flash memory can be used as an IDE drive because it implements the IDE command set on chip. So, a CompactFlash chip is smart where as the SmartMedia memory is dumb!

In order to build a working system, you need the following packages &#8211;

  * The Kernel Source &#8211; I used 2.4.20
  * Busybox &#8211; collection of utilities
  * Lilo &#8211; the boot loader
  * gcc &#8211; for compiling sources

### Step 1: Compiling the kernel

I downloaded the kernel sources from [http://www.kernel.org](http://www.kernel.org/). The package is named linux-2.4.20.tar.gz which is the latest stable release as of this writing. I put the downloaded file in the /usr/src directory and executed the following command to unpack the files &#8211;

> tar xvzf linux-2.4.20.tar.gz

The sources come with programs to help you configure kernel parameters. So, do the following &#8211;

> cd /usr/src/linux-2.4.20/  
> make xconfig

This starts up a GUI application with various buttons. You can select the features you want enabled or disabled. For the sake of simplicity, I compiled all drivers needed by my hardware into the kernel as opposed to compiling them as modules. Disabling all unnecessary support helps speedup the boot process. For instance, if you don&#8217;t have a scsi drive, there is no point in compiling those drivers. My linux setup required me to select the following options (there are other selection too, but there were the ones that I abosultely needed to bootup) &#8211;

  * Processor Type: Cyrix III/VIA C3/VIA C5
  * Networking Options: Unix domain sockets and TCP/IP networking
  * Block devices: RAM disk support, Initial RAM disk support
  * Network Devices: 10/100 Mbps &#8211; VIA Rhine
  * ATA/IDE/MFM/RLL Support: Include IDE/ATA-2 Support
  * File systems: ext2, /dev (automatically mount at boot)

Go through all the options and disable the ones that you don&#8217;t need. Then save and quit. Back on the command prompt, type the following commands:

> make dep  
> make bzImage

If the build is successful, the kernel image is put in the /usr/src/linux-2.4.20/arch/i386/boot directory.

### STEP 2: Build Busybox

Busybox is a multi-call binary. This means that it combines many programs into a single executable. To execute a command, you simply pass the name as a parameter to busybox. For instance, the &#8216;ls&#8217; utlity can be invoked by call &#8216;busybox ls&#8217;.

Busybox provides us with the &#8216;init&#8217; program which is run when the kernel starts up. Along with this, it also provides various other utilities such as ls, mount, ifconfig, df, swapon, swapoff etc. A complete list can be found at [http://www.busybox.net](http://www.busybox.net/). Download the source and unpack in any directory. Edit Configure.h to enable or disable the programs you want to keep. Remember that we want to build a minimal system, so only the utilities that are absolutely essential should be compiled in order to keep the size down. Here is a quick breakdown of the tasks:

  1. Download busybox-0.60.5.tar.gz (or whatever is the latest)
  2. Unpack: tar xvzf busybox-0.60.5.tar.gz
  3. Configure: edit Configure.h
  4. Specify static compile: This is done in the Makefile. I have chosen not to have any external library dependencies. This reduces the number of things that can go wrong. You may choose the other route.

### Step 3: Prepare the destination

My linux system boots from a 32 MB CompactFlash. This memory shows up as a regular hard disk on my development system. I created two partitions on this (/dev/hdc) using fdisk.

  * /dev/hdc1: ext2, 16 MB, used for storing the system files.
  * /dev/hdc2: ext2, 16 MB, used for storing user files.

Once the partitions have been created, ext2 filesystem is created using the following command: make2fs /dev/hdc1

### Step 4: Build the root filesystem

First, our CompactFlash is mounted by executing the command:

mount /dev/hdc1 /mnt

Then create the following directories &#8211;

<pre>drwxr-xr-x    2 root     root         1024 Dec  6 17:13 boot
drwxr-xr-x    2 root     root         1024 Dec  5 19:02 dev
drwxr-xr-x    2 root     root         1024 Dec  7 06:16 etc
drwxr-xr-x    2 root     root         1024 Dec  4 17:13 proc
drwxr-xr-x    2 root     root         1024 Dec  6 17:13 tmp
drwxr-xr-x    2 root     root         1024 Dec  6 18:27 var</pre>

Copy the kernel image that was compiled in STEP 1 to the boot directory. Be careful here and copy to /mnt/boot and not to /boot. Also copy the boot.b from /boot to /mnt/boot.

Populate the dev directory by copying nodes from your development system.

  * cp -dpR /dev/hdc /mnt/dev/
  * cp -dpR /dev/hdc1 /mnt/dev/

In order to add busybox and create links for the utility programs, change directory to where the busybox code resides. Do &#8216;make PREFIX=/mnt install&#8217; This will add the files to your rootsystem which is mounted at /mnt.

Switch to /mnt/etc directory. We need to create an inittab file. The &#8216;init&#8217; program uses this file to figure out what to do. It must contain the following text:

<pre>::sysinit:/etc/rcS
::respawn:/bin/ash
::ctrlaltdel:/bin/umount -a -r</pre>

This tells &#8216;init&#8217; to run /etc/rcS script on startup, start an &#8216;ash&#8217; shell (you may have selected a different shell during the configuration of busybox) and to unmount all when ctrl-alt-del is pressed. The programs that you want run automatically on startup should be put in the rcS script.

/etc/lilo.conf is used by the bootloader to set things up. The listing is shown below:

<pre>boot=/dev/hdc
disk=/dev/hdc
  bios=0x80
map=/boot/map
install=/boot/boot.b
default=linux
linear
prompt
timeout=100

image=/boot/bzImage
        label=linux
        root=/dev/hdc1
        read-only</pre>

The &#8216;bios=0x80&#8217; line tells lilo that when we set the secondary IDE as the boot drive in the BIOS, 0x80 will point to /dev/hdc instead of the usual /dev/hda. It&#8217;s a little confusing, read more about this in the lilo manual.

Now you are ready to install the bootloader on /dev/hdc. Change directory to /mnt/boot and execute the following:

> lilo -r /mnt/ -C etc/lilo.conf

Now pray that everything goes alright! If successful, move on&#8230;

Your filesystem should look something like [this](http://sumit-old.tampahost.net/projects/indashpc/rootfs.txt).

### Step 5: Boot from CompactFlash

You are now ready to reboot. A little extra prayer may help at this point! Go into the BIOS and set the secondary IDE as the first boot disk. Watch and hope that your system boots up. Don&#8217;t give up if it does not work on the first or second try. There is plenty of help available on the internet. On my working system, it takes 5 seconds from the lilo prompt to the command promt. Impressive, isn&#8217;t it? 😉

[Here](http://sumit-old.tampahost.net/projects/indashpc/bootup.txt) is a screenshot of my embedded bootup process.

<a name="references"></a>

### <a name="references"></a>References

<a name="references"></a><a name="references"></a>

  * <a name="references"></a>[The Linux Bootdisk HOWTO](http://www.linux.org/docs/ldp/howto/Bootdisk-HOWTO/)
  * [Using CompactFlash Cards in Your Embedded Linux System](http://www.linuxdevices.com/articles/AT2635434978.html)
  * [Busybox Website](http://www.busybox.net/)