---
layout: page
title: LED Strip Control
short_description: Wifi LED Strip Control
long_description: DIY LED strip control using wifi-enabled microcontroller
---

These custom lights consist of ESP8266 Wi-Fi microcontrollers wired to RGB LED strips. Light color, brightness, and pattern can be controlled through a Home Assistant home automation system using ESPHome.

<style>
* {
  box-sizing: border-box;
}

.column {
  float: left;
  width: 50%;
  padding: 5px;
}

/* Clearfix (clear floats) */
.row::after {
  content: "";
  clear: both;
  display: table;
}
</style>

<div class="row">
  <div class="column">
    <img src="{{site.baseurl}}/assets/images/LEDStripController2.jpg" alt="overview" style="display: block; margin: auto;">
  </div>
  <div class="column">
    <img src="{{site.baseurl}}/assets/images/LEDStripController3.jpg" alt="closeup" style="display: block; margin: auto;">
  </div>
</div> 

<h3>Hardware:</h3>
* ESP8266
* P9813 LED controller
*  3.3V Voltage Regulator 950mA LD1117V33 (LD33V)

<h3>ESPHome Code:</h3>
```cpp
    esphome:
      name: led_strip_1
      platform: ESP8266
      board: esp01_1m

    wifi:
      ssid: "***"
      password: "***"

      # Enable fallback hotspot (captive portal) in case wifi connection fails
      ap:
        ssid: "Led Strip 1 Fallback Hotspot"
        password: "***"

    captive_portal:

    # Enable logging
    logger:

    # Enable Home Assistant API
    api:

    ota:

    light:
      - platform: fastled_spi
        chipset: P9813
        data_pin: GPIO0
        clock_pin: GPIO2
        num_leds: 1
        rgb_order: RGB
        name: "LED Strip 1"
```

<h3>Citations:</h3>
* <https://esphome.io/components/light/fastled.html>
* <https://github.com/FastLED/FastLED>