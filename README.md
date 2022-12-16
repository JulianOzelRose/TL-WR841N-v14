# TL-WR841N
The TP-Link TL-WR841N is a relatively inexpensive 300mbps WiFi router. This makes it a perfect device for reverse engineering and hardware hacking projects. My goal here was to gain shell access via the router's UART debugging port.
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
![photo_2022-12-16_14-12-06 (5)](https://user-images.githubusercontent.com/95890436/208197839-11598118-c562-45e7-9051-d94d1c914e86.jpg)
The UART/Serial port is primarily used during the manufacturing process for debugging purposes only. You can find debugging ports on virtually any IOT device. With the proper tools, it is possible to access the shell using this port. From there, you can dump the firmware, flash hacked firmware, get device info, give yourself remote or backdoor access, and more. The UART pin layout is GND, TX, RX, and VCC.
# Testing UART Port Voltages
To avoid shorting both our TL-WR841N router and our serial device, we must first confirm that we are using the correct voltages. UART ports are typically either 3.3V, or 5V. Thankfully, TP-Link has gone ahead and labled the pins for us -- so there is no ambiguity over which pin is RX, TX, GND, or VCC. Using my multimeter, I tested and confirmed a 3.3V VCC, GND, and TX pins. RX reads at 0.0V, because it is essentially a listening port, just waiting for data. We can safely assume RX recieves at 3.3V, just like the rest of the serial port. Once the voltage is confirmed to be 3.3V, I soldered four header pins over the UART copper pads.
# Connecting UART to Serial USB Converter
![photo_2022-12-16_14-12-06 (3)](https://user-images.githubusercontent.com/95890436/208198729-9004ca52-7f22-4dff-9c04-627a84ff9245.jpg)
After confirming that our UART port work at 3.3V and soldering header pins, we can now connect our UART Serial Converter.
