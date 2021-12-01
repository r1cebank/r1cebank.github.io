---
title: How Sega Nu works (part 1)
copyright_author_href: 'https://github.com/r1cebank'
date: 2021-02-09 21:58:49
categories:
- Learning
- Arcade
- Sega Nu
tags:
- arcade
- windows
cover: /covers/sega-nu-pdaft.jpg
---
Is been couple month since I started working on the Sega Nu, this is the first arcade unit I have worked on. Working on the system has totally changed my understanding of how those system works.

Before start working on the Nu, my experience with it is very limited. I started to have interest on this system after I learnt this is the system used for my favorite arcade game: Hatsune Miku: Project Diva Arcade Future Tone. I always wanted to own one of the cabinet and make it work offline in my future home.
{% asset_img 1498579824.8449.jpg Sega nu cabinet %}
The documentation on this system is very limited, there are couple articles written about the system's spec, and there are one or two forum posts on the system's disassembly.

I also found one japanese blog that kinda documented the system's internal workings, but not very detailed. [SEGA Nu - 日記β](http://d4.princess.ne.jp/blog/art/00007.html#cm)

But combining all the info I got from forum posts and blog posts, I was able to paint a picture what the system is like before I got my hands on one:

- It is running Windows 8 Embedded
- I has one OS SSD and one Data HDD
- Some partitions are encrypted with Bitlocker
- The game images were not in encrypted partition but require keychip to mount them
- The bitlocker volumes are autounlocked at boot
- The keychip is an USB dongle that includes a secure element
- Keychip might self-destruct if plugged in a normal machine

I was able to get my hands on a Nu machine with keychip for PDAFT (Project Diva Arcade Future Tone), and my initial plan to understand the machine is:

- Crack bitlocker
- Extract SEGA device drivers (JVS, Keychip, PCIe, etc)
- Extract the game software
- Reverse engineer the game software
- Reverse engineer the keychip

After I got the machine, I was attempting to extract the bitlocker hashes from the machine, but got no luck since its a TPM locked drive and not password locked. So I changed strategy to use TPM sniffing to extract the FVEK (Full Volume Encryption Key) during a normal system bootup.

The system were very dirty and I managed to clean it and solder trace wires on the TPM chips.
{% asset_img image.png TPM chip trace %}
But during a bootup, the system went directly to recovery mode and the subsequent bootup failed, the motherboard seems to be dead. (No Video, no POST beep)

I was extremely upset due to the death of the motherboard, and was trying to source another Nu machine.

Was lucky to get my hands on another machine after a couple weeks searching, so after shipping and FedEx delays, I got my hands on the new machine and an initial boot shows success, the machine is able to mount the game image and give me an error that makes sense to my current setup.
{% asset_img image-1.png initial boot %}
So my plan has changes this time around:

- Extract bitlocker keys with PCILeech
- Unlock the drives
- Dump the stuff

PCILeech has been very successful, and I was able to get all the recovery keys for the bitlocker volume here. This is a great step forward since right now I can freely mess with the filesystem and hardware without worrying getting stuck in recovery mode.

After backing up the system drive, I decided to decrypt all the volumes and let the sytem to boot without bitlocker. (I thought this will be fine since the bitlocker volumes are unlocked automatically anyways, to the OS they are just regular drives)
{% asset_img image-7.png decrypt all the drives %}
I didn't know at that time, this decision is going to be catastrophic, and since I didn't make raw backups of the encrypted drive, there is no easy way to recover from this.

When I booted up the system with all drives decrypted, everything seems to be normal, Windows launched, sgpresboot.exe launched and sgboot.exe launched with its `STEP00__01` text telling which stage of initialization the system is in. Usually the system will jump from step 3 straight to step 15 and later will launch the game executable. This time it was stuck at step 4, after couple minutes of waiting, it displayed an message that I have never expected.

    FORMAT DISK
    DO NOT POWER OFF SYSTEM
    

{% asset_img image-4.png me not realizing what that message means %}
Upon  reboot, I noticed the system is stuck at this reboot look, everytime it will reformat the last partition of the SSD, enable bitlocker on it, and reboot. I thought the system is gonna reencrypt bitlocker on all drives which is fine. To my suprise this is not the case, now the main bootstrap executable will not run anymore and will keep formatting the last drive and reboot.

Now I am in posession of a machine that will not launch the game executable, it will reboot forever. I have the decrypted backup of the OS, but because it has formatted the drives, recoverying is not trival.

I tried formatting drives and encrypt them and reboot, the systems seems able to recover. Later I realized there is a file write filter enabled that is redirecting all my write calls.

Even after disabling the filter and reencrypt, I am still not able to restore the system back to its original state. I still haven't given up, but I have learnt much more about the system now.

- It is running Windows 8.1 Embedded Industry Standard
- Keyboard filter is enabled and filter all hotkeys
- [MMCSS](https://docs.microsoft.com/en-us/windows/win32/procthread/multimedia-class-scheduler-service) is enabled to give priority to multimedia applications.
- Enhanced Write Filter (EWF) is enabled on system drive and data drives to prevent change.
- At boot, C:\Windows\SEGA\System\sgpreboot.exe is launched and will execute sgboot.exe at SystemUser privilage
- SystemUser's password cannot be changed since sgpreboot.exe uses an algorithm to calculate the password and call LaunchProcess with password, changing SystemUser's password will cause sgpreboot.exe to crash
- SystemUser's password can be found in the C:\Windows\SEGA\tmp\AUTOUNATTEND.XML, hashed with base64
- Bitlocker cannot be disabled, sgboot.exe will check the status (confirmed in the disassembly) and will treat the device as not initialized if found bitlocker disabled
- A bug in the initialization will cause the machine to never able to reinitialize the bitlocker drive correctly and system will caught in a loop (boot->check->format->reboot)

With the above knowledge, I will make sure to backup the encrypted drive before start working on the machine so I will always have a way to recover in case of a mess up.
{% asset_img image-8.png back up all the drives %}
Now I am off to buy a new Nu.
