---
layout: post
title: "3 nodes in 3U: Hyperconverged Proxmox + Ceph cluster"
date: 2025-06-19 14:45 +0200
media_subpath: /assets/img/proxmox-3nodes-3U
image: cover.png
published: true
---

This writeup documents how I planned and built my current 3 nodes Proxmox cluster, with an NVMe-based Ceph cluster for virtual machine storage.

# Goals
The primary goal this new server will fulfill is that of my main hypervisor, inheriting the dozen or so virtual machines I already run on my current monolithic server. The current server is overbuilt for its tasks as well, so I'm also looking to downsize a little (if only for the power consumption).

Physical requirements are as follows, by order of importance:
- Must be **rack mountable**, and take up less space than the current 5.5U tall Fractal Define 7 that currently houses my server.
- Must be **quiet**, as my current living arrangements make it so that my bed is about 3m away from my rack.
- Must be a **cluster**, ideally highly available, so that I can take out one node and tinker with it without bringing my whole infrastructure down.

As for features:
- At least **10G networking**, because it's dirt cheap, I'm used to it, and, more importantly, numbers go brrrr.
- **Fast, local storage** for virtual machines. Backups will be dumped into TrueNAS.
- Hot swap drive bays. I don't *plan* to use them, buuuut it might be useful for later expansion :)

And as an overarching rule: ***don't break the bank***. I have not set any concrete limit to how much I am willing to spend, but I'm not willing to drop thousands either.

# Hardware

## Motherboard
My homelab is built for experimentation. I'm not running anything crucial to others, so it can go down or be a little unstable, and the only affected person is me. Thus, I like using exotic hardware, and this build is no exception. Everything revolves around the idea of bang-for-the-buck: stuff can be expensive, but they have to be worth it.

![Erying motherboard with an i5-12650H](erying.png){:.left width="400"}
The heart of each node is the Erying Skyline B660i, a desktop motherboard with a soldered-on laptop chip. In this case, the CPU is an i7-12650H, a 10c/16t chip from only a few years ago. They also had offerings with 13th and 14th gen CPUs, but I preferred staying away from trouble and not roll the dice on [potentially damaged chips](https://www.theverge.com/2024/7/26/24206529/intel-13th-14th-gen-crashing-instability-cpu-voltage-q-a).

The rest of the motherboard is nothing to scoff at either: two Realtek NICs, one of which is 2.5G, DDR4 DIMM slots for full-size RAM, and 3 (!) M.2 slots. Two are PCIe Gen 4 x4, and the third one, PCIe Gen 3 x4. It also supports front panel USB type C, for high speed transfers, should that become necessary later.

## Case
![1U server case with integrated power supply and rails](1U%20case%202.png){: .right width="400"}
This is a crucial part, as it will decide what else I can put in each node. I have opted for a seemingly unbranded 1U server chassis, that I got on Ebay, alongside its matching rails and a 400W Flex-ATX power supply.

## Networking
![10 Gbit/s dual network adapter](10gb.png){: .right width="300"}
Networking is a solved issue. I know exactly what I want: 10 Gbit/s or greater, with SFP+ ports, to run fibreoptics in-between the nodes. Why fibre? Rule of cool. Plus, I can keep the runs when I inevitably dip my toes in even faster networking. That, and working with Cat 6a cables is too much of a pain when making custom runs.

AliExpress is full of inexpensive, older generation cards that can accomplish pretty much anything you'd want out of a network card (unless it's C-States lower than C2 :angry:). I went for an Intel X550 controller on some unbranded PCB. They're old gen PCIe gen 2 x8 cards, that pull about 10W when in use.

Storage
-------
I wanted balling storage for my cluster. Reading ahead in the Ceph documentation, I knew that the lowest drive size would be all the storage I would have to play with, so the best idea was to get 3 identical drives in make and model. Beyond that, I wanted something that could match the 10G link between the servers, a drive that could actually sustain a high speed R/W operation. Thus, my only option was to go NVMe.

