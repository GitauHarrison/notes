# PostgreSQL

Welcome to the PostgreSQL tutorial! This tutorial will walk you through the basics of using PostgreSQL, relational database concepts and the SQL language. The only prerequisite for you to follow along is basic ability to use a computer.

For reference, these are the topics we will cover in this tutorial:

- [PostgreSQL Overview](postgresql.md) (this article)
- [Install PostgreSQL and DBeaver](install_postgresql_and_dbeaver.md)
- [Getting Started with PostgreSQL](getting_started_with_postgresql.md)


## What is a Database

Before we can dive into PostgreSQL, let us begin by understanding what a database is. A database is an organized collection of structured information (processed) or data (raw) that is typically stored electronically, say in a computer. Anyone can work with a database. The tool used to control a database is referred to as a database management system (DBMS). A DBMS serves as an interface between the database and its end users or programs, allowing users to retrieve, update or manage how the information is organized an optimized. Examples of popular DBMS include PostreSQL, MySQL, Microsoft SQL Server, Oracle Database, SQLite among others


## Types of Databases

The main four types of database management systems are:

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

Today, with the growth of Internet of Things, businesses are able to collect massive amounts of data. Forward-thinking organizations are making use of databases to go beyond basic data storage and transactions to analyze vast quantities of data. As a result, they are able to make better business decisions and become more agile and scalable.


## History of PostgreSQL

PostgreSQL started as a project in 1986 at Berkeley Computer Science Department, University of California. It was originally referred to as Postgres whose inspiration came from the Ingres database. They main goal was simply to add support for multiple datatypes. In 1996, Postgres was renamed to PostgreSQL to clearly show support for the Structured Query Language (SQL). 