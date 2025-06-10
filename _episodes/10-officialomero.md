---
title: "Official OMERO images"
teaching: 20
exercises: 30
questions:
- "How do I run a multi-container application such as OMERO?"
objectives:
- "Create a segregated Docker network"
- "Run the official OMERO Docker images."
keypoints:
- "Use docker networks to isolate an application, and to provide name-based lookups"
- "The official OMERO Docker images can be configured by defining environment variables to set most omero properties"
- "The OMERO.server image magically takes care of initialising and upgrading the database"
- "Use Docker volumes to store persistent data such as the OMERO database and data directory"
---

The previous lessons built up a Dockerfile from scratch.
The production OMERO images are slightly different.

## OMERO on Docker Hub

The official OMERO Docker images are built with Ansible, primarily because we need to support the installation of OMERO on multiple platforms including physical servers and virtual machines.
Using Ansible means only one set of instructions has to be maintained.


## Running OMERO.server

Pull all the required images:
~~~
docker pull openmicroscopy/omero-server:5.6.15
docker pull openmicroscopy/omero-web:5.29.0-1
docker pull postgres:16
~~~
{: .bash}

### Database and Networking

A way to join containers together is to use port forwarding, and configure each container to connect to the required port. However, this is insecure.

Instead you can create a separate Docker network for you application. This ensures:
- only containers within the same network can talk to each other (unless you forward a port)
- Docker runs an internal DNS server that let's you reference other images by name

If you do not specify a network Docker provides a default network, but this does not support communication between containers.

Create a new network:
~~~
docker network create my-omero-network
~~~
{: .bash}
~~~
1c9195622f603ac9820aa2717534cf9a206c6cad76875a0406abc04f0ebfcbbd
~~~
{: .output}

Start your PostgreSQL server, specifying the network you created as a parameter:
~~~
docker run -d --name my-db-server --network my-omero-network -e POSTGRES_USER=omero -e POSTGRES_DB=omero -e POSTGRES_PASSWORD=SeCrEtPaSsWoRd postgres:15
~~~
{: .bash}

