---
layout: post
title: Using SSH in Windows without PuTTY
date: 2024-04-11 00:19 +0200
categories: []
tags: []
image: /assets/img/windows-ssh/cover.png
published: true
---

A short and sweet brief on how to use SSH commands in Windows Terminal (available in Windows 10 and up), without the use of a third-party terminal, like PuTTY.

## Why do we use PuTTY?
Because it's easy to recommend, and to use. It's a historical piece of software that has been present for so long that it is a staple of forum posts, Reddit comments and other YouTube videos when tackling SSH connections. So why change that habit?

I personally like having features embedded in my operating system. While I appreciate how versatile and customisable a Linux installation can be, there are things that are not present in barebones installs that I need, which is in part why I learned to use preseed files to [automate my Linux installations]({% post_url 2024-02-29-automating-debian-install %}). On the contrary, there are features built into Windows that are available, but not turned on by default, which leaves a spot open for third-party software to come into play, and an SSH client is one of those features.

## Turning on the Windows SSH client
The procedure is the same for Windows 10 and 11. The screenshots were capturd on Windows Server 2022, which is visually the same as Windows 10.

1. In the search bar, look for "Optional feature" ![alt text](/assets/img/windows-ssh/search bar.png){: w="500"}

2. Click *Add a feature* and look for OpenSSH **Client**, and *Install*
![alt text](/assets/img/windows-ssh/openssh client.png){: w="400"}
![alt text](/assets/img/windows-ssh/installed.png){: w="400"}

    > Installing the OpenSSH **Server** will not grant you the ability to use the Terminal to initiate SSH connections, but will make your machine able to respond via SSH. It might be useful, but not what we're after here.
    {: .prompt-info}

3. Open a new command prompt/powershell/Windows Terminal instance, and try to remote into a machine

```powershell
# Just an example
C:\Windows\system32> ssh root@192.168.0.1

Linux debian 6.1.0-25-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.106-3 (2024-08-26) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Jan  1 19:00:00 1970 from 192.168.0.1
root@docker:~#
```

> If you had any terminal opened before installing the OpenSSH client, they will not be able to initiate connections until you close them and open them anew
{: .prompt-tip}

## Wrapping up
There is really nothing more to it, Windows has an integrated SSH client that is disabled by default, but can easily be turned back on. It also possesses many more "hidden" features, accessible througt the same *optional features* dialogue.