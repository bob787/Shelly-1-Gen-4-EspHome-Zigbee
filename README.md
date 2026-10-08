**Shelly Zigbee ESPhome notes**


**Why did I do this?**

When Shelly launched their Gen 4 relays with zigbee,  I was genuinely excited. I prefer Zigbee devices because they are inherently local. They can never be paywalled, locked out, feature reduced, or cancelled. 


When I finally got one, I learned the truth. They work fine on Zigbee, but firmware updates (At least through ZHA) were gated to Wifi. That’s right… If you want new firmware, you need to run two protocols. If you turn off Wifi and change your mind later, your only recourse is to reach inside a live electrical box and press the relay button.


I wanted a Zigbee only device with a ZHA upgrade path. I settled on ESPhome because its simple and recently adopted more comprehensive zigbee controls. I got my Shelly 1 Gen 4 paired with ZHA and the relay works in attached mode.


This project was inspired by ***automatous-io’s fantastic Shelly Matter over Thread repo, [https://github.com/automatous-io/shelly-1-gen4-matter-thread](https://github.com/automatous-io/shelly-1-gen4-matter-thread). Shelly documentation for this relay is here, [https://kb.shelly.cloud/knowledge-base/shelly-1pm-gen4](https://kb.shelly.cloud/knowledge-base/shelly-1pm-gen4) **


***Build the Flashing harness**

***90% of the work in this process is backing up the stock firmware and building a flashing harness. All the parts, links, and prices are in the table below. If you are reading this, I’ll bet you already have most of them on hand.**


***Flashing harness parts:**

| **\#** | **Part** | **Sample Link** | **Price (10/2026)** |
| - | - | - | :-: |
| **1** | **USB to UART converter** | [https://www.amazon.com/dp/B07WX2DSVB?ref=ppx\_yo2ov\_dt\_b\_fed\_asin\_title](https://www.amazon.com/dp/B07WX2DSVB?ref=ppx_yo2ov_dt_b_fed_asin_title)  | $15 |
| **2** | **1.27mm to 2.54mm pin pitch adapter** | [https://www.amazon.com/dp/B09DGJVST6?ref=ppx\_yo2ov\_dt\_b\_fed\_asin\_title](https://www.amazon.com/dp/B09DGJVST6?ref=ppx_yo2ov_dt_b_fed_asin_title)  | $7 |
| **3** | **1.27mm and 2.54mm pin headers** | [https://www.amazon.com/dp/B0CTKCWNGK?ref=ppx\_yo2ov\_dt\_b\_fed\_asin\_title&th=1](https://www.amazon.com/dp/B0CTKCWNGK?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1)    [https://www.amazon.com/HiLetgo-20pcs-2-54mm-Single-Header/dp/B07R5QDL8D/ref=sr\_1\_3](https://www.amazon.com/HiLetgo-20pcs-2-54mm-Single-Header/dp/B07R5QDL8D/ref=sr_1_3)  | ~$10 |
| **4** | **Dupont cables** | [https://www.amazon.com/Elegoo-EL-CP-004-Multicolored-Breadboard-arduino/dp/B01EV70C78/ref=sr\_1\_3](https://www.amazon.com/Elegoo-EL-CP-004-Multicolored-Breadboard-arduino/dp/B01EV70C78/ref=sr_1_3)?  | $7 |


***The Shelly 1 Gen 4 has 1.27mm pitch pins while USB to UART adapters have 2.54mm pitch pins. I had a cheap adapter board, but I’ve seen sewing pins, 3d printed pogo pin alignment boards, and many other ways to get this to work. You need a secure way to connect 4 pins from Shelly to the USB adapter and a quick reversible way to bridge the Shelly’s GPIO00 Pin to Ground to enter bootloader mode. **


**Pinout**

| Shelly 1 Gen4 | USB to UART |
| - | - |
| Pin 1 (ESP\_DBG\_UART) | - |
| Pin 2 (TXD) | RXD |
| Pin 3 (RXD) | TXD |
| Pin 4 (3.3V) | 3.3V |
| Pin 5 (RESET) | - |
| Pin 6 (GPIO0 - BOOT) | - |
| Pin 7 (GND) | GND |
|  |  |



Here is what mine looked like:



I used a female to male dupont cable to bridge GPIO)) to ground to enter bootloader mode during flashing













 










 Here is another angle on the setup


**OEM Firmware Backup:**

The process to backup the stock firmware and flash the new device is well covered here, [https://github.com/automatous-io/shelly-1-gen4-matter-thread/blob/main/docs/FLASHING.md](https://github.com/automatous-io/shelly-1-gen4-matter-thread/blob/main/docs/FLASHING.md) 


Here are some notes that can supplement some of the less clear points:

1. The OEM firmware is locked to the Mac address of the relay. Firmware from another relay can not be used

2.  Remember to put the Shelly into boot loader mode by grounding GPIO00 (Male dupont connector)

3. Get Port number (Mac Terminal)

ls /dev/cu.\*


	a. On the Mac mini, the port was /dev/cu.usbserial-BH010DUA

4. Get mac address (Mac Terminal)

python3 -m esptool --port /dev/cu.usbserial-BH010DUA read\_mac

5. Output on Shelly 1 Gen 4 Blue

esptool.py v4.12.0

Serial port /dev/cu.usbserial-BH010DUA

Connecting....

Detecting chip type... ESP32-C6

Chip is ESP32-C6 (QFN40) (revision v0.2)

Features: Wi-Fi 6, BT 5 (LE), IEEE802.15.4, Single Core + LP Core, 160MHz, Embedded Flash 8MB

Crystal is 40MHz

MAC: \<redacted\>

BASE MAC: \<redacted\>

MAC\_EXT: ff:fe

Uploading stub...

Running stub...

Stub running...

MAC: \<redacted\>

BASE MAC: \<redacted\>

MAC\_EXT: ff:fe

Hard resetting via RTS pin...


6. Read firmware with esptools

esptool.py --chip esp32c6 --port /dev/cu.usbserial-BH010DUA --baud 115200 \\

  read\_flash 0x0 ALL shelly-1-gen4-stock-\<redacted\>.bin

Had to change Slashes that changed to colons back to dashes


7. Verify with esptools

esptool.py --chip esp32c6 --port /dev/cu.usbserial-BH010DUA --baud 460800 verify\_flash 0x0 ~/shelly-1-gen4-stock-\<redacted\>.bin


8. Sample Output. Look for “digest matched”

esptool.py v4.12.0

Serial port /dev/cu.usbserial-BH010DUA

Connecting....

Chip is ESP32-C6 (QFN40) (revision v0.2)

Features: Wi-Fi 6, BT 5 (LE), IEEE802.15.4, Single Core + LP Core, 160MHz, Embedded Flash 8MB

Crystal is 40MHz

MAC: \<redacted\>

BASE MAC: \<redacted\>

MAC\_EXT: ff:fe

Uploading stub...

Running stub...

Stub running...

Changing baud rate to 460800

Changed.

Configuring flash size...

Verifying 0x800000 (8388608) bytes @ 0x00000000 in flash against /Users/\<redacted\>/shelly-1-gen4-stock-\<redacted\>.bin...

-- verify OK (digest matched)

Hard resetting via RTS pin...



**ESPHome Code**


**Zigbee implementation: In Q426, zigbee in EspHome is an evolving standard. I chose to use the older but proven luar123's external component ([https://github.com/luar123/zigbee\_esphome](https://github.com/luar123/zigbee_esphome)). ** Its "basic mode" maps a switch to the standard `on_off` cluster, which ZHA shows as a proper switch. 


EspHome has native Zigbee component in development (as of Q426) where cluster: on\_off is supported. When it eventually arrives in an upcoming release, this code can become a bit simplier.


Here is the code I used:


esphome:

  name: shelly-1-gen4-zigbee

  friendly\_name: Shelly 1 Gen4 Zigbee


esp32:

  variant: esp32c6

  flash\_size: 8MB

  toolchain: platformio

  framework:

    type: esp-idf


external\_components:

  - source: github://luar123/zigbee\_esphome

    components: \[zigbee\]


logger:

  level: INFO

  hardware\_uart: UART0         \# logs on the programming header


zigbee:

  id: zb

  router: true

  power\_supply: 1

  components: all


switch:

  - platform: gpio

    id: relay

    name: "Relay"

    pin: GPIO5

    restore\_mode: RESTORE\_DEFAULT\_OFF


binary\_sensor:

  \# SW terminal: internal (no name)

  - platform: gpio

    id: sw\_input

    pin: GPIO10

    filters:

      - delayed\_on\_off: 50ms

    on\_state:                  \# rocker switch; use on\_press for a momentary button

      then:

        - switch.toggle: relay


  \# Onboard button: short press toggles, hold 5-15 s to leave/rejoin network

  - platform: gpio

    id: device\_button

    pin:

      number: GPIO4

      inverted: true

      mode: INPUT\_PULLUP

    on\_click:

      - min\_length: 50ms

        max\_length: 1s

        then:

          - switch.toggle: relay

      - min\_length: 5s

        max\_length: 15s

        then:

          - zigbee.reset: zb


status\_led:

  pin:

    number: GPIO0

    inverted: true



**Flashing the ESPHome firmware**

**I used EspConnect per the direction here, [https://github.com/automatous-io/shelly-1-gen4-matter-thread/blob/main/docs/FLASHING.md](https://github.com/automatous-io/shelly-1-gen4-matter-thread/blob/main/docs/FLASHING.md). But, EspHome Web, [https://web.esphome.io/](https://web.esphome.io/) would work too. **


**Pairing with ZHA**

1. ZHA adoption took about 30 seconds

2. Entities exposed: On/Off switch for the relay. The PM version of this relay could add the power measurements




**Future versions**

I have a PM version of the Shelly 1 Gen 4 (The red one). I believe the code can be adapted to expose these measurements. I will likely wait for the ESpHome native zigbee cluster On/off is supported. Stay tuned to this repo for future developments


