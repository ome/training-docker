---
title: "Running Docker images"
teaching: 10
exercises: 25
questions:
- "How do you run an image?"
- "How can you access it?"
objectives:
- "Run a Docker image interactively."
- "Run a Docker image in the background."
keypoints:
---

This is an introduction to running and accessing pre-built Docker images.

## Where do images come from?

## Running Docker images interactively

First fetch an image:
~~~
docker pull centos:7
~~~
{: .bash}
~~~
7: Pulling from library/centos
Digest: sha256:eba772bac22c86d7d6e72421b4700c3f894ab6e35475a34014ff8de74c10872e
Status: Downloaded newer image for centos:7
~~~
{: .output}
- `centos`: The name of the Docker image that you want to run
- `:7`: The version/tag of the Docker image (default is `:latest`)

Then run it:
~~~
docker run -it centos:7 bash
~~~
{: .bash}
~~~
[root@3b02ac0aebba /]#
~~~
{: .output}

You now have a bash shell inside your Docker container.

- `-it`: Run "interactively"
- `bash`: The arguments to pass to the Docker image (most images have a default so this can often be omitted)

Make some changes inside the container:
~~~
echo hello world > hello.txt
ls
~~~
{: .bash}

~~~
anaconda-post.log  etc        lib         media  proc  sbin  tmp
bin                hello.txt  lib64       mnt    root  srv   usr
dev                home       lost+found  opt    run   sys   var
~~~
{: .output}

The default user in the `centos` image is `root`, so you can install things:
~~~
yum install -y wget
~~~
{: .bash}

Exit the container:
~~~
exit
~~~
{: .bash}

Now run the same command again:
~~~
docker run -it centos:7 bash
~~~
{: .bash}
List the files again:
~~~
ls
~~~
{: .bash}

~~~
anaconda-post.log  etc   lib64       mnt   root  srv  usr
bin                home  lost+found  opt   run   sys  var
dev                lib   media       proc  sbin  tmp
~~~
{: .output}

`hello.txt` is missing. This is a completely new container.


## Running Docker images in the background (`-d`/`--detach`)

Running images interactively is a useful for testing or development, for example if you want to test some installation instructions for OMERO, or check whether OMERO will compile with an updated system library.

Often you will want to run an image in the background.

First `pull` the `nginx` image:
~~~
docker pull nginx
~~~
{: .bash}
~~~
Using default tag: latest
latest: Pulling from library/nginx
bc95e04b23c0: Pull complete
110767c6efff: Pull complete
f081e0c4df75: Pull complete
Digest: sha256:004ac1d5e791e705f12a17c80d7bb1e8f7f01aa7dca7deee6e65a03465392072
Status: Downloaded newer image for nginx:latest
~~~
{: .output}

Pulling an image will download the image if it doesn't exist, or update it if it does. The latter is a like `git pull` on a branch.

Now run it in the background using the `-d`/`--detach` flag:
~~~
docker run -d --name my-nginx nginx
~~~
{: .bash}
~~~
dcf62aa4694b3a0fc846b7670aaec8c8c6aac09a0f59443e49966cf680be8802
~~~
{: .output}

All containers have an ID. `--name` is used to give a human-readable name `my-nginx` to the running container.

List all running containers:
~~~
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                               NAMES
dcf62aa4694b        nginx               "nginx -g 'daemon ..."   About a minute ago   Up About a minute   80/tcp                              my-nginx
~~~
{: .output}

What happens if you run another container with the same name?
~~~
docker run -d --name my-nginx nginx
~~~
{: .bash}
~~~
docker: Error response from daemon: Conflict. The container name "/my-nginx" is already in use by container "dcf62aa4694b3a0fc846b7670aaec8c8c6aac09a0f59443e49966cf680be8802". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
~~~
{: .output}

