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

Docker containers are intended to be ephemeral which means it should be possible to regularly stop, delete and replace them without loosing anything important.

This means that in contrast to a physical server where all storage is persistent you must carefully consider where you application writes to. Despite the inconvenience this is a good thing.

Persistent data should be stored in a [docker volume](https://docs.docker.com/engine/admin/volumes/volumes/), which is an abstraction of a chunk of a file-system.

> ## What happens if you run this?
>
> ~~~
> docker run -it my-omeropy-image omero login -s nightshade.openmicroscopy.org
> docker run -it my-omeropy-image omero user list
> ~~~
> {: .bash}
> > The second command prompts you to login again. It is a completely separate Docker container, so the login state from the first command is not passed to the second container.
> > {: .solution}
{: . challenge}

## Docker named volumes

Create a new volume for storing the OMERO login state:
~~~
docker volume create omero-session-vol
~~~
{: .bash}
~~~
omero-session-vol
~~~
{: .output}
List volumes:
~~~
docker volume ls
~~~
{: .bash}
~~~
DRIVER              VOLUME NAME
local               omero-session-vol
~~~
{: .output}

When you run an image you can mount a docker volume into the container using the `--mount` argument. Mount `omero-session-vol` on `/home/omero/omero`:
~~~
docker run -it --mount source=omero-session-vol,target=/home/omero/omero my-omeropy-image omero login -s nightshade.openmicroscopy.org
~~~
{: .bash}
~~~
WARNING:omero.util.TempFileManager:Invalid tmp dir: /home/omero/omero/tmp
Traceback (most recent call last):
  File "/home/omero/OMERO.py/lib/python/omero/util/temp_files.py", line 172, in tmpdir
    self.create(target)
  File "/home/omero/OMERO.py/lib/python/omero/util/temp_files.py", line 240, in create
    dir.makedirs(0700)
  File "/home/omero/OMERO.py/lib/python/path.py", line 1244, in makedirs
    os.makedirs(self, mode)
  File "/usr/lib64/python2.7/os.py", line 157, in makedirs
    mkdir(name, mode)
OSError: [Errno 13] Permission denied: '/home/omero/omero/tmp'
Could not access session dir: /home/omero/omero/sessions
~~~
{: .output}

The volume defaults to being owned by root. You can fix this by creating the mountpoint with the correct permissions in the Dockerfile
~~~
RUN useradd omero
WORKDIR /home/omero
USER omero
RUN omego download python --ice 3.6 --sym OMERO.py

RUN mkdir /home/omero/omero
~~~
{: .source}
Rebuild, and run. The session information should now be kept between container runs:
~~~
docker run -it --mount source=omero-session-vol,destination=/home/omero/omero my-omeropy-image omero login -s nightshade.openmicroscopy.org
docker run -it --mount source=omero-session-vol,destination=/home/omero/omero my-omeropy-image omero user list
~~~
{: .bash}

> ## `-v` and  `--mount`
>
> `--mount` is a new argument introduced in version 17.06. Docker recommend using `--mount` instead of `-v`, though both methods are mostly equivalent.
>
> If you are using an old version of Docker replace the `--mount ...` argument with `-v omero-session-vol:/home/omero/omero` in the above example.
{: .callout}


## Other volume options

Volumes can be mounted with several other options, e.g. `readonly`:
~~~
--mount source=omero-session-vol,destination=/home/omero/omero,readonly
~~~
{: .bash}
See the [documentation on volumes](https://docs.docker.com/engine/admin/volumes/volumes/) for more information.

## Bind mounts

Docker allows you to [mount a local path on the host directly into the container](https://docs.docker.com/engine/admin/volumes/bind-mounts/).
For example, you could use this to mount a directory of test files into a container for testing. However, this makes the container dependent on the file structure of the host, and also requires manually dealing with user permissions. Named volumes are easier to use.

Example or mounting your local home directory into a container:
~~~
docker run -it --mount type=bind,source="$HOME",destination=/external-home,readonly centos:7 ls -l /external-home/
~~~
{: .bash}

## Volume management

- List volumes: `docker volume ls`
- Delete a volume: `docker volume rm VOLUME-NAME`
- Delete all unattached volumes: `docker volume prune`


{% include links.md %}
