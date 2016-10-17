# WNM499: Midterm

## Title: Xmas LED Show
### [Optional] Logo

## Problem & Solution
In the late 1800's Thomas Edison invented the incandescent light bulb. From this invention the idea for Christmas lights soon followed.
The earliest lights were not programmable and simply always stayed on. Light shows with programmable logic arrived during the 1960's 
via the advent of the microcontroller. Today, you can purchase christmas lights almost everywhere however they come pre-programmed 
with only a handful of patterns. The manufactures of christmas lights do not allow the consumer to reprogram their products.

As programmable software is uniquitous is all aspects of our lives, one can't help but wonder at the possibility of extending software
towards making a personalized connection. When you buy a manufactured product that runs some amount of software, the end-user gets stuck
with non-upgradable software. Sometimes we find software that gives pre-programmable options to fit our personality, however the options
are never enough.

With the growing community of programmable internet-enabled devices and inexpensive costs to access lights, a custom setup
can be achieved to allow the end-user to program their own christmas lights. The Raspberry Pi is a high-powered mini-computer the size
of a credit card with built-in wireless internet connectivity and digitally programmable inputs and outputs. The Pi's wireless
connectivity will allow for controlling via the internet and will be accomplished using a mobile app built with Ionic. The system will be
built for private networks and will rely on a wireless router. In this system the mobile app will allow programmable light
patterns with almost an infinite amount of possibilities.

![High-Level System Design](https://raw.githubusercontent.com/ComputerEnchiladas/arduino/master/RGB-LED-via-Serial/High-Level%20Design.jpg)

The design above shows the basic setup. The user begins by using the mobile app and explores programming the led's. When the user has 
defined a light pattern then data is transmitted via web sockets through the router and to a socket server in the Raspberry Pi. The Pi
power's an individual LED to provide visual feedback, and then propagates the signal via Serial communication to a microcontroller;
Arduino Uno. The Arduino will drive an RGB-LED strip which can be extended.

## Prototype
![Prototype](https://raw.githubusercontent.com/ComputerEnchiladas/arduino/master/RGB-LED-via-Serial/Setup.png)
The prototype above will consist of getting the system setup with just one LED. The LED is RGB which means it can provide multiple colors.
With the mobile app the single LED will be programmed with multiple patterns and manage the changing of patterns.

