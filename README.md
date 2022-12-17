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
The UART/serial port is primarily used during the manufacturing process for debugging purposes. You can find UART ports on virtually any IOT device. With the proper tools, it is possible to access the shell using this port. From there, you can dump the firmware, flash hacked firmware, get device info, give yourself remote/backdoor access, and more. The UART pin layout is ```GND```, ```TX```, ```RX```, and ```VCC```.

![photo_2022-12-16_14-12-06 (5)](https://user-images.githubusercontent.com/95890436/208197839-11598118-c562-45e7-9051-d94d1c914e86.jpg)
# Testing UART Port Voltages
To avoid shorting both our TL-WR841N router and our serial USB device, we must first confirm that we are using the correct voltages. This is an important step, as using the wrong voltage can damage the router, the serial USB device, or both. UART ports are typically either 3.3V, or 5V. Thankfully, TP-Link has gone ahead and labled the pins for us -- so there is no ambiguity over which pin is ```RX```, ```TX```, ```GND```, or ```VCC```.

Using my multimeter, I tested and confirmed a 3.3V ```VCC```, ```GND```, and ```TX``` pins. ```RX``` reads at 0.0V, because it is essentially a listening port, just waiting for information. We can safely assume ```RX``` recieves at 3.3V, just like the rest of the serial port. Once the voltage is confirmed to be 3.3V, the next step is to connect the 3.3V serial USB converter.

# Connecting UART to Serial USB Converter
After confirming that the UART port works at 3.3V and soldering a set of header pins, we can safely connect our USB serial converter. Next we use a serial communication program to interface with the device and read the output. For this step, we connect ```GND-GND```, ```RX-TX```, and ```TX-RX```. It is not necessary to connect ```VCC```, so long as we set our serial USB device to use 3.3V.

![photo_2022-12-16_14-12-06 (3)](https://user-images.githubusercontent.com/95890436/208198729-9004ca52-7f22-4dff-9c04-627a84ff9245.jpg)

# Shell Access
Once we have secured a connection between the serial USB converter to the router's UART port, we can now modify the settings in PuTTy for serial communication. The main value we need to know is the UART port's baud rate. It is possible to use a logic analyzer to figure out what the baud rate is. 

However, there are several baud rates that are very common, so it is easy to guess common values until one works. In this particular case, the router's UART runs at ```115200``` baud rate. Other baud rates will produce garbage on the screen. We will also use the following settings: 8 data bits, 1 stop bits, no parity, no flow control. After gaining access to the shell, I used the Linux command ```echo $USER``` to confirm root access of the device. We know we have achieved this, as the device returns ```root```.



https://user-images.githubusercontent.com/95890436/208254191-7d6bdf81-cf2c-470f-bc04-1bbeaad90dae.mp4


