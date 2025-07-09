---
layout: post
title: "3 nodes in 3U: Hyperconverged Proxmox + Ceph cluster" 
date: 2025-06-17 10:06 +0200
media_subpath: /assets/img/proxmox-3nodes-3U
image: cover.png
published: false
---

## Summary
This writeup documents how I planned, built, and set up a 3-node Proxmox cluster with fast networking, and even faster storage. The whole cluster is redundant, highly available, and scaleable beyond 3 nodes if needed.

## Current situation
Here's how things are currently running in my homelab:
I have **one machine**.

It's an Epyc Rome 7302 based system, on a Supermicro H11SSL-NC motherboard, with 128 GB of DDR4 Registered ECC RAM, and runs Proxmox.
There are a number of expansion cards:
- A ConnectX 4 Lx network card, running a direct 10 Gbit/s link to my main computer
- A Quadro M4000, passed through to a Jellyfin instance for transcoding (Epyc has no built-in hardware video transcoder)
- And an Asus HyperM.2 gen 2 adapter, with three generic 2 TB NVMe drives, as well as a generic SAS2008 adapter, passed through to TrueNAS

All of that running on a single server has caused me trouble when I wanted to change the hardware in any way, as it meant taking down the entire infrastructure, which is less than ideal. So after I got baby's first half-height rack, I wanted to change that.


## Goals
The primary goal this new server will fulfill is that of my main hypervisor, inheriting my dozen or so virtual machines. The current server is overbuilt for its tasks, so I'm also looking to downsize a little (if only for the power consumption).

TrueNAS will be copied onto its own physical machine, as running it virtual would still cause some issues when tinkering. 

Physical requirements are as follows, by order of importance:
- Must be **rack mountable**, and take up less space than the current 5.5U tall Define 7 that currently houses my Epyc chip.
- Must be **quiet**, as my current living arrangements make it so that my bed is about 3m away from my rack.
- Must be a **cluster**, ideally highly available, so that I can take out one node and tinker with it without bringing my whole infrastructure down.

As for features:
- At least **10G networking**, because it's dirt cheap, I'm used to it, and, more importantly, numbers go brrrr.
- **Fast**, local **storage** for virtual machines. Backups will be dumped into TrueNAS.
- Hot swap drive bays. I don't *plan* to use them, buuuut it might be useful for later expansion :)

And as an overarching rule: ***don't break the bank***. I have not set any concrete limit to how much I am willing to spend, but I'm not willing to drop thousands on a single server either.


## Hardware
### Chassis
Unsurprisingly, this whole project was kick started by some late night eBay browsing, where I stumbled upon the one thing that, in my mind, would be the hardest to find for a decent price: the case. Despite listing *cluster* as a requirement, at that point I was still debating between two styles:
1. A low profile, 2-3U server with a recent and efficient yet powerful chip, on an AsRock Rack motherboard, and a dedicated GPU
2. A cluster of three nodes, with some lower power chips, and no dedicated transcoding hardware

But this listing sealed the deal:

![1U server case with integrated power supply and rails](1U%20case.png)

This was the Holy Grail. A 1U chassis, with hot swap front bays, front-facing power buttons, support for a full ATX motherboard, an included 400W power supply AND **rails**, for a flat 50 €.

Comparable offerings, from the likes of Lian-Li or Supermicro, compromise either heavily on functionality due to their age, or are newly built, but cost an arm and a leg. Some other brands like Rosewill and LogicCase seem to only be distributed in North America, and I wasn't about to pay the shipping and VAT adjustments from the US to Europe, nor to buy 3 samples from a TaoBao or AliExpress seller, and face the same financial issues.

![Xeon-based system with IPMI](xeon.png){: .right width="400"}

As if that wasn't enough, the seller also had a model of this chassis *with* a system inside of it.
It's a chipset C236 based motherboard, suitable for the Xeon E3-1200 V6 series of CPUs. While old and inefficient by today's standards, I couldn't help but think that, for 50 € extra on top of the chassis, I had found myself a decent platform to run TrueNAS off of. So I pulled the trigger on two empty cases, and a full one, for a total of 200 € (excl. s&h). Not a bad start.


