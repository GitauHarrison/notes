# Getting Started With Django

The approach we shall use to learn Django is project-based. We shall create a sample application, and in the process learn the different aspects of using Django to build a robust web application.

For your reference, if you would like to skip to a particular section in the entire Django tutorial, you can do so by clicking on any of the links below:

- [Django Overview](00_overview.md)
- [Getting Started With Django](01_getting_started.md) (this article)
- [Applications and routes](02_applications_and_routes.md)
- [Templates in Django](03_templates.md)
- [Admin Page](04_admin_page.md)
- [Databases and Migrations](05_database_and_migrations.md) 


### Table of Contents

This article is broken down into the following subsections:

1. [Installation](#installation)
2. [Create Django Project](#create-django-project)
3. [Overview Of Django Project Files](#overview-of-django-project-files)
4. [Run The Application](#run-the-application)


## Installation

Before we can use Django, we need to install it. Keep in mind that Django requires Python, so if you do not have Python, head over to https://www.python.org/downloads/ to download one for your Operating System. I will be using Ubuntu throughout this tutorial, which comes with Python pre-installed.

### Prerequisites

- Python
- VS Code (or your preferred editor)

Confirm that you have Python by running the command below in your terminal:

```python
$ python3

# Output
Python 3.8.10 (default, Jun 22 2022, 20:18:18) 
[GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

My OS currently has Python3.8.10. In your terminal, I will be on VS Code's terminal, create a Django folder to get started by running:

```python
$ mkdir demo_project
```

![Create Project](/02_django/images/01_getting_started/django_project.png)

Navigate into the new folder:

```python
$ cd demo_project
```
![Change Directory](/02_django/images/01_getting_started/cd_demo_folder.png)

It is recommended to install Django in a virtual environment. Virtual environments are used to isolate the needs of one application from another, or from cluttering the OS. You can run the command below in your terminal:

```python
$ python3 -m venv venv
```

This creates a virtual environment called `venv` in `demo_project`. Thereafter, we will need to activate it.

```python
$ source venv/bin/activate
```

You will notice that your terminal changes to `(venv)demo_project$ `. Alternatively, we can use `virtualenvwrapper` to not only create but also to activate a virtual environment for us.

```python
$ mkvirtualenv venv
```

If you are not familiar with virtualenvwrapper, you can learn more [here](/virtualenvwrapper_setup.md).

Now, we are ready to install Django. Let us run this command in our active virtual environment:

```python
(venv)$ pip3 install django
```

![Install django](/02_django/images/01_getting_started/install_django2.png)

Verify that Django has been installed by running this command in your terminal:

```python
# Enter into the Python prompt
(venv)$ python3

# Output
Python 3.8.10 (default, Jun 22 2022, 20:18:18) 
[GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.

# Import django and get version
>>> import django
>>> print(django.get_version())
4.1.3
>>> 
```

I am currently using V4.1.3. An alternative way to know what version of Django you have installed is by running this one-liner command in the terminal:

```python
(venv)$ python3 -m django --version

# Output
4.1.3
```

## Create Django Project

Django has some commands we can use to create and manage our project. One of these commands is `django-admin`.

```python
(venv)$ django-admin
```

![Django admin command](/02_django/images/01_getting_started/djajngo_admin_command.png)

You can see that when we run the `django-admin` command we get a list of sub-commands. The one we are most interested in at the moment is the `startproject` command. It creates a complete project for us, with the proper structure and all the files we will need to get started. Let us use it below:

```python
(venv)$ django-admin startproject blog_app
```

This command creates a project called `blog_app` within our current directory. You can run the command `ls` in your terminal to see it. We will need to change the directory to the new project to see what structure it has.

```python
(venv)$ cd blog_app
```

This only navigates us to the new folder but does not show us the project's structure. Since we want to see this new project's structure, we can do either of the following:

- [Run the `tree` command](#run-the-tree-command)
- [Use VS Code's file explorer](#vs-code-file-explorer)

### Run the `tree` Command

In the terminal, I will run the command `tree` to see the project's structure.

```python
(venv)$ tree

# Output
.
├── blog_app
│   ├── asgi.py
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py

1 directory, 6 files
```

You may not have this command installed. To use it, you will need to install it first. Thankfully, there is some instruction on the terminal on how to do the installation. Follow it and then re-run the `tree` command.

Above, you can see that we are currently in our initial project directory, denoted by the `.`. Our actual project is called `blog_app` and it has a few more files in it. In the top-level directory, there is also the `manage.py` file which acts as the entry point to the application.

### VS Code File Explorer

We can visualize the project's structure using VS Code's file explorer. First, let us open a new VS Code window in our current `blog_app` subdirectory.

```python
(venv)blog_app $ code .
```

![Open vs code](/02_django/images/01_getting_started/open_vs_code.png)

You will see the full project structure such as the one below.

![Project structure](/02_django/images/01_getting_started/project_structure.png)


## Overview of Django Project Files

Our project structure looks like this

```python
├── blog_app
│   ├── asgi.py
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py
```

Our `blog_app` project is in a folder called `demo_project` if you can recall from the start of our tutorial. So, let us start there.

- **demo_project**: The container for our project. Its name is irrelevant to Django.
-  **blog_app**: The directory of the actual project. Its name shall be used to import anything inside of it (for example, `blog_app.urls`)
- **blog_app/asgi.py**: An entry-point for ASGI-compatible web servers to serve our project.
- **blog_app/\__init__.py**: An empty file that tells Python that this directory should be considered a Python package.
- **blog_app/settings.py**: Settings/configuration for this Django project.
- **blog_app/urls.py**: The URL declarations for this Django project.
-  **blog_app/wsgi.py**: An entry-point for WSGI-compatible web servers to serve our project.

In subsequent tutorials, we shall go into detail about the individual files so we have a better and more practical understanding of what each does.


## Run The Application

In Django, all we need to do to see our basic application is to run one single command, unlike Flask where we have to set up a few environment variables before firing up the server.

To start the Django server, run:

```python
(venv)$ python3 manage.py runserver
```

![Run server](/02_django/images/01_getting_started/run_server.png)

We do not have to worry about the warnings in the terminal at this point because we have not finished setting up the application. What we are interested in at the moment is the localhost URL. It is currently running on port 8000. Paste the link in a new tab on your favorite browser to see the application. 

![Django Live](/02_django/images/01_getting_started/django_live.gif)

To stop the server, you can press "Ctrl + C" in the terminal running the application.