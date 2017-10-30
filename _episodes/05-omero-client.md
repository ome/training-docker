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
In the previous image we installed `omego`.
To install OMERO.py we need additional dependencies, most notably Ice.
The "proper" way to do this is to `pip install zeroc-ice`, but since this takes a long time and requires installing a full compiler toolchain we can cheat and install a pre-compiled version.
~~~
FROM centos:7
MAINTAINER spli@dundee.ac.uk

RUN yum install -y epel-release
RUN yum install -y python-pip
RUN pip install omego

# This is the proper way to install Ice Python:
#RUN pip install zeroc-ice
# But this is a shortcut that installs a precompiled version
RUN pip install https://github.com/openmicroscopy/zeroc-ice-py-centos7/releases/download/0.0.3/zeroc_ice-3.6.4-cp27-cp27mu-linux_x86_64.whl

RUN useradd omero
WORKDIR /home/omero
USER omero
RUN omego download python --ice 3.6 --sym OMERO.py
~~~
{: .source}
If you are feeling lazy you can [download the Dockerfile](../code/my-omeropy-dockerfile/Dockerfile).

~~~
docker build -t my-omeropy-image .
docker run -it my-omeropy-image
~~~
{: .bash}
~~~
/home/omero/OMERO.py/bin/omero version
~~~
{: .bash}
~~~
ERROR:omero.gateway:No Pillow installed, line plots and split channel will fail!
5.4.0-ice36-b74
~~~
{: .output}

> ## Fix the error/warning
>
> `ERROR:omero.gateway:No Pillow installed` indicates there is a missing dependency. Can you modify the Dockerfile to remove this error?
>
> > ## Solution
> >
> > Add this line:
> > ~~~
> > RUN pip install pillow
> > ~~~
> > after the existing `RUN pip install` lines.
> > {: .source}
> {: .solution}
{: .challenge}

It's a bit annoying that you have to type the full path to `/home/omero/OMERO.py/bin/omero`, so add it to the `PATH` in the image using the Dockerfile `ENV` command:
~~~
ENV PATH="/home/omero/OMERO.py/bin:${PATH}"
~~~
{: .source}
Rebuild the image. You should now have a fully working OMERO.py client. Try connecting to `nightshade.openmicroscopy.org` and running some [CLI commands](https://docs.openmicroscopy.org/omero/5.4.0/sysadmins/cli/index.html), e.g.:
~~~
omero login -s nightshade.openmicroscopy.org
omero user list
~~~
{: .bash}


## Default commands
When you run `my-omeropy-dockerfile` the `bash` shell is automatically started. This is the default in the parent `centos:7` image.

Change the default command by adding `CMD` to the end of your Dockerfile:
~~~
CMD ["/opt/omero/OMERO.py/bin/omero"]
~~~
{: .source}

Re-build, and run:
~~~
docker build -t my-omeropy-image .
docker run -it my-omeropy-image
~~~
{: .bash}
~~~
OMERO Python Shell. Version 5.4.0-ice36-b74
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
docker run -it my-omeropy-image omero version
~~~
{: .bash}
~~~
5.4.0-ice36-b74
~~~
{: .output}

> ## ENTRYPOINT vs CMD
>
> You can make the Docker image behave as the `omero` command using an `ENTRYPOINT`, see the Docker documentation for details.
{: .callout}

> ## Advanced: Compile and install Ice
>
> The current image uses a pre-compiled version of Ice.
> Modify the image to install ice using the recommended Zeroc method (`pip install zeroc-ice`).
> You will need to install a compiler and all development libraries required to compile Ice.
{: .challenge}


## Best practice for writing Dockerfiles

Writing a Dockerfile is easy, but there are several guidelines you should follow when developing a Docker image for production. [See this article in the Docker documentation](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/) for more details.


{% include links.md %}
