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

A Dockerfile is a text file containing a set of instructions for reproducibly building a Docker image


## My first Dockerfile
Create a new directory, enter it, and create a file called `Dockerfile`:
~~~
mkdir my-first-dockerfile
cd my-first-dockerfile
<edit> Dockerfile e.g. use vi Dockerfile
~~~
{: .bash}
~~~
FROM rockylinux:9
LABEL maintainer="ome-devel@lists.openmicroscopy.org.uk"

RUN dnf install -y -q epel-release
RUN dnf install -y -q python-pip
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
Building 17.7s (8/8) FINISHED                                                                                                                                                                              
 => [internal] load build definition from Dockerfile                                                                                                                                                           0.0s
 => => transferring dockerfile: 199B                                                                                                                                                                           0.0s
 => [internal] load .dockerignore                                                                                                                                                                              0.0s
 => => transferring context: 2B                                                                                                                                                                                0.0s
 => [internal] load metadata for docker.io/library/rockylinux:9                                                                                                                                                0.0s
 => [1/4] FROM docker.io/library/rockylinux:9                                                                                                                                                                  0.0s
 => [2/4] RUN dnf install -y -q epel-release                                                                                                                                                                   5.0s
 => [3/4] RUN dnf install -y -q python-pip                                                                                                                                                                    10.4s
 => [4/4] RUN pip install omego                                                                                                                                                                                1.9s 
 => exporting to image                                                                                                                                                                                         0.3s 
 => => exporting layers                                                                                                                                                                                        0.3s 
 => => writing image sha256:5655f61f4aa8a546b788b9d34a3f48238e5f9cf4b2d7eb976aceffd0bd406878                                                                                                                   0.0s 
 => => naming to docker.io/library/my-docker-image   
~~~
{: .output}
And run
~~~
docker run -it my-docker-image omego version
~~~
{: .bash}
~~~
b'0.7.0'
~~~
{: .output}

Now build it again:
~~~
docker build -t my-docker-image .
~~~
{: .bash}
~~~
 Building 0.1s (8/8) FINISHED                                                                                                                                                                                    
 => [internal] load build definition from Dockerfile                                                                                                                                                           0.0s
 => => transferring dockerfile: 34B                                                                                                                                                                            0.0s
 => [internal] load .dockerignore                                                                                                                                                                              0.0s
 => => transferring context: 2B                                                                                                                                                                                0.0s
 => [internal] load metadata for docker.io/library/rockylinux:9                                                                                                                                                0.0s
 => [1/4] FROM docker.io/library/rockylinux:9                                                                                                                                                                  0.0s
 => CACHED [2/4] RUN dnf install -y -q epel-release                                                                                                                                                            0.0s
 => CACHED [3/4] RUN dnf install -y -q python-pip                                                                                                                                                              0.0s
 => CACHED [4/4] RUN pip install omego                                                                                                                                                                         0.0s
 => exporting to image                                                                                                                                                                                         0.0s
 => => exporting layers                                                                                                                                                                                        0.0s
 => => writing image sha256:5655f61f4aa8a546b788b9d34a3f48238e5f9cf4b2d7eb976aceffd0bd406878                                                                                                                   0.0s
 => => naming to docker.io/library/my-docker-image  
~~~
{: .output}
The build is much quicker, and you may have noticed the `=> CACHED` lines.


## More Dockerfile commands

The full list of Dockerfile commands is in the [Docker builder documentation](https://docs.docker.com/engine/reference/builder/).
Some of the most common ones are:
- `COPY`: Copies a file (e.g. a script, configuration file, or archive) into the Docker image
- `USER`: Change the user that a command is run as (useful for dropping privileges)
- `WORKDIR`: Change the current working directory
- `EXPOSE`: Lists ports that should be *exposed* to the outside world
- `VOLUMES`: Directories that should be managed separately from the container (e.g. persistent data that should be kept after the container exits)


{% include links.md %}
