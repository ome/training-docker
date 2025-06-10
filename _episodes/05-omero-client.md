---
title: "Run an OMERO client in Docker"
teaching: 10
exercises: 25
questions:
- "How do I create a Docker image for a complex application?"
objectives:
- "Create a Dockerfile for a complex application"
keypoints:
---

This builds on the previous lesson to create a fully working Docker image for the OMERO python client.

## Installing OMERO.py's dependencies
We install OMERO.py in a virtual environment.
To install OMERO.py we need additional dependencies, most notably the Ice Python bindings.
We install a pre-compiled version matching our Docker image i.e. ``rockylinux`` and the Python version installed i.e. ``3.9``.

~~~
FROM rockylinux:9
LABEL maintainer="OME"

RUN dnf install -y epel-release
RUN dnf install -y python-pip

RUN dnf install -y python3

RUN python3 -mvenv /opt/omero/web/venv3

# But this is a shortcut that installs a precompiled version to install Ice Python
RUN /opt/omero/web/venv3/bin/pip install https://github.com/glencoesoftware/zeroc-ice-py-rhel9-x86_64/releases/download/20230830/zeroc_ice-3.6.5-cp39-cp39-linux_x86_64.whl
RUN /opt/omero/web/venv3/bin/pip install omero-py

RUN useradd omero
WORKDIR /home/omero
USER omero
~~~
{: .source}


~~~
docker build -t my-omeropy-image .
docker run -it my-omeropy-image
~~~
{: .bash}
~~~
/opt/omero/web/venv3/bin/omero version
~~~
{: .bash}
~~~
OMERO.py version:
5.20.0

~~~
{: .output}

Connect to `workshop.openmicroscopy.org` using[CLI commands](https://omero.readthedocs.io/en/stable/sysadmins/cli/index.html), e.g.:
~~~
/opt/omero/web/venv3/bin/omero login -s workshop.openmicroscopy.org
~~~
{: .bash}


## Default commands
When you run `my-omeropy-dockerfile` the `bash` shell is automatically started. This is the default in the parent `rockylinux:9` image.

Change the default command by adding `CMD` to the end of your Dockerfile:
~~~
CMD ["/opt/omero/web/venv3/bin/omero"]
~~~
{: .source}

Re-build, and run:
~~~
docker build -t my-omeropy-image .
docker run -it my-omeropy-image
~~~
{: .bash}
~~~
OMERO Python Shell. Version 5.20
Type "help" for more information, "quit" or Ctrl-D to exit
omero>
~~~
{: .output}

This time you enter the OMERO shell directly.

Can you pass command-line arguments?
~~~
docker run -it my-omeropy-image version
~~~
{: .bash}
~~~
docker: Error response from daemon: oci runtime error: container_linux.go:265: starting container process caused "exec: \"version\": executable file not found in $PATH".
~~~
{: .output}
If you pass a command line to `docker run` it overwrites the default, so you need to pass the whole command line including `omero`:
~~~
docker run -it my-omeropy-image /opt/omero/web/venv3/bin/omero version
~~~
{: .bash}
~~~
OMERO.py version:
5.20.0
~~~
{: .output}

> ## ENTRYPOINT vs CMD
>
> You can make the Docker image behave as the `omero` command using an `ENTRYPOINT`, see the [Docker documentation](https://www.docker.com/blog/docker-best-practices-choosing-between-run-cmd-and-entrypoint/) for details.
{: .callout}


## Best practice for writing Dockerfiles

Writing a Dockerfile is easy, but there are several guidelines you should follow when developing a Docker image for production. [See this article in the Docker documentation](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/) for more details.


{% include links.md %}