![Samsung 17mm 2.5" drive](nvme.png){: .left width="300"}

I picked up 3 identical 3.2 TB Samsung U.2 drives (`MZWLK3T2HCJL-000U3`) on Ebay. While they do rack up 40.000h (~4.5 years) of use, they bear only between two and four percent wear. Plus, some wuick testing on Windows revealed they could actually sustain over 2 GB/s read and over 1.5 GB/s write.

Their one downside is power consumption, and thus heat. They are rated to draw up to 25W, and need some draft over them to keep them under control. They typically only draw half that power, though.

## Power

![HDPlex 250W Gallium Nitride PSU](psu.png){: .right width="400"}

Speaking of power consumption, in an effort to make each node as efficient as possible, I purchased a new power supply: an HDPlex 250W GaN power supply.

It's fully passive, and is much smaller than even a Flex-ATX PSU like the one I got with the chassis. Not only does it fit in 1U, but the included cables are flexible enough to bend into shape with the lid closed. A 90° adapter for the 24 pin would put my mind at ease though. The flexible cables, along with the remote power input, makes it so that I can fit it somewhere else in the chassis and not only the designated place for the PSU, which helps massively with cable management. In 1U, space is at a premium, and cables accumulate very fast.

Cooling
------
Though I didn't think I'd need it, I have opted for watercooling each node. Even a slim air cooler is massively overkill for the CPUs I'm running, but their downside is that they need access to fresh air, otherwise they end up recycling warm air, and don't cool effectively.

I had thought about punching a hole in the lid, above the space the cooler would be in, so it could draw fresh air, but quickly remembered there would be equipments above it, that might extend over that hole. So it was either keeping the lids open, which is very prone to accidents and looks incredibly unfinished, or watercooling.

![1U all in one liquid cooler](dynatron.png){: .left width="400"}

I have opted for the `Dynatron L25-U`, a 5-fan, all-in-one liquid cooler designed specifically for 1U chassis. They also have a 3-fan version (the `Dynatron L3`), but I figured more airflow in a constrained space would be beneficial anyway. Naturally, the included fans are loud and power hungry (8W per fan!), but after adjusting the fan curve so that they idle at 25% and never go above 50%, they're, dare I say, quiet.

Drawbacks of liquid cooling are obvious, but I did not really leave myself any choice with the constraints I laid out early on.

Finished result
====
Some glory pictures of the finished product

Software
========
Short note, the software is your classic homelab suite. [Proxmox](https://www.proxmox.com/en/) as the hypervisor, with [Ceph](https://ceph.io/en/). A separate [TrueNAS](https://www.truenas.com/truenas-community-edition/) instance will handle everything that is not VM storage: bulk data, backup jobs, media library, you name it.

All VMs will be stored on Ceph, for easy and extremely quick migration. Since Ceph is distributed between every node, there is no need for data replication: it *already is* replicated! If I were to use the drives separately on each node, when a migration is triggered, all VM data would have to be copied first. It wouldn't take long on SATA SSDs backed by a 10G network, but 10 seconds is still faster than 2 minutes :) 

# Price breakdown
The big question, that I didn't want to answer, is that of price. While this list is concise, every component was bought separately, over an extended period of time. Expenses were spread over months, which made it easier to carry on testing components and building the nodes.

Each motherboard was purchased for just around 270 €, thanks to discounts on AliExpress. The network cards were 25 € each, also from AliExpress. Each case was 50 €, and the U.2 drives were 500 € for the three. The Dynatron L25-U is a 120 € AIO, and the HDPlex PSU retails for 145 €.

I also bought random knick-knacks to connect everything. Per node, I purchased: a 24 pin extension (3 €), an M.2 to U.2 adapter (20 €), a flexible PCIe riser (7 €), front USB 2.0 and 3.0 (5 € each), transceivers (8 € x2), and some fibre (5 €).

All in all, that's a grand total of roughly **2500 €**, or a cost of about 840 € per node. About that "not spending a grand" thing...

# Troubles
## Rejected components
While looking for hardware that fit the bill, I also had some failures. that I don't count in the total price.

![i7-1280p motherboard from an unknown seller on AliExpress](1280p.png){: .left width="250"}

The first motherboard I tried, that was also a special laptop-ATX motherboard, was [the one featured by Wendell on Level1Techs](https://www.youtube.com/watch?v=kMidSN9vfow). At $150 it was about half as expensive, but unfortunately it bears an Achille's heel: the processor on board is an Engineering Sample CPU. And apparently, Proxmox did not want anything to do with that, and refused to be stable past a day. Windows Server went up to 5 days though, after which I stopped the stability test. Too bad, as it bore an i7-1280p.

<br><br><br><br>

![Arsylid G195 illustration](g195.png){: .right width="250"}

Another huge trouble I ran into was that of cooling. As I mentioned earlier, air coolers wouldn't work in a closed chassis. The reason I can affirm that is that I have tried multiple. The `ID-Cooling IS-30`, the `ARSYLID G195`, the `Jonsbo HP-400S`, and even considered the `Noctua L9S`. Only one fits with the lid closed (the `ARSYLID G195`), but with no fresh air intake, and the ridiculously small thermal mass of 19mm of air cooler, the CPU thermal throttled after a few seconds on *any* load spike.


## Manual work
The chassis I chose was meant for one and only one motherboard, which means it doesn't bear a slot for an IO Shield, as the openings in the back were moulded around the back ports of the motherboard. So if I want to slot a new motherboard in, I need to cut an opening myself.

![Back of the 1U rackable chassis](chassis_back.png){: .center width="500"}