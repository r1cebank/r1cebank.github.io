---
title: Sega Nu Preservation
copyright_author_href: https://github.com/r1cebank
date: 2023-12-22 16:21:27
categories:
- Gaming
- Arcade
tags:
- arcade
- windows
- sega
- reverse-engineering
cover: /covers/bitlocker.png
---

Recently I've been asked by many people about how to preserve their SEGA Nu/Alls console, and I want to take this chance to leave some instructions so people can attempt before reaching out and waiting for me slowly reply them 🥹.

As we all know, the SEGA Nu or Alls are protected by Bitlocker with TPM + Recovery Key. Since there is a risk of system configuration change that triggers Bitlocker recovery, many of you (including myself) want a way to crack Bitlocker first and ensure in the event of TPM wipe, we are still able to unlock the drive with Recovery Key.

From the questions I've received here are two options I present to people that want to preserve their SEGA arcade consoles.

- Recovery Key Retrieval
- Re-Enroll TPM

The typical route is recovery key retrieval then re-enroll any device TPM (rekey)

## Console Difference

{% tabs test1 %}
<!-- tab SEGA Nu -->
SEGA Nu Recovery Key is per machine, and you will need to extract the key yourself in order to go to the next step.
<!-- endtab -->

<!-- tab SEGA Alls -->
All SEGA Alls machine share the same recovery key, you can skip to Re-Enroll TPM step if you wish.
<!-- endtab -->

{% endtabs %}

## To extract the FVEK (Full Volume Encryption Key)

Before we can extract the recovery key of the Bitlocker drive, we need to extract the FVEK of the drive.

To get the recovery key from the machine here is what you need:
1. A working machine, (recovery keys are set at the initial boot of the nu, they are machine specific) we need to extract the key from TPM
2. Once a working machine and able to boot to the game screen (keychip error or something else), we can use either a PCI device to extract the system memory, or using the reboot method to extract.
3. PCI device you will need is probably a PCILeech compatible device (I recommend https://shop.lambdaconcept.com/home/50-screamer-pcie-squirrel.html), once live system memory is extracted the key and recovery key can be found using data recovery software.

Once you have the memory dump, its time to profit!

{% asset_img profit.jpg profit meme %}

I've made a easy to use docker image for you to use, just use the command below and make sure you mount the directory with the memory dump.

```
docker run --rm -v [memory dump dir]:/data:ro r1cebank/volatility --plugins=/plugins -f nu2.raw --profile Win8SP1x64 bitlocker
```

The tool will give you an output like this:

```
====================NU1====================
Volatility Foundation Volatility Framework 2.5

Address : 0xfa8004d88aa0
Cipher  : AES-128
FVEK    : 1fc9e728e2776df0390ae2a7d9d37b06

Address : 0xfa8004d88d50
Cipher  : AES-128
FVEK    : 1fc9e728e2776df0390ae2a7d9d37b06

Address : 0xfa8004d349f0
Cipher  : AES-256
FVEK    : 672be81c6dd56199bafad55aab4cc4d3fbb313edd64f9e2db457149422de6d58
```

The tool will attempt to find keys based on the profile, the highest probable key is listed at the top.

Once you have the FVEK, you can mount the drive with bdemount or dislocker, I won't list details on how to use those tools here.

https://github.com/Aorimn/dislocker
https://manpages.ubuntu.com/manpages/xenial/man1/bdemount.1.html

After you have the FVEK, the fun begins.

## To get the Recovery Key

{% note danger modern %}
PLEASE BACKUP DRIVE BEFORE THE STEPS BELOW
{% endnote %}

Prerequisite: You will need a read-write mounted partition.

1. Please backup `C:\Windows\SEGA\System\sgpreboot.exe` and replace it with cmd.exe under system32
2. Unplug drive and put it in Nu matched with the drive, bitlocker will auto unlock and boot into windows
3. Command prompt will be shown with SystemUser privileges.
4. Use `runas` to launch another shell with SystemUser using password {% btn https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc771525(v=ws.11),Microsoft Learn (runas) %}
5. Execute `manage-bde -protectors C: -get`
6. This gives you the numerical recovery key which you can use.
7. Plugin the drive in another system and restore the backed up sgpreboot.exe

## To re-enroll TPM

If you want to migrate the disk to a different machine, you will be faced with the Bitlocker recovery screen every time you power up the machine. To eliminate this, you need to re-enroll TPM on the drive to ensure the new machine's TPM can unlock the drive.

{% note danger modern %}
PLEASE BACKUP DRIVE BEFORE THE STEPS BELOW
{% endnote %}

1. Please backup `C:\Windows\SEGA\System\sgpreboot.exe` and replace it with cmd.exe under system32
2. Unplug drive and put it in Nu matched with the drive, bitlocker will auto unlock and boot into windows
3. Command prompt will be shown with SystemUser privileges.
4. Use `runas` to launch another shell with SystemUser using password {% btn https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc771525(v=ws.11),Microsoft Learn (runas) %}
5. Execute `manage-bde -disable` {% btn https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/manage-bde-protectors,manage-bde %} to remove existing key protector (TPM)
6. Execute `manage-bde -protectors c: -add -tpm` to enable TPM protector on your new machine
7. Execute `ewfmgr C: -commit` to write the overlay data on protected volume. {% btn https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/customize/unified-write-filter,Microsoft Learn (ewfmgr)%}
8. Reboot to verify no recovery key is needed.

Once we checked that no recovery key is needed, you can plugin and restore the sgpreboot.exe back and boot your machine with the game :D