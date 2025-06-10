---
layout: page
title: Setup
permalink: /setup/
---

You must have access to a running Docker server and a Docker Hub account for this workshop. This should be setup in advance.

## Docker

The open-source Community Edition of Docker [can be downloaded here](https://store.docker.com/search?offering=community&q=&type=edition) and installed on your own computer, including on [OS X](https://store.docker.com/editions/community/docker-ce-desktop-mac) and  [Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows). You should download and install the latest stable version (this workshop has been tested on **28.2.2**). 

If you are a running a recent version of Linux then Docker may be available in the distribution repositories, though it is likely to be an older version. Consider installing a [newer version](https://store.docker.com/search?offering=community&q=&type=edition). On Linux Docker commands can only be run as root. You can change this by adding yourself to the `docker` group.


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
docker version
Client: Docker Engine - Community
 Version:           28.2.2
 API version:       1.50
 Go version:        go1.24.3
 Git commit:        e6534b4
 Built:             Fri May 30 12:09:15 2025
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          28.2.2
  API version:      1.50 (minimum version 1.24)
  Go version:       go1.24.3
  Git commit:       45873be
  Built:            Fri May 30 12:07:23 2025
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.7.27
  GitCommit:        05044ec0a9a75232cad458027ca83437aae3f4da
 runc:
  Version:          1.2.5
  GitCommit:        v1.2.5-0-g59923ef
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
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
