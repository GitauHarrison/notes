# Install Elasticsearch in Ubuntu 20.04 In Localhost

Since support for full-text search is not standardized like relational database are, there are several open-source full-text engines. Examples include [Sphinx](http://sphinxsearch.com/), [Apache Solr](https://lucene.apache.org/solr/), [Elasticsearch](https://www.elastic.co/elasticsearch/) among others. Thankfully, all these engines can work within a Flask application due to the fact that Flask is not opinionated. Using search capabilities of one of the relational databases would be great but since SQLAlchemy does not support this functionality, we will have to content with learning how to handle the searching with raw SQL statement. 

We can intentionally choose to use any of the full-text search engines to do the job for us. I will show you how you can install Elasticsearch and perform basic searches. If you would like to check out other articles on Elasticsearch, you can click on any of the links below:

- [Install Elasticsearch in Ubuntu 20.04 In Localhost](install_elasticsearch_localhost.md) (this article)
- [Install And Configure ElasticSearch In A Live Linux Server](install_elasticsearch_linode.md)
- [Implement Search Functionality In Your Flask App](implement_elasticseach.md)

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
-- 
-- The job identifier is 5858 and the job result is failed.
Apr 20 11:42:01 harry CRON[84787]: pam_unix(cron:session): session opened for user harry by (uid=0)
Apr 20 11:42:01 harry CRON[84790]: (harry) CMD (cd /home/harry/software_development/personal/python/flask/current_projects/sample_elearning_app && venv/bin/flask se>
Apr 20 11:42:01 harry CRON[84787]: (CRON) info (No MTA installed, discarding output)
Apr 20 11:42:01 harry CRON[84787]: pam_unix(cron:session): session closed for user harry
Apr 20 11:43:01 harry CRON[85736]: pam_unix(cron:session): session opened for user harry by (uid=0)

# ...
```

If you followed the installation steps correctly, you may need to restart your machine to fix this error. At this point, elasticsearch will start everytime you reboot your system. To test your set up and installation, run this command on your terminal:
```python
(venv)$ curl localhost:9200
```
You should be able to see JSON-formatted data displayed in your terminal. Alternatively, paste http://localhost:9200/ to youre browser's address bar. JSON data will be dumped on your screen. 

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

As from version 8 of elasticsearch, you may see the warming `<console>:1: ElasticsearchWarning: Elasticsearch built-in security features are not enabled. Without authentication, your cluster could be accessible to anyone. See https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-minimal-setup.html to enable security.` Elasticsearch security features are not enabled by default. These features are free, but require configuration changes to enable them. This means that users don’t have to provide credentials and can get full access to the cluster. Network connections are also not encrypted. To protect your data, it is strongly encourage you to enable the Elasticsearch security features. 

How do you enable this feature? In Ubuntu, you will need to locate the file `elasticsearch.yml`. Run this to find it:

```python
sudo nano /etc/elasticsearch/elasticsearch.yml
```

Scroll down to the line that reads network.host

```python
# ---------------------------------- Network -----------------------------------
#
# By default Elasticsearch is only accessible on localhost. Set a different
# address here to expose this node on the network:
#
# network.host: 192.168.0.1
network.host: localhost
#
# By default Elasticsearch listens for HTTP traffic on the first free port it
# finds starting at 9200. Set a specific HTTP port here:
#
http.port: 9200
#
# For more information, consult the network module documentation.
#
```

Above, I have commented `network.host: 192.168.0.1` and added a new line `network.host: localhost`. Additionally, I have `http.port: 9200` to ensure the port used by elasticsearch is `9200`.

Scroll down further to the security section and add this line:

```python
xpack.security.enabled: true
```

Prese Ctrl + X, then Enter on your keyboard to save the changes. Restart `elasticsearch`.