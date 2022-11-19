# Configure PostgreSQL in a Live Linux Server

There is no shortage of how to deploy a Python-powered backend application. Platform-as-a-service solutions such as [Python Anywhere](https://help.pythonanywhere.com/pages/Flask/) and [Google App Engine](https://cloud.google.com/appengine/docs/standard/python3/building-app), among others, do a good job of lifting the burden to self-host an application so we can focus on building.

As wonderful as these platforms are, using them causes us to lose the experience of learning how to host, configure and maintain a server. This article is dedicated to discussing how you can configure your server to use the PostgreSQL database.

### Table of Contents

1. [Getting Started]()
2. [PostgreSQL Overview]()
3. [Deployment to a Virtual Machine]()
4. [Configuring PostgreSQL in Linode]()

## Getting Started

This article presumes that you already have an application built, and ready for deployment. I will be using a [pre-built flask application](https://github.com/GitauHarrison/configure-flask-to-use-postgresql) to demonstrate how to configure PostgreSQL. Also, it is presumed you are already familiar with using PostgreSQL. For example, you know how to create a user and add a database. If you are not familiar with any of these, I recommend that you start here before proceeding:

- [Start a Flask Server]() (getting started with Flask)
- [Install PostgreSQL]() (install and set up PostgreSQL in Linux)
- [Accessing PostgreSQL using psql]() (Working with the PostgreSQL database)


## PostgreSQL Overview

As an overview, let us see how we can work with PostgreSQL. First, let us learn how to create a user and connect them to an existing database.

```python
$ sudo -u postgres psql

# Output
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=#
```

This command is run on the terminal. It allows us to access the `postgres` database using the `postgres` user. PostgreSQL normally comes with this user and database pre-configured right out of the box after installation. From here we can issue SQL commands such as this to create a superuser:

```python
postgres=# CREATE USER muthoni SUPERUSER;
CREATE ROLE

# Output
 Role name   |  List of roles Attributes     | Member of 
-------------+-------------------------------+-----------
 muthoni     |                               | {}
 postgres    | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

Alternatively, we can create this superuser directly from the terminal by running:

```python
$ sudo -u postgres createuser --superuser muthoni
```

This command is similar to the one above it. They both create a user called muthoni. What we would want to do immediately is to enforce the use of passwords by all users to protect the integrity of the database. You can learn how to do that [here](https://github.com/GitauHarrison/notes/blob/master/databases/access_postgresql/psql.md#create-user-password). To complete the setup process, this new user will need to have access to a pre-built database. The user can then interact with the database moving forward as follows:

```python
$ psql -d test_db -U muthoni
# Enter "muthoni"'s password
# We are accessing the testdb using muthoni

Password for user muthoni: 
psql (14.5 (Ubuntu 14.5-1.pgdg20.04+1))
Type "help" for help.

testdb=>
```

Database access can be managed by making changes to the `pg_hba.conf` file. This is the client authentication control file, typically stored in the database cluster's data directory. HBA stands for host-based authentication. You can locate this file by running:

```python
$ locate pg_hba.conf
```

You can then make edits to it by opening it in your favorite editor. I will use nano below:

```python
$ sudo nano path/to/pg_hba.conf
```

The general format of the `pg_hb.conf` file is a set of records, one per line. Each record specifies a _connection type_, a _client IP address range_, a _database name_, a _user name_, and an _authentication method_ to be used for connections matching these parameters.

```python
local         database     user            auth-method [auth-options]
local         all          postgres        peer

host         database      user            address        auth-method  [auth-options]
local        all           postgres        IP Address     peer
```

Locally, the PostgreSQL configuration is already done for us, so there is not much we need to do to get started with this database. You can refer to the [pg_hba.conf file documentation](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html) to learn more.

## Deployment to a Virtual Machine

When you deploy to a Linux server, you are going to be using a virtual machine hosted by your provider, in this case, I will use Linode. To begin, ensure that you have created an account on Linode. If you haven't already, [click here to sign up](https://linode.gvw92c.net/15oBBg). Occasionally, Linode will offer you free hosting credits as soon as you sign up. So, be sure to check that out and take advantage of it.

Kindly check the article on how to [Deploy Your Flask App on Linode Server](https://www.gitauharrison.com/articles/deployment/linode/deploy-flask-app-on-linode-server) to learn how it is done. I have already covered this in great detail. The application in the article uses SQLite, which is file-based, and therefore, very easy to use.


## Configuring PostgreSQL in Linode

As mentioned earlier, PostgreSQL comes preconfigured when you install it on your local machine. Everything works from localhost. With a live application, however, you are most likely to use your own IP address. This means that changes need to be made to accommodate our new needs.

### Install PostgreSQL

As is done on a local machine, we need to install PostgreSQL on your Linux server. If you followed through with the article [Deploy Your Flask App on Linode Server](https://www.gitauharrison.com/articles/deployment/linode/deploy-flask-app-on-linode-server), you can recall that we set up an entirely new virtual machine. It was like installing Ubuntu for the first time. We, therefore, need to install PostgreSQL too.

```python
# Create the file repository configuration:
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repository signing key:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Update the package lists:
sudo apt-get update

# Install the latest version of PostgreSQL.
# If you want a specific version, use 'postgresql-12' or similar instead of 'postgresql':
sudo apt-get -y install postgresql
```

You can confirm that it was successfully installed by running:

```python
$ psql -V
# Output
psql (PostgreSQL) 14.4 (Ubuntu 14.4-1.pgdg20.04+1)
```

Remember to start the PostgreSQL server. This is done by the command:

```python
$ sudo service postgresql start

# Output
* Starting PostgreSQL 14 database server
```

### Create A User

Now that there is an existing connection to the PostgreSQL server, we now have access to the `psql` command that allows us to interact with PostgreSQL from the terminal. For example, if we want to access the `postgres` database using the `postgres` user, we can run:

```python
$ sudo -u postgres psql
# Output
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=#
```

Since we are using the `sudo` command, we will be asked to provide our local machine password to continue. Notice that the prompt has changed to `postgres=#`.

`postgres` is the default user that comes with PostgreSQL. What we can do now is create our own user and give the user a password.

```python
postgres=# CREATE USER muthoni;
CREATE ROLE
```

This command creates a user called `muthoni`. The `CREATE USER` query is an alias (which means "an assumed identity") of `CREATE ROLE <name> WITH LOGIN;`.

Should we want `muthoni` to bypass all checks, we can make her a super user.

```python
postgres=# ALTER USER muthoni SUPERUSER;
CREATE ROLE
```

To ensure that `muthoni` is protected, we can give her a password.

```python
postgres=# ALTER USER muthoni WITH PASSWORD 'difficult-password';
ALTER ROLE
```

### Create A Database

There needs to be a database file to complete the setup process. Currently, in our new virtual machine, none exists. So, we need to create one. This can be achieved by running:

```python
postgres=# CREATE DATABASE demo_postgres_db WITH OWNER muthoni;
```

This makes `muthoni` the owner of the new database `demo_postgres_db`. If you would like to list all the databases currently in existence, plus see a few extra details, you can run:

```python
postgres=# \l
```
### Connect New User to the Database

It is now possible to access the `database demo_postgres_db` as `muthoni`. The command below allows for this:

```python
$ psql -d demo_postgres_db -U muthoni
```

You will most likely get the error `#psql: error: connection to server on socket "/var/run/postgresql/.s.PGSLQ.5432" failed: FATAL: Peer authentication failed for user "muthoni"`. This is a problem with the `pg_hba.conf` file. We need to locate it.

### Enforce Password Authentication

To locate the `pg_hba.conf` file, you can run:

```python
$ locate pg_hba.conf

# Output
/etc/postgresql/14/main/pg_hba.conf
/usr/share/postgresql/14/pg_hba.conf.sample

```

We are going to make some changes to it. This can be done by opening it in your favorite editor. I will use `nano` below:

```python
$ sudo nano /etc/postgresql/14/main/pg_hba.conf
```

The content of the file is largely commented out. Scroll to the bottom of the file until you get to "Database administrative login by Unix domain socket". You will see these two lines:

```python
# Line 1
local      all      postgres      peer

# Line 2
local      all      all           peer
```

Using the direction keys on your keyboard, navigate to where "peer" is and change these two instances to "md5". The new lines will become:

```python
# Line 1
local      all      postgres      md5

# Line 2
local      all      all           md5
```

With these changes, we need to restart the PostgreSQL server:

```python
$ sudo service postgresql restart
```

Let us try login as `muthoni`:

```python
$ psql -d demo_postgres_db -U muthoni

# Enter "muthoni"'s password
# We are accessing the testdb using muthoni
Password for user muthoni: 

psql (14.5 (Ubuntu 14.5-1.pgdg20.04+1))
Type "help" for help.

demo_postgres_db=>
```
I should point out that now even the user `postgres` will require a password to gain access to a database. If you are wondering what "peer" or "md5" means, these authentication methods are used to provide various levels of access.

- **Trust**: anyone who can connect to the server is authorized to access the database
- **Peer**: user a client's OS user name as database user name to access it
- **md5**: password-based authentication, hashed using md5


### Database Configuration In The Flask App

The `config.py` file is used to define what configurations are needed. Of interest is `DATABASE_URL`. This variable's value is sourced from the environment variables found in the `.env` file.

```python
# config.py

import os

basedir = os.path.abspath(os.path.dirname(__file__))
class Config(object):
    """All application configurations"""

    # Secret key
    SECRET_KEY = os.environ.get('SECRET_KEY')

    # Database configurations
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \ # Use postgresql if present else
        'sqlite:///' + os.path.join(basedir, 'app.db')            # default to file-based SQLite 
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

Using the format `DATABASE_URL=[engine]://[user name]:[user password]@[host]:[port]/[database]`, we can update the `.env` file as follows:

```python
#.env

SECRET_KEY=harry
DATABASE_URL=postgresql://muthoni:muthoni@localhost:5432/demo_postgres_app
```

To ensure that we do not have any issues in production, we modify the `config.py` file to use `config.get()` method instead of `os.environ.get()`.

```python
# config.py

# ... 
import json
    
    
with open('/etc/test_app.json') as config_file:
    config = json.load(config_file)
    
    
class Config(object):
    SECRET_KEY = config.get('SECRET_KEY')
    SQLALCHEMY_DATABASE_URI = config.get('DATABASE_URL')
    MAIL_USERNAME = config.get('MAIL_USERNAME')
    MAIL_PASSWORD = config.get('MAIL_PASSWORD')
    # ...
```

Additionally, and you may have noticed, the secret environment variables are located in the file `/et/test_app.json` and not the `.env`. This is so because it can get very tricky quickly to work with environment variables in web servers. Once again, if you have been following through with the article [Deploy Your Flask App on Linode Server](https://www.gitauharrison.com/articles/deployment/linode/deploy-flask-app-on-linode-server), you already have this file. What we need to do now is to update it. Open it in `nano`:

```python
$ sudo nano /etc/test_app.json
```

Using your keyboard arrows, modify the variable `DATABASE_URL` to:

```python
DATABASE_URL=postgresql://muthoni:muthoni@your_ip_address:5432/demo_postgres_db
```

I have changed the host from `localhost` to `project_ip_address`. The default PostgreSQL port `5432` is maintained in the live server. Make sure the changes are saved by pressing `ctrl + X` . Type "y" for "yes" to accept the changes.

### Configure PostgreSQL to Use New IP Address

**pg_hba.conf file**

The next step would be to modify the `pg_hba.conf` file in the Linux server.

```python
$ locate pg_hba.conf
$ sudo nano /etc/postgresql/14/main/pg_hba.conf
```

Then make the following changes:

```python
# "local" is for Unix domain socket connections only
local   all             all                                     md5
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
host    all             all             0.0.0.0/0               md5
# IPv6 local connections:
host    all             all             ::0/0                   md5
```

I have basically added the address `0.0.0.0/0` to the file for `IPv4` and `::0/0` on `IPv6`. `0.0.0.0/0` represents all IPv4 addresses, and `::0/0` represents all `IPv6` addresses. The [auth-pg-hba-conf documentation](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html) is a good resource to refer to.

Save the changes by pressing `ctrl + X` and typing "y" for "yes".

**postgresql.conf file**

The next file that needs to be modified is the `postgresql.conf`. This file has the general configurations on PostgreSQL.

```python
$ locate postgresql.conf
$ sudo nano /etc/postgresql/14/main/postgresql.conf
```

Find the following:

```python
listen_addresses = " " (it is probably commented out)
port =
```

Navigate to these lines and update them as follows:

```python
listen_addresses = 'app_IP_address' 
port = 5432
```

That's all! Now, let us restart the PostgreSQL server.

```python
$ sudo service postgresql restart
```

### Update Firewall

A firewall is used to monitor and control incoming and outgoing network traffic based on predetermined security rules. So far, based on the [Linode deployment article](https://www.gitauharrison.com/articles/deployment/linode/deploy-flask-app-on-linode-server), we have a few things set up. What we need to do now is to update our firewall to access traffic on port `5432`.

```python
$ sudo ufw allow 5432
$ sudo ufw enable
# type "y" for "yes" when prompted
```

To finalize things, we now need to restart nginx and reload supervisor .

```python
# Nginx
$ sudo systemctl restart nginx.service

# Supervisor
your_project_folder$ sudo supervisorctl reload
```

Should you encounter an issue running the command above, such as `error: <class 'FileNotFoundError'>, [Errno 2] No such file or directory: file: /usr/lib/python3/dist-packages/supervisor/xmlrpc.py line: 560`, consider running the two commands below to fix it:

```python
your_server:~/your_project_folder$ sudo supervisord -c /etc/supervisor/supervisord.conf

your_server:~/your_project_folder$ sudo supervisorctl -c /etc/supervisor/supervisord.conf
    
# Output
supervisor> # Type "reload"

Really restart the remote supervisord process y/N? # Answer "y" for "yes"
# You should see "Restarted supervisord"
# Exit supervisor prompt using "ctrl + c"
```

Once done, you can exit your server. Run `exit` on the Linux server terminal.
