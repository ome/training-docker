---
layout: page
title: Setup
permalink: /setup/
---

You must have access to a running Docker server and a Docker Hub account for this workshop. This should be setup in advance.

## Docker

The open-source Community Edition of Docker [can be downloaded here](https://store.docker.com/search?offering=community&q=&type=edition) and installed on your own computer, including on [OS X](https://store.docker.com/editions/community/docker-ce-desktop-mac) and  [Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows). You should download and install the latest stable version (this workshop has been tested on **17.09**). This will also install Docker Compose.

If you are a running a recent version of Linux then Docker may be available in the distribution repositories, though it is likely to be an older version. Consider installing a [newer version](https://store.docker.com/search?offering=community&q=&type=edition). You must also [install docker-compose](https://github.com/docker/compose/releases) as this is not included in the Linux Docker packages. On Linux Docker commands can only be run as root. You can change this by adding yourself to the `docker` group.


## Docker Hub

A Docker Hub account will be required, [create one if necessary](https://hub.docker.com/). Make a note of your username and password.


## Docker preferences

On OS X Docker for Mac default to using 2 GB RAM, which is insufficient for running OMERO. Open the Docker preferences and increase this to at least 4 GB, then click on `Apply & Restart`.

<img alt="Docker advanced preferences" src="{{ page.root }}/fig/docker-preferences.png" />


## Check docker is working

Check the version of Docker:
~~~
docker version
~~~
{: .bash}
~~~
Client:
 Version:      17.09.0-ce
 API version:  1.32
 Go version:   go1.8.3
 Git commit:   afdb6d4
 Built:        Tue Sep 26 22:40:09 2017
 OS/Arch:      darwin/amd64

Server:
 Version:      17.09.0-ce
 API version:  1.32 (minimum version 1.12)
 Go version:   go1.8.3
 Git commit:   afdb6d4
 Built:        Tue Sep 26 22:45:38 2017
 OS/Arch:      linux/amd64
 Experimental: false
~~~
{: .output}

Check you can access Docker:
~~~
docker ps
~~~
{: .bash}
~~~
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                               NAMES
~~~
{: .output}

{% include links.md %}
