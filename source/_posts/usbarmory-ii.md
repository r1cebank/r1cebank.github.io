---
title: USB Armory II
copyright_author_href: 'https://github.com/r1cebank'
date: 2021-02-22 07:54:42
categories:
- Technology
tags:
- security
- hardware
- linux
- usbarmory
cover: /covers/usbarmorymkii.jpg
---
While browsing Crowd Supply I found a very interesting device called USB Armory. Its a tiny security minded computer in the form factor of a tiny USB stick. The hardware specs is not very amazing but compared with similar device like Raspberry Pi Zero, its not that bad.

## Hardware

- **SoC**: NXP i.MX6ULZ ARM® Cortex™-A7 900 MHz
- **RAM**: 512 MB DDR3
- **Storage**: internal 16 GB eMMC + external microSD
- **Bluetooth module**: u-blox ANNA-B112 BLE
- **USB-C ports**: DRP (Dual Role Power) receptacle + UFP (Upstream Facing Port) plug, USB 2.0 only (*no video support*)
- **LEDs**: two
- **Slide switch**: for boot mode selection between eMMC and microSD
- **External security elements**: Microchip ATECC608A + NXP A71CH
- **Physical size**: 66 mm x 19 mm x 8 mm (without enclosure, including USB-C connector)

![USB Armory Image](https://www.crowdsupply.com/img/c49b/usbarmorymkii-angle-01_png_open-graph.jpg)
[CrowdSupply Link](https://www.crowdsupply.com/f-secure/usb-armory-mk-ii)


The proposed applications are:
- Mass storage device with advanced features such as automatic encryption, virus scanning, host authentication, and data self-destruct
- Hardware Security Module (HSM)
- OpenSSH client and agent for untrusted hosts (e.g., Internet kiosks)
- Router for end-to-end VPN tunnelling
- Tor bridge
- Password manager with integrated web server
- Electronic wallet
- Authentication token
- Portable penetration testing platform
- Low-level USB security testing

---

After getting it, I found the most useful usecase for me is to run a password manager. After turning on secure boot, you are required now to sign all the bootloaders, serial downloaded binaries. Here is what I think about the USB Armory II:

The integrated network setting in the default images made it very easy for plug and play use. I didn't have much issue with connecting it with SSH. There are only 1 or 2 instances where the DHCP server didn't assign IP and I had to manually assign. The network sharing is very easy and only require one line.

    sudo /sbin/iptables -t nat -A POSTROUTING -s 10.0.0.1/32 -o wlp3s0 -j MASQUERADE

iptable to allow routing from usbarmory to host
The GitHub documentation is kinda generic and outdated, it's documentation on most part of the device has been acurate, sometimes you just need to adapt them.

Since this is a niche device, given the steep price, its not being sold well so the community is quite small. The latest questions/issues on GitHub are left unanswreed and the Google Group is quite small as well. This is a highly DIY device and require a lot of existing knowledge around Linux and software.

What I find a lot of people online criticizing this device is mainly around its similarity with the Raspberry Pi Zero W and the price. For $155, its very expensive compared with the Pi Zero, but I do not feel this is a fair comparison.

## Comparison with Raspberry Pi Zero

#### Raspberry Pi is not a secure device

I feel the USB Armory's main selling point is its security features. You are able to configure secure boot, use secure element, and use cryptographic accelerator. I am not saying you should get this because it got all the secure features, but you really should treat this as a difference device.

#### USB Armory is less powerful but much portable

I have used the Raspberry Pi Zero with USB shield, but its is a lot bigger than the USB Armory, its useful but looking a lot bulkier than the USB Armory.

#### Raspberry Pi is made to be open

The size of the Raspberry Pi Zero is not a bad thing, the space is mainly for its 40-pin IO header. The device is made to be open and to be hacked.

I am really excited to see what this little device can do, and if you are interested in DIY, and security, you should get one too.
