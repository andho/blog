---
date: '2013-02-19 20:41:34'
layout: post
slug: Installing ubuntu as o
status: publish
published: false
title: Log monitoring with graylog2 on ubuntu 10.10
wordpress_id: '105'
categories:
- Server Administration
tags:
- analytics
- gelf
- gelf client
- graylog2
- log monitoring
- server administration
- ubuntu
---

.. role:: code
   :class: inline-code

installing ubuntu as efi
why? I installed windows and my ubuntu installation wouldn't boot.
I could boot with supergrub2 disk
grub-install fails.
So i thought i would go with rod smiths guide and boot ubuntu from GPT partition.
Converted disk to GPT. Installed grub-efi
By default it installed grub-efi-i386.
When i tried grub install again, it complained about not finding grub-pc, which it shouldn't even try to find because it should
try to find grub-efi.

Mention something about efi stub loading and my ubuntu being 32 bit so that didn't work.

Emailed rod smiths for help. He told if my ubuntu install was 32 bit, I should explicitly specify that I want to install grub-efi-x64.
He also said that the grub-pc not found issue might be solved by purging all grub stuff.
I purged grub, then installed the grub-efi-x64 package. And ran grub install.
two problems.
I have to specify the target and install_DEVICE.
target is 'x86_64-efi' and install device is disk where efi is located. ESP?
This asked me if I'm okay with removing the /boot/grub folder. I said what the hell, just do it? I had backup.
Installed grub, rebooted.
Ubuntu logo shows up in refind screen. Go into boot. Show a grub shell. Ofcourse, I don't have any grub configuration.
Ofcourse I had backup, but me being "stupid" I tried to mount the disk image in OS X, but I didn't have losetup, so I tried to boot another linux live cd like Ubuntu or Damn small linux. This is the most frustrating this to do on a mac, creating a live cd. I don't remember how i got ubuntu live cd to work before, but I couldn't get it working this time. At some point my pen drive stopped working. I wasn't able to boot using an external hard disk.

Then it hit me. I could boot ubuntu from grub shell.

i followed the guide from tuxers.com/instigating-a-manual-boot-from-the-grub-prompt/

mounted the partition backed up partition to get the grub config. Used this to mount
the backup img: http://askubuntu.com/questions/69363/mount-disk-device-image/69447#comment321607_69447

but didn't use the grub.cfg in the backup.

Instead used `sudo grub-mkconfig -o /boot/grub/grub.cfg`. This created a new grub.cfg from what grub could find in the available
partitions. I don't why grub-install didn't do it, after deleting the already exising one (I was assuming it would).

After this I rebooted. It works. Next step, installing windows on GPT partition.
