# Psql

## Overview

**Psql** is a terminal-based front-end for PostgreSQL. It allows you to issue queries interactively. Once you have connected to the PostgreSQL server, you can begin querying immediately. Besides basic querries, you can also issue certain commands such as `\d` to list all tables in the database, `\c dbname` to connect to another database and `\q` to quit from the postgres shell. You can access a text editor inside `psql` using `\e`. Run `\?` to see what other commands are available for use.

There are two ways you can connect to PostgreSQL:

1. [Using `psql`](/databases/access_postgresql/psql.md) (this article)
2. [Using SQL client software application and database administration tool](/databases/access_postgresql/dbeaver.md)

For reference, these are the topics we will cover in the entire database tutorial:

- [PostgreSQL Overview](/databases/00_postgresql_overview.md)
- [Install PostgreSQL](/databases/01_install_postgresql.md)
- [Getting Started with PostgreSQL](/databases/03_getting_started_with_postgresql.md)


### Table of Contents

Throughout this article, we will look at how to:

1. [Connect to the PostgreSQL Server Using `psql`](#connect-to-the-postgresql-server-using-psql)
2. [Create Another User](#create-another-user)
3. [Create A Super User](#create-a-super-user)
4. [Create User Password](#create-user-password)
5. [Change User Password](#change-user-password)
6. [Connect to PostgreSQL As Another User](#connect-to-postgresql-as-another-user)

## Connect to the PostgreSQL Server Using `psql`

Now that you have installed postgreSQL, the first thing you want to do is to connect to its server. Every installation creates a default user called `postgres` who is associated with the default Postgres role. We will begin by connecting to the server using this user.

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

With the connection created, we can now use the `psql` command to access the interactive terminal for PostgreSQL. This command is normally used in conjunction with a user.

```python
$ sudo -u postgres psql
```

We are using the default `postgres` user to access PostgreSQL terminal. You will be asked to provide a password to continue since you are using the `sudo` command. When you do so, your terminal will change to this:

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

The `CREATE USER` query is an alias (which means "an assumed identity") of `CREATE ROLE <name> WITH LOGIN;`. 

If you are curious how many users you have in the database, you can run this command:

```python
postgres=# \du

# Output
 Role name   |                         List of roles Attributes                         | Member of 
-------------+--------------------------------------------------------------------------+-----------
 muthoni     |                                                                          | {}
 postgres    |               Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

## Create a Super User

From a security standpoint, it is very dangerous to have a user with superuser privileges because this user is able to bypass all checks. You are advised to avoid the superuser unless it is necessary or be VERY careful when working with this user. Let us create another user with superuser privileges:

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

Alternatively, you can quickly create a superuser from the default Ubuntu terminal:

```python
$ sudo -u postgres createuser --superuser <name>
```

## Create User Password

With the users in place, the next step would be to create passwords for them. This is to ensure that only them, and those privy to the credential, will be able to access the database. From `psql`, we can do this:

```python
postgres=# CREATE USER chico WITH PASSWORD 'hard-to-guess';
CREATE ROLE
```

If the user already exists, we can use the `ALTER` command:

```python
postgres=# ALTER USER muthoni WITH PASSWORD 'difficult-password';
ALTER ROLE
```

## Change User Password

In the event you forget the password used by a given user, or you would simply want to modify it as a good security practice, PostgreSQL provides a way to recover your account:

```python
$ sudo -u postgres psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=#\password muthoni
Enter new password for user "muthoni":
Enter it again:
postgres=#
```

You begin by accessing the default `postgres` user, then issue the command `\password` followed by the _name_ of a user. You will be prompted to enter and confirm a new password.

Or you may do everything from the default Ubuntu terminal as follows:

```python
$ sudo - postgres psql -c "ALTER USER muthoni PASSWORD 'new-password';"
```

You can then restart your PostgreSQL server:

```python
$ sudo service postgresql start
```

## Connect to PostgreSQL As Another User

So far, we have been using the default `postgres` user to access the interactive SQL terminal. Now that we have other users created and their passwords set up, let us try to log in as these users.

```python
$ sudo -u muthoni psql

# Output
sudo: unknown user: muthoni
sudo: unable to initialize policy plugin
```

Apparently, _muthoni_ is not recognized from the attempt above. What we are trying to do here is actually to connect to a database called _muthoni_ which at the moment does not exist. Note that when we run `sudo -u postgres psql` the psql prompt changes to `postgres=#` meaning we have connected to the `postgres` database using the default `postgres` user. The format used above is:

```python
$ sudo -u <database> psql
```

What we need to do is to connect to an existing database using an existing user. Two things are needed: database and user for a connection to be established. Below, I will show you two approaches to Access and Authenticate users when working with postgreSQL. If you would like to gain some background knowledge on the two methods, read [How to Secure PostgreSQL](/databases/02_how_to_secure_postgresql.md).


### Approach 1: Machine Access

The idea here is to connect to postgreSQL just as you would any other file in your Unix machine, since postgreSQL is a special file within the local file system. This means that whoever wants to access them has to have the access rights just as any other user of the machine.

The default user of my machine is _harry_. Therefore, running `sudo psql -u harry <database>` will take me directly to the said database without asking for a password.

```python
$ sudo psql -u harry testdb

# No password asked

psql (14.5 (Ubuntu 14.5-1.pgdg20.04+1))
Type "help" for help.

testdb=> 
```

I also have another user called _muthoni_. If I try to access an existing database using this user, access is denied:

```python
$ psql -d <database> -U muthoni

# Output
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  Peer authentication failed for user "muthoni"
```

This is a problem with the `pg_hba.conf` file. We need to locate this file since we will be making changes to it to fix the "Peer" error. Run this command to see where it is located:

```python
$ locate pg_hba.conf

# Output
/etc/postgresql/14/main/pg_hba.conf
/usr/share/postgresql/14/pg_hba.conf.sample
```

Two locations have been found. We now need to make some changes in the file. Open the file using `nano`:

```python
$ sudo nano /etc/postgresql/14/main/pg_hba.conf
```

The contents of the file is largely commented out. Scroll to the bottom of the file until you get to "Database administrative login by Unix domain socket". You will see these two lines:

```python
# Line 1
local      all      postgres      peer

# Line 2
local      all      all           peer
```

The general format of the `pg_hba.conf` file is a set of records, one per line. Blank lines are ignored, as is any text after the # comment character. Each record specifies a:
- connection type,
- client IP address range (if relevant for the connection type),
- database name,
- user name,
- authentication method to be used for connections matching these parameters. 

What we want to do is to change the authentication method from _peer_ to _md5_ (which enforces passwords). In line 1, all users can access the postgres database through the peer method. In line 2, all users can access all databases using the peer method.

Using the direction keys on your keyboard, navigate to where "peer" is and change these two instances to "md5". The new lines will become:

```python
# Line 1
local      all      postgres      md5

# Line 2
local      all      all           md5
```

With these changes, we need to restart the postgreSQL server:

```python
$ sudo service postgresql restart
```

Let us try login as postgres user:

```python
$ psql -U postgres

# Enter your postgres password.
# Here, we are connecting to the postgres database as postgres user

Password for user postgres: 
psql (14.5 (Ubuntu 14.5-1.pgdg20.04+1))
Type "help" for help.

postgres=# 
```

We can also login as _muthoni_:

```python
$ psql -d testdb -U muthoni

# Enter "muthoni"'s password
# We are accessing the testdb using muthoni

Password for user muthoni: 
psql (14.5 (Ubuntu 14.5-1.pgdg20.04+1))
Type "help" for help.

testdb=> 
```

If you are wondering what "peer" or "md5" means, these authentication methods are used to provide various levels of access.

- **Trust**: anyone who can connect to the server is authorized to access the database
- **Peer**: user a client's OS user name as database user name to access it
- **md5**: password-based authentication, hashed using md5

The `pg_hba.conf` file [documentation](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html) provides great insight.

### Approach 2: Remote Access Using TCP/IP


If you would like to change to another user, run the command below in the default terminal prompt:

```python
$ psql -h localhost -d <database> <user> -p 5432
```

The `-h` option stands for host, `-d` for database and `-p` for port. Port 5432 is the default port. To know what port you would like to connect to, simply run:

```python
postgres=# SHOW port

# Output
 port 
------
 5432
(1 row)
```

Take this example:

```python
$ psql -h localhost testdb muthoni -p 5234
```

I will be asked to enter _muthoni_'s password. Once accepted, the terminal will switch to _testdb_ prompt:

```python
psql (14.5 (Ubuntu 14.5-1.pgdg20.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

testdb=> 
```

If you would like to switch to another database using _muthoni_, simply run:

```python
testdb=> \c anotherdb

# Output
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "anotherdb" as user "muthoni".
anotherdb=> 
```

The 'c' stands for 'connect'. We are connecting to `anotherdb` as user _muthoni_. If you would like to switch to another database as another user, not _muthoni_, then you need to append the user's name to `\c`.

```python
testdb=> \c testdb harry
```

I will be asked to provide _harry_'s password since this user is password-protected. A successful login will connect _harry_ to _test_db_.