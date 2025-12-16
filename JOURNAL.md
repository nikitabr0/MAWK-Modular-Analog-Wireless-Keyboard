## 2025.12.16 - 4.5 hours:

### Research on analog keyswitches, MCU and Trackpoint.

Initially considered ESP32-S3, because of wireless capabilities and large amount of I/O, but eventually decided on the nRF52840, due to improved power consumption and full control over 2.4GHz radio, which allows to create a custom dongle.

Don't want to use XIAO nRF52840 or a similar breakout board because of space constraints and my desire to learn SMD soldering.

Cheapest nRF52840 in a QFN-73 package (only this one has USB-D pins): https://www.aliexpress.com/item/1005005069398861.html
Also there's one on LCSC (JLCPCB): https://www.lcsc.com/product-detail/C190794.html

Apparently I don't need an expensive J-Link or nRF52840 devboard for flashing a bare nRF52840 SMD chip, it can be done with RPi (for example RP20240) with firmware like pico-debug or picoprog.

Decided to go with "Gateron Double-Rail Magnetic Dawn" switches (https://www.aliexpress.com/item/1005006788772072.html), as they're cheap and have a standard (Cherry MX) physical dimensions.

TrackPoint uses PS/2 and is compatible with QMK and ZMK.

This TrackPoint module seems promising: https://holykeebs.com/products/sk8707-01-trackpoint-sensor
