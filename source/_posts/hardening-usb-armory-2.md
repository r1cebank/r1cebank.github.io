---
title:  Hardening USB Armory (part 2)
copyright_author_href: 'https://github.com/r1cebank'
date: 2021-03-16 22:26:15
categories:
- Tutorial
- Hardware
tags:
- security
- usbarmory
- linux
- luks
cover: /covers/usbarmorymkiilayout.webp
---
Earlier we talked about trying to hardening the USB Armory, but given all the information I have found online, I need to figure out some custom way of generating the image so the secure environment can be easily reproduced.

- Custom Kernel + initramfs (for pre-boot LUKS unlock)
- Seperate /boot partition (unencrypted but signed to unlock the rootfs)
- Makefile to create multi-partition images
- Makefile to create LUKS images
- Signed bootloader with the image

## Step 1. Initramfs

This is the hardest part and took me a couple days to figure out, mainly due to I had zero experience with Linux Kernel and I had to read a lot of documentation and outdated documentation to figure out what to do. To learn more about initramfs, here is a super dated documentation from kernel.org, but it should explain everything you needed to know.

[Ramfs, rootfs and initramfs — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/filesystems/ramfs-rootfs-initramfs.html)

[Initramfs/Guide - Gentoo Wiki](https://wiki.gentoo.org/wiki/Initramfs/Guide)

After many trial and errors, I figured out an way to embed the initramfs in the kernel zImage and have it run a prompt during boot.

- Build busybox
- Create the filesystem structure
- Mount devtmpfs and tmpfs
- Make sure /dev/console exist with mknod

At the end, the make target will look like this:

```makefile
initramfs: busybox-bin-${BUSYBOX_VER} init
    mkdir -pv initramfs/{bin,dev,sbin,etc,run,boot,proc,sys/kernel/debug,usr/{bin,sbin},lib/modules,mnt/root,root}
    cp -av busybox-${BUSYBOX_VER}/_install/* initramfs
    cp init initramfs
    chmod +x initramfs/init
    mkdir -p initramfs/dev
    cp prebuilt/${LINUX_VER}/dcp_derive initramfs/usr/sbin
    cp prebuilt/${LINUX_VER}/armoryctl initramfs/usr/sbin
    chmod +x initramfs/usr/sbin/dcp_derive
    chmod +x initramfs/usr/sbin/armoryctl
    cp -av prebuilt/${LINUX_VER}/modules initramfs/lib
    mkdir -p initramfs/lib/modules/${LINUX_VER}-0
    @if test "${IMX}" = "imx6ulz"; then \
        cp prebuilt/${LINUX_VER}/mxs-dcp.ko initramfs/lib/modules/${LINUX_VER}-0/extra; \
    fi
    @if test "${IMX}" = "imx6ul"; then \
        cp prebuilt/${LINUX_VER}/caam_keyblob.ko initramfs/lib/modules/${LINUX_VER}-0/extra; \
    fi
    cd initramfs/dev && \
        mknod -m 622 console c 5 1 && \
        mknod -m 622 tty0 c 4 0
    cp -av prebuilt/${LINUX_VER}/cryptsetup/* initramfs
    echo "leds_gpio" | sudo tee -a initramfs/etc/modules
    echo "led_class" | sudo tee -a initramfs/etc/modules
    echo "mxs_dcp" | sudo tee -a initramfs/etc/modules
    echo "algif_rng" | sudo tee -a initramfs/etc/modules
    echo "algif_skcipher" | sudo tee -a initramfs/etc/modules
    echo "algif_hash" | sudo tee -a initramfs/etc/modules
    echo "af_alg" | sudo tee -a initramfs/etc/modules
    echo "dm_crypt" | sudo tee -a initramfs/etc/modules
```

## Step 2. Init script

After the initramfs is loaded to ram and mapped to root, the kernel will run the init script in the root directory. The init script is responsible for loading modules, running pre-boot actions and at the end `switch_root` to switch out the current root with the / on the file system. Here is am example of the init scipt with LUKS unlocking:

```sh
#!/bin/busybox /bin/sh
export PATH=/usr/bin:/usr/sbin:/bin:/sbin

# Mount the dev directory
mount -t devtmpfs none /dev

# Mount the rest of the directory
mount -t proc none /proc
mount -t sysfs none /sys
mount -t debugfs none /sys/kernel/debug
mount -n -t tmpfs    tmpfs    /run

echo -e "\nBoot took $(cut -d' ' -f1 /proc/uptime) seconds\n" > /dev/kmsg

echo -e "********** Starting custom unlock script **********\n" > /dev/kmsg

echo -e "**********    Loading kernel modules     **********\n" > /dev/kmsg

echo -e "Loading: dm-crypt\n" > /dev/kmsg
modprobe -q dm-crypt
echo -e "Loading: af_alg\n" > /dev/kmsg
modprobe -q af_alg
echo -e "Loading: algif_hash\n" > /dev/kmsg
modprobe -q algif_hash
echo -e "Loading: algif_skcipher\n" > /dev/kmsg
modprobe -q algif_skcipher
echo -e "Loading: algif_rng\n" > /dev/kmsg
modprobe -q algif_rng
echo -e "Loading: mxs_dcp\n" > /dev/kmsg
modprobe -q mxs_dcp
echo -e "Loading: leds_gpio\n" > /dev/kmsg
modprobe -q leds_gpio
echo -e "Loading: led_class\n" > /dev/kmsg
modprobe -q led_class

echo -e "**********   Kernel modules loaded       **********\n" > /dev/kmsg
armoryctl led white on
sleep 2

armoryctl led blue on
printf {LUKS_PASSWORD} | cryptsetup luksOpen /dev/{ROOTFS_DEV}p2 rootfs -d -
armoryctl led blue off
mount /dev/mapper/rootfs /root

armoryctl led white off
# Switch root
exec switch_root -c /dev/console /root /sbin/init
```

You see we are loading modules and turning on LED, then run cryptsetup to open the partition and later mount it, at the end we call `switch_root` to start the actual init process

## Step 3. Cryptsetup

This one is a hard one, there are some guides online to build this into a static executable, meaning it can be copied and put on the initramfs without the need to copy all the dynamic modules. I had zero success in that  and decided to make the dynamic executable work.

So I figured, what if I just copy all the shared modules over and maybe it works? So that's exactly what I did. I used a tool called ldd
{% asset_img image-1.png cryptsetup required libraries %}
So this tool helped me to identify all the dynamic libraries it needs to use, so I copied them over to the initramfs and it works right?

No

During testing I realized the cryptsetup will allow me to unlock the drive, it just quietly quits. After checking verbose outputs and googling, I realized I need to load all the kernel modules that is required by cryptsetup, one of them is essential: dm-crypt.

So I copied it over and it worked!

So up till this point, we built an initramfs with all the kernel modules we need and is able to unlock the LUKS drive and boot from it. But USB armory is a usb device, there is not way we want the users to connect the debug tool and unlock it manually everytime. (some might think this is an added security feature, but I assure you user input during secure boot process is always never good idea.

We could however spin up a SSH server like dropbear and unlock the drive like that, but I want to leverage the DCP processor and secure processors on the device. 

## Step 4. DCP

The imx6ullz SoC on the USB Armory has an data co-processor, and it can handle various cryptographic functions. What makes me interested is one of the feature inside the DCP that can be useful to help us create unattended luks unlock.

During manufacturing an key is fused inside this dcp called OTPMK, this key is inaccessable by anyone and the DCP is able to use it to encrypt data. During a secure boot setup, the OTPMK is used, if DCP detects its not in a secure booted enironment, the Non-Volatile key is used.

Here is what I have planned, since we all need to boot with secure boot enabled, how about we embed the DCP encrypted keys in initramfs and let the DCP decrypt it during boot and that result will be out LUKS key.

But the key is exposed and can be read my anyone!

Well, yeah, but its not like they can use it for anything, without DCP to decrypt the key, there is no way to use it to decrypt the LUKS partition. We can also forget attacker injecting some script to dump the DCP decrypted key since once they modify the bootloader, zImage, the secure boot check will fail and the SoC will refuse to run.

There is a case where the attacker can obtain the device and boot it, then it bruteforce the SSH key on the device. Well, if they can get in that way, I guess you probably don't have a habit of using strong secure passwords and I think your pre-boot LUKS password will be weak as well.

This concluded my journey to hardening my USB Armory Mk.II, its really an awesome device and I had a lot of fun tinkering with it. If you are interested you can find everything in the GitHub repo below.

[r1cebank/usbarmory-debian-base_image](https://github.com/r1cebank/usbarmory-debian-base_image)
