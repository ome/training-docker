---
title: "Pulling and pushing to Docker Hub"
teaching: 10
exercises: 15
questions:
- "How do I publish my own images on Docker Hub?"
objectives:
- "Push an image to Docker Hub"
keypoints:
- "Use `docker tag` to tag an image with the Docker Hub name before pushing with `docker push`"
---

## Images on Docker Hub

Docker image on Docker Hub follow the pattern `<hub-user>/<repo-name>[:<tag>]`, e.g.:
~~~
docker pull openmicroscopy/omero-server:5.6.15
~~~
{: .bash}

A small number of images are official Docker images and have a hub-user `library` which can be omitted, e.g. these are equivalent:
~~~
docker pull rockylinux:9
docker pull library/rockylinux:9
~~~
{: .bash}

There are [two ways to publish an image on Docker Hub](https://docs.docker.com/docker-hub/repos/), either manually, or by setting up an automated build.

## Pushing to Docker Hub

First you must give your locally built image the same name that it would be published with on Docker Hub. Rename your `my-omeropy-image`, replacing `ometraining` with your Docker Hub username:
~~~
docker tag my-omeropy-image ometraining/my-omeropy-image:0.1
~~~
{: .bash}
You must create a repository on Docker Hub before you can push to it: [https://hub.docker.com/add/repository/](https://hub.docker.com/add/repository/).
<img alt="Docker Hub add repository" src="{{ page.root }}/fig/docker-hub-add-repository.png" />
<img alt="Docker Hub add repository" src="{{ page.root }}/fig/docker-hub-repository-added.png" />

Login to Docker Hub:
~~~
docker login
~~~
{: .bash}
~~~
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: ometraining
Password:
Login Succeeded
~~~
{: .output}
Push the image to Docker Hub:
~~~
$ docker push ometraining/my-omeropy-image:0.1
~~~
{: .bash}
~~~
The push refers to a repository [docker.io/ometraining/my-omeropy-image]
40d9bb63f99b: Pushed
91f63c570966: Pushed
74d0a95d2fc0: Pushed
128806d79d0a: Pushed
071366fe9ed5: Pushed
96359d46c527: Pushed
889193115614: Pushed
ee8e03e5ccba: Pushed
cf516324493c: Pushed
0.1: digest: sha256:9ac559fb2ee9b5bccfcb831d2359fe2d7f6a462ffeb66fc7840918a7dbe07d99 size: 2214
~~~
{: .output}
If you push was successful you should [see the image listed under `tags` on https://hub.docker.com/r/ometraining/my-omeropy-image/tags/](https://hub.docker.com/r/ometraining/my-omeropy-image/tags/) (replace `ometraining` with your username)

> ## The `latest` tag
>
> `latest` is a special default tag.
> It is not updated automatically when you push to a tag, you must push to it separately.
{: .callout}

You should now be able to pull someone else's image:
~~~
docker pull some-other-user/my-omeropy-image:0.1
~~~
{: .bash}


## Automated builds

Docker Hub can optionally be linked directly to GitHub (or BitBucket) so that pushing to a GitHub repository will automatically update the Docker Hub image (Docker Hub runs `docker build` on its own infrastructure).
This works in a similar way to how GitHub actions automatically runs tests when you push to GitHub.


> ## Alternative Docker registries
>
> When you pull an image there are actually 4 components: `<registry.address>/<user>/<repository>:<tag>`.
> The default registry is Docker Hub but many alternative registries are available, and you can setup your own.
{: .callout}

{% include links.md %}
