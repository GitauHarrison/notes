# Install Elasticseacrch with Debian Package (Ubuntu 20.04)

## Install Necessary Packages

Since `Elasticsearch` runs on top of `Java`, you need to install the Java Development Kit (JDK). You can check if Java is installed by running this command in your terminal:

```python
$ java -version
```

If you do not have java, you can install the default JDK by:

```python
$  sudo apt install openjdk-8-jre-headless 
```

Run `$ java -version` to check that `java` is installed. Then, allow access to your repositories using HTTPS by installing an [APT](https://en.wikipedia.org/wiki/APT_(software)) repository package:

```python
$ sudo apt install apt-transport-https
```

## Download and Install Elasticsearch

First, update the GPG Key for the Elasticsearch repository using the `wget` command to pull the public key ([from the documentation now](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html)):

```python
$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

Then, add the repository to your system:
```python
$ echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```

At this point, you can now install `elasticsearch` by first updating the package index then run the installation:

```python
$ sudo apt update
$ sudo apt install elasticsearch
```

## Start `elasticsearch`

Elasticsearch does not run until you start it. Also, when you reboot your machine, you need to rerun the `elasticsearch` service since it does not start automatically. To reload the `systemd` configuration, run:

```python
$ sudo systemctl daemon-reload
```

Enable the `elasticsearch` service:

```python
$ sudo systemctl enable elasticsearch.service
```

Now, you can start elasticsearch:

```python
$ sudo systemctl start elasticsearch.service
```

At this point, `elasticsearch` will start everytime you reboot your system. 

To test your set up, try running `http://localhost:9200/` on your browser's url bar. You should see some JSON data dumped on your screen. Better still, on your terminal, try:

```python
$ curl localhost:9200
```

This completes the setup, installation and how to start the elasticsearch service. Now you can try running your commands on the terminal and everything should work fine.

Example Commands in your Python interpreter:

```python
>>> from datetime import datetime
>>> from elasticsearch import Elasticsearch
>>> es = Elasticsearch()
>>> es.indices.create(index='my-index', ignore=400)
{'acknowledged': True, 'shards_acknowledged': True, 'index': 'my_index'}
```