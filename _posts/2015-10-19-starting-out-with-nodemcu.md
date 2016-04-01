---
layout: post
title: Starting out with nodemcu
date: 2015-10-19 23:00
category: nodemcu
keywords: nodemcu, microcontroller, electronics, diy
---

You've just received your nodemcu DevKit board and want to start working on
your ideas. How do you get started when you're using Windows?

<!-- more -->

I've been playing with nodemcu for about a week and despite a few hiccups,
it's great. If you use a Windows machine, you shouldn't have any problems -
and I'm quite confident that it will work fine on Linux or Mac as well.

[NodeMCU](https://en.wikipedia.org/wiki/NodeMCU) is a development kit centered
around the [ESP8266](https://en.wikipedia.org/wiki/ESP8266) micro-controller.
It features a 32-bit CPU running at 80Mhz, 64KB RAM and IEEE 802.11 b/g/n
Wi-Fi, SPI, I2C, 10-bit ADC and plenty of GPIO pins that you can use for all
your IoT needs. Besides the features offered by ESP8266, nodeMCU comes with
USB-TTL and pins that can be connected easily to jumper cables.

### Connect the nodemcu board to your computer
1. My nodemcu board didn't come with a USB cable, but you can use any micro
   USB cable (hint - the cable that came with your Android phone) 
2. Connect the board to the computer and wait for the drivers to install

My nodemcu board uses CH341 as the USB-to-serial adapter, for which Windows 7 was able to install the driver automatically. 

### Flash the nodemcu firmware
I've ordered my nodemcu board from eBay, but it didn't come with any firmware
installed. This is perfectly normal and there's nothing to worry about.

1. Download [nodemcu-flasher](https://github.com/nodemcu/nodemcu-flasher) from
   GitHub. You can find the actual binary in the Win32/Release or Win64/Release
   folder 
2. Download the latest [nodemcu
   firmware](https://github.com/nodemcu/nodemcu-firmware/releases). At the
   moment of this writing, there are two versions for each firmware version:
   the integer version which supports only integer operations and the float
   version which contains support for floating point calculations. From a 
   performance point of view, the integer version should be better. It's also
   smaller and most of the time, you will not need to work with floating point
   numbers.
3. Open nodemcu-flasher, go to the configuration section 4. Select the
   downloaded firmware in the first entry (address 0x0000) 5. Press Flash

After a few seconds, the flashing process should be completed. From my
experience, you will need to reboot the board in order to connect to it (I
usually unplug it from the computer, but you can also try the reset button
from the board)

### Start programming
At the moment, I've found two IDEs that can be used for nodemcu Lua development.

#### LuaLoader
GitHub: <https://github.com/GeoNomad/LuaLoader>

A very basic and low level IDE that can be used to interact with the nodemcu
development board. It allows you to upload Lua files, execute them and also
interact with the board interactively.

I've used it as my first IDE, but I became annoyed about it's interfaces and
missing features. It also has problems in uploading Lua files, which can cause
strange errors later on.

#### ESPlorer
GitHub: <https://github.com/4refr0nt/ESPlorer>

After struggling for a few days with LuaLoader, I've decided to try out
ESPlorer. The difference was huge - the interface is more intuitive and it
also seems to be more stable. 

> If you want to upload a Lua file to the board, that also contains
> comments, you should use the "Upload file" command instead of the classic
> "Send to ESP". From what I've noticed, this transfers the file at once,
> without any parsing or splitting, which avoids various issues related to
> comments or other complex Lua structures.

Happy coding!

