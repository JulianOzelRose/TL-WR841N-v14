# TL-WR841N
The TP-Link TL-WR841N is a relatively inexpensive 300mbps WiFi router. This makes it a perfect device for reverse engineering and hardware hacking projects. My goal here was to gain shell access via the router's UART debugging port, and see what can be done from there.

![photo_2022-12-16_15-26-34](https://user-images.githubusercontent.com/95890436/208206724-dc6ea069-f0d3-42e3-a3aa-e40ebf6da23d.jpg)
# Tools
#### Hardware
- Klein Tools Digital Multimeter MM300
- USB 2.0 to TTL UART Module Serial Converter CH340G 5V/3.3V
- Soldering Iron Kit from Pulsivo
- Female/Female Jumper Cables
- Header Pins
#### Software
- PuTTy
- XCTU
# Serial UART Port
The UART/Serial port is primarily used during the manufacturing process for debugging purposes only. You can find debugging ports on virtually any IOT device. With the proper tools, it is possible to access the shell using this port. From there, you can dump the firmware, flash hacked firmware, get device info, give yourself remote or backdoor access, and more. The UART pin layout is ```GND```, ```TX```, ```RX```, and ```VCC```.

![photo_2022-12-16_14-12-06 (5)](https://user-images.githubusercontent.com/95890436/208197839-11598118-c562-45e7-9051-d94d1c914e86.jpg)
# Testing UART Port Voltages
To avoid shorting both our TL-WR841N router and our serial device, we must first confirm that we are using the correct voltages. This is an important step, as using the wrong voltage can damage the router, the Serial USB device, or both. UART ports are typically either 3.3V, or 5V. Thankfully, TP-Link has gone ahead and labled the pins for us -- so there is no ambiguity over which pin is ```RX```, ```TX```, ```GND```, or ```VCC```.

Using my multimeter, I tested and confirmed a 3.3V ```VCC```, ```GND```, and ```TX``` pins. ```RX``` reads at 0.0V, because it is essentially a listening port, just waiting for data. We can safely assume ```RX``` recieves at 3.3V, just like the rest of the serial port. Once the voltage is confirmed to be 3.3V, I soldered four header pins over the UART copper pads.

# Connecting UART to Serial USB Converter
After confirming that our UART port work at 3.3V and soldering header pins, we can now connect our UART Serial Converter. The next step is to use a serial communication program to interface with the device and read the output. For this step, we connect ```RX-RX```, ```TX-RX```, and ```RX-TX```. It is not necessary to connect ```VCC```, so long as we set our Serial USB device to use 3.3V.

![photo_2022-12-16_14-12-06 (3)](https://user-images.githubusercontent.com/95890436/208198729-9004ca52-7f22-4dff-9c04-627a84ff9245.jpg)

# Shell Access
Once we have secured a connection between the Serial USB converter to the router's UART port, we can now modify the settings in PuTTy for serial communication. The main value we need to know is the UART port's baud rate. It is possible to use a logic analyzer to figure out what the baud rate is. However, there are several baud rates that are very common, so it is easy to guess common values until one works. In this particular case, the router's UART runs at ```115200``` baud rate. Other baud rates will produce garbage on the screen. Lastly, I used the Linux command ```echo $USER``` to confirm root access of the device.

https://user-images.githubusercontent.com/95890436/208212759-f1e04f3c-021b-427d-892f-6508d08d004a.mp4