### Motherboard and CPU
I had had in the back of my mind for a while that I wanted to get one of those Chinese-manufactured motherboards with included laptop CPUs on them, such as the offerings from Erying. And a few days later, Wendell from Level1Techs released [a video showcasing a $150 motherboard with an included ES i7-1280p](https://www.youtube.com/watch?v=kMidSN9vfow) which seemed perfect: a more recent CPU than my Epyc, with an iGPU, desktop DDR4 for those sweet deals on an end-of-life standard, a full x16 PCIe port for expansion (x8 electrically) and two M.2 slots, with an extra A+E key for a supplemental NIC or a wifi card. The one downside was that the chip was an engineering sample, which could prove to be slightly less stable that their global release counterparts. But I was certain it wouldn't come back to bite me, so I bought one.

![i7-1280p motherboard from an unknown seller on AliExpress](1280p.png){: .right width="400"}

#### First hiccup
It came back to bite me. I put the board to the test, building it into the chassis I had received, and all went well. The BIOS was classic, but had the features I wanted, namely VTX, VT-d, and bifurcation. I got into proxmox, started configuring it the way I like it, spun up a few Linux VMs, and went to sleep. But I woke up the next day to a frozen OS.

I was quick to blame the cheap SSD I installed it on, so I bust out some drive I knew was good, reinstalled everything, and left a terminal open with `watch -n 1 uptime` running, in order to check if it froze. And wouldn't you know it, it froze again about 12 hours later.

I tried countless things to get it working, but Linux refused to behave. The longest I had managed to keep it running for was nigh on two days, after which it froze again. The silver lining is that it stayed up for over 5 days running Windows Server, before I shut it off manually. You win some, you lose some.

#### Caving in
While I knew I was gambling with the previous board, I was certain that the concept was good in theory. Very few things came close to the power-to-price ratio of that motherboard combo, and I was willing to try it again, despite the $150 paperweight I had just bought. So I turned to Erying, which had earned some credit in the homelab community for making such motherboards.

I waited for a site-wide sale on AliExpress, and bought a motherboard from them: the Skyline G660 ITX.
![Erying motherboard with an i5-12650H](erying.png){:.left width="400"}

This one is based around the i7-12650H, 10c/16t laptop CPU. The motherboard couples it with two Realtek NICs, one of which is 2.5G, as well as DDR4 DIMM slots for full-size RAM, and 3 (!) M.2 slots. Two are PCIe Gen 4 x4, and the third one, PCIe Gen 3 x4. It also supports front panel USB type C, for high speed transfers, should that become necessary later.

This one worked much better out of the box, with basically no trouble, besides the UEFI that was set to Chinese on the first boot. Virtualisation, IOMMU, and Bifurcation were all present, and worked as expected. I even reused the install from the previous motherboard to gain some time, and it never froze in the week of testing I did. I thus considered it good, and moved on to the next step.

![10 Gbit/s dual network adapter](10gb.png){: .right width="400"}

<h3> Networking </h3>

Networking was almost already figured out: there is a plethora of 10Gbit/s NICs readily available on AliExpress, for extremely reasonable prices. I settled on an Intel X520 based, PCIe gen 2 x8 card , with SFP+ connectors. I specifically wanted SFP+ to use transceivers and fibre. Why? Rule of cool, and who knows whether I'll upgrade that to 25, or even 40 GBit/s in the future, which copper can't do.

<br/>

<h3> Storage </h3>
Storage was a little trickier. In the spirit of building reliable stuff, I knew I wanted some discarded enterprise SSDs, to run in Ceph. Good read and write performance, be they Sequential or Random, alongside that sweet TLC memory for high endurance. 

Consumer-grade drives boast high numbers, but I've been burned by marketing before, especially on QLC or DRAM-less drives. They spike up to 1 GB/s, only to fall back down to USB speeds after a few seconds, and remain there until the end of the transfer. So I got patient, and setup alerts on eBay.
One day, an offer came up:

![eBay listing for U.2 NVMe Enterprise SSDs](ssds.png){: .center width="500"}

3 SSDs. Discarded enterprise drives which, according to the seller, had "only" been used about 40.000 hours (4.5 years). So, despite the high total price, I pulled the trigger, as I just knew it was a great deal for 9.6 TB of raw flash storage. Spoiler alert: each drive only had 2-4% wear. A great deal indeed!

All the core components were now there, what I needed now was some cabling and adapters, so AliExpress to the rescue. PCIe risers, M.2 to Mini SAS HD and Mini SAS HD to U.2 for the drives, 10 Gbit/s transceivers, a Xeon CPU for the up-and-coming TrueNAS board, some fibre patch cables, a 24 pin extension, and voilà: a new node, fully built, ready to be tested, before buying everything else for the other two nodes.

Long story short, it went great, and I ended up making two clones of it :)


