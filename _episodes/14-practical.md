---
title: "Modify docker-compose to match a local setup"
teaching: 20
exercises: 20
questions:
- "How can I mount local folder e.g. local storage for in-place import"
objectives:
- "Import an image using `in-place` import"
keypoints:
---
This lesson was used using Ubuntu Virtual Machine.

For convenience, we will mount the ``/tmp`` folder. Usually you will mount a directory
corresponding to your data storage. You can set the permissions when mounting the volume
e.g. mount as read-only.

First we need to stop the server. 
~~~
docker compose -f docker-compose.yml down
~~~
{: .bash}

Now we mount the folder ``/tmp`` in the Virtual Machine to the ``/tmp`` folder in the container.

~~~
services:

  database:
    image: "postgres:16"
    environment:
      - POSTGRES_USER=omero
      - POSTGRES_DB=omero
      - POSTGRES_PASSWORD=SeCrEtPaSsWoRd
      - POSTGRES_HOST_AUTH_METHOD=trust
    networks:
      - omero-network
    ports:
      - "5432:5432"
    volumes:
      - ./pgdata:/var/lib/postgresql/data

  omeroserver:
    # This container uses the tag for the latest omero server release of OMERO 5
    # We mount the local ``/tmp`` folder into the ``/tmp`` folder in container
    image: "openmicroscopy/omero-server:5"
    environment:
      - CONFIG_omero_db_host=database
      - CONFIG_omero_db_name=omero
      - ROOTPASS=omero-root-password
    networks:
      - omero-network
    ports:
      - "4063:4063"
      - "4064:4064"
    volumes:
      - omero-volume:/OMERO
      - /tmp:/tmp

  omeroweb:
    # This container uses the tag for the latest web release of OMERO 5
    image: "openmicroscopy/omero-web-standalone:5"
    environment:
      - OMEROHOST=omeroserver
      - CONFIG_omero_web_csrf__trusted__origins=["http://localhost:8080"]
    networks:
      - omero-network

  nginx:
    image: library/nginx:1.26
    networks:
      - omero-network
    ports:
      - "8080:80"
      - "8443:443"
    volumes:
      - ./nginx/log:/var/log/nginx
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro${VOLOPTS-}
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro${VOLOPTS-}
      #- "./certs:/etc/nginx/ssl:ro${VOLOPTS-}"


networks:
  omero-network:
    # If it fails and your network has a non-standard MTU this may
    # need to be set if auto-detection fails
    driver_opts:
        com.docker.network.driver.mtu: 1450


volumes:
  omero-volume:
~~~
{: .bash}

Run in detach mode i.e. you will be able to close the terminal and disconnect and the server will still be working.

~~~
docker compose -f docker-compose.yml up -d
~~~
{: .bash}


We now want to import an image.
OMERO offers several import options:

* Import by moving an image from one location to the server where OMERO is running. This is commonly done by users
via the Desktop client.
* Import "in-place". This means that a symbolic link between OMERO and the image's location i.e. the image does not move.

For more details please visit [import scenarios](https://omero.readthedocs.io/en/stable/sysadmins/import-scenarios.html)

We will focus on the "in-place" import option, this will match a local setup
i.e. an OMERO server running on a machine with a storage mounted. In our case a Ubuntu Virtual Machine with OMERO running using Docker containers and the mounted storage being ``/tmp``.


First we download a sample PNG image in the ``/tmp`` folder of the VM.

~~~
wget https://downloads.openmicroscopy.org/images/PNG/will/WesternBlots/Hela-frctns-DM1a-5.12.png -O /tmp/Hela-frctns-DM1a-5.12.png
~~~
{: .bash}


Since we have mounted the ``/tmp`` folder of the VM into the ``/tmp`` folder of the Docker container


The OMERO.server runs as the ``omero-server`` user within the Docker container.
We need to access the container as that user

Check the name(s) of the containers:

~~~
docker ps
~~~
{: .bash}

Use the name of the Docker image running the OMERO.server e.g. ``ubuntu-omeroserver-1``.

We will now perform an "in-place" import as the newly created user named ``user1``.

~~~
docker exec -ti -u omero-server  ubuntu-omeroserver-1 /opt/omero/server/venv3/bin/omero import --transfer=ln_s -s localhost -u user1 /tmp/Hela-frctns-DM1a-5.12.png
~~~
{: .bash}




