---
layout: post
title: Unlocking the power of biometrics on Linux
date: 2025-01-31 15:50 +0100
published: false
---

Earlier this year, I got a new work laptop, a Lenovo G16 IRL. As they tend to, laptops come with Windows preinstalled, with its activation key tied to the motherboard. However, for the past 8 months, I had been using Fedora Linux on my previous laptop, which I was very happy with. But this laptop has something more: a Windows Hello certification

## What is Windows Hello?
Windows Hello is a certification introduced by Microsoft that tells consumer a certain piece of equipment can be used to biometrically authenticate a user. The most prevalent forms of those equipments is the fingerprint scanner, and the facial recognition camera.

## Fingerprint reader
The easy way to get a fingerprint reader working on Linux is to use [fprint](https://fprint.freedesktop.org/). Unfortunately, the fingerprint scanner included in the G16 is not contained in the [officially supported hardware list](https://fprint.freedesktop.org/supported-devices.html). However, Lenovo has released a [proprietary driver](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-edge-laptops/thinkpad-e14-gen-4-type-21eb-and-21ec/downloads/ds563477-fpc-fingerprint-driver-for-ubuntu-2004-ubuntu-2204-thinkpad-e14-gen-4-e15-gen-4?category=Fingerprint+Reader) for this very fingerprint sensor (`10a5:9800`).

```bash
$ lsusb

Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 002 Device 002: ID 0bda:0420 Realtek Semiconductor Corp. 4-Port USB 3.0 Hub
Bus 002 Device 003: ID 0bda:8153 Realtek Semiconductor Corp. RTL8153 Gigabit Ethernet Adapter
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 003 Device 002: ID 0bda:5420 Realtek Semiconductor Corp. 4-Port USB 2.0 Hub
Bus 003 Device 003: ID 046d:c548 Logitech, Inc. Logi Bolt Receiver
Bus 003 Device 004: ID 04f2:b7b4 Chicony Electronics Co., Ltd Integrated Camera (1920x1080)
Bus 003 Device 005: ID 0bda:1100 Realtek Semiconductor Corp. USB2.0 HID
Bus 003 Device 006: ID 10a5:9800 FPC FPC Sensor Controller L:0002 FW:27.26.23.50
Bus 003 Device 007: ID 8087:0026 Intel Corp. AX201 Bluetooth
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
```



## IR Camera
https://github.com/boltgolt/howdy/issues/987