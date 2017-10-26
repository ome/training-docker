---
title: "Run an OMERO client in Docker"
teaching: 10
exercises: 25
questions:
- "How do I create a Docker image for a complex application?"
- "How can you run it?"
objectives:
- "Create complex Dockerfiles."
keypoints:
---

This builds on the previous lesson to create a fully working Docker image for the OMERO python client.

## Installing OMERO.py's dependencies
In the previous image we installed `omego`.
To install OMERO.py we need additional dependencies, most notably Ice.
The "proper" way to do this is to `pip install zeroc-ice`, but since this takes a long time and requires installing a full compiler toolchain we can cheat and install a pre-compiled version.
~~~
{ % include code/my-omeropy-dockerfile/Dockerfile % }
~~~
{: .source}

~~~
docker build -t my-omeropy-image .
docker run -it my-omeropy-image
~~~
{: .bash}
~~~
/opt/omero/OMERO.py/bin/omero version
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
> > {: .source}
> {: .solution}
{: .challenge}


You should now have a fully working OMERO.py client. Try connecting to `nightshade.openmicroscopy.org` and running some [CLI commands](https://docs.openmicroscopy.org/omero/5.4.0/sysadmins/cli/index.html).


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
If you pass a command line to `docker run` it overwrites the default, so you need to explicitly include `/home/omero/OMERO.py/bin/omero`:
~~~
docker run -it my-omeropy-image /home/omero/OMERO.py/bin/omero version
~~~
{: .bash}
~~~
5.4.0-ice36-b74
~~~
{: .output}

This can be made more user-friendly by adding `/home/omero/OMERO.py/` to the `PATH` in the image using the Dockerilfe `ENV` command:
~~~
ENV PATH="/home/omero/OMERO.py/bin:${PATH}"
~~~
{: .source}
~~~
docker run -it my-omeropy-image omero version
~~~
{: .bash}
~~~
5.4.0-ice36-b74
~~~
{: .output}


> ## Advanced: Compile and install Ice
>
> The current image uses a pre-compiled version of Ice.
> Modify the image to install ice using the recommended Zeroc method (`pip install zeroc-ice`).
> You will need to install a compiler and all development libraries required to compile Ice.
{: .challenge}



Advanced: https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/

{% include links.md %}
