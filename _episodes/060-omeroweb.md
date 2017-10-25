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
docker run -it my-omeropy-image
~~~
{: .bash}
~~~
OMERO Python Shell. Version 5.4.0-ice36-b74
Type "help" for more information, "quit" or Ctrl-D to exit
omero>
~~~
{: .output}
Try starting web:
~~~
omero> web start
~~~
{: .source}
~~~
ERROR: Django not installed!
omero>
~~~
{: .output}


> ## Additional dependencies for OMERO.web
>
> Install OMERO.web's Python dependencies. Ignore Nginx for now.
> The [OMERO.web documentation](https://docs.openmicroscopy.org/omero/5.4.0/sysadmins/unix/install-web/walkthrough/omeroweb-install-with-server-centos7-ice3.6.html) should be helpful.
>
> > ## Solution
> >
> > Add this line:
> > ~~~
> > RUN pip install -r /home/omero/OMERO.py/share/web/requirements-py27.txt
> > ~~~
> > {: .source}
> {: .solution}
{: .challenge}

It would be nice if OMERO.web could automatically run when the container is started, instead of requiring a manual step.
The `CMD` line can be changed to do this:
~~~
CMD ["/opt/omero/OMERO.py/bin/omero", "web", "start", "--foreground"]
~~~
{: .source}

Build the updated docker image, and run it.
OMERO.web listens on port 4080 be default, so forward this:
~~~
docker run -d --name my-omeroweb -p 14080:4080 my-omeropy-image
and you should be able to see OMERO.web in your web browser [http://localhost:14080](http://localhost:14080)


## OMERO.web configuration

As you will know from the OMERO web documentation Nginx is required to serve static files (e.g. javascript and CSS).
However you can run OMERO.web in development mode for testing, bypassing the requirement for Nginx.

> ## Set OMERO.web to run in development mode
>
> Edit the Dockerfile to configure OMERO.web to use the [built-in development server](https://docs.openmicroscopy.org/omero/5.4.0/developers/Web/Deployment.html#using-the-lightweight-development-server).
>
> > ## Solution
> >
> > Add these lines after the `RUN omego download python ...` line:
> > ~~~
> > RUN /home/omero/OMERO.py/bin/omero config set omero.web.application_server development
> > RUN /home/omero/OMERO.py/bin/omero config set omero.web.debug True
> > ~~~
> > {: .source}
> {: .solution}
{: .challenge}


{% include links.md %}
