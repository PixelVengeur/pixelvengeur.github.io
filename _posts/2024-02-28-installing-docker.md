---
layout: post
title: Installing Docker and Docker Compose
date: 2024-02-28 18:17 +0100
categories: [Tutorial]
tags: [docker]
image: /assets/img/installing-docker/preview.png
---

## What is Docker?
Docker is a piece of software that runs on your machine that allows you to run *containers*. Containers are small applications that run isolated from each other. They can prepend to any task a full computer could, because inside the container, there usually is a whole operating system running (most likely a flavour of Linux). While it allows you to run virtualised operating systems, Docker is __not__ a hypervisor. It's an isolator.

It's important to note that containers are not specific to Docker, containerisation is standardised. You may also have heard about Kubernetes or Podman, which use the exact same containers available to run with Docker.

Containerisation of applications has become a turning point in on-demand scalability of infrastructure. In the case of a homelab, containerisation allows us to develop, build and share applications extremely quickly and easily, and avoid the allmighty "but it works on my machine".

If it works on your machine and not the client's machine, then we ship your machine to the client, using Docker.
#### Docker Compose
`Compose` is an extension of Docker, that allows you to declare infrastructure as code. Instead of running commands in the terminal, Compose uses the YAML language in a text file to declare containers, networks, and volumes, and create everything all at once upon reading the file.

## Installing Docker

For a basic install, I'd recommend you use the convenience script, that does everything for you. It will pull the necessary resources, install both Docker and Docker compose, and clean up after itself.

This document may not be up to date at the time of reading, so I'd suggest you consult [the official Docker documentation on the convenience script](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script).

```bash
# Download and save the convenience script
curl -fsSL https://get.docker.com -o get-docker.sh
# Perform a dry run to make sure everything is in order
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


## Running your first container
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
If you run Debian like I do, the docker daemon will start automatically on machine boot, and start all the containers that were running before you last powered off the machine (if any). If you do not run a Debian-based distro, you explicitely need to tell `systemd` to start the daemon (if your distro uses `systemd`)

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

> Docker and Docker Compose are now installed and working, good job!
{: .prompt-tip}