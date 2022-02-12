# How to install Docker in Ubuntu 20.04

[Docker](https://www.docker.com/) is an application that simplifies the process of managing application processes in _containers_. Containers let you run your applications in resource-isolated processes. They’re similar to virtual machines, but containers are more portable, more resource-friendly, and more dependent on the host operating system.

In other words, containers are built on a lightweight virtualization technology that allows an application, along with its dependencies and configuration to run in complete isolation, but without the need to use a full blown virtualization solution such as virtual machines, which need a lot more resources and can sometimes have a significant performance degradation in comparison to the host. 

A system configured as a container host can execute many containers, all of them sharing the host's kernel and direct access to the host's hardware. This is in contrast to virtual machines, which have to emulate a complete system, including CPU, disk, other hardware, kernel, etc.

## Installing Docker CE

There are two editions of Docker, a free community edition (CE) and a subscription based enterprise edition (EE). I will show you how to install the Docker CE.

The Docker installation package available in the official Ubuntu repository may not be the latest version. To ensure we get the latest version, we’ll install Docker from the official Docker repository. To do that, we'll:

* add a new package source
* add the GPG key from Docker to ensure the downloads are valid
* install the package

Update your existing list of packages:

```python
$ sudo apt update
```

Install a few prerequisite packages which let `apt` use packages over `HTTPS`:

```python
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

Add the GPS key for the official Docker repository to your system:

```python
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add Docker repository to APT soureces:

```python
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

Update the package database with the Docker packages from the newly added repo:

```python
$ sudo apt update
```

Make sure to install from the Docker repo instead of the the default Ubuntu repo:

```python
$ apt-cache policy docker-ce

# Output
docker-ce:
  Installed: (none)
  Candidate: 5:19.03.13~3-0~ubuntu-focal
  Version table:
     5:19.03.13~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.12~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.11~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.10~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.9~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
```
Notice that docker-ce is not installed, but the candidate for installation is from the Docker repository for Ubuntu 20.04 (focal).

## Install Docker Engine

Update the `apt` package index, and install the latest version of Docker Engine and `containerd`:

```python
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

## Configure Docker to start on boot

Most current Linux distributions (RHEL, CentOS, Fedora, Ubuntu 16.04 and higher) use `systemd` to manage which services start when the system boots. Ubuntu 14.10 and below use `upstart`.

Enable docker when the computer boots:

```python
$ sudo systemctl enable docker

# Output
Synchronizing state of docker.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable docker
```
To disable this behavior, use `disable` instead:

```python
$ sudo systemctl disable docker
```

Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it’s running:

```python
$ sudo systemctl status docker

# Output
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2020-10-17 04:40:01 EAT; 3min 47s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 366577 (dockerd)
      Tasks: 14
     Memory: 37.8M
     CGroup: /system.slice/docker.service
             └─366577 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Oct 17 04:40:00 harry dockerd[366577]: time="2020-10-17T04:40:00.940348825+03:00" level=warni>
Oct 17 04:40:00 harry dockerd[366577]: time="2020-10-17T04:40:00.940355380+03:00" level=warni>
Oct 17 04:40:00 harry dockerd[366577]: time="2020-10-17T04:40:00.940362377+03:00" level=warni>
Oct 17 04:40:00 harry dockerd[366577]: time="2020-10-17T04:40:00.940542982+03:00" level=info >
Oct 17 04:40:01 harry dockerd[366577]: time="2020-10-17T04:40:01.044364957+03:00" level=info >
Oct 17 04:40:01 harry dockerd[366577]: time="2020-10-17T04:40:01.078308607+03:00" level=info >
Oct 17 04:40:01 harry dockerd[366577]: time="2020-10-17T04:40:01.121180654+03:00" level=info >
Oct 17 04:40:01 harry dockerd[366577]: time="2020-10-17T04:40:01.121317249+03:00" level=info >
Oct 17 04:40:01 harry dockerd[366577]: time="2020-10-17T04:40:01.137864732+03:00" level=info >
Oct 17 04:40:01 harry systemd[1]: Started Docker Application Container Engine.
```

Verify that Docker Engine is installed correctly by running the hello-world image:

```python
$ sudo docker run hello-world

# Output
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

To check the Docker version installed, run:

```python
$ docker --version
```

Also, for more information on your installed Docker version, run:

```python
$ docker version # running as non-root user

# Output
Client: Docker Engine - Community
 Version:           19.03.13
 API version:       1.40
 Go version:        go1.13.15
 Git commit:        4484c46d9d
 Built:             Wed Sep 16 17:02:52 2020
 OS/Arch:           linux/amd64
 Experimental:      false
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.40/version: dial unix /var/run/docker.sock: connect: permission denied
```

What this means is that you do not have the persion to connect to the Docker daemon socket. Basically, you are not the `root` user with access to connect to the Docker daemon socket. Run `docker version` as root:

```python
$ sudo docker version

# Output
Client: Docker Engine - Community
 Version:           19.03.13
 API version:       1.40
 Go version:        go1.13.15
 Git commit:        4484c46d9d
 Built:             Wed Sep 16 17:02:52 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.13
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       4484c46d9d
  Built:            Wed Sep 16 17:01:20 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.3.7
  GitCommit:        8fba4e9a7d01810a393d5d25a3621dc101981175
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```
The Docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket is owned by the user `root` and other users can only access it using `sudo`. **The Docker daemon always runs as the `root` user**.

## Manage Docker as non-root user

If you don’t want to preface the `docker` command with `sudo`, create a Unix group called `docker` and add users to it. When the Docker daemon starts, it creates a Unix socket accessible by members of the docker group.

> The docker group grants privileges equivalent to the root user. For details on how this impacts security in your system, [read here](https://docs.docker.com/engine/security/#docker-daemon-attack-surface).

> To run Docker without root privileges, see [Run the Docker daemon as a non-root user (Rootless mode)](https://docs.docker.com/engine/security/rootless/). Rootless mode is currently available as an experimental feature.

To create a docker group and add your user, we will do 4 things:
1. Create the docker group:
```python
$ sudo groupadd docker
```
2. Add your user to the docker group.
```python
$ sudo usermod -aG docker $USER
```
3. Log out and log back in so that your group membership is re-evaluated. Run the following command to activate the changes to groups:
```python
$ newgrp docker 
```
4. Verify that you can run `docker` commands without `sudo`:
```python
$ docker run hello-world
```
This command downloads a test image and runs it in a container. When the container runs, it prints an informational message and exits.

If you initially ran Docker CLI commands using `sudo` before adding your user to the docker group, you may see the following error, which indicates that your `~/.docker/` directory was created with incorrect permissions due to the sudo commands

```python
WARNING: Error loading config file: /home/user/.docker/config.json -
stat /home/user/.docker/config.json: permission denied
```
To fix this problem, either remove the `~/.docker/` directory (it is recreated automatically, but any custom settings are lost), or change its ownership and permissions using the following commands:

```python
$ sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
$ sudo chmod g+rwx "$HOME/.docker" -R
```

## Continue ...

Now that you know how to install Docker, learn how you can dockerize your flask application and deploy it to the Docker Registry [here](deploy_to_docker.md).