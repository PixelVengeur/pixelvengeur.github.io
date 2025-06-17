---
layout: post
title: "3 nodes in 3U: Hyperconverged Proxmox + Ceph cluster" 
date: 2025-06-17 10:06 +0200
---

## Summary
This writeup documents how I planned, built, and set up a 3-node Proxmox cluster with fast networking, and even faster storage. The whole cluster is redundant, highly available, and scaleable beyond 3 nodes if needed.


## Current situation
Here's how things are currently running in my homelab:
I have one machine. An Epyc Rome 7302 based system, on a Supermicro H11SSL-NC motherboard, with 128 GB of RAM, and runs Proxmox.
There are a number of expansion cards:
- A ConnectX 4 Lx network card which I got for free from work, running a 10 Gbit/s link to my main computer
- A generic SAS2008 adapter (mine happens to be branded Fujitsu), passed through to a virtual TrueNAS instance
- A Quadro M4000, also a hand me down from work, passed through to a Jellyfin instance for transcoding (Epyc has no built-in hardware video transcoder)
- And an Asus HyperM.2 gen 2 adapter, in which reside three generic 2 TB NVMe drives, also passed through to TrueNAS

All of that running on a single server has caused me trouble when I wanted to change the hardware in any way, as it meant taking down the entire infrastructure. Which is less than ideal.


## Goals
The primary goal this cluster will fulfill is that of my main hypervisor. TrueNAS should be V2P'd into its own box, as running it virtual would still cause some issues when tinkering. The current server is overbuilt, so I'm looking to partially downsize as well. 135W idle is a big oof.

Physical requirements are as follows, by order of importance:
- Must be rack mountable, and take up less space than the current 5.5U tall Define 7 that currently houses my Epyc chip
- Must be quiet, as my current living arrangements make it so that my bed is about 3m away from my rack
- Must be a cluster, ideally highly available, so that I can take out one node and tinker with it without bringing my whole infrastructure down, and I don't know how to administrate one and would like to learn

As for features:
- At least 10G networking, because it's dirt cheap, I'm used to it, and, more importantly, numbers go brrrr
- Fast, local storage for virtual machines. Backups will be dumped into TrueNAS
- Hot swap drive bays. I don't *plan* to use them, buuuut it might be useful for later expansion :)

And as an overarching rule: don't break the bank. I have not set any concrete limit to how much I am willing to spend, but I'm not willing to drop thousands on a home server (yet).


## Hardware
### The chassis
Unsurprisingly, this whole project was kick started by some late night eBay browsing, where I stumbled upon the one thing that, in my mind, would be the hardest to find for a decent price: the case. I was still hesitant between two styles:
1. A low profile, 2-3U server with a GPU, and a recent and efficient yet powerful chip on an AsRock Rack motherboard (for IPMI)
2. A cluster of three nodes, with some lower power chips and no dedicated transcoding hardware

But this listing sealed the deal:

![1U server case with integrated power supply and rails](assets/img/proxmox-3nodes-3U/1U%20case.png)

This was the Holy Grail. A 1U chassis, with hot swap front bays, with front-facing power buttons, with support for a full ATX motherboard, an included 400W power supply AND rails, for a flat 50 €.

Comparable offerings, from the likes of Lian-Li or Supermicro, compromise either heavily on functionality due to their age, or are newly built, but cost an arm and a leg. Some other brands like Rosewill and LogicCase seem to only be distributed in North America, and I wasn't about to pay the shipping and VAT adjustments from the US to Europe, nor to buy 3 samples from a TaoBao or AliExpress seller, and face the same financial issues.

As if that wasn't enough, the seller also had a model of this chassis *with* a system inside of it.
![Xeon-based system with IPMI](assets/img/proxmox-3nodes-3U/xeon.png)
It's a chipset C236 based motherboard, suitable for the Xeon E3-1200 V6 series of CPUs. While old and inefficient by today's standards, I couldn't help but think that, for 50 € extra on top of the chassis, I had found myself a decent platform to run TrueNAS off of. So I pulled the trigger on two empty cases, and a full one, for a total of 200 € (excl. s&h). Not a bad start.

