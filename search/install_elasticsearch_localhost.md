# Install Elasticsearch in Ubuntu 20.04 In Localhost

Since support for full-text search is not standardized like relational database are, there are several open-source full-text engines. Examples include [Sphinx](http://sphinxsearch.com/), [Apache Solr](https://lucene.apache.org/solr/), [Elasticsearch](https://www.elastic.co/elasticsearch/) among others. Thankfully, all these engines can work within a Flask application due to the fact that Flask is not opinionated. Using search capabilities of one of the relational databases would be great but since SQLAlchemy does not support this functionality, we will have to content with learning how to handle the searching with raw SQL statement. 

We can intentionally choose to use any of the full-text search engines to do the job for us. I will show you how you can install Elasticsearch and perform basic searches. If you would like to check out other articles on Elasticsearch, you can click on any of the links below:

- [Install Elasticsearch in Ubuntu 20.04 In Localhost](install_elasticsearch_localhost.md) (this article)
- [Install And Configure ElasticSearch In A Live Linux Server](install_elasticsearch_linode.md)
- [Implement Search Functionality In Your Flask App](implement_elasticseach.md)


### Table Of Content

- [Overview](#overview)
- [Install Necessary Packages](#install-necessary-packages)
- [Download and Install `Elasticsearch`](#download-and-install-elasticsearch)
- [Start Elasticsearch](#start-elasticsearch)
   - [Memory Usage](#memory-usage)
- [Testing](#testing)
- [Basic Security](#basic-security)
   - [Enable Security](#enable-security)
   - [Create Passwords For Built-in Users](#create-passwords-for-built-in-users)
   - [Create Users And Their Passwords And Assign Roles](#create-users-and-their-passwords-and-assign-roles)
   - [Reset User Password](#reset-user-password)
   - [Delete User](#delete-user)


## Overview

You probably are aware that `pip` allows us to install Python packages, and, therefore, it would make so much obvious sense to simply install `elasticsearch` by running:

```python
(venv)$ pip3 install elasticsearch
```
We can test out the service by performing basic operations in a Python interpreter:

```python
$ python3
# ...
>>> from elasticsearch import Elasticsearch
>>> es = Elasticsearch('http://localhost:9200')
```
We have created an instance of the class `Elasticsearch` and passed a connection URL to connect to the service. Since data in Elasticsearch is written in _indexes_, let us try to make one. 

```python
>>> es.index(index='test_index', id=1, body={'test': 'this is the first test'})
```
Elasticsearch works with data in JSON format. We have written an object with the field `text` to an index called `test_index`. When you press `Enter` on your keyboard, you will get a `ConnectionError`. Your attempts to connect to the service fails. 

`pip3 install elasticsearch` only installs a Python client which is used for communication between your Python script and the existing Elasticsearch cluster. The assumption is that the cluster is currently running on  `localhost` with the default elasticsearch port `9200`.

There are a couple of steps we need to follow to ensure that we have configured our machine to run `Elasticsearch`:
* Install necessary packages needed to run `Elasticsearch`
* Download and install `Elasticsearch`
* Start the `Elasticsearch` service

## Install Necessary Packages

Since Elasticsearch runs on top of Java, you need to install the Java Development Kit (JDK). You can check if Java is installed by running this command in your terminal:

```python
$ java -version
```
If you do not have java, you can install the default JDK by:
```python
$ sudo apt install openjdk-8-jre-headless 
```

Run `java -version` to check that `Java` is installed.

Then next step would be to allow access to our repositories via HTTPS. Therefore, let us install an APT transport package:

```python
$ sudo apt install apt-transport-https
```

## Download and Install `Elasticsearch`

First, update the GPG Key for the Elasticsearch repository using the wget command to pull the public key:

```python
$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
Add the repository to your system:

```python
(venv)$ echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```

Now, install `elasticsearch` by first updating the package index then run the installation:
```python
$ sudo apt update
$ sudo apt install elasticsearch
```

## Start Elasticsearch

Elasticsearch does not run until you start it. Also, when you reboot your machine, you need to rerun the elasticsearch service since it does not start automatically. 

To reload the `systemd` configuration, run:
```python
$ sudo systemctl daemon-reload
```

Enable the elasticsearch service:

```python
$ sudo systemctl enable elasticsearch.service
```

Now, you can start elasticsearch:
```python
$ sudo systemctl start elasticsearch.service
```

### Memory Usage

You may encounter the following error after running the command above:

```python
Job for elasticsearch.service failed because a fatal signal was delivered to the control process.
See "systemctl status elasticsearch.service" and "journalctl -xe" for details.
```

What this means is that we are unable to connect to the elastic search service. A quick run of the command `systemctl status elasticsearch.service` in the terminal will show:

```python
● elasticsearch.service - Elasticsearch
     Loaded: loaded (/lib/systemd/system/elasticsearch.service; enabled; vendor preset: enabled)
     Active: failed (Result: signal) since Thu 2023-04-20 11:41:55 EAT; 3min 6s ago
       Docs: https://www.elastic.co
    Process: 84416 ExecStart=/usr/share/elasticsearch/bin/systemd-entrypoint -p ${PID_DIR}/elasticsearch.pid>
   Main PID: 84416 (code=killed, signal=KILL)

Apr 20 11:41:50 harry systemd[1]: Starting Elasticsearch...
Apr 20 11:41:55 harry systemd[1]: elasticsearch.service: Main process exited, code=killed, status=9/KILL
Apr 20 11:41:55 harry systemd[1]: elasticsearch.service: Failed with result 'signal'.
Apr 20 11:41:55 harry systemd[1]: Failed to start Elasticsearch.
lines 1-11/11 (END)
```

Alternatively, you can run `journalctl -xe` on the terminal to see:

```python
# ...

elasticsearch Out of memory: Killed process 58476 (java) total-vm:7952508kB, anon-rss:357382>
# ...
```

To fix this, we will need to open the `./elasticsearch.yml` file:

```python
$ sudo nano /etc/elasticsearch/elasticsearch.yml
```

This will open the file in `nano`. Navigate to **Network** section and ensure your settings are:

```python
# /etc/elasticsearch/elasticsearch.yml: Network settings

# ---------------------------------- Network -----------------------------------
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: 127.0.0.1
#
# Set a custom port for HTTP:
#
http.port: 9200
```

Elasticsearch is now enabled to allow connections from localhost and to also listen to port `9200`. If you run `journalctl -xe`, you may notice that the error comes from memory issues. To fix this memory issue, we need to devot some memory to the Elasticsearch main service. Elasticsearch is a JVM application and it, therefore, utilizes memory devoted to the JVM (32-bit or 64-bit). The memory used by JVM usually consist of:

- **Heap space** (configured via -Xms and -Xmx)
- **Internal JVM** (usually tens of Mb)
- OS-dependent memory features like **memory-mapped files**.

Elasticsearch mostly depends on the **heap memory** whose setting is done _manually_ by passing `-Xms` and `-Xmx` (heap space) option to the JVM running the Elasticsearch service. To do this, open `./jvm.options` file:

```python
$ sudo nano /etc/elasticsearch/jvm.options
```

> `Xms` represents the initial size of total heap space
`Xmx` represents the maximum size of total heap space

In the opened JVM Configuration file, uncomment the value of `-Xms` and `-Xmx`. My current machine has 8GB RAM. The default values are `-Xms4g` and `-Xmx4g`.  As you modify these values, ensure that they are no more than 50% of your physical RAM.

```python
# -----------------
# RAM <= 1 GB
# -----------------
-Xms128m
-Xmx128m

# OR

-Xms256m
-Xmx256m

# -----------------
# RAM >= 2 GB but <= 4 GB
# -----------------
-Xms512m
-Xmx512m

# OR

-Xms750m
-Xmx750m

# -----------------
# RAM >= 4 GB but <= 8 GB
# -----------------
-Xms1024m
-Xmx1024m

# OR

-Xms2048m
-Xmx2048m

# -----------------
# RAM >= 8 GB
# -----------------

Decide how much heap space you want to allocated Elasticsearch
For better performance, you can allocated higher based on available resources
```
Ensure that the value of `-Xms` and `-Xmx` are equal.

```python
# --- Wrong ---
-Xms128m
-Xmx256m

# --- Correct ---
-Xms128m
-Xmx128m
```
In the event you get this error:

```python
the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
```

Open `elasticsearch.yml` in `nano` and adjust your discovery settings as below

```python
# /etc/elasticsearch/elasticsearch.yml: Discovery setting

# -------------------------------- Discovery ---------------------------------
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
discovery.seed_hosts: []
```

Start Elasticsearch and check its status by running these commands:

```python
$ sudo systemctl start elasticsearch.service
$ sudo systemctl status elasticsearch.service
```

If you followed the installation steps correctly, you may need to restart your machine.To test your set up and installation, run this command on your terminal:

```python
(venv)$ curl localhost:9200
```
You should be able to see JSON-formatted data displayed in your terminal. Alternatively, paste http://localhost:9200/ to your browser's address bar. JSON data will be dumped on your screen. 

If you are curious what JSON is, it stands for JavaScript Object Notation. It is a standardized format that is commonly used to transfer data as text over a network. It is used by a lot of APIs and databases, and it is very easy for both humans and machines to read. JSON is more like Python dictionaries with key/value pairs. 

The process of encoding data into JSON format (dict to JSON) is called **Serialization**. Often the function `json.dumps(dict_object)` does this. The process of converting JSON back to native objects such dicts is called **Deserialization** using `json.loads(object)`.

## Testing

Create a virtual environment and install `elasticsearch` in this environment:

```python
$ mkvirtualenv test
(test)$ pip3 install elasticsearch
```
Start your interpreter:

```python
(test)$ python3
# ...
>>> from elasticsearch import Elasticsearch
>>> es = Elasticsearch('http://localhost:9200')
>>> es.index(index='test_index', id=1, body={'test': 'this is the first test'})

# Output
{'_index': 'my_index', '_type': '_doc', '_id': '1', '_version': 2, 'result': 'updated', '_shards': {'total': 2, 'successful': 1, 'failed': 0}, '_seq_no': 1, '_primary_term': 1}
```

## Basic Security

As from version 8 of elasticsearch, you may see the warming 
```python
<console>:1: ElasticsearchWarning: Elasticsearch built-in security features are not enabled. Without authentication, your cluster could be accessible to anyone. See https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-minimal-setup.html to enable security.
```

Elasticsearch security features are not enabled by default. These features are free, but require configuration changes to enable them. This means that users don’t have to provide credentials and can get full access to the cluster of nodes. Network connections are also not encrypted. To protect your data, it is strongly encourage you to enable the Elasticsearch security features. 


### Enable Security

How do you enable this feature? In Ubuntu, you will need to locate the file `elasticsearch.yml`. 

- First, stop Elasticsearch from running if it is:

   ```python
   $ sudo systemctl stop elasticsearc.service
   ```

- Then, open `elasticsearch.yml` file in `nano`:

   ```python
   $ sudo nano /etc/elasticsearch/elasticsearch.yml
   ```

- Scroll down to the line that reads network.host

   ```python
   # /etc/elasticsearch/elasticsearch.yml: Enable security

   # ---------------------------------- Node -----------------------------------
   #
   xpack.security.enabled: true
   #
   ```

- Press Ctrl + X, then Enter on your keyboard to save the changes.

If you restart the Elasticsearch service and run `curl localhost:9200` on your terminal, you will see this error:

```python
{"error":{"root_cause":[{"type":"security_exception","reason":"missing authentication credentials for REST request [/]","header":{"WWW-Authenticate":"Basic realm=\"security\" charset=\"UTF-8\""}}],"type":"security_exception","reason":"missing authentication credentials for REST request [/]","header":{"WWW-Authenticate":"Basic realm=\"security\" charset=\"UTF-8\""}},"status":401}
```

Alternatively, if you paste `localhost:9200` in your browser, you will notice a pop up asking for a username and password. Since we do not know what these credentials are, if we press "Cancel" or if we close the prompt, we will get the `missing authentication credentials for REST request` error seen above.

Surprised? Hadn't we run `curl localhost:9200` earlier and we got some JSON data back? No, it should not be a surprise. We have just enabled security on our cluster and usage is dependant on user authenication now.


### Create Passwords For Built-in Users

To communicate with the cluster, you must configure a username for the [built-in users](https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html). All requests that do not have a user name and a password will be rejected. To index documents, users much have write access.

- Access the `elasticsearch/` folder:

   ```python
   $ cd /usr/share/elasticsearch/
   ```
- In this folder, we can set passwods for built-in users by running:

   ```python
   # -------------------
   # auto
   # -------------------

   $ sudo bin/elasticsearch-setup-passwords auto

   # OR

   # -------------------
   # interactive
   # -------------------

   $ sudo bin/elasticsearch-setup-passwords interactive
   ```
   - The `auto` parameter outputs randomly-generated passwords to the console that you can change later if necessary.
   - The `interactive` parameter allows you to set your own passwords

I used `interactive` so I can set my own password. You will be taken through a series of users for whom you will set a password. One of the users is `elastic`. Now that I know a user, and I have a password for this user, I can access the Elasticsearch service. Paste localhost:9200 on your browser and key in your user credentials. You should be able to see some JSON data dumped on the screen.

In your terminal, where there is no popup, you will need to use the formart below to access Elasticsearch:

```python
$ curl 'http://usernameWithWriteAccess:password@localhost:9200'
```

Replace `usernameWithWriteAccess` with your user, eg `elastic`, and provide your password by replacing `password` in the URL after the colon (`:`). If you are working with environment variables,  you need to instantiate the `ELASTICSEARCH_URL` as seen below before restarting your Flask server:

```python
#.env: Instantiate Elasticsearch variable

ELASTICSEARCH_URL=http://usernameWithWriteAccess:password@localhost:9200
```

If you want to know what other elastic command you can use, run:

```python
$ cd /usr/share/elasticsearch/
$ cd bin

# You will see a list of available files that you can run against.
# See the elasticsearch documenation for more information.
```

### Create Users And Their Passwords And Assign Roles

To create custom users, run:

```python
$ cd /usr/share/elasticsearch/bin

# As su
$ sudo ./elasticsearch-users user add test_user - p test_user_password -r test_user_role
```

You will notice a warning telling you of available users and roles, but for now we will ignore this. The command above will create a user called `test_user` whose password is `test_user_password` of role `test_user_role`. To list the users you have created, run:

```python
# /usr/share/elasticsearch/bin: List available users

$ ./elasticsearch-users list

# Output
testuser       : test_role*
```

### Reset User Password

To reset an existing user's password, run:

```python
# /usr/share/elasticsearch/bin: Reset user password

$ ./elasticsearch-users passwd test_user

# You will be asked to enter a new password and confirm it
```

### Delete User

To delete a user in your cluster, run:

```python
# /usr/share/elasticsearch/bin: Delete a user

$ ./elasticsearch-users userdel test_user

# Check the users' list
```