---
title: "Run OMERO.web in Docker"
teaching: 10
exercises: 25
questions:
- "How can I run OMERO.web in Docker"
objectives:
- "Further develop the OMERO python image to run OMERO web."
keypoints:
---

This builds on previous lessons to create an OMERO.web image running in development mode.

Start with the image built in the previous lesson

~~~
RUN pip install omero-web
~~~
{: .bash}
~~~

~~~
docker run -it my-omeropy-image
~~~
{: .bash}
~~~
ERROR: OMERODIR not set
omero>
~~~
{: .output}


> ## Set OMERODIR
>
> Create a directory as the root user and set the correct permissions
> The [OMERO.web documentation](https://omero.readthedocs.io/en/stable/sysadmins/unix/install-web/walkthrough/omeroweb-install-rockylinux9-ice3.6.html) should be helpful. We ingore the installation of Nginx for now.

~~~
RUN mkdir /opt/omero/web/OMERO.web/
RUN chown -R omero:omero  /opt/omero/web/OMERO.web/
ENV OMERODIR=/opt/omero/web/OMERO.web/
~~~
{: .bash}

To start automatically OMERO.web when the container is started, instead of requiring a manual step, we need to modify
the `CMD` line to:
~~~
CMD ["/opt/omero/web/venv3/bin/omero", "web", "start", "--foreground"]
~~~
{: .source}
~~~
544 static files copied to '/opt/omero/web/OMERO.web/var/static', 2 post-processed.
Clearing expired sessions. This may take some time... [OK]
Starting OMERO.web... [2025-06-10 14:54:15 +0000] [130] [INFO] Starting gunicorn 23.0.0
[2025-06-10 14:54:15 +0000] [130] [INFO] Listening at: http://127.0.0.1:4080 (130)
[2025-06-10 14:54:15 +0000] [130] [INFO] Using worker: sync
[2025-06-10 14:54:15 +0000] [131] [INFO] Booting worker with pid: 131
[2025-06-10 14:54:15 +0000] [132] [INFO] Booting worker with pid: 132
[2025-06-10 14:54:15 +0000] [133] [INFO] Booting worker with pid: 133
[2025-06-10 14:54:15 +0000] [134] [INFO] Booting worker with pid: 134
~~~
{: .output}

The `--foreground` flag is necessary because OMERO.web defaults to running in the background as a daemon, but Docker will exit as soon as the foreground process completes.

Build the updated docker image, and run it.
OMERO.web listens on port 4080 be default, so we need to forward the port:
~~~
docker run -d --name my-omeroweb -p 14080:4080 my-omeropy-image
~~~
{: .bash}
and you should be able to see OMERO.web in your web browser [http://localhost:14080](http://localhost:14080) if you are running the installation locally.


## OMERO.web configuration

As you will know from the OMERO web documentation Nginx is required to serve static files (e.g. javascript and CSS).
However you can run OMERO.web in development mode for testing, bypassing the requirement for Nginx.

> ## Set OMERO.web to run in development mode
>
> Edit the Dockerfile to configure OMERO.web to use the [built-in development server](https://omero.readthedocs.io/en/stable/developers/Web/Deployment.html#using-the-lightweight-development-server).
>
> > ## Solution
> >
> > Add these lines after the `RUN omego download python ...` line:
> > ~~~
> > RUN /opt/omero/web/venv3/bin/omero config set omero.web.application_server development
> > RUN /opt/omero/web/venv3/bin/omero config set omero.web.debug True
> > ~~~
> > {: .source}
> {: .solution}
{: .challenge}


{% include links.md %}
