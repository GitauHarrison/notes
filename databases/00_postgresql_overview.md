# PostgreSQL Overview

Welcome to the PostgreSQL tutorial! This tutorial will walk you through the basics of using PostgreSQL, relational database concepts and the SQL language. The only prerequisite for you to follow along is basic ability to use a computer.

For reference, these are the topics we will be covering in the entire database tutorial:

1. [Postgres Overview](/databases/00_postgresql_overview.md) (this article)
2. [Install PostgreSQL](/databases/01_install_postgresql.md)
3. [Access PostgreSQL Using PSQL](/databases/access_postgresql/psql.md)
4. [Access PostgreSQL Using DBeaver](/databases/access_postgresql/dbeaver.md)
5. [How to Secure PostgreSQL](/databases/02_how_to_secure_postgresql.md)
6. [Getting Started With PostgreSQL](/databases/03_getting_started_with_postgresql.md)
7. [Sample Flask Project Using PostgreSQL](/databases/project_configure_postgres_in_flask.md)


## What is a Database

Before we can dive into PostgreSQL, let us begin by understanding what a database is. A database is an organized collection of structured information (processed) or data (raw) that is typically stored electronically, say in a computer. Anyone can work with a database. The tool used to control a database is referred to as a database management system (DBMS). A DBMS serves as an interface between the database and its end users or programs, allowing users to retrieve, update or manage how the information is organized and optimized. The DBMS acronym is sometimes extended to indicate the underlying database model. Examples of popular DBMS include PostreSQL, MySQL, Microsoft SQL Server, Oracle Database, SQLite among others.


## Types of Databases Models

The main four types of database models are:

1. Hierarchical database
2. Network database
3. Relational database
4. Object-Oriented database


### Hierarchical DBMS

Here, model data is organized in a tree-like structure, say from top - bottom or bottom - up. The data is represented using the parent-and-child relationship. You will find that a parent may have many children while a child may have only one parent.


### Network DBMS

The network model will typically allow a child to have many parents. The sole advantage of this DBMMS is to address the need to model more complex relationships such as the many-to-many type. Entities in this model can be accessed through several paths.


### Relational DBMS

It is the most widely used model, partly because it is the easiest. It prioritized normalizing data in rows and columns within a table. A relational database (RDB) has the ability to establish links or relationships between information by joining tables, which makes it easy to understand and gain insight about the relationship between various data points.


### Object-oriented DBMS

Here, data is created and modelled as objects. It offers support for classes of objects and it applies object-oriented core functionalities such as inheritance, polymorphism and so on. Programmers enjoy the consistency that comes with one programming environment because the database is integrated with the programming language and uses the same representation model.


## Why Use a Database

Today, with the growth of Internet of Things, businesses are able to collecting massive amounts of data. Forward-thinking organizations are making use of databases to go beyond basic data storage and transactions to analyze vast quantities of data. As a result, they are able to make better business decisions and become more agile and scalable.


## History of PostgreSQL

PostgreSQL started as a project in 1986 at Berkeley Computer Science Department, University of California. It was originally referred to as Postgres whose inspiration came from the Ingres database. They main goal was simply to add support for multiple datatypes. In 1996, Postgres was renamed to PostgreSQL to clearly show support for the Structured Query Language (SQL). 


## Advantages of Using PostgreSQL

One of the main reasons you would want to consider using PostgreSQL is its large set of features. It possess an incredibly large number of featueres related to performance, security, programming extensions, configurations among others. 

- For example, you can use a vast number of programming languages to write database functions. Example languages include Python, Java, SQL, JavaScript etc. 
- A host of data types are also supported by PostgreSQL. Priimitive datatypes such as Strings, Booleans, Numeric types are fully supported. Other datatypes that are supported include network addresses, geometric types, monetary types, multi-dimensional arrays among others.
- PostgreSQL allows you to define your own complex types so that you can align your data closely to your needs
- Full-text search feature is available in PostgreSQL. Search can be fine-tuned to match your expectations for relevancy.
- PostgreSQL has robust authentication, access control and privilege management systems suitable for organizations of any size. It helps define who can use the system and what each user is allowed to see or do.

Are you uncertain on why you should chose PostgreSQL coming from SQLite? Take some time to [review these arguments](https://www.twilio.com/blog/sqlite-postgresql-complicated).