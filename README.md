# Short docker intro (with 2 examples)

![Maintenance status](https://img.shields.io/badge/Maintained%3F-yes-brightgreen)
[![MIT license](https://img.shields.io/badge/license-MIT-brightgreen)](https://opensource.org/licenses/MIT)

## Objective

> This tutorial aims to give a brief overview over the docker software and technology. As practical part there will be a step by step instruction on how to maintain (several) local database management systems (in parallel) on one machine, using Docker.

<details>
  <summary>Table of Contents</summary>
  <p>

- [Short docker intro (with 2 examples)](#short-docker-intro-with-2-examples)
  - [Objective](#objective)
  - [What is docker?](#what-is-docker)
    - [On a high level](#on-a-high-level)
    - [More practical](#more-practical)
  - [When, why and what for do I need docker?](#when-why-and-what-for-do-i-need-docker)
  - [Getting started](#getting-started)
  - [Docker commands cheat-sheet](#docker-commands-cheat-sheet)
  - [SQL commands cheat sheet](#sql-commands-cheat-sheet)
  - [Example 1 - create and run a mysql rdbms](#example-1---create-and-run-a-mysql-rdbms)
    - [Prerequisits](#prerequisits)
    - [Download official mysql docker image](#download-official-mysql-docker-image)
    - [Run an image inside a docker container](#run-an-image-inside-a-docker-container)
    - [Access app inside docker container (port mapping)](#access-app-inside-docker-container-port-mapping)
    - [Docker volumes](#docker-volumes)
      - [Create a docker volume](#create-a-docker-volume)
      - [Create a new container and attach a docker volume](#create-a-new-container-and-attach-a-docker-volume)
      - [Insert test data](#insert-test-data)
      - [Test volume persistance](#test-volume-persistance)
  - [Example 2 - Run 2 mysql servers in parallel (v5.7 + v8.x)](#example-2---run-2-mysql-servers-in-parallel-v57--v8x)
  - [Create your own docker image](#create-your-own-docker-image)
  - [Dockerfile](#dockerfile)
  - [CREDITS](#credits)
  - [Links](#links)
  - [License](#license)
  
  </p>
</details>

## What is docker?

### On a high level
- Docker is an OS-level virtualization software that separates applications from it's infrastructure
- an alternative to hypervisor-based virtual machines
- it provides a standard way to run code/applications
- it's like a virtualized operating system of a server

### More practical
- docker packages and runs applications in a standardized virtual unit, called "container"
- a docker container encapsulates everything an application needs to run (and only those things!), including libraries, system tools, code, and runtime.
    - containers are isolated system processes on a host and are separated from one another (that's why they bundle their own software, libraries and configuration files)
    - containers can communicate with each other through well-defined channels

## When, why and what for do I need docker?

- docker allows developers to work in standardized environments using local containers (which provide all necessary applications, libraries and services to run)
- Containers are lightweight and contain everything needed to run the application, so there's no need to rely on what is currently installed on the host
- multiple containers can run simultaneously on a given host
- containers can be shared and everyone you share it with gets the same container (which always works in the same way!)
- provides developers and admins a reliable, low-cost way to build, test, ship, and run distributed applications at any scale

## Getting started

* Create a (free) docker user account ([https://hub.docker.com/](https://hub.docker.com/)).

* Download + install the [Docker Desktop App](https://www.docker.com/products/docker-desktop).

* Open your Terminal and execute `$ docker -v` to see if the installation was successful (*expected output: a docker version string like `Docker version 20.10.6, build 370c289`*).

## Docker commands cheat-sheet

* `$ docker -v` - show docker version string
* `$ docker pull image:tag` - pull an image (like cloning in git)
* `$ docker images` - list all LOCALLY available docker images
* `$ docker ps` - list currently running (booted) containers
* `$ docker ps -a` - list all active containers (running or stopped)
* `$ docker run image` - run a docker image inside a container
* `$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -p 3306:3306 -d mysql:5.7` - starts a docker container
    * `--name` - tag to set a new *custom container name* (e.g. "some-mysql")
    * `-e` - sets an *environment variable* (e.g. to store mysql db root password)
    * `-d` - set a specific *tag* for an image
    * `-p 3306:3306` -p port flag: map **host port** '3306' (first number) to **container port** '3306' (second number) inside the **container**
    * ***ATTENTION***: if this container is deleted and recreated later on, the data inside will be LOST (to avoid that create and attach a local docker volume using the `-v` flag)
* `$ docker stop <image-name>` - stops a running container
* `$ docker start <image-name>` - starts a container again
* `$ docker rm <image-name>` - remove a local docker **image** by name
* `$ docker rmi <image-ID>` remove a local docker **image** by ID (get image ID using `$ docker images`) Works only, if **NO** other docker container uses that image
* `$ docker volume --help` - help for docker volume commands
* `$ docker volume ls` - list all created docker volumes
* `$ docker volume create <new_volume_name>` - create a new docker volume
* `$ docker volume inspect <volume_name>` - show detailed information of an existing docker volume
* `$ docker rm <volume_name>` - remove an existing docker volume
* `$ docker run --name some-mysql8 -e MYSQL_ROOT_PASSWORD=my-secret-pw -v mysql8_data:/var/lib/mysql -p 3307:3306 -d mysql:8` - starts a docker container with an attached local docker volume (an accessible file system)
  * using `-v mysql8_data:/var/lib/mysql` appends an existing local docker volume named "mysql8_data" to the container and links it to the given path INSIDE the container
  * if the container is deleted and recreated later on, the ***DATA WON'T BE LOST*** (because the data is stored inside the attached local docker volume)

## SQL commands cheat sheet

```sql
create database `docker-mysql8`;
use `docker-mysql8`;
create table collegues (name varchar(255));
insert into collegues(name) values ("CEO");
insert into collegues(name) values ("Product Manager");
insert into collegues(name) values ("Product Owner");
insert into collegues(name) values ("Marketing");
insert into collegues(name) values ("Sales");
```

## Example 1 - create and run a mysql rdbms

Install and run 2 different mySQL rdmbs versions (5.7 and 8.x) next to each other in a docker container

### Prerequisits

* log into your [docker hub user account](http://hub.docker.com/sso/start)

* once logged in search for the official mysql docker image (named "mysql")

* you will be prompted a link to the official [mysql docker image](https://hub.docker.com/_/mysql).

* to get the mysql docker image (that includes the mysql rdbms app) and create a docker container that runs the image inside, we have to **pull the official mysql docker *image*** using the `$ docker pull image-name:tag-name` command

### Download official mysql docker image

> Loading a pre-build docker image is like *cloning a repo* in Git. For our example we'll load a pre-build docker image of a mysql rdbms (version 5.7).
 
To download/pull the docker image run
  * `$ docker pull mysql:5.7` - adding the tag `:5.7` tells docker to pull version 5.7 of the mysql rdbms
  * As a result the image will be copied into the local file system.
      
Run `$ docker images` to see if the pulled image is locally available

### Run an image inside a docker container

> Now that we pulled the mysql docker image we need to start/run it. Therefor we need a *docker container*.

* a container is a normal operating system process except that this process is **isolated** from every other process. It has its **own file system**, **networking** and **isolated process tree**, all separate from the host.

* to learn how we can boot up our pulled image inside a docker container we'll consult the official *mysql docker image online documentation*  ([https://hub.docker.com/_/mysql](https://hub.docker.com/_/mysql), section: "How to use this image" → "Start a mysql server instance")

* the documentation suggests the following command to **boot up/run** the image inside a container:

  * `$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag`, where
  
  * `--name` assigns a *name* to the new container (here: "some-mysql")
  * `-e` sets an *environment variable* to store the mysql **root password** for the database connection (demo root db password is: my-secret-pw)
  * `-d` runs the container in detached mode (in the background)
  * `:tag` sets the 'tag' for the mysql version (in our version '5.7' is the one we want to install - could be named anything else...)

* type and execute `$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:5.7` (just changed 'tag' to '5.7')

* execute `$ docker ps -a` to see if the docker container is up and running

> Now that we know how to *create* and *boot/run* a local docker container! But to **use** the container we need to know how to *ACCESS / connect* to the mysql server *inside* the docker container!

### Access app inside docker container (port mapping)

> Knowing how to search and pull a docker image and how to create and boot it inside a docker container, we will now see how to *CONNECT* to the container to *ACCESS* it's contents (in our case the mysql server)!

* to *ACCESS* the container content it is necessary to *CONNECT* to it by creating a **port mapping** between a host/client port (3306) and a container port (3306) => Without the port mapping, we wouldn’t be able to access the application inside the container! Let's perform some preparatory steps first:
  
  * step 1: stop the currently running example container with `$ docker stop some-mysql`

  * step 2: delete the example container by executing `$ docker rm some-mysql`

  * step 3: run `$ docker ps -a` to check the currently running containers (*some-mysql* should NOT be listed as a running docker container anymore!)

* we use the same *run* command as before but with an additional port mapping flag (-p): `$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -p 3306:3306 -d mysql:5.7`, where
  
  * `-p 3306:3306` maps *host port* '3306' (first number) to *container port* '3306' (second number)
  
* now that we've got an accessible container lets **connect to the mysql rdbms** inside the container and fill it  with some test data

  * open up a mysql client (we use the [TablePlus app](https://www.tableplus.io/download))

  * **create a new db connection** (right click -> New -> Connection -> MySQL) and provide neccessary connection information:
    * *Name*: Mysql
    * *Ver*: MySQL 5.x (dropdown menu)
    * *Host*: 127.0.0.1
    * *Port*: 3306
    * *User*: root
    * *Password*: my-secret-pw
  * press 'TEST' button -> if test ok (all fields are green) press 'Connect' button
  * **create sample data** using TablePlus's App (SQL query interface):

    ```sql
    create database docker;
    use docker;
    create table people (name varchar(255));
    ```

  * Press 'database button' (the stacked db icon) to connect to the database itself
    * in opening dialog select 'docker' (our database name)
    * Add some random names for people db in 'Name' field of table

* if we now **STOP and RESTART** the container, the data in the DB will still be available!
  > `$ docker stop some-mysql` - stops the container
  >
  > `$ docker start some-mysql` - restarts the container

* but if we **STOP, REMOVE and then RE-RUN** the container the *DB DATA WILL BE LOST* (this is because the data is stored in a container, thus in a file system that we do not have direct access to) !!

  > `$ docker stop some-mysql` - stops the container
  >
  > `$ docker rm some-mysql` - deletes the container
  >
  > `$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -p 3306:3306 -d mysql:5.7` - re-runs the container (recreate + reconnect container)
  >
  > `$ docker ps -a` - shows if re-run was successful
  >
  > open TablePlus and reconnect the db

* The solution to **NOT LOSE ANY DATA** (even when deleting the docker container) is: attaching a [docker volume](https://docs.docker.com/storage/volumes/) to the container.

### Docker volumes

> Think of a docker volume as a *virtual hard drive* (a file system we have access to) that is attached to the docker container (the entity where the docker image is running inside).

* docker volume needs to be attached when container is **CREATED**

* to see all commands related to docker volumes, type `$ docker volume --help`.
> At this point we have to delete the example docker container again (see steps above) and re-create it - only this time attached to a docker volume.

#### Create a docker volume

* create a docker volume with `$ docker volume create <volume_name>`

  * in our example we use `$ docker volume create mysql_data`

* list docker volumes by running `$ docker volume ls` (should list the new docker volume "mysql_data")

* ***But, which path inside the container do I need to attach the volume to?***
  
  * let's check official mysql image online documentation ([https://hub.docker.com/_/mysql](https://hub.docker.com/_/mysql), section: "Where to store data"):

  * documentation points out to use `-v /my/own/datadir:/var/lib/mysql` added to the `$ docker run ...` command, where

    * `-v` volume flag
    * `/my/own/datadir` volume name (as given with *$ docker volume create <volume_name>*)
    * `/var/lib/mysql` exact path ***inside the docker container*** where to attach the volume to.

#### Create a new container and attach a docker volume

* add previously created docker volume `mysql_data` to the `docker run` command and execute it:

  ```zsh
  $ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -v mysql_data:/var/lib/mysql -p 3306:3306 -d mysql:5.7
  ```
  
  * this re-runs (recreate and reconnect) the docker container, this time with a docker volume to store the data attached to it
  
* Is new container running? `$ docker ps -a`

#### Insert test data

> Let's use the mysql server inside the docker container to create a new database and add some test data to it.

* open TablePlus app
* add a new db connection (enter connection information as shown above)
* execute following SQL queries to create and fill database table:

  ```sql
  create database docker;
  use docker;
  create table peolpe(name varchar(255));
  insert into people(name) values ("Jessi");
  insert into people(name) values ("Alex");
  insert into people(name) values ("Alba");
  insert into people(name) values ("Arno");
  insert into people(name) values ("Frieda");
  ```

#### Test volume persistance

> Let's test the VOLUME PERSISTENCE by deleting the new container and bringing it back up again connected to the same docker volume.

```zsh
$ docker stop some-mysql
$ docker rm some-mysql
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -v mysql_data:/var/lib/mysql -p 3306:3306 -d mysql:5.7
```

* open TablePlus and reconnect the db
* you'll see, that the database 'docker' is still there and the table 'people' including the test data is still intact!

> At this point we have a mysql server connected with our local port 3306 and using a persistent file system we have access to (docker volume). For a normal local setup this is more than enough!
> 
> But the real power behind docker is shown in the next example. 

## Example 2 - Run 2 mysql servers in parallel (v5.7 + v8.x)

> The real power behind docker is that everything is containerized. This gives us the ability to run different versions of the mysql server next to each other and in parallel.
> 
> So for our next example we will create and run an additional docker container running a mysql v8.x image. It will run right next to our already created docker container (mysql v5.7).

* `$ docker pull mysql:8` pull down mysql version 8.x
* `$ docker volume create mysql8_data` create new docker volume "mysql8_data" that we use for mysql version 8
* `$ docker volume ls` check if volume has been created
* `$ docker run --name some-mysql8 -e MYSQL_ROOT_PASSWORD=my-secret-pw -v mysql8_data:/var/lib/mysql -p 3307:3306 -d mysql:8`
  
  * run mysql:8 image inside a container named "some-mysql8"
  * pass a root password and set it as an environment variable
  * attach container to a docker volume named "mysql8_data" (that connects to '/var/lib/mysql' inside the container) and
  * connect the local port 3307 to the container port 3306

* `$ docker ps -a` see running docker containers

> That's it. We're done! We now run 2 different mysql server versions next to each other in 2 separate docker containers.

## Create your own docker image

> This section describes how to create your own docker image

Instead of *downloading* an official (pre-built) docker image you can create your own by using the `$ docker built command`.

An image includes everything you need to run an application - the code or binary, runtime, dependencies, and any other file system objects required.

When we tell Docker to build our image by executing the `$ docker build` command, Docker reads the required built instructions from a Dockerfile and executes them one by one (in succession ) and creates a Docker image as a result.

Example: [create a simple node.js application in a docker container](https://docs.docker.com/language/nodejs/build-images/)

## Dockerfile

 Using `docker build`, users can create an automated build that executes several command-line instructions in succession (see: [Dockerfile best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)). Images are built automatically by reading the ordered instructions from a `Dockerfile`.
 
 A `Dockerfile` is a text document that *contains this ordered set of command line instruction* needed to *assemble an image* ([Dockerfile reference](https://docs.docker.com/engine/reference/builder/)). The name of the Dockerfile really is just `Dockerfile` (without any file extension).

Example Dockerfile:

```docker
FROM node:12.16.3

WORKDIR /code

ENV PORT 80

COPY package.json /code/package.json

RUN npm install

COPY . /code

CMD ["node", "src/server.js"]
```

## CREDITS

- [https://laracasts.com/series/guest-spotlight/episodes/2](https://laracasts.com/series/guest-spotlight/episodes/2)
  
  > [Jose Soto](https://tighten.co/authors/jose-soto/) shows step by step how to maintain local database management systems on your machine, using Docker.

- [https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)

## Links

For more complete info, please see:

- [Official docker homepage](https://www.docker.com/) and [Docker hub](https://hub.docker.com/)

- [(Official) docker images](https://hub.docker.com/search?q=&type=image)

- [Online docker documentation](https://docs.docker.com/)

- [How to remove local docker images](https://linuxhandbook.com/remove-docker-images/)

## License

Published under the [MIT Licence](LICENSE.md)