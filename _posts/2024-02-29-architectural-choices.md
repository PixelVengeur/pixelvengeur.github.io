---
layout: post
title: Architectural choices
date: 2024-02-29 18:42 +0100
categories: [Self-hosted]
tags: [docker, linux, proxmox]
image: /assets/img/architectural-choices/preview.png
---


## What is this about?
What do I use in my infrastructure, how do I use it, and why do I use it


## Hypervisor
Given the (at the time of writing) recent acquisition of VMWare by Broadcom, and the switch to a subscription service, they, in my opinion, should not be thought about for a lab environment. Not only is there no free trial anymore, but the last version you can get is ESXi 8, and you'll be stuck there, no possibility of evolution. You might want to setup a virtualised ESXi host in another hypervisor to get familiar with it if you want/need to, but I would avoid using VMWare as my main hypervisor today if you're starting out.

With that out of the way, there are basically only two options left: **Xen Orchestra**, for a vSphere-like experience, and **Proxmox**.

### Proxmox
I chose proxmox... Well if I'm honest, because of [Craft Computing](https://www.youtube.com/c/CraftComputing). When I learned about Proxmox, I did not even know that XCP-ng was a thing, nor that VMWare was a thing either. All I knew about was Type 2 hypervisors, like Microsoft Hyper-V and Virtualbox.

It supports everything I've ever wanted to achieve: virtualisation (duh), PCIe passthrough, IPMI, ZFS and best of all, it's Debian based.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/9/9e/Hyperviseur.svg/707px-Hyperviseur.svg.png){: width="500"}
_Type 1 and Type 2 hypervisors_

### XCP-ng
I used XCP-ng and Xen Orchestra quite interchangeably, so let's fix that.

- **XCP-ng** is the hypervisor, it runs baremetal on the hardware, and its role is to manage everything related to the virtual machines.
- **Xen Orchestra** is a third-party web UI, that runs on top of XCP-ng, and 
## Linux distros
With my hypervisor soroted out, what do I run in my virtual machines?

### Debian Linux
Let's start simple with Debian Linux. It's my favourite and the one I know best, despite not being the first one I delved into. The very first one I tried was Linux Mint. Because just like every curious soul, I looked up "best linux to start with", and sawthat it was Ubuntu. So of course I picked the second most popular at the time, Linux Mint.

