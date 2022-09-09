# Database Management Tools

These are software applications that help users to manage SQL server infrastructure. They allow users to configure, monitor, manage and administer SQL servers and databases. As a web developer or as a database administrator, you will be dealing with SQL statements to explore the database for various reasons:

- Querying the database
- Build and execute SQL code
- Generating reports
- Making a backup
- Diagonising an application for database-related problems

It is therefore important to choose and have the right tool that can speed up database-related tasks and make you more productive. There are many tools available in the market that can be used to work with a database. To name two, we have [pgAdmin](https://www.pgadmin.org/) and [DBeaver](https://dbeaver.io). I will use DBeaver throughout the subsquent articles as we strive to understand SQL in-depth.

There are two ways you can connect to PostgreSQL:

1. [Using `psql`](/databases/access_postgresql/psql.md)
2. [Using SQL client software application and database administration tool](/databases/access_postgresql/dbeaver.md) (this article)

For reference, these are the topics we will cover in the entire database tutorial:

- [PostgreSQL Overview](postgresql.md)
- [Install PostgreSQL](install_postgresql.md)
- [Getting Started with PostgreSQL](getting_started_with_postgresql.md)

### Table of Contents

Throughout this article, we will look at how to:

1. [Install DBeaver](#install-dbeaver)


## Install DBeaver

DBeaver is free and open source universal database tool for developers and database administrators. It supports several databases including MySQL, Oracle and Postgresql. It can be used as an alternative of MySQL-Workbench and PGAdmin. It has a user-friendly interface that makes it more useful for its new users. Using DBeaver, you can create local databases and also configure the maximum result size to avoid session hanging issues in case the query results take time to process big queries. The community edition of DBeaver is free, though if you'd like to do NoSQL then you may opt for the enterprise version.

### Using the Terminal

OpenJDK is the default Java development and runtime available on Debian repository. Run the following commands to have Java installed and set as default Java on your Ubuntu:

```python
sudo apt update
sudo apt -y install default-jdk
```

Once that is done, you can check the version by running:

```python
$ java --version

# Output

openjdk 11.0.16 2022-07-19
OpenJDK Runtime Environment (build 11.0.16+8-post-Ubuntu-0ubuntu120.04)
OpenJDK 64-Bit Server VM (build 11.0.16+8-post-Ubuntu-0ubuntu120.04, mixed mode, sharing)
```

Add DBeaver repository to your Debian / Ubuntu by running the commands below:

```python
wget -O - https://dbeaver.io/debs/dbeaver.gpg.key | sudo apt-key add -

echo "deb https://dbeaver.io/debs/dbeaver-ce /" | sudo tee /etc/apt/sources.list.d/dbeaver.list
```

After adding the repo, update the apt list and install DBeaver CE:

```python
sudo apt update

sudo apt install dbeaver-ce
```


Once the installation is complete, you can start DBeaver by typing `dbeaver-ce` in the terminal. You should see DBeaver launching.

![DBeaver Terminal Launch](/images/databases/postgresql/dbeaver_terminal_launch.gif)

Or, you can simply search for DBeaver within your apps and click on it to launch.

![Lauch DBeaver from Apps](/images/databases/postgresql/launch_dbeaver_from_apps.gif)


## Using the Installer

If you find using the terminal a bit technical, you can do the following:

Download the installer file from https://dbeaver.io/download/. Navigate to the Linux subsection and choose the _Linux Debian Package(installer)_.

![Linux Debian Installer](/images/databases/postgresql/linux_installer.png)

The file will be downloaded to your local machine storage. Double-click the downloaded file to start the installation. You will see this:

![Install DBeaver from Software Center](/images/databases/postgresql/install_dbeaver_software_center.png)

Click on "Install" to begin the installation process. Once done, you can open the app from within your list of desktop apps.