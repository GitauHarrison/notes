You will learn how to use Python and the Flask web Framework to build a personal blog. For this reason, you need to ensure you have Python installed.

### Install Python

How do you know whether you have Python installed in your computer? Open your terminal and type

```python
$ which python3
```
If you do not get any output displayed on your terminal, that means that you do not have Python installed in your system. You need to install it. Visit the [Python Official Site](https://www.python.org/downloads/) and install the latest release according to your system. The three major systems include Windows, Linux and Mac OS X.

However, if you get something like:

```python
$ which python3

# Output
/usr/bin/python3
```
This means you have Python3 in your system. To make sure your Python installation is functional, run `python3` in your terminal as below:

```python
$ python3

# Output
Python 3.8.5 (default, Jul 28 2020, 12:59:40) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```
The Python interpreter is in an interactive prompt where you can type in Python statements. But for now, we won't type in anything. We wanted to make sure that the interpreter is working well. 

Type in `exit()` and press `Enter` to exit the interpreter. Alternatively, press `ctrl + Z`.

I recommend that you have Ubuntu as your main Operating System as you start this Web Development journey. Ubuntu, a distribution of Linux, is optimized for development and it takes a creator approach rather having you as a consumer of the Operating System. It comes preinstalled with Python. That means you do not need to install it again.

>To install Ubuntu, you will need to create an Ubuntu Bootable USB stick. Learn how to create a bootable USB stick [on Windows](https://ubuntu.com/tutorials/create-a-usb-stick-on-windows#1-overview) or [on Ubuntu](https://ubuntu.com/tutorials/create-a-usb-stick-on-ubuntu#1-overview). With this Bootable USB stick, you can:
>* Install or upgrade Ubuntu
>* Test out the Ubuntu desktop experience without touching your PC configuration
>* Boot into Ubuntu on a borrowed machine or from an internet cafe
>* Use tools installed by default on the USB stick to repair or fix a broken configuration


### Install Flask

Python3 comes with a tool called `pip3` which does all the installation of Python packages. Python has an official and public Python Package repository called [PyPI](https://pypi.org/)(Python Package Index) from where anybody can install Python packages.

To install a package on your machine, all you need to do is:

```python
$ pip3 install <package-name>
```

Note that if your Python interpreter is installed globally in your machine for all users, chances are your regular user account will not have the permission to make modifications to it. The only way to overcome this is to run the installation as an administrator using the `sudo` command. From there onwards, the installed packege will be available to everyone.

Whenever you are working on a project, chances are your project will have multiple versions. Think about this, you have just finished a project using `flask`'s version 0.1. By the time you want to start another project, another version of `flask`, say version 0.2 is released. Obviously, you will install version 0.2 of `flask` to keep your project current. 

This may go on and on, not just with `flask` but maybe with another package. What you will end up with is your computer will have mutliple versions of packages. To keep your computer system clean, it is recommended that you create something called _virtual environments_. A virtual environment is a complete copy of the Python interpreter. The envrionments help isolate your development needs from that of your machine, and, therefore, helps to keep your system very clean and uncluttered.

### Getting Started

Let us start by creating a directory that will host all our project files. I will call this directory _personal_blog_.

```python
$ mkdir personal_blog # mkdir (short for make directory) command is used to create an empty directory
```

With an empty project directory created, we need to move into it:

```python
$ cd personal_blog # cd (change directory)command is used to change one directory from another
```

#### Creating a Virtual Environment

There are two ways you can use to create a virtual environment:

1. Manual way
2. Using virtualenv wrapper

##### Manual way

```python
$ python3 -m venv venv
```

Here, we are asking Python to run the `venv` package which creates virtual environment called `venv`. 

![Python Virtual Environment](/images/python_virtual_env.png)

After the command completes, you are going to have a directory called _venv_ where all your virtual environment will be saved.

With your virtual environment created, you have to tell your system that you want to use it. This is done by _activating_ it. To activate your new virtual environment, run:

```python
$ source venv/bin/activate

# your terminal will change to something like:
(venv) $ ...
```

##### Using Virtual Environment Wrappers

[`virtualenvwrapper`](https://virtualenvwrapper.readthedocs.io/en/latest/) is a set of extensions to `virtualenv` tool. The extensions include wrappers for creating and deleting virtual environments and otherwise managing our development workflow, making it easier to work on more than one project at a time without introducing conflicts in their dependencies.

I personally prefer using `virtualenvwrapper` due to how easy it is to use. During creation of virtual environments, it not only creates it but it also activates it automatically. 

For example, to create a virtual environment called _venv_, all I need to do is run the `mkvirtualenv` command to create and activate the new environment:

```python
$ mkvirtualenv venv

# Seen on the terminal:
(venv) $ ...
```

If you would like to use `virtualenvwrapper`, learn how to install and use it [here](/virtualenvwrapper_setup.md).

#### Install Flask

We will use `pip3` to install `flask`:

```python
$ pip3 install flask
```

Confirm that your virtual environment has `flask` installed by running your Python interpreter and importing it:

```python
$ python3 # run your Python interpreter
>>> import flask
>>> 
```

If you do not get any error, it means your flask extension was installed well. You can also check the version of `flask` installed. Run:

```python
$ flask --version # ensure you exited the Python interpreter above by pressing ctrl + Z
```