## Miscellaneous troubles
### No I/O Shield slot
![alt text](chassis_back.png)
Small inconvenience in the grand scheme of things, but the chassis doesn't have an IO shield slot. It was made in a partnership with AsRock Rack, for a select few motherboards that all had the same back IO. So, angle grinder and Dremel to the rescue, and I had a hole the size of an IO Shield.


### Cooling
You might notice I have left out a big part of what makes buildign in 1U difficult: the cooling. Well, it wouldn't be that difficult, if it needn't be silent, I could just have used the 5 4028 fans that were included with the case, slapped on a passive heatsink for LGA1700 and bob's your uncle. But I could not, as I still needed eardrums for my day to day life, as well a crumb of sleep at night.

So I bought a variety of so-called "low profile" coolers, that would fit in 1U. The `ID-Cooling IS-30`, the `ARSYLID G195`, the `Jonsbo HP-400S`, and even considered the `Noctua L9S`, but quickly noticed that it wouldn't fit with the chassis closed. As a matter of fact, only one of those fit with the chassis closed, and it's the 19mm tall `ARSYLID G195`.

![Arsylid G195 illustration](g195.png){: .right width="350"}

Problem is, after about half an hour, it just blows hot air through its fins due to lack of exhaust. I could cut off a hole in the chassis atop where the cooler resides, but it wouldn't work anymore the moment I racked something on top of the server.
Thus, for a while, I just ran the servers without the lid on, with each node separated by 1U. It worked great, but felt cheesy. That's not what I had settled to make, I'm taking up twices the rack space I had planned for, each activity spike is audible due to the CPU fans ramping up, and it just looked unfinished. Enters liquid cooling.

While I had thought at forst of an external solution, where I'd plug each node into an exterior pump and radiator combo, after buying fittings, quick connects and various other supplies, I realised that it just wouldn't fit with the rest of the thardware inside. So, as a last ditch effort, I started to envision putting a radiator inside each node. A few hours later, and I had found the `Dynatron L25-U`

![1U all in one liquid cooler](dynatron.png){: .left width="400"}

I hesitated a while, not only due to the price tag of 130 €, but also the five 4020 fans, that I knew would render useless any choice I had made towards silence. I still bought one, as a test, thinking I'd replace the fans with Noctua A4x20 if need be.

At the time of writing, it has been handling the 12650H incredibly well for over a month. After tuning the fan curve to make it stay between 25 and 40% duty cycle, as well as getting a dedicated fan hub to power these hungry boys, it stays whisper quiet, and only ramps up a little if the load is sustained. No need for a Noctua swap!

Another added bonus is that it physically separates the chassis in two, and creates airflow over the rest of the server by sucking the air from one side, and exhausting it the other. It's a little more power hungry than an air cooler, and all 5 fans run at 100% on boot for a few seconds, but that's it for the shortcomings. Routing the tubing was a delicate task, but once done and zip tied in place, it's as if it wasn't even there during operation.

