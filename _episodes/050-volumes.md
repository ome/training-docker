---
title: "Storing data Docker volumes"
teaching: 15
exercises: 15
questions:
- "Why should I use Docker volumes?"
- "Where can I store and access data?"
objectives:
- "Explain why volumes are necessary."
- "Manage host and Docker volumes"
- "Why should I use docker volumes instead of host volumes?"
keypoints:
---

Docker containers are intended to be ephemeral. This means it should be possible to regularly stop, delete and replace them without loosing anything important.

This means that in contrast to a physical server where all storage is persistent you must carefully consider where you application writes to. Despite the inconvenience this is a good thing.

Persistent data should be stored on a volume, which in Docker is an abstraction of a chunk of file storage.

## Types of volumes

## Volume management

{% include links.md %}
