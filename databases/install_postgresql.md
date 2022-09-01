# Install PostgreSQL

Now that you have a brief understanding of what a database is, you know the history of PostgreSQL, and are familiar with some of the advantages of using PostgreSQL, you are ready to start using it. We will begin by learning how to install it in our operating system. We will also be using the DBeaver client software to help us get off the ground quickly.

For reference, these are the topics we will cover in this tutorial:

- [PostgreSQL Overview](postgresql.md)
- [Install PostgreSQL and DBeaver](install_postgresql_and_dbeaver.md) (this article)
- [Getting Started with PostgreSQL](getting_started_with_postgresql.md)

### Table of Contents

In this article, we will focus on installing PostgreSQL, creating a connection to it and optionally know how to uninstall it.

1. [Install PostgreSQL in Linux Using the Terminal](#install-postgresql-in-linux-using-the-terminal)
2. [Create Connection to PostgreSQL](#create-connection-to-postgresql)
3. [Uninstall PostgreSQL in Linux Using the Terminal](#uninstall-postgresql-in-linux-using-the-terminal)


## Install PostgreSQL in Linux Using the Terminal

Most Linux distributions such as Debian and Ubuntu have PostgreSQL integrated with their package environments by default. However, Ubuntu "snapshots" a specific version of PostgreSQL that is then supported throughout the lifetime of that Ubuntu version. Other versions of PostgreSQL are available through the PostgreSQL apt repository. It is recommended that you install PostgreSQL this way since it ensures a proper integration with the operating system, including automatic patching and other management functionality.

If you try to search for an available version of PostgreSQL after a fresh install of Ubuntu by running `psql -V`, you will notice that the command `psql` will not be found. You will be told that it can be installed by running:

```python
sudo apt install postgresql-client-common
```

![No postresql](/images/databases/postgresql/no_postgresql.png)

When installed, running `psql -V` would show this outcome:

```python
psql (PostgreSQL) 14.4 (Ubuntu 14.4-1.pgdg20.04+1)
```

Install and build from source by visiting the [Linux downloads (Ubuntu)](https://www.postgresql.org/download/linux/ubuntu/) for more information.


## Create Connection to PostgreSQL

User privilege and privilege access management (PAM) is a serious security concept when working with databases. What privilege access does is to empower organizations to reduce the threat that may come due to data breaches. For example, a business might have two employees with administrator access to a database. Only these users have the authority to work with the database and can delete or setup objects in a database.

From the [PostgreSQL Overview](postgresql.md) article, you may have noticed that one of the advantages of using PostgreSQL is its ability to handle access control through roles and privileges. 

There are two ways we can access PostgreSQL. Check them using the links below:

1. [Using `psql`](/databases/access_postgresql/psql.md)
2. [Using SQL client software application and database administration tool](/databases/access_postgresql/dbeaver.md)

You can choose one over the other based on your personal preferrance, but it is good to know of both ways.


## Uninstall PostgreSQL in Linux Using the Terminal

For whatever reason, if you wish to uninstall PostgreSQL from your system, you can do the following:

```python
$ dpkg -l | grep postgres

# Output
ii  pgdg-keyring                                  2018.2                               all          keyring for apt.postgresql.org
ii  postgresql                                    14+242.pgdg20.04+1                   all          object-relational SQL database (supported version)
ii  postgresql-14                                 14.4-1.pgdg20.04+1                   amd64        The World's Most Advanced Open Source Relational Database
ii  postgresql-client-14                          14.4-1.pgdg20.04+1                   amd64        front-end programs for PostgreSQL 14
ii  postgresql-client-common                      241.pgdg20.04+1                      all          manager for multiple PostgreSQL client versions
ii  postgresql-common                             241.pgdg20.04+1                      all          PostgreSQL database-cluster manager
```

The command above will list all dependant packages of postgres (or postgresql). Then, you can uninstall these packages individually:

```python
$ sudo apt --purge remove package_name package_name
```

`purge` completely erases traces of the configuration files of a program from the system. It is particularly useful when you want to "start all over again" with an application because you messed up the configuration. See this discussion on [StackOverfolow](https://askubuntu.com/a/187891/789542).

Now that everything has been uninstalled, you can verify the uninstallation process by running:

```python
$ psql -V
```
