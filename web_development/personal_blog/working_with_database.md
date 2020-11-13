The completed project used in this article can be referenced [here](https://github.com/GitauHarrison/personal-blog-tutorial-project/commit/12693f33c3fdfb35258ac9f3e76862fd558a25bc).

In this chapter, I will introduce the important concept of databases. We will use a database to store all user information that we get through our forms. Data in a database is typically persistent and can be retrieved when needed. What I mean is when we close our application and reopen it, we do not have to worry that any user data will be lost. Our database safely stores these data.

### Flask Databases

Flask supports a number of databases with no particular preference. It is intentionally unopinionated, which provides us with the freedom to choose whichever database we want to use. Despite the support for databases, Flask does not support them natively.

Databases can be group into two broad categries: _relational databases_ and those that are not relational. Those that do not follow any relational model are called _NoSQL_, meaning they do not implement the relational query language [SQL](https://en.wikipedia.org/wiki/SQL). For applications that have structured data such as a list of users, or blog posts, relational databases are much better and more recommended. NoSQL databases do much better with data that is less structured. For this reason, we will learn how to implement relational databases for our blog.

We will use [SQLAlchemy](https://www.sqlalchemy.org/). It is an [Object Relational Mapper](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping) where classes can be mapped to a database to develop a clean database schema. SQLAlchemy supports a number of popular database engines, including MySQL, PostrgeSQL and SLQLite. This is super convinient because as you develop your application, you can use a serverless database such as SQLite and upon production, you can easily switch to a more robust database engine such as Postgre without haveing to change your application. 

Flask has a couple of extensions that help us work with databases. In this chapter, I will introduce two to you:

* [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/)
* [Flask-migrate](https://flask-migrate.readthedocs.io/en/latest/)

When a database is created, it will not stay empty. Data will fill it. This means that our database will need to be constantly updated with new changes. So, with a relational database, when this change takes place, we need to migrate our current schema to the new modified structure. 

Let us go ahead and install both extensions in our virtual environment:

```python
$ pip3 install flask-sqlachemy
$ pip3 install flask-migrate
```

SQLite does not need to run any server. Additionally, since our application is still very small, this database engine is most appropriate for our use. For these two reasons, we will use it in our application. Let us add some configurations to our `config` module:

```python
import os
basedir = os.path.abspath(os.path.dirname(__file__))

class Config(object):
    #...
    SQLALCHEMY_DATABAE_URI=os.environ.get('DATABAE_URL') or 'sqlite:///' + os.path.join(basedir, 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS=Flase
```

We are taking the location of the database from the environment variable DATABAE_URL. If this does not exist, we provide a safety net where we configure our database called `app.db` int the main directory of our application, which is stored in the variable `basedir`.

Every time a change is made to the database, we will get a signal to our application. We do not need this signals every time we make changes to our database for now. So, I have set `SQLALCHEMY_TRACK_MODIFICATIONS` to `Flase`.

As done earlier, we need to register our database to the application instance.

app/__init__.py: Initialize database in application
```python
from flask import Flask
from config import Config
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app = Flask(__name__)
app.config.from_object(Config)
db = SQLAlchemy(app)
migrate = Migrate(app, db)

from app import routes, models
```

We have imported our models into the application instance.

