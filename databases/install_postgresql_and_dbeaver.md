# Install PostgreSQL and DBeaver

Now that you have a brief understanding of what a database is, you know the history of PostgreSQL, and are familiar with some of the advantages of using PostgreSQL, you are ready to start using it. We will begin by learning how to install it in our operating system. We will also be using the DBeaver client software to help us get off the ground quickly.

For reference, these are the topics we will cover in this tutorial:

- [PostgreSQL Overview](postgresql.md)
- [Install PostgreSQL and DBeaver](install_postgresql_and_dbeaver.md) (this article)
- [Getting Started with PostgreSQL](getting_started_with_postgresql.md)

### Table of Contents

1. [Install PostgreSQL in Linux Using the Terminal](#install-postgresql-in-linux-using-the-terminal)
2. [Working with PostgreSQL](#working-with-postgresql)
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


## Working with PostgreSQL

User privilege and privilege access management (PAM) is a serious security concept when working with databases. What privilege access does is to empower organizations to reduce the threat that may come due to data breaches. For example, a business might have two employees with administrator access to a database. Only these users have the authority to work with the delete or setup objects in a database.

From the [PostgreSQL Overview](postgresql.md) article, you may have noticed that one of the advantages of using PostgreSQL is its ability to handle access control through roles and privileges. 

There are two ways you can use PostgreSQL:

- Using `psql` - a terminal-based front-end for PostgreSQL. It allows you to issue queries interactively.
- Using SQL client software application and database administration tool such as DBeaver and pgAdmin.

### Connecting to the PostgreSQL Server

Now that you have installed postgreSQL, that first thing you want to do is to connect to its server. Every installation creates a default user called `postgres` who is associated with the default Postgres role. We will begin by connecting to the server using this user.

```python
$ psql
```

You may notice that running the command above produces an error:

```python
# Ouput

psql: error: connection to the server on socket "/var/run/posgresql..." failed: No such file or directory. Is the server running locally and accepting connections on the socket?
```

What this means is that our server is currently not running. To start it, we will run:

```python
$ sudo service postgresql restart

# Output
* Restarting PostgreSQL 14 database server
```

With the connection created, we can now access the `psql` command to access the interactive terminal for PostgreSQL. This command is normally used in conjunction with a user.

```python
$ sudo -u postgres psql
```

We are using the default `postgres` user to access PostgreSQL terminal. You will be asked to provide a password to continue since you are using `sudo` command. When you do so, your terminal will change to this:

```python
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=# 
```

### Create Another User

At this stage, we are ready to start issuing SQL commands. The first action we will take is to create another user. We will use SQL statements to accomplish this. If you are not familiar with SQL commands, do not worry. The subsequent chapters are dedicated to help you understand SQL.

```python
postgres=# CREATE USER muthoni;
CREATE ROLE
```

The CREATE USER query is an alias (which means "an assumed identity") of CREATE ROLE <name> WITH LOGIN; If you are curious how many users you have in the database, you can run this command:

```python
postgres=# \du

# Output
 Role name   |                         List of roles Attributes                         | Member of 
-------------+--------------------------------------------------------------------------+-----------
 muthoni     |                                                                          | {}
 postgres    |               Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

From a security stanpoint, it is very dangerous to have a user with superuser privileges because this user is able to bypass all checks. You are advised to avoid the superuser unless it is necessary or be VERY careful.

### Create a Super User

Let us create another user with superuser privileges:

```python
postgres=# CREATE USER wangare SUPERUSER;
CREATE ROLE

# Output
 Role name   |                         List of roles Attributes                         | Member of 
-------------+--------------------------------------------------------------------------+-----------
 muthoni     |                                                                          | {}
 postgres    |               Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 wangare     |               Superuser                                                  | {}
```

The query above is the same as `CREATE ROLE <name> LOGIN SUPERUSER;`.

Alternatively, you can simple create a superuser from the default Ubuntu terminal:

```python
$ sudo -u postgres createuser --superuser <name>
```

### Create User Password

With the uses in place, the next step would be to create passwords for them. This is to ensure that only them, and those privy to the credential, will be able to access the database. From `psql`, we can do this:

```python
postgres=# CREATE USER chico WITH PASSWORD 'hard-to-guess';
CREATE ROLE
```

If the user already exists, we can the ALTER command:

```python
postgres=# ALTER USER muthoni WITH PASSWORD 'difficult-password';
ALTER ROLE
```

### Change User Password

You may forget your user password, or you would simple want to modify it as a good security practice. Let us do this:

```python
$ sudo -u postgres psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=#\password muthoni
Enter new password for user "muthoni":
Enter it again:
postgres=#
```

Or you may do everything from the default Ubuntu terminal as follows:

```python
$ sudo - postgres psql -c "ALTER USER muthoni PASSWORD 'new-password';"
```

You can then restart your PostgreSQL server:

```python
$ sudo service postgresql restart
```


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
