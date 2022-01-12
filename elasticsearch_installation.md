# Install Elasticsearch in Ubuntu 20.04

Since support for full-text search is not standardized like relational database are, there are several open-source full-text engines. Examples include [Sphinx](http://sphinxsearch.com/), [Apache Solr](https://lucene.apache.org/solr/), [Elasticsearch](https://www.elastic.co/elasticsearch/) among others. Thankfully, all these engines can work within a Flask application due to the fact that Flask is not opinionated. Using search capabilities of one of the relational databases would be great but since SQLAlchemy does not support this functionality, we will have to content with learning how to handle the searching with raw SQL statement. 

We can intentionally choose to use any of the full-text search engines to do the job for us. I will show you how you can install Elasticsearch and perform basic searches.

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

Run `java --version` to check that `Java` is installed.

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

At this point, elasticsearch will start everytime you reboot your system. To test your set up and installation, run this command on your terminal:
```python
(venv)$ curl localhost:9200
```
You should be able to see JSON formatted data displayed in your terminal. Alternatively, paste http://localhost:9200/ to youre browser's address bar. JSON data will be dumped on your screen.

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