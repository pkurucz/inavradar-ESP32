![Logo](https://github.com/mistyk/inavradar-ESP32/raw/master/docs/logo.png)

# LoRa based inter UAV communication

INAV-Radar is an addition to the [INAV](https://github.com/iNavFlight/inav) flight control software, it relays information about UAVs in the area to the flight controller for display on the OSD. INAV-Radar does this by using [LoRa](https://en.wikipedia.org/wiki/LoRa) radio to broadcast position, altitude, speed and plane name. It also listens for other UAVs so INAV OSD  can display this information as a HUD.

![OSD](https://github.com/mistyk/inavradar-ESP32/raw/master/docs/osd.jpg)

## News

Tests on aplha 7 look promising!

If you feel brave engough to be a tester, just ask us in the Facebook group. [Contact](#contact)

Thanks to Olivier we now have 'the HUD' it's nealy augmented reality!

## Index
[Hardware](#hardware)

[Development](#development)

[Testing](#testing)

[Wireing](#Wireing)

[FC settings](#FC-settings)

[ESP32 commands](#commands)

[Contact](#contact)

## Hardware

Current development is done using these cheap ESP32 LoRa modules.

There are different variants for 433MHz and 868/915MHz:

[Banggood: ESP32 Lora 868/915MHz (2 Pcs)](https://www.banggood.com/de/2Pcs-Wemos-TTGO-LORA32-868915Mhz-ESP32-LoRa-OLED-0_96-Inch-Blue-Display-p-1239769.html?rmmds=search&cur_warehouse=CN)

[Banggood: ESP32 Lora 433MHz](https://www.banggood.com/de/Wemos-TTGO-LORA-SX1278-ESP32-0_96OLED-16-Mt-Bytes-128-Mt-bit-433Mhz-For-Arduino-p-1205930.html?rmmds=search&cur_warehouse=CN)

Other variants (e.g. Heltec) or without OLED display and different antenna connectors should also work.

Also please keep track of your countries regulations regarding radio transmissions.

## Development

Everything here is WORK IN PROGRESS!

The software is based on two components:
- ESP32 LoRa part is found in this repo.
It's developed using [PlatformIO](https://platformio.org/) plugin for [Atom](https://atom.io/) editor.
- INAV OSD part repo is found [here](https://github.com/KingKone/inav).
It's a fork from the INAV repo and instructions how to build can be found [here](https://github.com/iNavFlight/inav/blob/master/docs/development/Building%20in%20Docker.md).

INAV-Radar is a experimental firmware based on INAV and is currently no part of the INAV flight control software. INAV repo can be found [here](https://github.com/iNavFlight/inav).

## Testing

For testing there is no need to install Atom and PlatformIO, just use the [esptool](https://github.com/espressif/esptool) for flashing.

Your system needs the driver for the USB UART bridge:
[Windows+MacOS](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers)
 or [Alternative MacOS](https://github.com/adrianmihalko/ch340g-ch34g-ch34x-mac-os-x-driver)

Also you will need [Python 3.4 or newer](https://www.python.org/downloads/) installed on your system.

Be sure to check 'Add Python to PATH':
![Python Setup](https://github.com/mistyk/inavradar-ESP32/raw/master/docs/python.png)


The latest stable esptool.py release can be installed via pip in your command prompt:

Windows:
```
c:\> pip install esptool
```

MacOS:
```
$ pip3 install esptool
```

Download the air-to-air test firmware from the [releases page](https://github.com/mistyk/inavradar-ESP32/releases)
and extract it. Run this command to flash it onto your ESP32 Lora module (Windows and MacOS):

You may change the --port to match your operating system. If you are using Windows check the [device manager](https://github.com/mistyk/inavradar-ESP32/raw/master/docs/devManager.PNG).

Windows:
```
c:\> cd (your air-to-air directory here)
c:\> esptool.py --port COM11 write_flash -z --flash_mode dio 0x1000 bootloader_dio_40m.bin 0x8000 default.bin 0xe000 boot_app0.bin 0x10000 firmware.bin 0x291000 fs.bin
```

MacOS:
```
$ cd (your air-to-air directory here)
$ esptool.py --port /dev/tty.SLAB_USBtoUART write_flash -z --flash_mode dio 0x1000 bootloader_dio_40m.bin 0x8000 default.bin 0xe000 boot_app0.bin 0x10000 firmware.bin 0x291000 fs.bin

```

The output should look something like this:
![Windows CMD output](https://github.com/mistyk/inavradar-ESP32/raw/master/docs/cmd.PNG)

## Wireing

To connect the ESP32 to the FC:
- wire up +5V and GND
- TX from FC to ESP RX pin 17
- RX from FC to ESP TX pin 23


## FC settings

Backup your FC settings, flash the current [testing version of INAV](https://github.com/mistyk/inavradar-ESP32/releases).

Dump your backup back into the cli.

Activate MSP on the corresponding UART, the speed is 57600.

OSD and HUD settings:
```
osd_layout 0 2 0 0 V        // Crosshair and the HUD ON
osd_layout 0 43 0 0 H       // north map OFF
osd_layout 0 44 0 0 H       // takeoff map OFF
osd_layout 0 45 0 0 H       // radar map OFF

set osd_camera_uptilt = 0   // the HUD: camera angle
set osd_camera_fov_h = 135  // the HUD: field of view horizontal
set osd_camera_fov_v = 85   // the HUD: field of view vertical
```

Please also flash the extra Vision OSD fonts for signal strenth and the homing crosshair.

The HUD has an entry in the stick menu (OSD->HUD) where you can change this configuration at runtime.

Thanks for testing! 😄 👍

## Commands

If something is not working correctly, please open a serial terminal (Windows e.g. [PuTTy](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)).

Attention !!! On Windows with Putty you must use [CRTL]-[J] instead of the [ENTER] key.

Type in "debug" to activate debug mode. Reset the device (lower button) connected to the terminal and send us a copy of the output.

Also there are some other commands:

```
================= Commands =================
status                  - Show whats going on
help                    - List all commands
config                  - List all settings
config loraFreq n       - Set frequency in Hz (e.g. n = 433000000)
config loraBandwidth n  - Set bandwidth in Hz (e.g. n = 250000)
config loraSpread n     - Set SF (e.g. n = 10)
config uavtimeout n     - Set UAV timeout in sec (e.g. n = 10)
config fctimeout n      - Set FC timeout in sec (e.g. n = 5)
config debuglat n       - Set debug GPS lat * 10000000 (e.g. n = 501004900)
config debuglon n       - Set debug GPS lon * 10000000 (e.g. n = 87632280)
reboot                  - Reset MCU and radio
gpspos                  - Show last GPS position
debug                   - Toggle debug output
localfakeplanes         - Send fake plane to FC
lfp                     - Send fake plane to FC
radiofakeplanes         - Send fake plane via radio
rfp                     - Send fake plane via radio
movefakeplanes          - Move fake plane
mfp                     - Move fake plane
```

## Contact

[Facebook Group](https://www.facebook.com/groups/360607501179901/)

[Patreon](https://www.patreon.com/inavradar)
