---
title: Sega Nu keychip (part 1)
copyright_author_href: 'https://github.com/r1cebank'
date: 2020-12-03 20:58:06
categories:
- Gaming
tags:
- arcade
- hardware
- security
- sega
cover: /covers/nukeychip.bmp
---
## Introduction
In the Sega Nu systems, system boot and application start is gated by the mysterious "keychip". A lot of systems sold online do not include the keychip since they are property of Sega and needed to be returned once the operator terminate contract with Sega. I was lucky enough to find two keychip being auctioned and I want to document my reverse engineering efforts. Hopefull later people are able to use this information to reconstruct keychip for the game they want to launch and keep these arcade machine running.

Sega Nu is the machine behind Hatsune Miku Project Diva Arcade Future Tone (PDAFT). 

This is an machine I have always wanted and I have enjoyed so much playing while I was in Japan.

I recently acquired a Sega Nu Future Tone with keychip, the following documentation will include what attempts I have made to crack the machine and reverse engineered its parts.

Here is what the keychip looks like on the front, there is the part number and serial and the SBZV indicating this is for PDAFT.

{% note info simple %}
Sega uses the 4 letter code to indicate which game the keychip is meant to be used for, when you acquire the keychip please make sure you are getting the right one for your game.
{% endnote %}

{% asset_img nukeychip.bmp Nu keychip %}

## Components
* [Winbond 25X40CLS1G SPI Memory](https://www.winbond.com/resource-files/w25x40cl_f%2020140325.pdf)
* NXP 7001C Secure Processor
* [NXP LPC11U14F ARM Processor](https://www.nxp.com/docs/en/data-sheet/LPC11U1X.pdf)

## Interfaces
* USB for host communication
* Custom drivers located in the system drive called `sgkeychip.sys`
* During USB sniffing, it looks like its communicating over serial

This is what the internal looks like for the keychip

{% asset_img pcbfront.bmp PCB front %}

and the back

{% asset_img pcbback.bmp PCB front %}

Using this key, when the game is launched on the Nu, the game program is able to successfully decrypt the game partition image and you are able to dump the entire content of the drive if you hacked the Nu to allow keyboard and mouse control.

It is also speculated that the key will self destruct when plugged in normal machines, that is unlikely but since this is a rare item I will not analyze it on the regular machine. To reverse engineer the protocol this device I am going to use USB sniffer to capture the traffic between the Nu and this keychip hoping to figure out how it works and if there is a way to create our own key.