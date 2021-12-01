---
title: Hardening the USB Armory (part 1)
copyright_author_href: 'https://github.com/r1cebank'
date: 2021-02-26 23:00:32
categories:
- Tutorial
- Hardware
tags:
- security
- usbarmory
- linux
- luks
cover: /covers/usbarmorymkii.webp
---
Since last time I wrote about the USB Armory II, I have spent a considerable amount of time working on the device. I have to say it is a very capable device, able to support many tools that I want to run on it. But working with USB Armory with Secure Boot turned on, I have found some issues that is not documented in the wiki.

## Problem

The secure boot process will only protect the initial boot process, validating the kernel, but its still easy to modify the filesystem and inject files in to the filesystem. The rootfs is also not encrypted so the content is not safe, if you store a LUKS key to use for auto unlock encrypted partition it can be exposed.

## 'Kinda ok' solution

The first step is to use LUKS to encrypt the SD card and since I use it to store files required to various service at start up, I need to setup auto unlock when the system boots up. But since the rootfs is not encrypted, storing the key inside the rootfs is not making it more secure, anyone know how to read a crypttab will be able to locate the keyfile and unlock the drive. Because the device included a crypto processor with sealed keys, another approach to make this work is to store the encrypted keys on the drive and during mount, call the crypto processor to decrypt it and unlock the drive. 

But remember anyone is able to mount and tamper with our rootfs right? They can just easily slip a script for it to be ran at startup and call the crypto processor, decrypt the keys, and dump the decrypted keys to the rootfs.

## The solution

Since we already have secure boot, the rest of the task to ensure security of the device lies with if we can lock down the rootfs. If we are able to use LUKS to encrypt the partition and use kernel + initramfs to validate and unlock the drive. We are able to prevent regular-skilled attack on the device. Because they are not able to bypass secureboot and install their own initramfs and kernel, they cannot tamper with the LUKS unlock sequence and if we store encrypted password there and use DCP to decrypt and unlock LUKS we are able to prevent any tamper to the unlock sequence to dump the password. To add an extra layer of security we can also enable boot ssh unlock using dropbear, then no key is stored on the device encrypted or plaintext, hence prevent unlock even if the device is lost and attacker figured out a way to decrypt the embedded encrypted keys.

Example using dcp_derive to feed decrypted keys to cryptsetup to unlock the partition.

    #!/bin/sh
    
    /usr/bin/dcp_derive dec $(cat /etc/dcp_pass) 27 | cryptsetup luksOpen /dev/mmcblk0p1 mmc0_crypt -d -
    
    mount /dev/mapper/mmc0_crypt /mnt/secure
    

## Is it it possible?

I asked about this in the google group for USB Armory and I got an response from one of the developers.
{% asset_img image-10.png developer response %}
So it appears that it is totally possible to do this, its going to be complicated, but its possible.

They didn't provide me with any more information on how to achieve this but I guessed it would be the following:

- Enable Secure Boot with U-boot or armory-boot (armory-boot doesn't support initramfs, so we probably have to stick with u-boot)
- Have the signed bootloader to verify kernel image + initramfs
- Initramfs will run init first to decrypt the LUKS roofs and run root_switch after

I am going to try to test with another usbarmory with u-boot (if I am not able to make armory-boot to support initramfs)
