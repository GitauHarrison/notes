# How to configure Python Environment to use Virtualenv Wrapper

Working on a python project in an isolated python environment is recommended so that python modules and packages don’t meddle with that of other projects or even that of the operating system. Thus, a virtual environment is needed eminently.

I will show you how to set up your python environment to use virtualenvironment wrapper in Ubuntu 20.04. In this simple tutorial, we will do four things:

* Install `pip3`
* Install `virtualenv`
* Install `virtualenvwrapper`
* Edit `.bashrc/` file or `.bash_profile` file, or both

##### Prerequisites

We will use `pip3`, the default python package installer for Python3. 

#### Setting Up a Virtual Environment

Open your terminal in your home directory (or any other directory you may choose).First, you need to create a special directory which will be hidden that will store all your virtual environments

Create a hidden virtual environments folder:

```python
mkdir .virtualenv # the dot(.) denotes that the file is hidden
```

Install `pip3`:

```python
$ sudo apt install python3-pip
```

Confirm installation of `pip3`:

```python
$ pip3 --version # checks for the version of pip3
$ which pip3 # shows you the location of pip3 in your system
```

Install `virtualenv` using `pip3`:

```python
$ pip3 install virtualenv # install virtualenv
$ which virtualenv # find where it is located
```

Now, it is time to install `virtualenvwrapper`:

```python
$ pip3 install virtualenvwrapper
```
We will modify our `.bashrc` file by adding a row that will adjust every new virtual environment to use Python3. We will need to point virtual environments to the directory we created above (.virtualenv)

Open the `.bashrc` file using your `vim` editor:

```python
$ vim .bashrc
```

If you notice that you do not have `vim` in your system, make sure you install it through `$ sudo apt install vim`

`.bashrc` file will open in your terminal. Something you should note about the `vim` editor is that it does not use the conventional editing commands such as `ctrl + C` etc. Rather, it uses its own special commands to allow you interact with it. Learn how to navigate the `vim` editor [here](https://www.linux.com/training-tutorials/vim-101-beginners-guide-vim/). 

For now, there are two basic commands you need to know:
* How to enter into `insert` mode so you can make edits
* How to save and quit

Now that your editor is open, we need to start making changes. Scroll to the bottom of the file by pressing the down arrow key. At the very bottom of your editor, press the letter _i_ to enter into _insert_ mode (_i_ could mean 'insert').

###### Insert Mode
Add these rows:

```python
#Virtualenvwrapper settings:
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_VIRTUALENV=/home/gitau/.local/bin/virtualenv
source ~/.local/bin/virtualenvwrapper.sh
```
Here, we have created variables to hold the path to our Python3 and virtual environment. 

Note that you should replace the line
```python
export VIRTUALENVWRAPPER_VIRTUALENV=/home/gitau/.local/bin/virtualenv
```
with your computer name:

```python
export VIRTUALENVWRAPPER_VIRTUALENV=/home/<your-computer-name>/.local/bin/virtualenv
```

###### Save and Quit

Type `:wq` and press `Enter` This will allow you to save your changes and quit the editor. Close your terminal and reopen it.

You have completed setting up your virtual environment. 

#### Create Your Virtual Environment

`virtualenvwrapper` allows you to create and activate your virtual environment at the same time. To create your virtual environment, use the command `mkvirtualenv`:

```python
$ mkvirtualenv <name-of-your-virtual-environment> # for example mkvirtualenv chatapp
```
Your terminal will change from:

```python
gitau@harry:~/software_development/python/flask_lesson_notes$
```
To something like:
```python
(chatapp) gitau@harry:~/software_development/python/flask_lesson_notes$
```

To deactivate your virtual environment, run:
```python
$ deactivate
```

#### Reactivate Your Virtual Environment
If you want to activate the virtual evironment you just created, run:

```python
$ workon chatapp
```

`workon` command allows you to activate an already created virtualenvironment. You can also use the same command to list all available virtual environments in your system by running:

```python
$ workon
```

#### Remove Your Virtual Environment

At some point, if you want to delete your virtual environment, simply use the `rmvirtualenv` command:

```python
$ rmvirtualenv chatapp
```

#### Copy Your Virtual Environment

To copy an existing virtualevironment, use `cpvirtualenv`:

```python
$ cpvirtualenv chatapp
```