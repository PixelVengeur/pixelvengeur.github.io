---
layout: post
title: NixOS debuts!
date: 2024-03-04 02:56 +0100
categories: []
tags: [nixos]
image: 
published: false
---

- Declarative
- Reproducible
- Idempotent?
- Hardware conf seperate from software conf
- Backup config? KDE?

### Negatives
- Hot plugging peripherals feels unfinished. I have a USB-C dock and a monitor that I usually plug into my laptop when I sit down with it, and, while unplugging them is seamless, plugging them back in seems to reliably crash the entire Plasma desktop environment. The wallpaper disappears, so does teh taskbar, the cursor icon is locked into the last one it displayed, and I can't interact with anything anymore. When the system boots, the wallpaper is not displayed correctly on the second monitor either.

![The wallpaper is shifted to the right](/assets/img/nixos-debuts/nixos-1.png)
_Second monitor wallpaper at boot_

Two things are to consider:
  - My laptop is a piece of absolute garbage, and its embedded firmware is not only an unsupported buggy mess, but every update I installed after getting it in 2018 broke something. All those symptoms are OS agnostic, it is the laptop that's to blame.
  - It most likely is a Plasma issue, and not a NixOS one. I'll re-test with GNOME to see if it persists.

> These are all KDE issues. No idea why. Switching to GNOME 45 fixed them all.
> It was also only a couple of lines in the config file, thank you NixOS for being like this <3