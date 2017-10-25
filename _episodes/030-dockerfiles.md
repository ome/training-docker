---
title: "Creating new Docker images"
teaching: 15
exercises: 15
questions:
- "How can you create your own Docker image?"
objectives:
- "Create a new Docker image from scratch"
- "Understand what a Dockerfile is."
keypoints:
---

The previous lesson showed how to run existing docker images.
This lesson will demonstrate how to build your own images.

## What is a Dockerfile?

A Dockerfile is a text file containing a set of instructions for reproduceably building a Docker image


## My first Dockerfile
TODO: find a better application to demonstrate this?
~~~
{ % include code/my-first-dockerfile/Dockerfile % }
~~~
{: .source}

Build it. `-t` let's you give the image a name
~~~
docker build -t my-docker-image .
~~~
{: .bash}
~~~
...
 ---> 60af0824a7d1
Removing intermediate container b705e824a9f2
Successfully built 60af0824a7d1
Successfully tagged my-docker-image:latest
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


## More Dockerfile commands

The full list of Dockerfile commands is in the [Docker builder documentation](https://docs.docker.com/engine/reference/builder/).
Some of the most common ones are:
- `COPY`: Copies a file (e.g. a script, configuration file, or archive) into the Docker image
- `USER`: Change the user that a command is run as (useful for dropping privileges)
- `WORKDIR`: Change the current working directory
- `EXPOSE`: Lists ports that should be *exposed* to the outside world
- `VOLUMES`: Directories that should be managed separately from the container (e.g. persistent data that should be kept after the container exits)


{% include links.md %}
