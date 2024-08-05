---
title: "Why Fedora Silverblue?"
layout: post
date: 2024-03-05
description: |
  I've been using Fedora Silverblue for a few years.  When a friend wanted to try a Linux distro, I ultimately landed on recommending Silverblue.  This is why.
tags:
- linux
aliases:
- /posts/2024-03-05-why-fedora-silverblue
---

This post is largely based on a write-up I gave to a friend about why I recommended Fedora Silverblue as their first Linux distribution.  I assume no knowledge of Linux here.

I've been using Linux-based operating systems (termed "Linux distributions", "Linux distros", or just "distros") since about 2011.  I've hopped between many distros, and for some years would switch distros every few months.  Lately, though, I've been quite content to use only one distro for my daily desktop use, **Fedora Silverblue**.

![Fedora Silverblue](/images/silverblue.png)

## Background

By most reports, the most popular Linux distro is Ubuntu.  I've had mixed experiences with Ubuntu, and don't recommend it to people.  Another popular—perhaps the second most popular—Linux distro is Fedora Workstation.  Fedora Workstation is pretty good, and I used it for several years before switching to Fedora Silverblue, which is its immutable variant.

## Why Silverblue

Silverblue is an "atomic" or "immutable" Linux distro.  These two terms mean the same thing in this context, which is "cannot change".  And this sounds like a really stupid idea—what good is a computer if you can't change anything?—but that's not the full story.  Only the *operating system bits* can't change.  So you can run programs and games, change your settings, put files in your `Documents` or `Downloads` folders, etc, all exactly as you'd expect.

The "atomic" bit matters for *OS updates*.  Sometimes—not often, but sometimes—Windows Update will bork your Windows install.  The same thing happens with macOS and its update mechanism.  Same with most Linux distros.  However, Silverblue is different.  It uses "snapshots" of the operating system bits and snapshots cannot be changed.  When you start your computer, the most up-to-date snapshot is used.  When the operating system updates, this is done by creating a new snapshot with the contents of the running snapshot + updated code.  This can happen in the background while you're using your computer, and when you restart your computer the snapshot with the updates in it is used and you're :sparkles:automatically:sparkles: up-to-date.  Ergo, there's never any sitting and staring at the blue "Wait while we install updates" screen like Windows, macOS, and many Linux distros have.

Also, if an update has somehow gone terribly wrong, you can tell Silverblue to "roll back" to the previous snapshot, and the OS is back to where it was pre-update.  I think I've had to do this once or twice, it's no big deal (but a failed update is a nightmare for almost any other operating system).

You can also "rebase" atomic distros to other atomic distros.  There's one terminal command you run, and Silverblue will install that other distro (e.g. Fedora Kinoite, pictured below) to a new snapshot to be started the next time you reboot.  You can bounce back and forth between atomic distros in this way which is kinda fun (you get to experience other kinds of Linux distro) but also kinda annoying (you have to keep telling it which distro you want to use next, because it'll assume you want to keep using the same one).

![Fedora Kinoite](/images/kinoite.jpg)

## The Downside

Linux distros have been historically bad about differentiating what is an "operating system bit" and what is a "user application".  For instance, in Fedora Silverblue, Firefox is considered an operating system bit—and so only updates between reboots—but Calendar is considered a user application and can update whenever through Software.  Pretty strange, right?  It used to be much worse.

Operating system bits are installed and updated through `.rpm` files in the Fedora variants; and this was, for many years, the *only* recommended way to install *any* software on Fedora.  And so, if you go to install (e.g.) Google Chrome from https://www.google.com/chrome, Google will offer your a `google-chrome.rpm` file.  The trouble with Silverblue is that, while you *can* install software from `.rpm` files, it's pretty annoying to do and that software will only be available after a reboot since it's considered a new operating system bit.  This is obviously pretty sucky and furthermore not recommended by the people who make Silverblue.

Instead, the people behind Silverblue prefer that you use their new way for installing user applications, *Flatpak*.  Applications packaged with Flatpak (termed "flatpaks") are managed separately from the operating system bits *and* have several other benefits aside from not having to reboot to install them.

Flatpaks are distributed through marketplaces, the biggest of which is [Flathub](https://flathub.org), which is where you can find Discord, Spotify, Steam, and a bunch of other applications you'll probably want.

The easiest way to install software from Flathub is Software.  This is really another downside, because Software is terribly slow and has a habit of throwing up arcane error messages.  On a fresh Silverblue install, Software will present you with a "Failed to install English" error when first launched.  This error is probably safe to ignore, but sometimes the errors shown by Software are actually important—it's pretty difficult to know which are inane and which are serious if you're new to Linux in general.

I'm hoping that Software gets better with time.  For now, I install flatpaks via the terminal myself, which is a bit intimidating if you're not accustomed to it.

## In Sum

I use and recommend Fedora Silverblue because it's stable and flexible.  It has allowed me to recover my OS from all kinds of gnarly messes while simultaneously giving me the flexibility to try other distros without going through the whole painful partition-and-install process.  Despite its shortcomings, I hope that Fedora Silverblue makes for a pleasant first Linux experience.
