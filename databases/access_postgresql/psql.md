# Psql

## Overview

**Psql** is a terminal-based front-end for PostgreSQL. It allows you to issue queries interactively. Once you have connected to the PostgreSQL server, you can begin querying immediately. Besides basic querries, you can also issue certain commands such as `\d` to list all tables in the database, `\c dbname` to connect to another database and `\q` to quit from the postgres shell. You can access a text editor inside `psql` using `\e`. Run `\?` to see what other commands are available for use.

There are the two ways you can connect to PostreSQL:

1. [Using `psql`](databases/access_postgresql/psql.md) (this article)
2. [Using SQL client software application and database administration tool](databases/access_postgresql/dbeaver.md)


## Connecting to the PostgreSQL Server Using `psql`

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

## Create Another User

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

## Create a Super User

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

## Create User Password

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

## Change User Password

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
