# Implement Search Functionality In Your Flask App

[Elasticsearch](https://www.elastic.co/what-is/elasticsearch) may mean different things to different people, depending on their level of familiarity with the technology. At its core, Elasticsearch is largely a distributed open-source search and analytics engine for all types of data built on Apache Lucene and developed in Java, using NoSQL, meaning it stores data in unstructured way and that we cannot use SQL to query it. It is at the heart of the ELK Stack (Elasticsearch, Kibana and Logtash) such that it has become synonymous with the name of the stack itself. 

The following are other sections in the Elasticsearch series. You can click on any of the links to learn more:

- [Install Elasticsearch in Ubuntu 20.04 In Localhost](install_elasticsearch_localhost.md)
- [Install And Configure ElasticSearch In A Live Linux Server](install_elasticsearch_linode.md)
- [Implement Search Functionality In Your Flask App](implement_elasticseach.md) (this article)

As you learnt in the [installation guide](install_elasticsearch_localhost.md), support for full text search is not standardized like relational databases are. Also, the fact that SQLAlchemy does not natively support the search functionality, we have to contend with the fact that we need to manually do this ourselves.

### Table of Contents

- [Overview](#overview)
    - [Documents](#documents)
    - [Indices](#indices)
    - [Inverted index](#inverted-index)
    - [Cluster](#cluster)
    - [Nodes](#nodes)
    - [Shards](#shards)
- [Build A Simple Flask App](#build-a-simple-flask-app)


## Overview

From the [Testing](install_elasticsearch_localhost.md#testing) section in the Localhost installation guide, you saw how to index a document and get corresponding JSON data back. However, you may be curious to understand how everything actually works. The [Elasticsearch documenation](https://www.elastic.co/what-is/elasticsearch) does a good job of trying to explain how indexing and parsing works. Let us cover some basic concepts of how it organizes data and its backend components.

### Documents

They are the basic unit of JSON data that can be indexed in Elasticsearch. If you have interacted with rows in relational databases, a document is more less like a row object. Each document has a unique ID and a given data type.

```python
body={'test': 'this is the first test'}
```

### Indices

An index is used to classify documents that have the same characteristics. It is what you would typically use to query against Elasticsearch to retrieve logically-related data. Say you have a blog, you can have an index for Articles, another for Comments et cetera. Each index has a name.

```python
index='test_index'
```

### Inverted Index

Elasticsearch uses a data structure called an inverted index that supports very fast full-text searches. An inverted index lists every unique word that appears in any document and identifies all of the documents each word occurs in. Let us look at this example to best understand what it is.

![Inverted index](/images/elasticsearch/inverted_index.png)

The inverted index does not really store items directly, but it instead splits each documents up into individual search terms and then maps each term to the documents they occur in. By using inverted indices, Elasticsearch can quickly find best matches for full-text searches from even large data sets.

When performing full-text searches, we are actually querying an inverted index and not the JSON documents that we defined when indexing the documents. A cluster can have at least one inverted index. That’s because there will be an inverted index for each full-text field per index. So if you have an index containing documents that contain five full-text fields, you will have five inverted indices.

### Cluster

A cluster is a group of one or more node instances that are connected together. The effectiveness of Elasticsearch is in the distribution of tasks to each node in the cluster.

### Nodes

A node is a single server that participates in the indexing and search capabilities of a cluster. 

- **Master node**: Responsible for cluster-wide operations such as creating and deleting an index and adding or remove a node
- **Data node**: Stores data and executes data related operations such as search and aggregation
- **Client node**: Forwards cluster requests to **master** and **data** nodes

### Shards

It is possible to subdivide an index into multiple pieces called shards. Each shard is a full-functional and independent index that can be hosted on any node. By distributing documents across many shards and distributing those shards across multiple nodes, Elasticsearch is able to protect against hardware failures and increase query capacities as nodes are added to a cluster. 

