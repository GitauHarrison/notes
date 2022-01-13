# How to Install and Configure Git in Ubuntu

## Overview

> Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.

Every Git clone is a full-fledged repository with complete history and full revision tracking capabilities, not dependent on network access or a central server. Branching and merging are fast and easy to do.

Typically, Git is used with hosting services such as GitHub, GitLab, Bitbucket, among others. They all offer distributed version control services. Unlike Git, which is strictly a commandline tool, GitHub, and the likes, provides a web-based graphical interface for 'version control'. This allows for collaboration and sharing of projects among people or groups of people.

## Installation

Before you start using Git, you have to make it available on your computer. Even if it’s already installed, it’s probably a good idea to update to the latest version. You can either install it as a package or via another installer, or download the source code and compile it yourself.

In this tutorial, I will recommend installing `git` from the Ubuntu Software repository. This tutorial is based on Ubuntu 20.04. Run the following commands in your terminal following the order below:
<br>

### Step 1

```python
$ sudo apt update
$ sudo apt upgrade
```
It is important to always update and upgrage your [APT](https://guide.ubuntu-fr.org/server/apt.html#:~:text=The%20apt%20command%20is%20a,upgrading%20the%20entire%20Ubuntu%20system.). Since your are running as root, you will be asked to enter your password. Do so and the installation will continue.


### Step 2
```python
$ sudo apt install git
```
This downloads and installs `git` on your system.

### Step 3
```python
$ git --version
```
Check what version of git has been installed in your system.
<br>

## Configuration

We will use the terminal to configure `git`. You will need to provide your _username_ and _email address_.


### Step 1
```python
$ git config --global user.username "<put here your username>"
```
This will set your global username

### Step 2
```python
$ git config --global user.email "<put here your email@example.com>"
```

### Step 3
```python
$ git config --list
```
This step confirms all the changes you just made in the `git` configuration file. If at any point you want to edit your changes, run `$ git config`

_Read more on `git` in their [documentation](https://git-scm.com/)_