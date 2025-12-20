## 2025.12.16 - 4.5 hours:

### Research on analog keyswitches, MCU and Trackpoint.

Initially considered ESP32-S3, because of wireless capabilities and large amount of I/O, but eventually decided on the nRF52840, due to improved power consumption and full control over 2.4GHz radio, which allows to create a custom dongle.

Don't want to use XIAO nRF52840 or a similar breakout board because of space constraints and my desire to learn SMD soldering.

Cheapest nRF52840 in a QFN-73 package (only this one has USB-D pins): https://www.aliexpress.com/item/1005005069398861.html
Also there's one on LCSC (JLCPCB): https://www.lcsc.com/product-detail/C190794.html

Apparently I don't need an expensive J-Link or nRF52840 devboard for flashing a bare nRF52840 SMD chip, it can be done with RPi (for example RP2040) with firmware like pico-debug or picoprog.

Decided to go with "Gateron Double-Rail Magnetic Dawn" switches (https://www.aliexpress.com/item/1005006788772072.html), as they're cheap and have a standard (Cherry MX) physical dimensions.

TrackPoint uses PS/2 and is compatible with QMK and ZMK.

This TrackPoint module seems promising: https://holykeebs.com/products/sk8707-01-trackpoint-sensor


## 2025.12.17 - 2.5 hours:

### Power management and TMR sensor research.

Power management:
Gonna use Microchip MCP73871. It got full battery and power-path management, so I just gotta connect VBUS (USB-C) -> IN and OUT -> VDDH (nRF52840) + other 5V things.
AliExpress: https://www.aliexpress.com/item/1005008336731440.html
LSCS: https://www.lcsc.com/product-detail/C511310.html

TMR sensor:
Allegro CT8150PC should work, but the specific placement relatively to the Gateron Double-Rail Magnetic switches is questinable. Has ±8mT sensitivity range. ChatGPT suggests that the magnet should remain in the 2.5-5mm range from the sensor for it to work reliably, while the key travel is 4mm. It's especially likely that it will come too clode to the sensor. Will have to actually test this before making the PCB design.
Will use SOT package.
Turns out they're really expensive. Found them only at LCSC (https://www.lcsc.com/product-detail/C22879138.html) for ~1.23€(1.5$)/pcs at 100+. Will try to find something cheaper.
Found a little cheaper at DigiKey (just 0.83€/pcs at 100+): https://www.digikey.ee/en/products/detail/allegro-microsystems/ct8150pc-is3/22114148

Analog multiplexers:
Since nRF52840 can use just 8 pins for SAADC, I'm gonna have to use multiplexers. Found Texas Instruments CD4051B, they have 3 address pins, 8 inputs/outputs and can be disabled.
I could wire all the address pins to 3 pins on nRF, MUXs' outputs are connected in groups of 3 to 3 ADC pins on nRF (so 1 ADC pin per 3 MUXs) and then I use 3 pins, each connected to 3 MUXs from different groups, on nRF to enable the MUXs and allow it to read each TMR sensor independently. So it's basically a matrix.
![image](Images/2025_12_17_ADC_MUXs_scheme_sketch.png)
Will go with SOP-16 package for easier assembly.
Cheapest at LCSC: https://www.lcsc.com/product-detail/C87646.html
AliExpress: https://www.aliexpress.com/item/1005009321962080.html


## 2025.12.18 - 2h:

### Alternative analog sensors, battery, module MCU

Looked into cheaper TMR and HE sensors, but didn't find any.

Decided to use a Li-Po battery like this: https://www.aliexpress.com/item/1005003462939882.html

Will use nRF52840 for modules as well, as I'll have to buy them in bulk anyways.
Probably will have to use multiple I2C buses to mitigate capacitance. Will add MOSFETs on SDA and SCL on the modules to keep them disconnected until the module MCU powers up completely, to prevent unexpected spikes. Modules are going to set an "advertisement" address on startup, the main MCU will contact that address for module type and then assign it a unique address for the session.
![image](Images/2025_12_18_I2C_buses_sketch.png)


## 2025.12.20 - 2.5h:

### SPI for high-speed modules (displays, trackpads), connectors

Decided to add SPI buses for higher-speed modules (displays and trackpads).
Will need to do SCLK, MOSI, MISO and 2-3 CS pins for each bus. Multiple CSs are to allow connecting multiple SPI modules on the same bus.
SPIs are kinda sensitive in terms of hot-plugging, so need a lot of precautions. Generally SCLK, MOSI, MISO don't matter as far as CS is high, but I'll add isolation just in case. They will be isolated until the slave is powered up and ready to estabilish the connection.

1. The module is connected, MCU powers up, all buses are isolated.
2. The slave opens I2C, goes live on it and tells the master, that it's a SPI module.
3. The master tells the slave to open the SPI and start communication.
4. Slave starts SPI.
5. It works!

All CS lines should have pull-ups (10-100 kOhms) on the master side. Good to add pull-downs on clock lines. 22-68 Ohm series resistors on SCLK amd MOSI close to the master MCU
Decided to use SN74CBTLV3384 for I2C and SPI isolation. It can enable it's channels in groups of 5, so I can keep SPI switched off even when I2C is active.
Cheapest at LCSC: https://www.lcsc.com/product-detail/C2653297.html
DigiKey: https://www.digikey.ee/en/products/detail/texas-instruments/SN74CBTLV3384PWR/378134

For edge connectors I'll use pogo pins. To allow any module to be connected from any side and other modules to be connected to it, I'll use this configuration:
![image](Images/2025_12_20_connectors_scheme.png)
