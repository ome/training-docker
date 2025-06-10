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
Compare this with the command in the previous lesson used to run the postgres container- the mapping between the Docker command-line parameters and the compose file should be clear.

Run docker-compose in the directory containing your `docker-compose.yml` file:
~~~
docker compose up
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
docker compose up
~~~
{: .bash}
and you should see both the database and OMERO.server running.

Finally add `omero-web` to `services`:
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

  omeroweb:
    image: "openmicroscopy/omero-web-standalone:latest"
    environment:
      - OMEROHOST=omeroserver
    networks:
      - omero-network
    ports:
      - "4080:4080"

networks:
  omero-network:

volumes:
  database-volume:
  omero-volume:
~~~
{: .source}
Run
~~~
docker compose up
~~~
{: .bash}
and you should be able to log in to OMERO.web at [http://localhost:4080](http://localhost:4080).


{% include links.md %}
