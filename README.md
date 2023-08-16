# TL-WR841N-v14
The TP-Link TL-WR841N is a relatively inexpensive 300mbps WiFi router. This makes it a perfect device for reverse engineering and hardware hacking projects. My goal here was to gain shell access via the router's UART debugging port and see what can be done from there. Please note that this guide is specific to v14.6 of the router -- some details may be slightly different for earlier or later hardware versions. Included in this repo is a partial file system dump of the router.

![IMG_9762](https://user-images.githubusercontent.com/95890436/208492928-3936bd76-56dc-43e2-8519-299447747682.jpg)


## Tools used
#### Hardware
- MM300 Digital Multimeter 
- CH340G USB TTL UART Serial Converter 
- Soldering kit from Pulsivo
- Jumper cables
- Header pins
#### Software
- PuTTy

## The serial UART port
The UART serial port is primarily used during the manufacturing process for debugging purposes. You can find UART ports on virtually any IoT device. With the proper tools, it is possible to access the shell using this port. From there, you can dump the firmware, flash hacked firmware, get device info, give yourself remote/backdoor access, and more. The UART pin layout is: ```GND```, ```TX```, ```RX```, ```VCC```.

![photo_2022-12-16_14-12-06 (5)](https://user-images.githubusercontent.com/95890436/208197839-11598118-c562-45e7-9051-d94d1c914e86.jpg)
## Testing UART port voltages
To avoid shorting both the TL-WR841N router and the serial USB device, we must first confirm that we are using the correct voltages. This is an important step, as using the wrong voltage can damage the router, the serial USB device, or both. UART ports are typically either 3.3V, or 5V. Thankfully, this particular version of the router has pre-labled pins -- so there is no ambiguity over which pin is ```RX```, ```TX```, ```GND```, or ```VCC```.

Using a multimeter, I tested and confirmed a 3.3V ```VCC```, ```GND```, and ```TX``` pins. The ```RX``` pin reads at 0.0V, because it is essentially a listening port, just waiting for information. We can safely assume ```RX``` receives at 3.3V, just like the rest of the pins. Once the voltage is confirmed to be 3.3V, the next step is to connect the 3.3V serial USB converter.

## Connecting UART to serial USB converter
After confirming that the UART port works at 3.3V and soldering a set of header pins, we can safely connect our USB serial converter. Next we use a serial communication program to interface with the device and read the output. For this step, we connect ```GND-GND```, ```RX-TX```, and ```TX-RX```. It is not necessary to connect ```VCC```, so long as we set our serial USB device to use 3.3V.

![208198729-9004ca52-7f22-4dff-9c04-627a84ff9245](https://user-images.githubusercontent.com/95890436/208494179-916da9d5-439a-450f-9ddd-7170eec90d49.jpg)


## Obtaining shell access
Once we have secured a connection between the serial USB converter to the router's UART port, we can now modify the settings in PuTTy for serial communication. The main value we need to know is the UART port's baud rate. It is possible to use a logic analyzer to figure out what the baud rate is. However, there are several baud rates that are very common, so it is often easier to guess common values until one works. In this particular case, the router's UART runs at **115200** baud rate. Other baud rates will produce garbage on the screen. We will also use the following settings:
- ```Data bits:``` 8
- ```Stop bits:``` 1
- ```Parity:``` None
- ```Flow control:``` None

After gaining access to the shell, I used the Linux command ```echo $USER``` to confirm root access of the device. We know we have achieved this, as the device returns ```root```.

https://github.com/JulianOzelRose/TL-WR841N-v14/assets/95890436/ef8f553b-256d-4cd7-80a0-edb2c1f17edf


## Bypassing write protection (read only console)
At first I was only able to access the shell, but not issue commands -- I was stuck in a read-only console. I learned that this is because TP-Link shorts the ```RX``` pin of the UART port as a security measure. There is a single voltage-limiting resistor just next to the ```RX``` pin on this particular model. Pulling that resistor out with a pair of tweezers did the trick, and I was then able to write to the console and issue commands. Take note of the missing resistor at ```R18``` just above the ```RX``` pin.

![IMG_9832 - Copy](https://user-images.githubusercontent.com/95890436/208493479-fe79a047-e249-4d27-b693-82f5032896c4.jpg)

# Sources
Because this router is very popular for reverse engineering, there are many guides on how to reverse it. I used the following sources to help me throughout this project.
I recommend checking them out for an even more in-depth explanation on the hardware reversing process.
- [Zero Day Initiative](https://www.zerodayinitiative.com/blog/2019/9/2/mindshare-hardware-reversing-with-the-tp-link-tl-wr841n-router)
- [River Loop Security](https://www.riverloopsecurity.com/blog/2020/01/hw-101-uart/)
- [Hackaday](https://hackaday.com/2019/09/12/dissecting-the-tl-wr841n-for-fun-and-profit/)
- [OpenWrt](https://openwrt.org/toh/tp-link/tl-wr841nd)

