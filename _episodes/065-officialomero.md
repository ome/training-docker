---
title: "Official OMERO images"
teaching: 10
exercises: 25
questions:
- "How do I run a multi-container application?"
- "How should I run OMERO Docker in production?"
objectives:
- "Know how to use the official OMERO Docker images."
keypoints:
---

The previous lessons built up a Dockerfile from scratch.
The production OMERO images are slightly different.

## OMERO on Docker Hub

The official OMERO Docker images are built with Ansible, primarily because we need to support the installation of OMERO on multiple platforms including physical servers and virtual machines.
Using Ansible means only one set of instructions has to be maintained.


## Running production OMERO.server

Pull all the required images:

~~~
docker pull openmicroscopy/omero-server:5.4.0
~~~
{: .bash}
~~~
5.4.0: Pulling from openmicroscopy/omero-server
Digest: sha256:4dcfb9b7612144c750ffccfffb743bcd1265d513838bb96fdc86e41a75447740
Status: Image is up to date for openmicroscopy/omero-server:5.4.0
~~~
{: .output}
~~~
docker pull openmicroscopy/omero-web:5.4.0
~~~
{: .bash}
~~~
5.4.0: Pulling from openmicroscopy/omero-web
Digest: sha256:31ea4c1d0bb329a32a9223c9be4352903865c1eb3cd28faf2f35fad5792415a8
Status: Image is up to date for openmicroscopy/omero-web:5.4.0
~~~
{: .output}
~~~
docker pull postgres:9.6
~~~
{: .bash}
~~~
9.6: Pulling from library/postgres
Digest: sha256:318757ed6291e6a1ef86312ac453b9b4a67b48495b59ca2dece909cb0c688c53
Status: Image is up to date for postgres:9.6
~~~
{: .output}

### Database and Networking

A naive way to join containers together is to use port forwarding, and configure each container to connect to the required port. However, this is insecure.

Instead you can create a separate Docker network for you application. This ensures:
- only containers within the same network can talk to each other (unless you forward a port)
- Docker runs an internal DNS server that lets you reference other images by name

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
docker run -d --name my-db-server --network my-omero-network -e POSTGRES_USER=omero -e POSTGRES_DB=omero -e POSTGRES_PASSWORD=SeCrEtPaSsWoRd postgres:9.6
~~~
{: .bash}

This will set a password on the default `postgres` user and database. If you want to create a separate user and database see the [documentation on Docker Hub](https://hub.docker.com/_/postgres/).

You can check the DNS resolution by attempting to ping my-db-server from another container outside or inside the network you created:
~~~
docker run -it --rm centos ping my-db-server
~~~
{: .bash}
~~~
ping: my-test-db-server: Name or service not known
~~~
{: .output}

~~~
docker run -it --rm --network my-omero-network centos ping my-db-server
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

### OMERO.server

Run OMERO.server. OMERO requires several configuration parameters to be set; these can be passed by defining environment variables with `-e`:
~~~
docker run -d --name my-omero-server --network my-omero-network -e CONFIG_omero_db_host=my-db-server -e CONFIG_omero_db_user=omero -e CONFIG_omero_db_pass=SeCrEtPaSsWoRd -e CONFIG_omero_db_name=omero -e ROOTPASS=omero-root-password -p 4063:4063 -p 4064:4064 openmicroscopy/omero-server:5.4.0
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

Since this is a new PostgreSQL server an OMERO database didn't exist, so it was initialised automatically. If an old version of the database (e.g. corresponding to OMERO 5.3) was present it would've been automatically upgraded.

Ports 4063 and 4064 are forwarded, so you should be able to login as `root` from an OMERO.client on your machine.

> ## Connect to the server using an OMERO Docker image
>
> In the previous session you created an Docker image `my-omeropy-image` containing OMERO.py which includes the CLI.
> Run this image and login to the OMERO.server.
>
> > ## Solution
> >
> > ~~~
> > docker run -it --rm --network my-omero-network my-omeropy-image
> > OMERO Python Shell. Version 5.4.0-ice36-b74
> > Type "help" for more information, "quit" or Ctrl-D to exit
> > omero> login -s my-omero-server -u root
> > Password:
> > Created session 982fedea-f91b-4b9e-bc8e-564b935b5286 > > (root@my-omero-server:4064). Idle timeout: 10 min. Current group: system
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

### OMERO.web

As with OMERO.server the production OMERO.web image requires a few parameters to configure the connection to OMERO.server:
~~~
docker run -d --name my-omero-web --network my-omero-network -e OMEROHOST=my-omero-server -p 4080:4080 openmicroscopy/omero-web:5.4.0
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
[2017-10-25 14:26:10 +0000] [57] [INFO] Starting gunicorn 19.7.1
[2017-10-25 14:26:10 +0000] [57] [INFO] Listening at: http://0.0.0.0:4080 (57)
[2017-10-25 14:26:10 +0000] [57] [INFO] Using worker: sync
[2017-10-25 14:26:10 +0000] [62] [INFO] Booting worker with pid: 62
[2017-10-25 14:26:10 +0000] [63] [INFO] Booting worker with pid: 63
[2017-10-25 14:26:10 +0000] [68] [INFO] Booting worker with pid: 68
[2017-10-25 14:26:10 +0000] [73] [INFO] Booting worker with pid: 73
[2017-10-25 14:26:10 +0000] [74] [INFO] Booting worker with pid: 74
~~~
{: .output}
OMERO.web is now running on port 4080, try connecting to it on [http://localhost:4080](http://localhost:4080).

There is one big problem- the Django static files aren't served.
A typical production installation of OMERO.web also requires an Nginx server.




## Additional notes

- There are [multiple types of network](https://docs.docker.com/engine/userguide/networking/#bridge-networks), for instance to allow container running on different hosts to communicate. In most cases the default type (`bridge`) if fine.
- Some of these Docker design choices are motivated by the [12 factor app](https://12factor.net/).

{% include links.md %}
