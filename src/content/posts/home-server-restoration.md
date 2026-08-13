---
title: Home server Restoration
published: 2026-01-04
description: 'How I recovered from a disk failure'
image: ''
tags:
- self-hosted
category: ''
draft: false
lang: ''
---
What a way to start the new year!

Disk on the thinkpad finally throws a SMART error for imminent failure (upon further inspection it was because the number of replaced sectors was above the threshold. I have a backup of the files but don’t want to spend time configuring the system again. Decided to clone the drive to a new one. I made a basic ext4 Linux partition on the new drive in MBR (same as the source drive: thinkpad doesn’t support UEFI).

# Clonezilla
Recommended software by the community for imaging and cloning. Decide to do a direct disk-disk clone since I don’t really need an image (will keep this imaging in mind for backups: got to look at how to optimise the size).

Initial attempt to clone failed with bad sectors being reported. Tried again in expert mode with the `—rescue` flag enabled. The operation did not complete due to some sectors of the block being inaccessible. (let it run for 3 hours for good measure but no dice)

# ddrescue
Another community recommendation which has better algorithms for dealing with these bad sectors compared to clonezilla.
Ran it twice. Once to generate the log file and once again to try and recover data from the bad sectors. The good part is that the bad sectors were very small (1024 bytes on a 320GB HDD) and I was able to recover 99% of data.

# Grub Trouble
After the cloning was successful, I tried booting into the os from the new drive. However, the boot failed with the grub text repeating on screen and not allowing me to input commands. The drive did not mount in my live iso and `dmesg` showed me an error related to geometry which is caused due to an issue in the partition. I fixed it by running `e2fsck`. My fedora live iso did not have `grub-install` so I used `grub2-install` and that allowed me to boot into grub. I set the Linux and initrd variables and booted. It worked! My system was operational again. Created a new grub config and reinstalled grub. Rebooted and everything is working great.

Time To Recovery:  14 hours (most of it spent just waiting for cloning to complete)

## References
1. https://unix.stackexchange.com/questions/115698/fix-ext4-fs-bad-geometry-block-count-exceeds-size-of-device#293462
2. http://superuser.com/questions/1237684/ddg#1512531
3. https://www.av8n.com/computer/htm/grub-reinstall.htm#sec-grub.cfg
4. https://linuxconfig.org/introduction-to-grub-rescue