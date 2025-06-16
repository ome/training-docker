---
title: "Orchestration with Docker Compose"
teaching: 20
exercises: 20
questions:
- "How can I manage applications with multiple components"
objectives:
- "Use Docker compose to deploy an OMERO installation"
keypoints:
---

Docker compose (and other orchestration tools) make it easier to deploy applications consisting of multiple Docker containers.

Docker compose deploys an application described by a YAML file. There are multiple versions of the docker-compose format/schema. This example uses version 3.


## Database

Start by creating a `docker-compose.yml` file for just PostgreSQL, including a private network and data volume:
~~~
services:

  database:
    image: "postgres:16"
    environment:
      - POSTGRES_USER=omero
      - POSTGRES_DB=omero
      - POSTGRES_PASSWORD=SeCrEtPaSsWoRd
    networks:
      - omero-network
    volumes:
      - "database-volume:/var/lib/postgresql/data"

networks:
  omero-network:

volumes:
  database-volume:
~~~
{: .source}
Compare this with the command in the previous lesson used to run the postgres container, the mapping between the Docker command-line parameters and the compose file should be clear.

Run ``docker compose`` in the directory containing your `docker-compose.yml` file:
~~~
docker compose -f docker-compose.yml up
~~~
{: .bash}
~~~
Creating network "dockercompose_omero-network" with the default driver
Creating volume "dockercompose_database-volume" with default driver
Creating dockercompose_database_1 ...
Creating dockercompose_database_1 ... done
Attaching to dockercompose_database_1
...
database_1  | LOG:  database system is ready to accept connections
~~~
{: .output}

Next add `omeroserver` to the `services` and `omero-volume` to `volumes`:
~~~
services:

  database:
    image: "postgres:16"
    environment:
      - POSTGRES_USER=omero
      - POSTGRES_DB=omero
      - POSTGRES_PASSWORD=SeCrEtPaSsWoRd
    networks:
      - omero-network
    volumes:
      - "database-volume:/var/lib/postgresql/data"

  omeroserver:
    image: "openmicroscopy/omero-server:5.6.15"
    environment:
      - CONFIG_omero_db_host=database
      - CONFIG_omero_db_user=omero
      - CONFIG_omero_db_pass=SeCrEtPaSsWoRd
      - CONFIG_omero_db_name=omero
      - ROOTPASS=omero-root-password
    networks:
      - omero-network
    ports:
      - "4063:4063"
      - "4064:4064"
    volumes:
      - "omero-volume:/OMERO"

networks:
  omero-network:

volumes:
  database-volume:
  omero-volume:
~~~
{: .source}
Run
~~~
docker compose -f docker-compose.yml up
~~~
{: .bash}
and you should see both the database and OMERO.server running.


We now want to add `omero-web` and `nginx` to `services`.
First we need to create a folder ``nginx`` in the directory where the ``docker-compose.yml`` file is.
This folder will be "mounted".

~~~
mkdir nginx
cd nginx
~~~
{: .bash}

Then add to the ``nginx`` folder a file ``default.conf``:

~~~

map $http_upgrade $connection_upgrade {
  default upgrade;
  '' close;
}

server {
  listen 80 default_server;
  server_name  $hostname;

  # Self-signed certificate for testing
  #listen 443 ssl;
  #ssl_certificate /etc/nginx/ssl/server.pem;
  #ssl_certificate_key /etc/nginx/ssl/server.key;

  resolver 127.0.0.11;

  location / {
    proxy_pass http://omeroweb:4080;
  }
}
~~~
{: .bash}

Add af file ``nginx.conf``:

~~~
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log debug;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
~~~
{: .bash}

We now add `omero-web` and `nginx` to `services`.
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
{: .source}
Run in detach mode i.e. you will be able to close the terminal and disconnect and the server will still be working.

~~~
docker compose -f docker-compose.yml up -d
~~~
{: .bash}
and you should be able to log in to OMERO.web at [http://localhost:8080](http://localhost:8080).


To stop the container

Run
~~~
docker compose -f docker-compose.yml down
~~~
{: .bash}

{% include links.md %}
