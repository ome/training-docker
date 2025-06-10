---
title: "Introduction to Docker"
teaching: 10
exercises: 0
questions:
- "What is Docker?"
objectives:
- "Explain containerisation."
- "Compare Docker with other forms of virtualisation."
- "Understand basic Docker terminology."
keypoints:
- "Docker is ..."
- "Images ..."
- "Containers ..."
---

This is a basic introduction to the key concepts behind [Docker](https://www.docker.com/) and some of the terminology used.

## What is containerisation?

Two things:
- A lightweight form of virtualisation
- A methodology for packaging and distributing applications


## What is Docker?

Docker is a platform for building and running applications in isolated containers.
It is often used to support reproducible build and deployment of applications, but can also be used in an interactively for testing or exploring new software.

It is also the name of the company behind the technology.


## How does Docker compare with virtual machines

Most virtualisation platform (e.g. [VirtualBox](https://www.virtualbox.org/), [VMWare](https://www.vmware.com/), [OpenStack](https://www.openstack.org/)) replicate all the hardware of a typical computer in software, allowing you to run most operating systems.
- Resource heavy

In contrast Docker is a form of lightweight virtualisation
- It takes advantage of Linux kernel features to isolate separate processes
- Low-level system calls or hardware access probably won't work

A bit like individual OMERO.servers for each user versusx one server with each user in their own private group.


## Images and containers

Typically a Docker image will only run one application (e.g. Nginx web server), in contrast to a virtual machine which may have external SSH access, cron jobs to perform regular clean-up tasks, and multiple user accounts, amongst other things.




{% include links.md %}
