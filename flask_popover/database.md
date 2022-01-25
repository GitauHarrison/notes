# Working With A Database

For your reference, these are the sections included in this tutorial:

1. [Section 1: Create a simple web application](/flask_popover.md#create-a-simple-application)
2. [Section 2: Web Forms](web_forms.md)
3. [Sectin 3: Working with Databases](database.md)
4. [Section 4: User Login](user_login.md)
5. [Section 5: User Posts](user_posts.md)
6. [Section 6: Implement Popover](popover.md)

When a user registers, we need to store their username, email and password in a database.  During subsequent visits. we will also need to retrieve a user's information from a database to authenticate them before they can access their account. Flask-sqlalchemy makes it possible to work with databases in Flask. We need to install this package in our virtual environment before we can create our database.

```python
(venv) $ pip3 install flask-sqlalchemy
```

I am going to use the SQLite database engine for the convinience of this tutorial. It is perfect for small applications, as each database is stored in a single file on a disk and there is no need to run a database server like MySQL and PostgreSQL.

### Configure Flask-SQLAlchemy

All our configurations are in the `config` module. We will update this module with two new items:

`config.py`: Database configuration
```python
import os

basedir = os.path.abspath(os.path.dirname(__file__))


class Config(object):
    # Form security
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'

    # Database
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

Flask-sqlalchemy extension takes the location of the database from `SQLALCHEMY_DATABASE_URI` environement variable. If this variable does not exist, I have provided a default database named `app.db` which will be located in the application's main directory.

Now we are ready to begin creating our application's database. Note that very time we make changes to the database, we need to run some _migrations_ to effect the changes. 
For example, if we decide that we want to add a new column to the `User` table to store the user's first name, we will need to create a new migration file. This new file will contain the new schema for the table.

The `flask_migrate` extension helps us do this. Ensure that you have installed it in your virtual environment.

```python
(venv) $ pip3 install flask-migrate
```

The database is going to be represented in the application by a database instance. This is done in `__init__.py` file.

`app/__init__.py`: Database instance
```python
# ...
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

# ...
db = SQLAlchemy(app)
migrate = Migrate(app, db)

from app import routes, models
```

The structure of the database will be defined in the `models` module.

### Database Models

The data that will be stored in the database will be represented by a collection of classes, usually called _models_. The ORM layer within SQLAlchemy will do the translations required to map objects created from these classes into rows in the database tables. Each model represents a single table in the database. In our case, we want to create a `User` model which will have columns such as `username`, `email` and `password`.


`app/models.py`: User model
```python
from app import db


class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))

    def __repr__(self):
        return f'User: {self.username}'
```
The `__repr__()` method is used to print the objects of this class.

### Create Migration

With the schema in place, we need to create this table and apply the changes to the database.

```python
(venv) $ flask db init

# Output

Creating directory /home/harry/software_development/python/practice_projects/start_flask_server/migrations ...  done
Creating directory /home/harry/software_development/python/practice_projects/start_flask_server/migrations/versions ...  done
Generating /home/harry/software_development/python/practice_projects/start_flask_server/migrations/script.py.mako ...  done
Generating /home/harry/software_development/python/practice_projects/start_flask_server/migrations/alembic.ini ...  done
Generating /home/harry/software_development/python/practice_projects/start_flask_server/migrations/README ...  done
Generating /home/harry/software_development/python/practice_projects/start_flask_server/migrations/env.py ...  done
Please edit configuration/connection/logging settings in '/home/harry/software_development/python/practice_projects/start_flask_server/migrations/alembic.ini' before proceeding.
```

This command creates a _migrations_ folder in the application's main directory. All database files will be stored in this folder. If you inspect this folder, you will notice that it comes with a _versions_ subfolder. All changes we make to the database will be recorded as "versions" and they will be stored in this folder.

_Note that the `flask` command relies on the `FLASK_APP` environment variable to locate the application. Ensure that you have set it in the `.flaskenv` file._ If you are not aware what this file is, kindly review the [start a flask server](https://github.com/GitauHarrison/notes/blob/master/start_flask_server.md) tutorial. In a nutshell, this file defines environment variables needed by the application as soon as the server is fired up.

Next, let us create our first database migration which will include a user's table that maps to our `User` model in the database.

```python
(venv) $ flask db migrate -m "user table"

# Output

INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'user'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_email' on '['email']'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_username' on '['username']'
  Generating /home/harry/software_development/python/practice_projects/start_flask_server/migrations/versions/ac423ee17380_user_table.py ...  done
```

Since this is the first migration, this command will add the entire `User` model to the migration script. Notice how Alembic has named this file `ac423ee17380_user_table.py`. This is the name of the migration.

The `flask-migrate` command does not make any changes to the database; it only generates the migration script. To apply these changes, we need to run the `flask db upgrade` command.

```python
(venv) $ flask db upgrade


# Output

INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> ac423ee17380, user table
```

The application we want to build will allow a user to post comments in the form of a chat conversation. The user will be the author of the comment. Each post will have the author's image, their username, the time they posted the comment, and the comment itself.

![Post](/images/flask_popover/post.png)

Let us expand the database to include a `Post` model and show the relationship it has with the `User` model.

`app/models.py`: Post model
```python
from app import db
from datetime import datetime


class User(db.Model):
    __table_name__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(64), index=True, unique=True)
    password_hash = db.Column(db.String(128))
    posts = db.relationship('Post', backref='author', lazy='dynamic') # < --- relationship

    def __repr__(self):
        return f'Body: {self.username}'


class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    body = db.Column(db.String(140))
    timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id')) # < --- author

    def __repr__(self):
        return f'Body: {self.body}'

```

The `user_id` field is initialized as a foreign key to the `user.id` field of the `User` model, referencing the `id` field of the `User` model. Technically, this is a one-to-many relationship where one user can author several posts in the application.

> It is an unfortunate inconsistency that in some instances such as in a `db.relationship()` call, the model is referenced by the model class, which typically starts with an uppercase character, while in other cases such as this `db.ForeignKey()` declaration, a model is given by its database table name, for which SQLAlchemy automatically uses lowercase characters and, for multi-word model names, snake case.

Since we have updates in the application models, we need to generate new database migrations.

```python
(venv) $ flask db migrate -m "post table"


# Output


INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'post'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_post_timestamp' on '['timestamp']'
  Generating /home/harry/software_development/python/practice_projects/start_flask_server/migrations/versions/3179c8a31797_post_table.py ...  done
```

This migration needs to be applied to the database:

```python
(venv) $ flask db upgrade


# Output

INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade ac423ee17380 -> 3179c8a31797, post table
```

### Shell Context

You can test the applicaton by running some commands on the terminal. Since we have a database, we can create a context of the application such that we have access to the database. Typically, on a normal Python interpreter, we will have to do the following:

```python
(venv) $ python3
Python 3.8.10 (default, Nov 26 2021, 20:14:08) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.

# Access configurations

>>> from app import app
>>> app.config['SECRET_KEY']
'you-will-never-guess'

# Access database

>>> from app import db
>>> db
<SQLAlchemy engine=sqlite:////home/harry/software_development/python/practice_projects/start_flask_server/app.db>

# Access User model

>>> from app.models import User
>>> user = User.query.all()
>>> user
[]
```
Everything will work fine but you have to explicitly import `app`, `db` and `User` model. If you try to run the above commands without properly importing the `app`, `db` and `User` model, you will get an error.

```python
(venv) $ python3
Python 3.8.10 (default, Nov 26 2021, 20:14:08)
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.

# Example of error

>>> import db
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ModuleNotFoundError: No module named 'db'
```

This is where the shell context comes in. It makes testing things out a lot easier. To begin, you have to make a shell context in your application's entry point. In the case of this tutorial, the entry point is `my_app.py`. Let us update this file to include the following:

`my_app.py`: Flask shell context
```python
from app import app, db
from app.models import User, Post


@app.shell_context_processor
def make_shell_context():
    return dict(
        db=db,
        User=User,
        Post=Post
    )
```
The `app.shell_context_processor` decorator is a function that returns a dictionary of variables (and not a list) that will be available in the shell session every time `flask shell` is run.

Now, to easily access the database, you can run the following command in the terminal:

```python
(venv) $ flask shell

Python 3.8.10 (default, Nov 26 2021, 20:14:08) 
[GCC 9.3.0] on linux
App: app [development]
Instance: /home/harry/software_development/python/practice_projects/start_flask_server/instance

# db object

>>> db
<SQLAlchemy engine=sqlite:////home/harry/software_development/python/practice_projects/start_flask_server/app.db>

# User model

>>> user = User.query.all()
>>> user
[]
```