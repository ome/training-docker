---
title: "Running Docker images"
teaching: 10
exercises: 25
questions:
- "How do you run an image?"
- "How can you access it?"
objectives:
- "Run a Docker image."
- "Fetch logs from running a container."
- "Obtain a shell inside a container."
keypoints:
- "Docker Hub is a repository of prebuilt Docker images"
- "`docker run` always creates a new container"
- "`docker ps` lists containers"
- "`docker logs` will fetch application logs in a well-designed container"
- "When developing or testing `docker exec` and `docker cp` may be useful"
---

This is an introduction to running and accessing pre-built Docker images.

## Where do images come from?

Docker (the organisation) hosts a central registry of images: [https://hub.docker.com/explore/](https://hub.docker.com/explore/). You can easily use third party registries, or run your own.

Docker Hub is like any other application or source code repository. There are some official images (e.g. `ubuntu`), but anyone can push anything to it under a prefix (user or organisation name).


## Running Docker images interactively

First fetch an image:
~~~
docker pull rockylinux:9
~~~
{: .bash}
~~~
9: Pulling from library/rockylinux
4c81ef64b3e1: Pull complete 
Digest: sha256:d7be1c094cc5845ee815d4632fe377514ee6ebcf8efaed6892889657e5ddaaa6
Status: Downloaded newer image for rockylinux:9
docker.io/library/rockylinux:9
~~~
{: .output}
- `rockylinux`: The name of the Docker image that you want to run
- `:9`: The version/tag of the Docker image (default is `:latest`)

Then run it:
~~~
docker run -it rockylinux:9 bash
~~~
{: .bash}
~~~
[root@2b5f11235d99 /]#
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
afs  bin  dev  etc  hello.txt  home  lib  lib64
lost+found  media  mnt  opt  proc  root  run
sbin  srv  sys  tmp  usr  var
~~~
{: .output}

The default user in the `rockylinux` image is `root`, so you can install things:
~~~
dnf install -y wget
~~~
{: .bash}

Exit the container:
~~~
exit
~~~
{: .bash}

Now run the same command again:
~~~
docker run -it rockylinux:9 bash
~~~
{: .bash}
List the files again:
~~~
ls
~~~
{: .bash}

~~~
afs  bin  dev  etc  home  lib  lib64
lost+found  media  mnt  opt  proc  root  run
sbin  srv  sys  tmp  usr  var
~~~
{: .output}

`hello.txt` is missing. This is a completely new container.

Exit the container:
~~~
exit
~~~
{: .bash}

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
b16f1b166780: Pull complete 
c9a20772aff4: Pull complete 
5c242ffc14bb: Pull complete 
bf2e7af999d2: Pull complete 
5cbad9890292: Pull complete 
4cf85f4d417b: Pull complete 
99f78d9a3fb1: Pull complete 
Digest: sha256:fb39280b7b9eba5727c884a3c7810002e69e8f961cc373b89c92f14961d903a0
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
~~~
{: .output}

Pulling an image will download the image if it does not exist, or update it if it does. The latter is a like `git pull` on a branch.

Now run it in the background using the `-d`/`--detach` flag:
~~~
docker run -d --name my-nginx nginx
~~~
{: .bash}
~~~
b9ba7a701e1f59a9366ac7b88899d81a9262aeb671ac1d0cab7344ddfe8bb3ef
~~~
{: .output}

All containers have an ID. `--name` is used to give a human-readable name `my-nginx` to the running container.

List all running containers:
~~~
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                               NAMES
b9ba7a701e1f        nginx               "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp                              my-nginx
~~~
{: .output}

What happens if you run another container with the same name?
~~~
docker run -d --name my-nginx nginx
~~~
{: .bash}
~~~
docker: Error response from daemon: Conflict. The container name "/my-nginx" is already in use by container "b9ba7a701e1f59a9366ac7b88899d81a9262aeb671ac1d0cab7344ddfe8bb3ef". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
~~~
{: .output}


## Listing containers (`ps`)
List running containers using the `ps` command:
~~~
docker ps
~~~
{: .bash}
~~~
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
d90cdd419eb6        nginx               "/docker-entrypoint.…"   4 seconds ago       Up 3 seconds        80/tcp                              my-nginx
~~~
{: .output}

List containers including those that aren't running:
~~~
docker ps -a
~~~
{: .bash}
~~~
CONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS                      PORTS                               NAMES
d90cdd419eb6        nginx                               "/docker-entrypoint.…"   52 seconds ago      Up 51 seconds               80/tcp                              my-nginx
a7a77a946eaa        rockylinux:9                             "bash"                   2 minutes ago       Exited (0) 2 minutes ago                                         great_khorana
~~~
{: .output}


## Stopping and removing containers (`stop`, `rm`)
Stop the Nginx container:
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
~~~
docker ps -a
~~~
{: .bash}
~~~
CONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS                      PORTS                               NAMES
a7a77a946eaa        rockylinux:9                           "bash"                   2 minutes ago       Exited (0) 2 minutes ago                                        great_khorana
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
8f52b7ddfd8fea65112945cd346b38024fa465938bf6b07f99a204a878b397ac
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
This time open [http://localhost:32768](http://localhost:32768) (change `32768` to whichever port Docker has chosen for you).

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

In general you should not need to use `docker exec` on a production container.


## Viewing container outputs and logs

A well designed container will print out its logs to stdout. These can be viewed with the `logs` command:
~~~
docker logs my-nginx
~~~
{: .bash}
~~~
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/06/10 12:02:07 [notice] 1#1: using the "epoll" event method
2025/06/10 12:02:07 [notice] 1#1: nginx/1.27.5
2025/06/10 12:02:07 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14) 
2025/06/10 12:02:07 [notice] 1#1: OS: Linux 5.10.76-linuxkit
2025/06/10 12:02:07 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2025/06/10 12:02:07 [notice] 1#1: start worker processes
2025/06/10 12:02:07 [notice] 1#1: start worker process 30
2025/06/10 12:02:07 [notice] 1#1: start worker process 31
2025/06/10 12:02:07 [notice] 1#1: start worker process 32
2025/06/10 12:02:07 [notice] 1#1: start worker process 33
172.17.0.1 - - [10/Jun/2025:12:02:49 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36" "-"
2025/06/10 12:02:50 [error] 32#32: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:12345", referrer: "http://localhost:12345/"
172.17.0.1 - - [10/Jun/2025:12:02:50 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://localhost:12345/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36" "-"
172.17.0.1 - - [10/Jun/2025:12:02:57 +0000] "\x16\x03\x01\x07\x12\x01\x00\x07\x0E\x03\x03+\xEB\xF2\x9A\xE7P\xDE_\xD1\xF3\xD6\x0E\xA1P?\xAE\x98v\x11J\xCA\xCC\xF4u\xC6\x86!\xDE\xD8\xEB\x98\x83 \xFF\x90\xDA\x5C\xCC\xA1c\xD9\x0F\xC4'\x9E\x04\xACu\xFE\xF7\x80=\xBB\xB0\xB6\xEF\x9B\x1B`ax\x0B\xC4\x13\xEA\x00 \xEA\xEA\x13\x01\x13\x02\x13\x03\xC0+\xC0/\xC0,\xC00\xCC\xA9\xCC\xA8\xC0\x13\xC0\x14\x00\x9C\x00\x9D\x00/\x005\x01\x00\x06\xA5zz\x00\x00\xFF\x01\x00\x01\x00\x00+\x00\x07\x06" 400 157 "-" "-" "-"
172.17.0.1 - - [10/Jun/2025:12:02:57 +0000] "\x16\x03\x01\x06\xF2\x01\x00\x06\xEE\x03\x03\xC4*" 400 157 "-" "-" "-"
172.17.0.1 - - [10/Jun/2025:12:02:57 +0000] "\x16\x03\x01\x07\x12\x01\x00\x07\x0E\x03\x03\x02yJ\xFAv\x17\xBC\xD7\x0E_C.\xFC\xB3\x06\x1C\xEB\x16\xA2\xBC\xA0Hkd\x22\xC6\xF9\x16\x82\xDC\xE2Z \xDF\xE7\xAC+\xC3\xF8'\x82T\xBE\xC3\xDD\x00Pu\x96r\xC4R\xEC\xDF\xA7\xF6\xD8{\xDEg4\xE2\xF4r\x03\x00 JJ\x13\x01\x13\x02\x13\x03\xC0+\xC0/\xC0,\xC00\xCC\xA9\xCC\xA8\xC0\x13\xC0\x14\x00\x9C\x00\x9D\x00/\x005\x01\x00\x06\xA5\xAA\xAA\x00\x00\x00\x1B\x00\x03\x02\x00\x02D\xCD\x00\x05\x00\x03\x02h2\x00\x12\x00\x00\x00" 400 157 "-" "-" "-"
172.17.0.1 - - [10/Jun/2025:12:02:57 +0000] "\x16\x03\x01\x06\xD2\x01\x00\x06\xCE\x03\x03m+\x92n\xF5\xAA;\xED\xBB9\x97\x05]V\xAA\xEF<\x0F\xAD\xBA\x99\xF2\x9F\x7F\x80\x15\x97{\x91\xDD\xAE\x14 \x97n`\xF85wf\xA4\xF2\xD2\x12n\x8C&a\xD4\x1E\xAC\x84\x9D\x18\xBA\xB2\x0CI1," 400 157 "-" "-" "-"
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

Create a new local file called [nginx-new-index.html](../data/nginx-new-index.html), and copy this into the Nginx html directory:
~~~
docker cp nginx-new-index.html my-nginx:/usr/share/nginx/html/index.html
~~~
{: .bash}
Refresh your browser, you should see your new index page.

There are other ways of sharing files with a docker container, these will be covered later.


{% include links.md %}
