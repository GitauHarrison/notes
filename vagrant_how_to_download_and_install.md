# Vagrant: How to Download and Install the Right Way

## Background

One of the reasons why you would consider Vagrant is because you seek to deploy your application so that other users can have access to it. It is a traditional method of deployment in the sense that it you need to install your application manually or through a scripted installer on a stock server machine.

There are other servers that can be used for deployment. Vagrant, unlike the rest, is free. 

## Requirements

To have vagrant running on your machine, you will need two tools:
* VirtualBox and 
* Vagrant (itself)

You will also need:
* Root previlidges (sudo) and
* A terminal window



## Creating an Ubuntu Server

If you are not sure whether you have vagrant or not, run this command on your terminal:
```python
$ vagrant --version
```

### Installing a `VirtualBox`

`VirtualBox` is a software utility that allows you to run an operating system inside an operating system. It does this by creating a virtual environment. Since Vagrant creates virtual operating systems, it needs a tool like `VirtualBox` to manage the virtual operating systems.

To install `VirtualBox`, run:
```python
$ sudo apt install virtualbox
```

To check that you actually have `VirtualBox` installed, run:
```python
$ which virtualbox
```

Successful installation will show you a PATH to `VirtualBox`.


## Installing Vagrant

There are two ways you can follow to install `vagrant`:

### Option 1:
```python
$ sudo apt install vagrant
```
Your system will download and install the software. However, the challenge with this is that there is a likelihood your system is not up to date with the latest version of the software.

To check for the latest update, visit the official [Vagrant Download Page here](https://www.vagrantup.com/downloads.html) and download the appropriate one for your OS.

To check the version installed in your system, run:
```python
$ vagrant --version
```

### Option 2:

From the official HashiCorp Download Page, you can find the latest version of `vagrant`. Before you download the latest version of `vagrant`, it is good practice to look for any updates you might have. Then download your version by running these two commands:
```python
$ sudo apt update
$ curl -O https://releases.hashicorp.com/vagrant/2.2.x/vagrant_2.2.x_x86_64.deb
```
_Note: Change the 2.2.x to the appropriate version you are downloading_

We are using `curl` to get the file and download it. For more information on `curl`, visit:
* https://ec.haxx.se/. 

Further reading on:
* https://linuxize.com/post/curl-command-examples/.

To install `vagrant`, you need to run:
```python
$ sudo apt install ./vagrant_2.2.X_x86_64.deb
```
## Verify Vagrant Installation

To verify that the installation was successful, run:
```python
$ vagrant --version
```
You should be able to see the exact version you downloaded and installed.



## Getting started with Vagrant

Since you are going to use `vagrant` to run your project locally, the first thing you want to do is to create a directory that will host your application. To do this, you can:
```python
$ mkdir my_new_project
$ cd my_new_project
```

Now that you are inside your new project, you need to initialize `vagrant`. You can accomplish this in two ways:

First, run:
```python
$ vagrant init ubuntu/bionic64
```
This will create an _Ubuntu disk image_ within your current working directory.

Alternatively, 

You can choose to create a `vagrant` configuration file before running Vagrant. This can be accomplished by creating an empty file:
```python
$ touch Vagrantfile
```
Using your favorite text editor, you can populate the file with your configurations. For example,
```python
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/bionic64"
    config.vm.network "private_network", ip: "192.168.33.10"
    config.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
    end
end
```
This file will be useful when you try to create and configure the virtual machine as specified in your config file.

Now that you have a configuration file, run:
```python
$ vagrant up
```
After completion, `vagrant` will mount the project directory at /vagrant in the virtual machine which allows you to work on your project's files on your host machine.

**NOTE: [What are vagrant boxes?](https://www.vagrantup.com/docs/boxes)**

You have seen me use:
```python3
$ vagrant init ubuntu/bionic64
```
What exactly is `<ubuntu/bionic64>`? 

Boxes are the package formats for `vagrant` environments. Factually, the list is endless on available boxes, as seen [here](https://app.vagrantup.com/boxes/search). You can search for a box of your choice. In this guide, I am using the disk image of Ubuntu 18.04 LTS. 

Here are the [official Ubuntu Cloud Images](https://cloud-images.ubuntu.com/) that have been customized to run on public clouds that provide Ubuntu Certified Images, Openstack and more.




## Using SSH Client

Thing is, you do not have an app on your desktop to connect to your server. You will need to work through the command-line/terminal  and connect to your server through an `SSH` client. 

If you are using a Vagrant machine, (which you are now) you can simply run:
```python
$ vagrant ssh
```
You can also choose to create a user to connect to your server. To do this, you will need to log into your server's root account using `ssh` and do the following:
```python
$ adduser --gecos "" ubuntu
$ usermod -aG sudo ubuntu
$ su ubuntu
```
This creates an _ubuntu_ user. What you should note is, at this point, if you ever `logout` of your server, you will need to repeat the above steps (summarized below) to get to your user:
```python
$ vagrant up
$ vagrant ssh
$ sudo su ubuntu
```
Alternatively, you can just run this command to get to your new user:
```python
$ vagrant ssh -c "/home/ubuntu; sudo su ubuntu"
```


## Secure your Server

To minimize the risk of your server being compromised, you need to tighten available loopholes an attacker might use to gain access. We will use the inbuilt `vim` editor (or another of your choice) to help us modify our `config` file as follows:
```python
$ sudo vim /etc/ssh/sshd_config
```
Make the following changes by editing the original file to:
```
PermitRootLogin no
PasswordAuthentication no
```

After you are done editing the `ssh` configuration, the service needs to be restarted of the changes to take effect:
```python
$ sudo service ssh restart
```
#### Install a firewall

Block access to the server on any ports that are not explictly enabled:
```python
$ sudo apt-get install -y ufw
$ sudo ufw allow ssh
$ sudo ufw allow http
$ sudo ufw allow 443/tcp
$ sudo ufw --force enable
$ sudo ufw status
```
