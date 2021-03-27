# How to Install and Configure Git in Ubuntu

It is best to install `git` from the Ubuntu Software repository. This tutorial is based on Ubuntu 20.04. Run the following commands in your terminal following the order below:
<br>

#### Installation

###### Procedure 1:

```python
$ sudo apt update
$ sudo apt upgrade
```
It is important to always update and upgrage your [APT](https://guide.ubuntu-fr.org/server/apt.html#:~:text=The%20apt%20command%20is%20a,upgrading%20the%20entire%20Ubuntu%20system.). Since your are running as root, you will be asked to enter your password. Do so and the installation will continue.


###### Procedure 2:
```python
$ sudo apt install git
```
This downloads and installs `git` on your system.

###### Procedure 3:
```python
$ git --version
```
Check what version of git has been installed in your system.
<br>
#### Configuration

We will use the terminal to configure `git`. You will need to provide your _username_ and _email address_.


###### Procedure 1:
```python
$ git config --global user.username "<put here your username>"
```
This will set your global username

###### Procedure 2:
```python
$ git config --global user.email "<put here your email@example.com>"
```

###### Procedure 3:
```python
$ git config --list
```
This step confirms all the changes you just made in the `git` configuration file. If at any point you want to edit your changes, run `$ git config`

**Read more on `git` in their [documentation](https://git-scm.com/)**