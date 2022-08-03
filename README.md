# Renogy Modbus on ESPHome

Here is a very rough draft of my notes for this project. This is an example configuration for ESPHome to communicate with a Renogy Wanderer 10A PWM Charge Controller. Pretty much any Renogy charge controller with an RS-232 port should work with more/less the same config, though some features and fields may not work with all Renogy charge controllers depending on their features.

Some day I plan on returning to this project and doing a more formal write-up, but hopefully this can help someone before then.

First, you need a RS-232/TTLconverter. I used this: [Sparkfun BOB-11189 RS-232 Transceiver Breakout](https://www.sparkfun.com/products/11189) which is just a breakout for the MAX 3232. Any equivalent RS232/TTL converter should work. The wiring was a little tricky to get right and I did not take a photo or write down my schematic before tearing it down and throwing it in a box. :( But I used this as my reference for the Renogy pinout: [https://github.com/collinturney/solarshed](https://github.com/collinturney/solarshed) and this for the protocol (look in the reference directory): [https://github.com/KyleJamesWalker/renogy_rover](https://github.com/KyleJamesWalker/renogy_rover).

Sparkfun also has breakout boards for the 6P6C jack used on the Renogy projects so you can breadboard easily and connect it with a nice clean 6-pin cable.

Bonus: The renogy jack also supplies 15V out--if you have an appropriate step-down converter you can use this to power the ESP. **DO NOT** connect the V+ lines directly to your ESP--most boards cannot step down from 15V so you will likely need an external converter here.

## Connections

(From memory--may not be accurate)

| Device Pin  | MAX3232 Pin |
| ----------- | ----------- |
| Renogy TX | R1 IN |
| Renogy RX | T1 OUT |
| Renogy GND | GND |
| ESP TX | T1 IN |
| ESP RX | R1 OUT |
| ESP RX | R1 OUT |
| ESP 3V3 | Vcc (3V-5.5V) |
| ESP GND | GND |

If your wiring doesn't work you can try swapping the RX/TX directions, but make sure you only connect your ESP to the TTL/CMOS side (**DO NOT** connect the ESP to the T OUT or R IN pins--the ESP cannot handle RS-232 voltages). 

### **WARNING**

**Please** double-check the schematics, pinouts, and datasheets they provide to avoid any possible damage--I take no responsibility for my memory!  When I was playing around with this I did some sanity checking with a multimeter to make sure I didn't fry an ESP with RS-232 level voltages--read the note in the first github link RE: RJ12 pinout.

## Configuration

- common/common.yaml : A port of my common esphome configuration I use for most projects.
- common/secrets.yaml : An example secrets file which gets used by my common configuration.
- modbus-test.yaml : An example configuration file for a TinyPICO ESP32 connected to a Renogy Wanderer Charge Controller with Modbus.
- renogy-modbus.yaml : The Renogy-specific configuration for UART, Modbus, Sensor registers, and switchable load (light, etc).

## Nota Bene

1. Your ESP's TX and RX pin may be different depending on which controller you're using and how you choose to set it up. 
2. The modbus controller address that worked for me was 0xFF, despite just about everyone else and everything stating it should be 0x00. I never had the time to figure out why, so try the other if one doesn't work.
3. The log level is set to DEBUG in the test file--helpful when benchtesting, noisy when in use.
4. I have the update interval set at 3s which may be a bit aggressive for everything. I am using skip_updates on the less relevant fields like daily statistics. You might want to play with these values for your use case. ESPHome will automatically group up simultaneous queries if the memory addresses are adjacent, which reduces the overhead in the payload and speeds up communication. I remember seeing various warnings in the logs about overlapping/repeating addresses and requests being sent before the previous completed, but I do not remember how well-optimized I got this before I shelved the project, so it warrants some playing around. If you're not planning on using a bunch of the extra data fields they can just be omitted, but it's cool how much data this little thing tracks.

## Disclaimer

This is far from well-tested at this point. Use any information here at your own risk. I take no responsibility for errors, accuracy, or anything you choose to do with this information. If you spot something you know to be false, feel free to let me know so I can correct it.

## Credits

Thanks to these two for posting highly useful information that helped me get this far, as well as a handful of others on GitHub who I no doubt forgot since last year:

- [https://github.com/collinturney/solarshed](https://github.com/collinturney/solarshed)
- [https://github.com/KyleJamesWalker/renogy_rover](https://github.com/KyleJamesWalker/renogy_rover)

Thanks to [Renogy](https://www.renogy.com/) for releasing a really cool, really reasonably-priced line of Solar Charge Controllers with a standards-based communication protocol **AND** releasing the documentation around their specific registers to the community. This is AWESOME of them. They have a forum where some of this was discussed (I believe the original conversations were on a previous host, but still) [https://pc.renogy-dchome.com/](https://pc.renogy-dchome.com/#/)

Thanks to [ESPHome](https://www.esphome.io/), its developers both past and present and Nabu Casa for making such an awesome project that makes it so quick and easy to play with ESP32 microcontrollers.