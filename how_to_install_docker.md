# How to install Docker in Ubuntu 20.04

[Docker](https://www.docker.com/) is an application that simplifies the process of managing application processes in _containers_. Containers let you run your applications in resource-isolated processes. They’re similar to virtual machines, but containers are more portable, more resource-friendly, and more dependent on the host operating system.

In other words, containers are built on a lightweight virtualization technology that allows an application, along with its dependencies and configuration to run in complete isolation, but without the need to use a full blown virtualization solution such as virtual machines, which need a lot more resources and can sometimes have a significant performance degradation in comparison to the host. 

A system configured as a container host can execute many containers, all of them sharing the host's kernel and direct access to the host's hardware. This is in contrast to virtual machines, which have to emulate a complete system, including CPU, disk, other hardware, kernel, etc.

### Installing Docker CE

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

Now, install Docker:

```python
$ sudo apt install docker-ce
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