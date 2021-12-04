---
title: Creating luks partition from existing partition
copyright_author_href: 'https://github.com/r1cebank'
date: 2021-02-01 06:45:57
categories:
- Technology
- Learnings
tags:
- linux
- security
- luks
cover: /covers/luks.webp
---
I've been building a self-contained server from an Intel Nuc, recently I decided to move my dropbox to self-hosted Seafile solution. One of the problem is, I did not encrypt the external storage when I did the system setup. When googled about encrypting existing partitions, I came up with a solution.

**Make sure you back up all of your data**

1. Shrink the partition using resize2fs (make sure calculate the new block size and leave around 32M)
2. Run `cryptsetup reencrypt --encrypt /dev/sdXY --reduce-device-size 32M`
3. The re-encryption is going to be slow, on my SATA mechanical drive its around 12MiB/s, make sure you use screen

When I ran the above command, it reached around 80% after two days, but I accidentally exited the process and stopped the re-encryption! Oh No!

Of course I did not back up all my data (not because I am careless, all the data there is recoverable by re-downloading). I tried to mount the drive, it worked but its definately missing data. After carefully reading the man page for `cryptsetup` I discovered an command to resume the encryption.

`cryptsetup reencrypt --resume-only /dev/sdaXY`

Run the above command if it your computer crashes or the encryption process is interrupted. (Don't count on this tho, back up, back up!)
