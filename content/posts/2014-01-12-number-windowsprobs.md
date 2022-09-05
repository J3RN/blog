---
title: "#WindowsProbs"
date: 2014-01-12T17:06:21-0500
expirydate: 2022-09-04
description: |
  J3RN tries to figure out how to save an HP laptop from a boot loop issue. He fails.
draft: false
tags:
- linux
- windows
---

Linux is Great
--------------
It's been a long time since I've used Windows - probably a few years. It's been
even longer since I legitimately cared about fixing Windows problems and didn't
instead opt for wiping the partition and doing a fresh reinstall.

Trust me, I'm a Developer
-------------------------
As a computer scientist, naturally I run into people who confuse "developer"
with "IT Professional" and ask me to fix their computers for them. My general
catch-all for this is "I don't fix Windows problems." In the case they use a
Mac, I expect macOS to lock them out of doing most harmful things to their
system. If they happen to be a Linux user (which, to date, none have been), I
would happily hack on their machine until it has recovered some amount of
usability.  But more often than not, it's Windows. And I don't fix Windows
problems.

The J3RN Way™
-------------
Today it was discovered that if you pester me long enough, you might 
just get me to at least give it a go. An HP 8570w Workstation Laptop. A 
powerhouse, on paper. A monstrous thing with a boot loop issue. 

Alright, let's do this The J3RN Way™. I'll save your important documents,
but we're going to wipe the system and reinstall.

I need to boot from disk or USB and can't. F10 to get to BIOS settings. Well, 
they don't do jack shit unless you disable SecureBoot (according to a forum 
post) and enable Legacy mode (which I can confirm actually does things). 

Alright, we can boot from USB. Let's boot to a Manjaro Linux LiveCD so
I can move all these important files to my external hard drive affectionately 
named "DAISI." Leaping into the file manager (Thunar, I belive it is), quickly 
mount all available drives.

Five Partitions in Search of an Exit
------------------------------------
Wait, all of these just have Linux top-level directories on them... Alright, 
let's crack open some GParted - my favorite last-resort partition manager! It 
complains about the partition table once, and again, and perhaps a third time.
Alright, we're in. /dev/sda appears to have about 5 partitions, mostly in NTFS.
Sounds like your general factory install of Windows. On /dev/sda? Weird.

Alright, /dev/sda4 is a big hunk of NTFS. I can only imagine this is where 
the nice people at HP thought my buddy here would like to store his files.
Let's mount it. "How about 'No!'," screams <code>mount</code>, "This is a RAID 
0 volume, and I can't handle that shit!" I like to paraphrase sometimes. Not
often. Well, I don't know much about RAID either, so I can't do much about
that.

So my friend says he has his Windows system recovery CD. Well, the built-in 
feature tries to load for about five seconds, then just crashes out to a blue
screen with a ":(", then reboots immediately, so it may be worth a try. Good 
work on the sad face, Microsoft. Classy. I liked it. Now all you need is to 
make your system less easy to utterly destroy, and we might be friends.

Booting with the disk, a whole lot of nothing happens. Jumping back into the 
BIOS settings, I search for anything that could be blocking the ability to boot
from disk. Not being able to find anything about it is hardly conclusive - who
knew you needed to enable Legacy mode to boot from a USB drive?

Anyway, I pop out the disk and have a look at it. It is entitled "Applications 
and Drivers" and additionally says "This software comes preinstalled." This 
disk isn't Windows recovery - it's a disk full of manufacturer bloatware, most 
likely. Thanks, HP. That was real useful.

The Windows of Our Discontent
-----------------------------
I sit back in my chair and look at this dysfunctional heap of hardware. I tried
to save some files, and couldn't. I tried to reinstall and couldn't. This
machine was at an intersection of evils: No protection against the end-user
destroying the system, and no way to service the system once it was broken.

So thus I have been brought to my knees. I beg of you, HP, Dell, Lenovo, Asus, 
and other PC hardware manufacturers - Just ship Linux! A new machine with an 
LTS version of Ubuntu pre-installed would simply be a dream.

Oh, and fix your disgusting UEFI interface. It sucks.
