---
title: "Creating new Docker images"
teaching: 15
exercises: 15
questions:
- "How can you create your own Docker image?"
objectives:
- "Create a new Docker image."
keypoints:
- "A Dockerfile is a set of instructions for building a Docker image"
- "Convert a Dockerfile into an image using `docker build`"
- "`docker build` has a built-in cache to greatly speed up builds"
---

The previous lesson showed how to run existing docker images.
This lesson will demonstrate how to build your own images.

## What is a Dockerfile?

A Dockerfile is a text file containing a set of instructions for reproducably building a Docker image


## My first Dockerfile
Create a new directory, enter it, and create a file called `Dockerfile`:
~~~
mkdir my-first-dockerfile
cd my-first-dockerfile
<edit> Dockerfile
~~~
{: .bash}
~~~
FROM centos:7
MAINTAINER spli@dundee.ac.uk

RUN yum install -y -q epel-release
RUN yum install -y -q python-pip
RUN pip install omego
~~~
{: .source}
- `FROM`: The name of a base image
- `MAINTAINER`: The email of the developer or owner
- `RUN`: Runs a shell command

Build it. `-t` let's you give the image a name
~~~
docker build -t my-docker-image .
~~~
{: .bash}
~~~
Sending build context to Docker daemon  2.048kB
Step 1/5 : FROM centos:7
 ---> 196e0ce0c9fb
Step 2/5 : MAINTAINER spli@dundee.ac.uk
 ---> 737e0a466759
Step 3/5 : RUN yum install -y -q epel-release
 ---> Running in 3e57c2d63ecc
warning: /var/cache/yum/x86_64/7/extras/packages/epel-release-7-9.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for epel-release-7-9.noarch.rpm is not installed
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-4.1708.el7.centos.x86_64 (@CentOS)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
 ---> 45b20e1cc15a
Removing intermediate container 3e57c2d63ecc
Step 4/5 : RUN yum install -y -q python-pip
 ---> Running in 582e51a06c3b
warning: /var/cache/yum/x86_64/7/epel/packages/python2-pip-8.1.2-5.el7.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Public key for python2-pip-8.1.2-5.el7.noarch.rpm is not installed
Importing GPG key 0x352C64E5:
 Userid     : "Fedora EPEL (7) <epel@fedoraproject.org>"
 Fingerprint: 91e9 7d7c 4a5e 96f1 7f3e 888f 6a2f aea2 352c 64e5
 Package    : epel-release-7-9.noarch (@extras)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
 ---> 0630ca1f87a9
Removing intermediate container 582e51a06c3b
Step 5/5 : RUN pip install omego
 ---> Running in 289f0bf4dad2
Collecting omego
  Downloading omego-0.6.4.tar.gz
Collecting yaclifw>=0.1.1 (from omego)
  Downloading yaclifw-0.1.2.tar.gz
Collecting argparse (from yaclifw>=0.1.1->omego)
  Downloading argparse-1.4.0-py2.py3-none-any.whl
Installing collected packages: argparse, yaclifw, omego
  Running setup.py install for yaclifw: started
    Running setup.py install for yaclifw: finished with status 'done'
  Running setup.py install for omego: started
    Running setup.py install for omego: finished with status 'done'
Successfully installed argparse-1.4.0 omego-0.6.4 yaclifw-0.1.2
You are using pip version 8.1.2, however version 9.0.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
 ---> 1e9496172007
Removing intermediate container 289f0bf4dad2
Successfully built 1e9496172007
Successfully tagged my-docker-image:latest
spli@ docker-carpentry (gh-pages)$
~~~
{: .output}
And run
~~~
docker run -it my-docker-image omego version
~~~
{: .bash}
~~~
0.6.4
~~~
{: .output}

Now build it again:
~~~
docker build -t my-docker-image .
~~~
{: .bash}
~~~
Sending build context to Docker daemon  2.048kB
Step 1/5 : FROM centos:7
 ---> 196e0ce0c9fb
Step 2/5 : MAINTAINER spli@dundee.ac.uk
 ---> Using cache
 ---> 737e0a466759
Step 3/5 : RUN yum install -y -q epel-release
 ---> Using cache
 ---> 45b20e1cc15a
Step 4/5 : RUN yum install -y -q python-pip
 ---> Using cache
 ---> 0630ca1f87a9
Step 5/5 : RUN pip install omego
 ---> Using cache
 ---> 1e9496172007
Successfully built 1e9496172007
Successfully tagged my-docker-image:latest
~~~
{: .output}
The build is much quicker, and you may have noticed the ` ---> Using cache` lines.


## More Dockerfile commands

The full list of Dockerfile commands is in the [Docker builder documentation](https://docs.docker.com/engine/reference/builder/).
Some of the most common ones are:
- `COPY`: Copies a file (e.g. a script, configuration file, or archive) into the Docker image
- `USER`: Change the user that a command is run as (useful for dropping privileges)
- `WORKDIR`: Change the current working directory
- `EXPOSE`: Lists ports that should be *exposed* to the outside world
- `VOLUMES`: Directories that should be managed separately from the container (e.g. persistent data that should be kept after the container exits)


{% include links.md %}
