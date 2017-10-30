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
version: "3"

services:

  database:
    image: "postgres:9.6"
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
docker-compose up
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
version: "3"

services:

  database:
    image: "postgres:9.6"
    environment:
      - POSTGRES_USER=omero
      - POSTGRES_DB=omero
      - POSTGRES_PASSWORD=SeCrEtPaSsWoRd
    networks:
      - omero-network
    volumes:
      - "database-volume:/var/lib/postgresql/data"

  omeroserver:
    image: "openmicroscopy/omero-server:5.4.0"
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
docker-compose up
~~~
{: .bash}
and you should see both the database and OMERO.server running.

Finally add `omero-web` to `services`:
~~~
version: "3"

services:

  database:
    image: "postgres:9.6"
    environment:
      - POSTGRES_USER=omero
      - POSTGRES_DB=omero
      - POSTGRES_PASSWORD=SeCrEtPaSsWoRd
    networks:
      - omero-network
    volumes:
      - "database-volume:/var/lib/postgresql/data"

  omeroserver:
    image: "openmicroscopy/omero-server:5.4.0"
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
    image: "openmicroscopy/omero-web-standalone:master"
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
docker-compose up
~~~
{: .bash}
and you should be able to log in to OMERO.web at [http://localhost:4080](http://localhost:4080).

If you are feeling lazy you can [download the complete docker-compose.yml file](../code/docker-compose/docker-compose.yml)

> ## Modifying the OMERO configuration
>
> Modify `docker-compose.yml` to include some non-default configuration, for example enable a public user in OMERO.web
>
> > ## Solution
> >
> > Add the following to the `omeroweb` `environment`:
> > ~~~
> > - CONFIG_omero_web_public_enabled=True
> > - CONFIG_omero_web_public_user=root
> > - CONFIG_omero_web_public_password=omero-root-password
> > - CONFIG_omero_web_public_url__filter=^/
> > ~~~
> > {: .bash}
> > This is intended to be a quick example, you should obviously avoid using `root` as your public user in production.
> {: .solution}
{: .challenge}

{% include links.md %}