Stop the container:
~~~
docker stop my-nginx
~~~
{: .bash}
~~~
my-nginx
~~~
{: .output}
And delete it:
~~~
docker rm my-nginx
~~~
{: .bash}
~~~
my-nginx
~~~
{: .output}

## Connecting to a port (`-p`/`--publish`)

In the output of `docker ps` you might have noticed `80/tcp`. This indicates Nginx is listening on port 80 inside the container, but to access it from outside the container you must setup port-forwarding using the `-p`/`--publish` argument.

Syntax: `-p external-host-port-number:internal-container-port-number`
~~~
docker run -d --name my-nginx -p 12345:80 nginx
~~~
{: .bash}
~~~
829209f5ed461af738ac9bd75dc36c36cfdf7c43281f403939b666c1c020a4ab
~~~
{: .output}
List running containers again:
~~~
docker ps
~~~
{: .bash}
~~~
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
829209f5ed46        nginx               "nginx -g 'daemon ..."   2 seconds ago       Up 1 second         0.0.0.0:12345->80/tcp               my-nginx
~~~
{: .output}

This time you can see `0.0.0.0:12345->80/tcp`, which indicates port 12345 on the server running docker will be forwarded to port 80 in the container. Open [http://localhost:12345](http://localhost:12345) and you should see a `Welcome to nginx!` page.

If you have multiple ports you can pass the `-p` argument multiple times. Docker can automatically setup port forwarding for you if you pass the `-P` flag:
~~~
docker rm -f my-nginx
docker run -d --name my-nginx -P nginx
docker ps
~~~
{: .bash}
~~~
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
69f7daadf908        nginx               "nginx -g 'daemon ..."   4 seconds ago       Up 3 seconds        0.0.0.0:32768->80/tcp               my-nginx
~~~
{: .output}
This time open [http://localhost:32768](http://localhost:32768) (change 32768 to whichever port Docker has chosen for you).

## Accessing the container directly

Most containers are written to only run one application so won't have SSH enabled. If you need to access a container, e.g. for debugging, use the `exec` command.

For example, cat the `nginx.conf` file in the container:
~~~
docker exec my-nginx cat /etc/nginx/nginx.conf
~~~
{: .bash}
~~~

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

...
~~~
{: .output}

You can run bash interactively inside the container:
~~~
docker exec -it my-nginx bash
~~~
{: .bash}
~~~
root@69f7daadf908:/#
~~~
{: .output}
~~~
id
exit
~~~
{: .bash}
~~~
uid=0(root) gid=0(root) groups=0(root)
~~~
{: .output}

## Viewing container outputs and logs

A well designed container will print out it's logs to stdout. These can be viewed with the `logs` command:
~~~
docker logs my-nginx
~~~
{: .bash}
~~~
172.17.0.1 - - [20/Oct/2017:12:52:05 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.12; rv:56.0) Gecko/20100101 Firefox/56.0" "-"
172.17.0.1 - - [20/Oct/2017:12:52:05 +0000] "GET /favicon.ico HTTP/1.1" 404 169 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.12; rv:56.0) Gecko/20100101 Firefox/56.0" "-"
2017/10/20 12:52:05 [error] 7#7: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:32768"
~~~
{: .output}

> ## Following docker logs in real-time
>
> All docker commands have a built-in `--help` argument. Can you run `docker logs` so that when you visit http://localhost:NNNNN you see a log message immediately?
>
> > ## Solution
> >
> > ~~~
> > docker logs -f my-nginx
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}


## Adding content to a running docker container
The docker `cp` command can copy files to and from a running container.
For example, copy the nginx.conf file to your computer:
~~~
docker cp my-nginx:/etc/nginx/nginx.conf .
ls
~~~
{: .bash}

Create a local file, and copy this into the Nginx html directory


~~~
docker cp data/nginx-new-index.html my-nginx:/usr/share/nginx/html/index.html
~~~
{: .bash}
Refresh your browser, you should see your new index page.

There are other ways of sharing files with a docker container, these will be covered later.


{% include links.md %}
