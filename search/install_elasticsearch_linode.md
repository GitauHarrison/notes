# Install And Configure ElasticSearch In A Live Linux Server

Is it any different configuring your app on [Linode](https://linode.gvw92c.net/15oBBg) compared to your localhost? This article will give you the confidence to properly set up your live Linux server to ensure that Elasticsearch runs on your app. If you would like to check out other articles on Elasticsearch, you can click on any of the links below:

- [Install Elasticsearch in Ubuntu 20.04 In Localhost](install_elasticsearch_localhost.md)
- [Install And Configure ElasticSearch In A Live Linux Server](install_elasticsearch_linode.md) (this article)
- [Implement Search Functionality In Your Flask App](implement_elasticseach.md)

The good news is that there is not much of a difference between setting up your live Linux server and your localhost. I recommend that you check out the [installation guide for setting up Elasticsearch in your localhost](install_elasticsearch_localhost.md) before proceeding. 

Once you have installed Elasticsearch, you would want to update its configuration file to include your network host.

```python
$ sudo nano /etc/elasticsearch/elasticsearch.yml

# Network settings

# ---------------------------------- Network -------------------------------
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: 127.0.0.1
#
# Set a custom port for HTTP:
#
http.port: 9200
```

The command above will open the `elasticsearch.yml` file in `nano`. Change `127.0.0.1` to the IP address of your application in your Linux server. For instance, you can change it to:

```python
network.host: 12.123.123.12
```

Elasticsearch is now enabled to allow connections from your IP address and to also listen to port `9200`. Additionally, if you would like to update the list of your discovery hosts, you can list them here:

```python
$ sudo nano /etc/elasticsearch/elasticsearch.yml

# Discovery

# -------------------------------- Discovery -----------------------------
# Pass an initial list of hosts to perform discovery when this node 
# is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
discovery.seed_hosts: ["12.123.123.12"]
```

Above, I have added my example IP address to the list of seed hosts. At this stage, I can start the Elasticsearch service and test my setup and installation:

```python
(venv)$ curl 12.123.123.12:9200
```

You should be able to see JSON-formatted data displayed in your terminal. Alternatively, paste http://12.123.123.12:9200/ to your browser's address bar. JSON data will be dumped on your screen. You can check out the [testing](install_elasticsearch_localhost.md#testing) section to try out the service given the new updates. Besides, you can also implement [basic security](install_elasticsearch_localhost.md#basic-security) for the service as seen the section.

Remember to update `ELASTICSEARCH_URL` in `/etc/your_app_config.json` file (this file lists all your secret environment variables):

```python
(venv)$ sudo nano /etc/your_app_config.json

# Edit as needed
{
    "ELASTICSEARCH_URL": "ELASTICSEARCH_URL=http://usernameWithWriteAccess:password@localhost:9200"
}
```

Notice that I am using `http` and not `https`. At this stage, I have not been able to enable the use of `https` since using it throws the SSL Error. To ensure that the server is working correctly, you can issue a `Model.reindex()` command in your shell to update the indices used by the service before testing the search feature.