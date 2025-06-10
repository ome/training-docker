---
layout: page
title: Setup
permalink: /setup/
---

You must have access to a running Docker server and a Docker Hub account for this workshop. This should be setup in advance.

## Docker

Install Docker Engine. Read the installation page corresponding to your operation system:
[Linux installation](https://docs.docker.com/engine/install/),
[Mac installation](https://docs.docker.com/desktop/setup/install/mac-install/),
[Windows instatllation](https://docs.docker.com/desktop/setup/install/windows-install/)
You should download and install the latest stable version.
This workshop has been tested on **28.2.2**. 


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
