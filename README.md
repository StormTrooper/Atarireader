# Atari Cartridge Reader

## Overview

This is the source code and hardware design for an Arduino-based device to extract ROM images from Atari 2600 cartridges.
Original design from [Brian]. (https://netninja.com/2016/02/18/reading-atari-cartridges-with-an-arduino/)
Source code at [github](https://github.com/BrianEnigma/Arduino)


## Hardware

I have updated code to use Arduino Mega. The cartridge connection itself comes from [a 24 position edgeboard connector from Digikey](https://www.digikey.com/product-detail/en/EBC12DCWN/S3304-ND/927256).

This is a fairly easy build, given that the cartridges work at 5V levels, consisting of directly connecting address and data lines between the cartridge slot and the Arduino. 

### Connections

You can find the [Atari cartridge pinouts online](http://www.atariage.com/2600/faq/index.html?SystemID=2600#pinouts), but they basically amount to the following:

```
(Looking at the bottom of the cartridge -- i.e. edge connectors first)
                        Top
 D3   D4   D5   D6   D7   A12  A10  A11  A9   A8  +5V   SGND
--1- --2- --3- --4- --5- --6- --7- --8- --9- -10- -11- -12-
 GND  D2   D1   D0   A0   A1   A2   A3   A4   A5   A6   A7
                        Bottom

Dx = Data line x
Ax = Address line x
+5V = +5 volts
SGND = Shield Ground
GND = Ground
```

Obviously, the two grounds connect to ground. 5V connects to the Arduino's 5V voltage regulator. A12 is a chip select and is also connected to the 5V regulator.

The address lines connect to the following Arduino pins. Remember that pins A10-A12 are out of order on the connector and that the pinout above is for the card edge itself (in the cartridge), so you'll have to do some mirroring to ensure you're using the correct card edge plug pins.

- cartridge A0 to Arduino's (analog) A0
- cartridge A1 to Arduino's (analog) A1
- cartridge A2 to Arduino's (analog) A2
- cartridge A3 to Arduino's (analog) A3
- cartridge A4 to Arduino's (analog) A4
- cartridge A5 to Arduino's (analog) A5
- cartridge A6 to Arduino's (digital) A6
- cartridge A7 to Arduino's (digital) A7
- cartridge A8 to Arduino's (digital) A8
- cartridge A9 to Arduino's (digital) A9
- cartridge A11 to ground
- cartridge A10 to Arduino's (digital) A10
- cartridge A12 to +5V

The data lines map in the following way. 

- cartridge D0 to Arduino's (digital) D2
- cartridge D1 to Arduino's (digital) D3
- cartridge D2 to Arduino's (digital) D4
- cartridge D3 to Arduino's (digital) D5
- cartridge D4 to Arduino's (digital) D6
- cartridge D5 to Arduino's (digital) D7
- cartridge D6 to Arduino's (digital) D8
- cartridge D7 to Arduino's (digital) D9

The top two pins are hard-wired, meaning we can only access 2^10 address space at a time. This is sufficient for smaller cartridges and as a proof-of-concept, but you'll need to manually hard-wire pins to perform banking if you want to use this particular setup for larger ROMs. More advanced Arduino boards provide more pins. The setup is similar, but left as an exercise for the reader.

**Build Hardships**

I ran into several issues during the build, detailed more fully in the blog post.

1. Not enough pins. With two more pins (or a fancy multiplexing scheme), I can't read large ROM cartridges.
2. The edge connector pins have a breadboard-incompatible footprint. I really wanted to build this on a temporary breadboard, but the edge connector's through-hole pins only have one effective row of pins between them. This is not a wide enough footprint to fit over the center of a breadboard (where a DIP-package chip would normally sit). I ended up using a prototyping space, soldering wires to the pins, and using those to connect to the Arduino.
3. The edge connector is just barely tall enough to reach the cartridge. In fact, I could not push it all the way down onto the board and solder on the back side, as you do with most through-hole components. I had to put a piece of masking tape on the back of the board, gently put the card edge connector into the proto-space, without pushing too hard. The legs in this configuration, like stilts, give it a couple of extra millimeters. I then had to solder on the same side as the connector.

In the future, I would definitely see if I could find a taller connector with better pin spacing.

## Serial Dump Software

You can find the serial dump software, which gets loaded into the Arduino, at `Atarireader/Atarireader.ino`.


## Serial Capture

You can capture the ROM dump using the Arduino IDE's serial monitor (and copy/paste into a text file) or by using a serial terminal emulator such as Minicom.

## Reassembly Software

The `reassemble.rb` script will take a capture file and translate it into a binary file.

## Future Design Considerations

This project was built on a workbench in an afternoon as a proof-of-concept. If I wanted to do a more formal job with this design I would look toward the following:

- Taller card edge connector or some way to physically lift it better.
- Design and 3D print a latch-release that fits around the card edge connector. Some cartridges have fully exposed card edges, but most need pins to push in a latch release before the edge connector is exposed. It's easy enough to manually do this with a plastic spudger tool, but it would be nice to build that into the design.
- Use a GPIO extender to twiddle the higher address lines so that I can reach them from code without manually bank switching by manually grounding or energizing those pins.