### The motherboard and CPU
I had had in the back of my mind for a while that I wanted to get one of those Chinese-manufactured motherboards with included laptop CPUs on them, such as the offerings from Erying. And a few days later, Wendell from Level1Techs released [a video showcasing a $150 motherboard with an included ES i7-1280p](https://www.youtube.com/watch?v=kMidSN9vfow) which seemed perfect: a vaslty more recent CPU that my Epyc, with an iGPU, desktop DDR4 for those sweet deals on an end-of-life standard, a full x16 port for expansion and two M.2 slots, with an extra A+E key for a supplemental NIC or a wifi card. The one downside was that the chip was an engineering sample, which could prove to be slightly less stable that their global release counterparts. But that won't come back to bite me, so I bought one.

![i7-1280p motherboard from an unknown seller on AliExpress](assets/img/proxmox-3nodes-3U/1280p.png)

### First hiccup
Long story short, it was too good to be true. I put it to the test, building it into the chassis I had received, and all went well. The BIOS was classic, but had the features I wanted, namely VTX, VT-d, and bifurcation. I got into proxmox, started configuring it the way I like it, spun up a few Linux VMs, and went to sleep. I woke up the next day, to a frozen OS.

Weird, that's the first time it has happened to me at all with Proxmox. I was quick to blame the cheap SSD I installed it on, so I bust out some drive I knew was good, reinstalled everything, and left a terminal open with `watch -n 1 uptime` running, in order to check if it froze. And wouldn't you know it, it froze again about 12 hours later.

I tried countless things to get it working, but Linux refused to behave. The longest I had managed to keep it running for was nigh on two days, after which it froze again. The silver lining is that it stayed up for over 5 days running Windows Server, before I shut it off manually. You win some, you lose some.

### Caving in
While I knew I was gambling with the previous board, I was certain that the concept was good in theory. Very few things came close to the power-to-price ratio of that motherboard combo, and I was willing to try it again, despite the $150 paperweight I had just bought. So I turned to Erying, which had earned some credit in the homelab community for making such motherboards.

I waited for a site-wide sale on AliExpress, and bought a motherboard from them: the Skyline G660 ITX.
![Erying motherboard with an i5-12650H](assets/img/proxmox-3nodes-3U/erying.png)
This one is based around the i7-12650H, a 12th generation 10 cores, 16 threads laptop CPU. The motherboard couples it with two Realtek NICs, one of which is 2.5G, as well as DDR4 DIMM slots for full-size RAM, and 3 (!) M.2 slots, two of which are PCIe Gen 4 x4, and the other one, PCIe Gen 3 x4. It also supports front panel USB type C, for high speed storage, should that become necessary later.

And this one worked much better, with basically no trouble, besides the UEFI that was set to Chinese on the first boot. Virtualisation, IOMMU, and Bifurcation were all present, and worked as expected. I even reused the install from the previous motherboard to gain some time, and it never froze in the week of testing I did. I thus considered it good, and moved on to the next step.

### Networking, storage, and miscellaneous items.
Networking was almost already figured out: there is a plethora of 10Gbit/s NICs readily available on AliExpress, for extremely reasonable prices. My choice was on an Intel X520 based, PCIe gen 2 x8 card , with SFP+ connectors. I specifically wanted SFP+ to use transceivers and fibre. Why? Rule of cool, and who knows whether I'll upgrade that to 25, or even 40 GBit/s in the future, which copper can't do.

Storage was a little trickier. In the spirit of building reliable stuff, I knew I wanted some discarded enterprise SSDs, to run in Ceph. Good read and write performance, be they Sequential or Random, alongside that sweet TLC memory for high endurance. Consumer-grade drives boast high numbers, but I've been burned by marketing before, especially on QLC or DRAM-less drives. They spike up to 1 GB/s, only to fall back down to USB speeds after a few seconds. So I got patient, and setup alerts on eBay.

One day, an offer came up
![eBay listing for U.2 NVMe Enterprise SSDs](assets/img/proxmox-3nodes-3U/ssds.png)
3 SSDs, one per node, discarded enterprise stuff which, according to the seller, had "only" been used a few thousand hours. So, despite the price, I pulled the trigger.

All the core components were there, all I needed was some cabling and adapters, so AliExpress to the rescue. PCIe risers, M.2 to Mini SAS HD and Mini SAS HD to U.2 for the drives, 10 Gbit/s transceivers, a Xeon CPU for the up-and-coming TrueNAS board, some fibre patch cables, and a 24 pin extension since the included PSU cable doesn't reach the new motherboard and voilà: a new node, fully built, ready to be tested, before buying everything else for the other two nodes.


## Software


## Miscellaneous troubles
### Hardware incompatibility
#### No I/O Shield slot
#### Cooling
You might notice I have left out a big part of what makes buildign in 1U difficult: the cooling. Well, it wouldn't be that difficult, if it needn't be silent, I could just have used the 5 4028 fans that were included with the case, slapped on a passive heatsink for LGA1700 and bob's your uncle. But I could not, as I still needed eardrums for my day to day life, as well a crumb of sleep at night.

So I bought a variety of so-called "low profile" coolers, that would fit in 1U. The ID-Cooling IS-30, the ARSYLID G195, the Jonsbo HP-400S, and even considered the Noctua L9S, but quickly noticed that it wouldn't fit with the chassis closed. As a matter of fact, only one of those fit with the chassis closed, and it's the 19mm tall ARSYLID G195. Problem is, after about half an hour, it just blows hot air through its fins, and the CPU throttles anytime there's a spike. So for a while, I just ran the coolers without the lid on, with each node separated by 1U. It worked great, but felt cheesy. That's not what I had settled to make, I'm taking up twices the rack space I had planned for, and it just looked unfinished.

### Space constraints

### Intel microcode
#### Proxmox Helper scripts + RIP tteck

### Power consumption
#### C-states & ASPM
#### Removing the 10G switch, 10G networking, and U.2 drives

## Customisation
### 3D Printed front IO