# How To Deploy a Flask App on Docker Containers

Containers are built on a lightweight virtualization technology that allows an application, along with its dependencies and configuration to run in complete isolation, but without the need to use a full blown virtualization solution such as virtual machines, which need a lot more resources and can sometimes have a significant performance degradation in comparison to the host. A system configured as a container host can execute many containers, all of them sharing the host's kernel and direct access to the host's hardware. This is in contrast to virtual machines, which have to emulate a complete system, including CPU, disk, other hardware, kernel, etc.

![Docker vs VMs](images/docker_vs_vms.png)

In spite of having to share the kernel, the level of isolation in a container is pretty high. A container has its own file system, and can be based on an operating system that is different than the one used by the container host. For example, you can run containers based on Ubuntu Linux on a Fedora host, or vice versa. While containers are a technology that is native to the Linux operating system, thanks to virtualization it is also possible to run Linux containers on Windows and Mac OS X hosts. This allows you to test your deployments on your development system, and also incorporate containers in your development workflow if you wish to do so.

While Docker isn't the only container platform, it is by far the most popular. There are two editions of Docker, a free community edition (CE) and a subscription based enterprise edition (EE). In this tutorial, Docker CE will be sufficient.

To work with Docker CE, you first have to install it on your system. There are installers for Windows, Mac OS X and several Linux distributions available at the [Docker website](https://www.docker.com/community-edition). If you are working on a Microsoft Windows system, it is important to note that Docker CE requires Hyper-V. The installer will enable this for you if necessary, but keep in mind that enabling Hyper-V prevents other virtualization technologies such as VirtualBox from working. 

### Install Docker CE
Learn how to install `docker` [here](how_to_install_docker.md).

## Build a Container Image

The first step in creating a container for our app is to _build an **image**_ for it. A container image is a template that is used to create a container. It contains a complete representation of the container file system, along with various settings pertaining to networking, start up options, etc.

I will show you how to generate a container through a script. 

The command that creates scripted container images is `docker build`. This command reads and executes build instructions from a file called `Dockerfile`, which I will need to create. The Dockerfile is basically an installer script of sorts that executes the installation steps to get the application deployed, plus some container specific settings.

Dockerfile: Script installer
```python
FROM python:3.8-apline

RUN adduser -D practice_blog

WORKDIR /home/python/flask_tutorial/practice_blog

COPY requirements.txt requirements.txt
RUN python3 -m venv practice_blog
# Upgrade pip below
RUN practice_blog/bin/python3 -m pip install --upgrade pip
RUN practice_blog/bin/pip3 install requirements.txt
RUN practice_blog/bin/pip3 gunicorn

COPY app app
COPY migrations migrations
COPY blog.py config.py boot.sh ./
RUN chmod +x boot.sh

ENV FLASK_APP blog.py

RUN chown -R practice_blog:practice_blog ./
USER practice_blog

EXPOSE 5000
ENTRYPOINT ["./boot.sh"]
```
Each line in the `Dockerfile` is a command. The `FROM` command specifies the base container image on which the new image will be built. The idea is that you start from an existing image, and or change some things, and you end up with a derived image. Images are referenced by a _name_ and a _tag_, separated by a _colon_. The tag is used as a versioning mechanism, allowing an image to provide more than one variant. Above, the image used is `python`, which is the official Docker image for Python. The tags for this image allow you to specifiy the interprter versioin and the base operating system. The `3.8-alpine` tag selects a Python 3.8 interpreter installed on Alpine Linux. The Alpine Linux distribution is often used instead of the more popular ones such as Ubuntu because of its small size. You can see what tags are available for the Python image in the [Python Image Reposistory](https://hub.docker.com/_/python?tab=tags).

The `RUN` command executes an arbitrary command in the context of the container. This would be similar to you typing the command in a shell prompt. The `adduser -D practice_blog` command creates a new user named `practice_blog`. Most container images have `root` as the default user, but it is not a good practice to run an application as `root`, so we need to create our own user.

The `WORKDIR` command sets a default directory where the application is going to be installed. When I created the `practice_blog` user above, a home directory was created, so now I'm we are making that directory the default. The new default directory is going to apply to any remaining commands in the `Dockerfile`, and also later when the container is executed.

The `COPY` command transfers files from your machine the container file system. This command takes two or more arguments, the _source_ and the _destination_ files or directories. The source files must be relative to the directory where the Dockerfile is located. The destination can be an absolute path, or a path relative to the directory set in a previous `WORKDIR` command. Above, we are copying the `requirements.txt` file to the `practice_blog` user's home directory path in the container file system.

With `requirements.txt` in the container file system, we can create a virtual environment using `RUN` command. Above, I have created a virtual environment called `practice_blog` in which I install all the requirements needed for our application. It is also possible to explictly install packages in your container. In our example, we are installing `gunicorn` which we are going to use as a web server. Altenatively, you can have `gunicorn` added to the `requirements.txt` file.

The `COPY` commands that follow install the application to the container, by copying the `app` package, the `migration` repository with the database migrations and the top-level files `blog.py` and `config.py`. Additionally, We are also copying a new file `boot.sh`.

The `RUN chmod` command ensures that this `boot.sh`  file is set as an executable file. If you are in a Unix based file system and your source file is already marked as executable, then the copied file will also have the executable bit set. I added an explicit set because on Windows it is harder to set executable bits. If you are working on Mac OS X or Linux you probably don't need this statement, but it does not hurt to have it anyway.

The `ENV` command sets an environment variable inside the container. We need to set `FLASK_APP` which is required to use the `flask` command.

The `RUN chown` command sets the owner of all the directories and files that were stored in _/home/python/flask_tutorial/practice_blog_ as the new `practice_blog` user. Even though we created the this user near the top level of the Dockerfile, the default user for all the commands remain the `root` user. So, all the files need to be switched to the `practice_blog` user so that this user can work with them when the container is started.

The `USER` command in the next line makes the new `practice_blog` user the default for any subsequent instructions, and also for when the container is started.

The `EXPOSE` command configures the port that the container will be using for its server. This is necessary so that Docker can configure the network in the container appropriately. We have chosen the standard flask port `5000`, but this can be any port.

The `ENTRYPOINT` command defines the default command that should be executed when the container is started. This is the command that will start the web server. To keep things well organized, we will now make a separate `boot.sh` script file:

boot.sh: Docker container start up script
```python
#!/bin/sh
source practice_blog/bin/activate
flask db upgrade
flask translate compile
exec gunicorn -b :5000 --access-logfile - --error-logfile - practice_blog:app
```
Above, we activate the virtual environment, upgrade the database through the database migration framework, compile the language translations, and finally run the server with `gunicorn`.

In a shell script, `exec` triggers the process running the script to be replaced with the command given, instead of starting it as a new process. This is important because Docker associates the life of the container to the first process that runs on it. In cases like this one where the start up process is not the main process of the container, we need to make sure that the main process takes the place of that first process to ensure that the container is not terminated by Docker.

In Docker, anything that container writes to `stdout` or `stderr` will be captured and stored in the logs for the container. For that reason, the `--access-logfile` and `--error-logfile` are both configured with a `-`, which sends the log to the standard output so that they are stored as logs by Docker.

With the Dockerfile created, now we can build a container image:
```python
$ docker build -t practice_blog:latest .
```

The `-t` argument in `docker build` command sets the name and tag for the new image. The `.` indicates the base directory where the container is to be built. This is the directory where the _Dockerfile_ is located. The build process is going to evaluate all the commands in the Dockerfile and create the image, which will be stored on our own machine.

To obtain a list of all the images that you have locally, run the `docker images` command:

```python
$ docker images

# Output
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
practice_blog       latest              39fa5cda1776        10 seconds ago      206MB
python              3.8-alpine          8744555ae7bb        6 days ago          42.3MB
```
The listing will include your image as well as the base image on which it was built. **Any time you make changes to the application, you can update the container image by running the command again**.

## Start a Container

With the image already created, we can run the container version of the application using `docker run` command, which usually takes a large number of arguments.

```python
$ docker run --name practice_blog -d -p 8000:5000 --rm practice_blog:latest

# Output
581ac718128eab9b58fe883ce114dba44f74bf2d937d8b5de45dec2f7567531e
```

The `--name` option provides a name for the new container. The `-d` option tells Docker to run the container in the background without which the container runs as a foregroung application, blocking your command prompt. The `-p` option maps container ports to the host ports. The first port is the port on the host computer, and the one on the right is the port inside the container. The above example exposes the `5000` port in the container on port `8000` in the host, so you will access the application on `8000`, even though internally the container is using port `5000`. The `--rm` option will delete the container once it is terminated. While this is not required, containers that finish or are interrupted (I mean successful or not) are usually not needed anymore, so they can be automatically deleted. The last argument is the name and tag of our container image.

After you run the command above, you can access the application on http://localhost:8000.

To see what container is running, use `docker ps`:

```python
$ docker ps

# Output
CONTAINER ID        IMAGE                  COMMAND             CREATED             STATUS              PORTS                    NAMES
08b68c8afe5f        practice_blog:latest   "./boot.sh"         23 seconds ago      Up 22 seconds       0.0.0.0:8000->5000/tcp   practice_blog
```
You can use a more useful variant such as `docker ps -a` instead of `docker ps` if you do not see any output. Note that the CONTAINER ID is shortened compared to what was generated by `docker run` command. 

If you want to stop a running container, use `docker stop <CONTAINER ID>` command:

```python
$ sudo docker stop 08b68c8afe5f

# Output
08b68c8afe5f
```
##### Deleting Images and Containers

To delete a container, use `docker rm <CONTAINER ID>`:

```python
$ docker rm 08b68c8afe5f
```

Additionally, if you have several containers as seen in your `docker ps -a` list, and would like to delete all of them at once rather than having to run`docker rm <CONTAINER ID>` for each one of them, run:

```python
$ docker rm $(docker ps -a -a -f status=exited)
```

You can know the status of your container by checking the column `STATUS` when you run `docker ps`:

```python
$ docker ps

# Output
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

To delete an existing container image, check the image's REPOSITORY and IMAGE ID using `docker images` before running `docker rmi` command:

```python
$ docker rmi <REPOSITORY> <IMAGE ID>
```
Multiple images can be deleted as follows:

```python
$ docker rmi $(docker images -a -q)
```
##### Docker Cleaning Tools

As you work with Docker, it’s also easy to accumulate an excessive number of unused images, containers, and data volumes that clutter the output and consume disk space. Docker gives you all the tools you need to clean up your system from the command line. 

###### Purge all Unused or Dangling Images, Containers, Volumes, and Networks
Images, containers and other volume data not successfully built or created to completion are defined as _dangling_. You can use a single command to lean up any resources — images, containers, volumes, and networks — that are dangling (not associated with a container):

```python
$ docker system prune
```
To additionally remove any stopped containers and all unused images (not just dangling images), add the -a flag to the command:
```python
$ docker system prune -a
```

### Realistic Web Application

An application normally has a number of configurations that are sourced from environment variables. For example, the Flask secret key, database URL and email server options are all imported from environment variables.

We need to set these envrionment variables inside the container. The `ENV` command was used earlier in the `Dockerfile` to set environment variables, and it is handy for variables that are going to be static. For variables that depend on installation, however, it is not convinient to have them as part of the build process, because you want to have a container image that is _portable_. If you want to give your application to another person as container image, you want that person to be able to use it as is, and not have to build it with different variables.

So, build time environment variables can be useful, but there is also a need to have run-time environment variables that can be set via the `docker run` command, and for these variables, `-e` option can be used.

Setting email configuration in Docker container
```python
$ docker run --name practice_blog -d -p 8000:5000 --rm -e SECRET_KEY=my-secret-key \
    -e MAIL_SERVER=smtp.gmail.com -e MAIL_PORT=587 -e MAIL_USE_TLS=true \
    -e MAIL_USERNAME=<your-gmail-username> -e MAIL_PASSWORD=<your-gmail-password> \
    microblog:latest
```

### Using Third-Party "Containerized" Services

Assuming you have SQLite as your database of choise, what we know is that the application saves all its data on a file on disk on the local machine. What do you think is going to happen when the to the SQLite file when you stop and delete the container? The file is going to disappear!

The file system in a container is _ephemeral_, meaning that it goes away when the container goes away. You can write data to the file system, and the data is going to be there if the container needs to read it, but if for any reason you need to recycle your container and replace it with a new one any data that the application saved to disk is going to be lost forever.

A good design is to make the container application _stateless_. If you have a container that has application code and no data, you can throw it away and replace it with a new one without any problem, the container becomes truly disposable, which is great in terms of simplifying the deployment of upgrades.

But this means that the data must be put somewhere outside of the application container. This is where the Docker Registry comes into play.

> The Registry is a stateless, highly scalable server side application that stores and lets you distribute Docker images. By putting images in a registry, you can store static and immutable application bits, including all of their dependencies, at a framework level. You then can version and deploy images in multiple environments and thus provide a consistent deployment unit.

The Docker Container Registry contains a large variety of container images, for example the `python` container image discussed above. Docker maintains images for many other languages, databases and other services in the Docker registry. Companies and regular users alike can publish our own container images.

### Adding a MySQL Container

What you will do:
* Get `MySQL` image from the MySQL public registry
* Update the _Dockerfile_ to add a `MySQL` client package (`pymysql`)
* Link server to application

MySQL has public container images available on the Docker registry. MySQL relies on environment variables that need to be passed to `docker run`. These configure passwords, database names etc. We will use the officially maintained image by mySQL team [here](https://hub.docker.com/r/mysql/mysql-server/).

Start MySQL server

```python
$ docker run --name mysql -d -e MYSQL_ROOT_PASSWORD=yes -e MYSQL_DATABASE=practice_blog \
 -e MYSQL_USER=practice_blog -e MYSQL_PASSWORD=<database-password> mysql/mysql-server:5.7

# Output
Unable to find image 'mysql/mysql-server:5.7' locally
5.7: Pulling from mysql/mysql-server
03e20c09154c: Pull complete 
989c25a93b15: Pull complete 
1fe2b817a6cb: Pull complete 
0807146aa37b: Pull complete 
Digest: sha256:96f7f199868eaaf9dd9c3cff47021831f5525047b41b0c6a8bf1187936a3e9d2
Status: Downloaded newer image for mysql/mysql-server:5.7
48d838a161eb84ba32f62a46448ecbe741786a9860fe39e58d4926a1ca33014c

 # Use of \ allows for a rather long command to go to a new line. Do not include it in your command
```
On a Docker installed machine, you will get a fully installed MySQL server with a randomly generated root password, a brand new database called `practice_blog`, and a user with the same name that is configured with full permissions to access the database. Enter a proper password as value for the `MySQL_PASSWORD` environment variable. 

>`mysql` in `mysql/mysql-server:5.7` is the _docker registry account used by MySQL_. `mysql-server:5.7` is the image from MySQL.

You should have the MySQL container running. Check:
```python
$ docker ps

# Output
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS                   PORTS                    NAMES
48d838a161eb        mysql/mysql-server:5.7   "/entrypoint.sh mysq…"   7 minutes ago       Up 6 minutes (healthy)   3306/tcp, 33060/tcp      mysql
8b748206500d        practice_blog:latest     "./boot.sh"              2 hours ago         Up 2 hours               0.0.0.0:8000->5000/tcp   practice_blog
```

With the MySQL server started, we need to add a MySQL client package. We will use `pymysql` and add it to the _Dockerfile_.

Dockerfile: Add MySQL client
```python
# previous commands
RUN practice+_blog/bin/pip3 install gunicorn pymysql
# previous commands
```
With this change to our Dockerfile (or the application itself), the container image needs to be rebuilt:

```python
$ docker build -t practice_blog:latest .
```
We will need to start our `practice_blog` again, this time linking it to the database container so that both can communicate through the network:

```python
$ docker run --name practice_blog -d -p 8000:5000 --rm -e SECRET_KEY=my-secret-key \
    -e MAIL_SERVER=smtp.gmail.com -e MAIL_PORT=587 -e MAIL_USE_TLS=true \
    -e MAIL_USERNAME=<your-gmail-username> -e MAIL_PASSWORD=<your-gmail-password> \
    --link mysql:dbserver \
    -e DATABASE_URL=mysql+pymysql://practice_blog:<database-password>@dbserver/practice_blog \
    practice_blog:latest
```
The `--link` option tells Docker to make another container accessible to this one. The argument contains two names separated by a colon. The first part is the name or ID of the container to link. The second part defines the hostname that can be used in this container to refer to the linked one. `dbserver` if a generic name that represents the database server.

With the link to the two containers established, we can set the `DATABASE_URL` environment variable so that SQLAlchemy is directed to use the MySQL database in the other container. The database URL is going to use `dbserver` as the hostname, `practice_blog` as the database name and user, and the password that you selected when you started MySQL.

Sometimes the MySQL container takes a few seconds to be fully running and ready to accept database connects. If you start MySQL container  and then start the application container immediately after, when the `boot.sh` script tries to run `flask db upgrade` it may fail due to the database not being ready to accept connects.

boot.sh: Retry database connection
```python
#!/bin/sh
source practice_blog/bin/activate
while true; do
    flask db upgrade
    if [[ "$?" == "0" ]]; then
        break
    fi
    echo Upgrade command failed, retrying in 5 secs...
    sleep 5
done
flask translate compile
exec gunicorn -b :5000 --access-logfile - --error-logfile - practice_blog:app
```

The loop checks the exit code of the `flask db upgrade` command, and if it is non-zero it assumes that something went wrong, so it waits five seconds and then retries.