The container is configured by defining environment variables with the `-e` parameter.
The above example creates a user and database named `omero`, and sets a password. See the [documentation on Docker Hub](https://hub.docker.com/_/postgres/) for more parameters.

You can check the DNS resolution by attempting to ping my-db-server from another container.
First try from a container on the default docker network:
~~~
docker run -it --rm rocky ping my-db-server
~~~
{: .bash}
~~~
ping: my-test-db-server: Name or service not known
~~~
{: .output}
Then try it on the network you created:
~~~
docker run -it --rm --network my-omero-network rocky ping my-db-server
~~~
{: .bash}
~~~
PING my-db-server (172.18.0.2) 56(84) bytes of data.
64 bytes from my-db-server.my-omero-network (172.18.0.2): icmp_seq=1 ttl=64 time=0.082 ms
64 bytes from my-db-server.my-omero-network (172.18.0.2): icmp_seq=2 ttl=64 time=0.122 ms
^C
--- my-db-server ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.082/0.102/0.122/0.020 ms
~~~
{: .output}

> ## Docker links
>
> Older versions of docker used the `--link` argument to link containers together. This is deprecated; you should use networks instead.
{: .callout}

### OMERO.server

Run OMERO.server. OMERO requires several configuration parameters to be set; these can be passed by defining environment variables with `-e`:
~~~
docker run -d --name my-omero-server --network my-omero-network -e CONFIG_omero_db_host=my-db-server -e CONFIG_omero_db_user=omero -e CONFIG_omero_db_pass=SeCrEtPaSsWoRd -e CONFIG_omero_db_name=omero -e ROOTPASS=omero-root-password -p 4063:4063 -p 4064:4064 openmicroscopy/omero-server:5.6.15
~~~
{: .bash}
Other ways of setting configuration variables for OMERO include passing in a configuration file, but this has the disadvantage of being harder to deploy when using remote Docker servers.

OMERO parameters (`-e`):
- `CONFIG_omero_db_host`: `omero.db.host`
- `CONFIG_omero_db_user`: `omero.db.user`
- `CONFIG_omero_db_pass`: `omero.db.pass`
- `CONFIG_omero_db_name`: `omero.db.name`
- `ROOTPASS`: OMERO root password

Run docker logs to check the server is running:
~~~
docker logs -f my-omero-server
~~~
{: .bash}
~~~
Running /startup/50-config.py
Running /startup/60-database.sh
postgres connection established
Initialising database
...
Running /startup/99-run.sh
Starting OMERO.server
No descriptor given. Using etc/grid/default.xml
~~~
{: .output}
You should now have a functional OMERO.server.

Ports `4063` and `4064` are forwarded, so you can login as the OMERO `root` user from an OMERO.client on your machine.

> ## Database initialisation and upgrades
>
> Since this is a new PostgreSQL server an OMERO database didn't exist, so it was initialised automatically. If an old version of the database (e.g. OMERO 5.4) was present it would be automatically upgraded.
{: .callout}

> ## Problem: Connect to the server using an OMERO Docker image
>
> In the previous session you created an Docker image `my-omeropy-image` containing OMERO.py which includes the CLI.
> Run this image and login to the OMERO.server.
>
> > ## Solution
> >
> > ~~~
> > docker run -it --network my-omero-network my-omeropy-image omero login -s my-omero-server -u root
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

### OMERO.web

As with OMERO.server the production OMERO.web image requires a few parameters to configure the connection to OMERO.server:
~~~
docker run -d --name my-omero-web --network my-omero-network -e OMEROHOST=my-omero-server -p 4080:4080 openmicroscopy/omero-web:5.29.0-1
~~~
{: .bash}
Since the most common configuration is to connect to a single OMERO.server this Docker image has a special environment variable, `OMEROHOST`, that will automatically configure OMERO.web to connect to that server.

~~~
docker logs -f my-omero-web
~~~
{: .bash}
~~~
Running /startup/50-config.py
Running /startup/60-default-web-config.sh
Running /startup/99-run.sh
Starting OMERO.web
...
604 static files copied to '/opt/omero/web/OMERO.web/lib/python/omeroweb/static', 2 post-processed.
Clearing expired sessions. This may take some time... [OK]

~~~
{: .output}
OMERO.web is now running on port 4080, try connecting to it on [http://localhost:4080](http://localhost:4080).

There is one big problem- the Django static files aren't served.
There are two options at present.

A typical production installation of OMERO.web also requires an Nginx server. You can either an Nginx container with the OMERO.web statics bundled and configured to proxy to the OMERO.web container, the easiest option is to use a Django plugin called [WhiteNoise](http://whitenoise.evans.io/en/stable/) that serves static files directly through Django:
~~~
docker run -d --name my-omero-web-standalone --network my-omero-network -e OMEROHOST=my-omero-server -p 4081:4080 openmicroscopy/omero-web-standalone:latest
~~~
{: .bash}
You should be able to connect to OMERO.web on port 4081: [http://localhost:4081](http://localhost:4081).



### Check what's running
~~~
docker ps
~~~
{: .bash}
~~~
CONTAINER ID        IMAGE                                        COMMAND                  CREATED              STATUS              PORTS                               NAMES
15d4d60826e8        openmicroscopy/omero-web-standalone:latest   "/usr/local/bin/en..."   40 seconds ago       Up 39 seconds       0.0.0.0:4081->4080/tcp              my-omero-web-standalone
31931074e5b6        openmicroscopy/omero-web:5.29.0-1               "/usr/local/bin/en..."   51 seconds ago       Up 49 seconds       0.0.0.0:4080->4080/tcp              my-omero-web
0bbfb6ccef0a        openmicroscopy/omero-server:5.6.15            "/usr/local/bin/en..."   About a minute ago   Up About a minute   0.0.0.0:4063-4064->4063-4064/tcp    my-omero-server
b3603499d161        postgres:16                                "docker-entrypoint..."   About a minute ago   Up About a minute   5432/tcp                            my-db-server
~~~
{: .output}
~~~
docker network ls
~~~
{: .bash}
~~~
NETWORK ID          NAME                DRIVER              SCOPE
d8339aacac4c        bridge              bridge              local
04228510796c        host                host                local
4502df24113e        my-omero-network    bridge              local
19a36ee67994        none                null                local
~~~
{: .output}


## Persistent data

Docker containers should not contain any important state, such as the OMERO and PostgreSQL data directories. Instead you should use [volumes](https://docs.docker.com/engine/admin/volumes/volumes/) which provide an abstraction of a file-system.

You will usually need to consult the documentation for the Docker images you are using to find out where persistent data is stored. A well written Dockerfile should use the `VOLUME` command to indicate the locations of persistent data. When you run a container Docker automatically creates volumes for you.

~~~
docker volume ls
~~~
{: .bash}
~~~
DRIVER              VOLUME NAME
local               11b5de6192a4ff33d543fab74c974d45818291b17ab645aa6b8c2476d1945b1d
local               ad1226f1baaa3a2c3075cff363d0e6957b1f254dfa55ffcbd0b7927af30f3699
local               b1f172df2a2ccc260185030a3eed570ebfd86be617166c038a57793a3df54ebf
...
~~~
{: .output}

The default volume names aren't very useful, but can give a volume a name if you create it yourself.

### Using volumes with OMERO

First delete the existing omero and database containers (keep the standalone web container running):
~~~
docker stop my-omero-server my-db-server my-omero-web
docker rm my-omero-server my-db-server my-omero-web
~~~
{: .bash}

Create data volumes for the PostgreSQL and OMERO data directories:
~~~
docker volume create my-db-data
docker volume create my-omero-data
~~~
{: .bash}

Mount `my-db-data` on `/var/lib/postgresql/data` when running PostgreSQL:
~~~
docker run -d --name my-db-server --mount source=my-db-data,destination=/var/lib/postgresql/data --network my-omero-network -e POSTGRES_USER=omero -e POSTGRES_DB=omero -e POSTGRES_PASSWORD=SeCrEtPaSsWoRd postgres:16
~~~
{: .bash}
Mount `my-omero-data` on `/OMERO` when running OMERO.server:
~~~
docker run -d --name my-omero-server --mount source=my-omero-data,destination=/OMERO --network my-omero-network -e CONFIG_omero_db_host=my-db-server -e CONFIG_omero_db_user=omero -e CONFIG_omero_db_pass=SeCrEtPaSsWoRd -e CONFIG_omero_db_name=omero -e ROOTPASS=omero-root-password -p 4063:4063 -p 4064:4064 openmicroscopy/omero-server:5.6.15
~~~
{: .bash}


## Additional notes

- There are [multiple types of network](https://docs.docker.com/engine/userguide/networking/#bridge-networks), for instance to allow container running on different hosts to communicate. In most cases the default type (`bridge`) is fine.
- Some of these Docker design choices are motivated by the [12 factor app](https://12factor.net/).
- If you want to contribute to or discuss the existing images have a look at the open issues:
  - [OMERO.web](https://github.com/openmicroscopy/omero-server-docker/issues)
  - [OMERO.server](https://github.com/openmicroscopy/omero-web-docker/issues)

{% include links.md %}
