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

This is a basic introduction to the key concepts behind Docker and some of the terminology used.

## What is Docker?

Docker is a platform for building and running applications in isolated containers.
It is often used to support reproducible build and deployment of applications, but can also be used in an interactively for testing or exploring new software.


## How does Docker compare with virtual machines

Most virtualisation platform (e.g. VirtualBox, VMWare, OpenStack) replicate all the hardware of a typical computer in software, allowing you to run most operating systems.
- Resource heavy

In contrast Docker is a form of lightweight virtualisation
- Takes advantage of Linux kernel features to isolate separate processes
- Low-level system calls or hardware access probably won't work

A bit like individual OMERO.servers for each user vs one server with each user in their own private group.


## Images and containers

Typically a Docker image will only run one application (e.g. Nginx web server), in contrast to a virtual machine which may have external SSH access, cron jobs to perform regular clean-up tasks, and multiple user accounts, amongst other things.




{% include links.md %}