I quickly learned that most distros share common ground, and that Linux Mint, Ubuntu, Pop! OS and the likes were all based on another one: [Debian Linux](https://www.debian.org/index.en.html). So I went to the source, and started using Debian as my main distro, due to its nigh unambiguity among enthusiasts. But there is another popular choice for enterprise Linux users: ~~Arch~~ **Red Hat**. 

### Red Hat
Red Hat is the parent company of Red Hat Enterprise Linux (**RHEL**). From my experience, it's the most widespread Linux distro in enterprise and production environments, mainly because of the support they offer. They also were among the first to push for that market, branding themselves as production-ready, secure, reliable, and compatible with everything you need to run (which, in fairness to them, has almost always been true for me). If you so desire, there are many ways you can play with an RHEL-based distro, outlined hereafter.

#### [Fedora Linux](https://fedoraproject.org/)
Fedora is the first step in the RHEL environment. It is built upon the previous RHEL release, thus bringing in validated code that already runs in production, and merges in other open-source projects, where independant developers add new features to the OS.

It's the most up to date version, but also the most bleeding edge, stability of integration with the newest of technologies is not guaranteed. It is still based on previous validated code, so Fedora is still a great choice for everyday use, be it as your desktop or server distro of choice.

#### [CentOS Stream](https://www.centos.org/centos-stream/)
It's the version of Linux that the RHEL releases are based on, where external sources contribute their own code to the Fedora code in order to ensure integration with their product, be it hardware or software.

#### [Red Hat Enterprise Linux](https://access.redhat.com/products/red-hat-enterprise-linux)
The fully-featured, current RHEL release, with long support cyclces, and optional customer support should you run into trouble. It contains the code of CentOS Stream (and thus Fedora), that RHEL direct employees have tested and validated, as well as tweaked to ensure maximum reliability.

Starts at $400 without the support, or $900 with it, for 1 license, for 1 year :melting_face:

![](https://www.redhat.com/rhdc/managed-files/CentOS_Graphic_Full_Desktop_0.svg)
_RHEL integration diagram_

Those three options are managed and funded by Red Hat themselves, in a continuous development cycle.

#### [Rocky Linux](https://rockylinux.org/)
Your last option is Rocky Linux. Rocky Linux is not supported by Red Hat, and yet is a production-ready, downstream version of RHEL (as opposed to CentOS and Fedora, who are upstream of RHEL). It fulfills the role that CentOS used to serve, which is to be the fully-featured RHEL release, without the support, but offered for free. And it also happens to be the next distro I have my eyes on deploying in my home environment.

## But why not Windows Server?
Windows Server is full of contradictions for me. The more I delved into IT, the more my feelings towards the Microsoft approach grew strongly, both in good and bad. I could shorten it to the "it just works but you don't know why exactly" mentality, but it's much more than that. I actually *like* the fact that I don't have to spend hours configuring an OS to achieve one functionality. I like the wizards that it offers for almost every task. I like the fact that I could have a single server for everything I need, and not be worried about it. I like ReFS, their 

But at the same time, not only do I not have the money for it, even if I did, it would [limit my hardware resources](https://learn.microsoft.com/en-us/windows-server-essentials/get-started/hardware-limits) and take up more of them exponentially the more services I need. And that's if it detects my hardware correctly at all (I've had previous issues with a SAS3008 controller). With the aforementioned wizards and automatic configuration tools, I lose all the granular control I get with other softwares and other operating systems. And their push towards cloud computing in Azure frankly revulses me, as a self-hosting enthusiast.

I did end up [deploying a Windows Server VM]({% post_url 2024-04-07-truenas-and-active-directory %}), thanks to a rogue key I obtained from work, and for what I use it for, it's great. Active Directory is awesome for centralised authentication, and the DHCP and DNS services do everything I need them to do. So indeed, it works, and it works well. But for 700 € per license with a hardware cap, it better do.


## Containerisation engine
I'm using Docker, for the same reason I use Debian, it's what I know and know how to use. But, Docker is not the only kid on the block. There are two other widespread technologies, achieving the same results: Podman, and Kubernetes.

### Podman
![](https://raw.githubusercontent.com/containers/common/main/logos/podman-logo-full-vert.png){: width="150"}{: .left}

Podman didn't use to support everything Docker could do, but in these past few years, the project has matured and evolved to the point that it nearly acts as a drop-in replacement. The commands are the same, just substituting `docker` for `podman`, however, it doesn't support compose files out of the box *yet*.

Compose files are my absolute favourite way of deploying containers, bar none. They're easy to read and write, they're idempotent, and easily backupable. While Podman does support deploying compose files, it does so by using **docker** compose, and thus still relies on Docker. It's not a dealbreaker, but so far Docker has not failed me, so I have no desire to switch as of yet.

### Kubernetes
![](https://raw.githubusercontent.com/cncf/artwork/main/projects/kubernetes/stacked/white-text/kubernetes-stacked-white-text.png){: width="150"}{: .left}

While I put Kubernetes as a Docker alternative, it's much, much more than that. Kubernetes is able to deploy and manage containers, like Docker and Podman, but that would be simply scratching the surface of what it's able to do.

Kubernetes is an orchestrator, and allows for horizontal scaling. In practical terms, it means that it can run and scale container deployments by itself, across multiple machines, depending on the workflow. Lots of traffic? It will deploy other containers and load balance the traffic between them. Little traffic? The opposite, it will remove unneeded containers to save on resources.

All of that comes at a cost, and in the case of Kubernetes, I've found it was the steep learning curve. Even if you know your way around Docker, Kubernetes is a whole different environment. While the docs do contain a [Getting Started guide](https://kubernetes.io/docs/tutorials/kubernetes-basics/), that is well-written for what it's worth, the whole k8s stack is massively overkill for a home environment. Kubernetes is a godsend for professionals and companies alike, and a great technology at that, but I do feel like it's too much for simple container management. ~~Not at all because it's too complicated for me.~~


## Wrapping up
While I only scratched the surface of what a homelab could be used for, I hope this writeup helped clear a few questions you might have had. I also hope you might have learned a few things along the way, and have discovered soem things you might want to get into!