---
layout: post
title: Installing Docker and Docker Compose
date: 2024-02-28 18:17 +0100
categories: [Tutorial]
tags: [docker]
image: /assets/img/installing-docker/preview.png
---

## Installing Docker

For a basic install, I'd recommend you use the convenience script, that does everything for you. This document may not be up to date at the time of reading, so I'd suggest you consult [the official Docker documentation on the convenience script](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script).

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh --dry-run
```
Once you have verified the output of the command, run the second one without `--dry-run`

```bash
sudo sh get-docker.sh
```

Go fetch a hot beverage while the install script does its job. You have earned it.

Once Docker is installed, by default, you can only run docker commands as the super user `root`. I usually allow the unprivileged user to execute commands, to avoid running stuff as root as much as possible. To do so, add the unprivileged user to the `docker` group. If the docker group was not created, create it yourself.

> Adding your user to the `docker` group gives it root-level privileges.
> [Running the docker daemon as a non-root](https://docs.docker.com/engine/security/rootless/) user is possible, but limits use cases.
{: .prompt-danger}

```bash
# In case the group wasn't created
sudo groupadd docker
sudo usermod -aG docker $USER
```
If you run a VM, reboot it for the privileges to apply correctly. Sometimes a simple log off/log on can suffice, most of the time, you need to reboot.


## Running a first container
Once rebooted and logged in, to test your installation, run the following command

```bash
docker run hello-world
```
What the command we just run did is very well explained by its output:

```bash
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

> You just deployed your first container!
{: .prompt-tip}


## Automatically start containers on boot
If you run Debian like I do, the docker daemon will start automatically on machine boot, and start all your containers (if any). If you do not run a Debian-based distro, you explicitely need to tell `systemd` to start the daemon (if your distro uses `systemd`)

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```