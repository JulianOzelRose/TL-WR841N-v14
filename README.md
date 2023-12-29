# TL-WR841N-v14
The TP-Link TL-WR841N is a relatively inexpensive 300mbps WiFi router. This makes it a perfect device for reverse engineering and hardware hacking projects. My goal here was to gain shell access via the router's UART debugging port and see what can be done from there. Please note that this guide is specific to v14.6 of the router — some details may be slightly different for earlier or later hardware versions. Included in this repo is a partial file system dump of the router, as well as a firmware dump.

![IMG_9762](https://user-images.githubusercontent.com/95890436/208492928-3936bd76-56dc-43e2-8519-299447747682.jpg)

## Tools
#### Hardware
- CH340G
- CH341A
- Digital multimeter
- Header pins
- Jumper wires 
- Soldering kit
#### Software
- AsProgrammer
- PuTTy
- Tftpd64

## The serial UART port
The UART serial port is primarily used during the manufacturing process for debugging purposes. You can find UART ports on virtually any IoT device. With the proper tools, it is possible to access the shell using this port. From there, you can dump the firmware, flash hacked firmware, get device info, give yourself remote/backdoor access, and more. The UART pin layout is: ```GND```, ```TX```, ```RX```, ```VCC```.

![photo_2022-12-16_14-12-06 (5)](https://user-images.githubusercontent.com/95890436/208197839-11598118-c562-45e7-9051-d94d1c914e86.jpg)
## Testing UART port voltages
To avoid shorting both the TL-WR841N router and the serial USB device, we must first confirm that we are using the correct voltages. This is an important step, as using the wrong voltage can damage the router, the serial USB device, or both. UART ports are typically either 3.3V, or 5V. Thankfully, this particular version of the router has pre-labeled pins — so there is no ambiguity over which pin is ```RX```, ```TX```, ```GND```, or ```VCC```.

Using a multimeter, I tested and confirmed a 3.3V ```VCC```, ```GND```, and ```TX``` pins. The ```RX``` pin reads at 0.0V, because it is essentially a listening port, just waiting for information. We can safely assume ```RX``` receives at 3.3V, just like the rest of the pins. Once the voltage is confirmed to be 3.3V, the next step is to connect the 3.3V serial USB converter.

## Connecting UART to serial USB converter
After confirming that the UART port works at 3.3V and soldering a set of header pins, we can safely connect our USB serial converter. Next we use a serial communication program to interface with the device and read the output. For this step, we connect ```GND-GND```, ```RX-TX```, and ```TX-RX```. It is not necessary to connect ```VCC```, so long as we set our serial USB device to use 3.3V.

![208198729-9004ca52-7f22-4dff-9c04-627a84ff9245](https://user-images.githubusercontent.com/95890436/208494179-916da9d5-439a-450f-9ddd-7170eec90d49.jpg)


## Obtaining shell access
Once we have secured a connection between the serial USB converter to the router's UART port, we can now modify the settings in PuTTy for serial communication. The main value we need to know is the UART port's baud rate. It is possible to use a logic analyzer to figure out what the baud rate is. However, there are several baud rates that are very common, so it is often easier to guess common values until one works. In this particular case, the router's UART runs at 115200 baud rate. Other baud rates will produce garbage on the screen. So we will use the following settings in PuTTy:
- **Baud rate:** 115200
- **Data bits:** 8
- **Stop bits:** 1
- **Parity:** None
- **Flow control:** None

After gaining access to the shell, I used the Linux command ```echo $USER``` to confirm root access of the device. We know we have achieved this, as the device returns ```root```.

https://github.com/JulianOzelRose/TL-WR841N-v14/assets/95890436/ef8f553b-256d-4cd7-80a0-edb2c1f17edf


## Bypassing write protection (read only console)
At first I was only able to access the shell, but not issue commands — I was stuck in a read-only console. I learned that this is because TP-Link shorts the ```RX``` pin of the UART port as a security measure. There is a single voltage-limiting resistor just next to the ```RX``` pin on this particular model. Pulling that resistor out with a pair of tweezers did the trick, and I was then able to write to the console and issue commands. Take note of the missing resistor at ```R18``` just above the ```RX``` pin.

![IMG_9832 - Copy](https://user-images.githubusercontent.com/95890436/208493479-fe79a047-e249-4d27-b693-82f5032896c4.jpg)

## Extracting the file system
To extract the router's file system, we will archive each directory, then upload to our PC using TFTP. Using the ```busybox``` command, we can see that the router uses a BusyBox with limited functionality.
While it has the ```tftp``` function which will be useful for uploading files from the router, there are no commands available to us to archive directories/files.
To be able to archive directories and upload them, we will need to first upload a BusyBox binary with more functions defined. You can find a link to the one I used [here](https://busybox.net/downloads/binaries/1.21.1/).
```
~ # busybox
BusyBox v1.19.2 (2022-08-16 12:08:08 CST) multi-call binary.
Copyright (C) 1998-2011 Erik Andersen, Rob Landley, Denys Vlasenko
and others. Licensed under GPLv2.
See source distribution for full notice.

Usage: busybox [function] [arguments]...
   or: busybox --list[-full]
   or: function [arguments]...

        BusyBox is a multi-call binary that combines many common Unix
        utilities into a single executable.  Most people will create a
        link to busybox for each function they wish to use and BusyBox
        will act like whatever it was invoked as.

Currently defined functions:
        arping, ash, brctl, cat, chmod, cp, date, df, echo, free, getty, halt,
        ifconfig, init, insmod, ipcrm, ipcs, kill, killall, linuxrc, login, ls,
        lsmod, mkdir, mount, netstat, pidof, ping, ping6, poweroff, ps, reboot,
        rm, rmmod, route, sh, sleep, taskset, tftp, top, umount, vconfig
```

Using the ```mount``` command, we can see that most of the router's file system is read only. While ```rootfs```, ```/proc/``` and ```/sys/``` are technically read-write,
Linux typically restricts any write operations to these directories for security reasons. The only location we can reliably write to is the ```/var/``` directory.
```
~ # mount
rootfs on / type rootfs (rw)
/dev/root on / type squashfs (ro,relatime)
proc on /proc type proc (rw,relatime)
ramfs on /var type ramfs (rw,relatime)
/sys on /sys type sysfs (rw,relatime)
```
To upload our new BusyBox, we connect our router to our computer, either with a Cat 5 cable or wirelessly. Using Tftp64, we prepare our new BusyBox binary for upload.
We then ```cd``` over to the ```var/tmp``` directory, and use ```tftp``` to download the new binary, change its access permissions with ```chmod```, and then execute.

https://github.com/JulianOzelRose/TL-WR841N-v14/assets/95890436/89f00c3e-eba8-4407-bf50-8c04f3d2f9a1

With our new BusyBox binary, we can now archive directories. Using ```ls``` command from the root directory, we can see there are a total of 11 directories, as ```linuxrc``` is a script.
So to extract the file system, we just need to archive each individual directory, and upload it via TFTP to our PC.

```
~ # ls
web      usr      sbin     mnt      lib      dev
var      sys      proc     linuxrc  etc      bin
```

https://github.com/JulianOzelRose/TL-WR841N-v14/assets/95890436/3f3a8536-2ee8-47a4-9731-62489bceeb0a

With ```/web/``` archived and uploaded, we just have 10 directories left. It should be noted that some of these directories, particularly ```/dev/```, ```/proc/```, and ```/sys/``` are
system-reserved directories that contain mostly symbolic links and other files which cannot be properly archived. In any case, we should be able to extract most of the file system using these methods.

## Extracting the firmware
With the file system successfully extracted, the next logical step is to extract the firmware. Upon closer inspection
of the board, it appears to use the cFeon 25QH32 series IC chip for storing its firmware. With the CH341A
programmer, there are two ways to go about extracting the firmware from the IC chip:

1. Place the programmer's SOP clips directly on the chip.
2. Desolder the chip, and resolder it onto the programmer's SPI board.

I had trouble getting a read on the IC chip using the first method. My hypothesis is that with this particular
version of the board, reading the firmware with the SOP clips also powers the microprocessor, which interferes
with the programmer's ability to get a proper read. Instead, I used the second method and desoldered the chip
from the router, and resoldered it directly onto my programmer's SPI board.

#### Firmware chip soldered onto CH341A's SPI board
![Firmware-IC-CH341A](https://github.com/JulianOzelRose/TL-WR841N-v14/assets/95890436/b81673fb-30d7-4051-a611-71ee7834658c)

The SPI programmer was then able to extract the firmware with no issues. AsProgrammer does a good job at auto-detecting the firmware chip ID,
so all that needed to be done was click 'Read IC', and voila — it worked. Now here is what that firmware dump looks like from a hex editor.
Notice the highlighted strings from the bootloader.

#### Firmware dump viewed from a hex editor
![Firmware-HxD](https://github.com/JulianOzelRose/TL-WR841N-v14/assets/95890436/ddb2f801-cb34-44ff-b928-49e70ca24c60)

## Note about CH341A and 3.3V chips
It should also be noted that the firmware chip on this board operates at 3.3V, and there is a known design flaw with
the CH341A programmer where the data lines operate at 5V, regardless of how the programmer is configured. There are plenty of
tutorials online on how to mod the CH341A to work at 3.3V, which I recommend doing to avoid shorting your board's firmware chip.
You can find one such tutorial [here](https://www.youtube.com/watch?v=HwnzzF645hA).

## Sources
I relied on the following sources to guide me throughout this project. These resources provide more in-depth explanations and insights into the hardware reversing process:

- [Zero Day Initiative - Hardware Reversing with the TP-Link TL-WR841N Router](https://www.zerodayinitiative.com/blog/2019/9/2/mindshare-hardware-reversing-with-the-tp-link-tl-wr841n-router)
- [River Loop Security - Hardware Hacking 101: Getting a root shell via UART](https://www.riverloopsecurity.com/blog/2020/01/hw-101-uart/)
- [Hackaday - Dissecting The TL-WR841N For Fun And Profit](https://hackaday.com/2019/09/12/dissecting-the-tl-wr841n-for-fun-and-profit/)
- [OpenWrt - TP-Link TL-WR841ND](https://openwrt.org/toh/tp-link/tl-wr841nd)

These guides were instrumental in helping me understand the various aspects of reverse engineering, hardware hacking, and working with router firmware.
I recommend checking them out for a deeper understanding of the processes involved.
