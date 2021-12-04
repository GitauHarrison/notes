# Moderate User Comments in Flask

As an administrator of a website, one of the things you would really want to do is to moderate what users say publicly to each other. This is a very common task, and having it implemented in your Flask website can really help sanitize the contents. In this tutorial, I will show you how you can implement this feature.

## Create A Simple Flask Application

We will need a simple application to test the Flask server. I have created one already [here](https://github.com/GitauHarrison/starting-a-flask-server). We will build on it and add a comment moderation feature. If you would like to know how the application in the link was built, you can read the [start a flask server tutorial](start_flask_server.md).

## Application Configurations

To receive comments in our application, we need to provided forms that users can fill. The information that users pass through the forms will be used to populate our comments section. So, how can we make forms in Flask?

Flask provides the `flask-wtf` extension, a wrapper around the [WTForms](https://wtforms.readthedocs.io/en/3.0.x/) package. Let us install it in our virtual environment:

```python
(comment_moderation)$ pip3 install flask-wtf
```

As our application grows, certain configurations will be needed. At the moment, we have had no need to use any configuration in our application. Most Flask applications expect certain configurations to be set. One such configuration is the `SECRET_KEY`. It is a variable whose value is a cryptographic key useful in the generation of signatures and tokens. Flask-WTF uses it to protect web forms against a nasty attack called [Cross Site Request Forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) or CSRF (pronounced as 'sea-surf'). This value, as the name suggests, is meant to be a secret.

If you look carefully, I have a module called `config` in the top-level directory. Following the principle of _separtion of concerns_, all the configurations that our application will need will be added here.

`config.py: Secret key configuration`
```python
import os


class Config(object):
    # Form
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'

```

I am sourcing the value of `SECRET_KEY` from an environment variable. If the value is missing, I have provided a safe fallback by hardcoding a string. The environment variable is a secret and should not be added to version control. I will create a new file called `.env` which will have the actual value of `SECRET_KEY`. `.gitigonore` file we already have ignores this file from version control, so it is safe to create it in the top-level directory.

```python
(comment_moderation)$ touch .env
```

I will use the following method to generate the value of the SECRET_KEY:

```python
(comment_moderation)$ python3 -c 'import os; print(os.urandom(16))'

# Random Output
b'\x1eh\xfcIWC\x91\xd7\xb3\xfd\x02dK\xe0\xb5z'
```

Hard to guess, right? I will add this value to my environment variable.

`.env: Add secret configuration keys`
```python
SECRET_KEY=b'\x1eh\xfcIWC\x91\xd7\xb3\xfd\x02dK\xe0\xb5z'
```

With the variable set, we can now update our application instance to read and apply our configurations.

`__init__.py: Regiser the config module in application instance`
```python
from flask import Flask
from flask_bootstrap import Bootstrap
from config import Config # < ------------ update

app = Flask(__name__)
app.config.from_object(Config) # < ------------ update

boostrap = Bootstrap(app)

from app import routes, errors

```

I have began by importing the `Config` class from the top-level directory. I have used the `app.config.from_object(Config)` to create an instance of our `Config` class. The lower name `config` comes from python to indicate the `config` module whereas the upper case is the actual `Config` class.

## Flask Web Forms

Flask-WTF uses Python classes to represent web forms. The form classes define the variable fields we would like to have in our forms.

### Define Comment Form

Let us begin by creating a simple comment form. The information we want from a user of our application are their username, their email addresses and the content they would like to share on our application. We will begin by creating an empty `forms` module in the `app/` sub-directory.

```python
(comment_moderation)$ touch app/forms.py 
```

Let us now create the class that defines all the fields we want in our comments form.

`forms.py: Create a user comment form`
```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField, TextAreaField
from wtforms.validators import DataRequired, Email


class CommentForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    email = StringField('Email', validators=[DataRequired(), Email()])
    comment = TextAreaField('Comment', validators=[DataRequired()])
    submit = SubmitField('Post')

```

`flask-wtf` has a base class called `FlaskForm` which is needed. The variable fields are defined using `StringField`, `TextAreaField` and the `SubmitField` from `wtforms`. To ensure that the user does not submit empty data, we attach `DataRequired`. This will prevent them from clicking the `Post` button if any of the fields above are empty. The ```Email``` validator will ensure that the email address is valid.

You will need to install `email-validator` to use the ```Email``` validator.

```python
(comment_moderation)$ pip3 install email-validator
```

### Display Comment Form

The next step is to display this form in our templates. We will use the `index.html` file to display our comment form. 

`index.html`
```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %} <!-- Update -->

{% block app_context %}
    <div class="row">
        <div class="col-md-12">
            <h1>Comment Moderation</h1>
        </div>  
    </div>
    <div class="row">
        <div class="col-md-6">
            {{ wtf.quick_form(form) }} <!-- Update -->
        </div>
    </div>
{% endblock %}
```

I have used Bootstrap to quicky display my comment form, though you can manually display the same form using raw HTML. 

With the form ready to be displayed, we will now update our `index()` view function to render it.

`routes.py: Render the comments form`

```python
from app import app
from flask import render_template
from app.forms import CommentForm # < ---- update


@app.route('/')
@app.route('/index')
def index():
    form = CommentForm() # < ---- update
    return render_template('index.html', form=form) # < ---- update

```

Start your flask server in the terminal by running the command `flask run`. Navigate to http://127.0.0.1:5000/ to see the comments form displayed.

![Comment Moderation Form](images/comment_moderation/comment_moderation_form.png)

## Working with a Database

After a user clicks the `Post` button, we will need to store the data in a database. This datat will persist even if we close the application. The database will also allow us to retrieve all the data about our users. 

Since this application is small in nature, we will use a SQLite database. That does not mean that as the project grows and becomes bigger SQLite will not be a suitable choice. It is still a great choice for bigger applications. Typically, SQLite stores data in a file on the disk, and there is no need to run a datbase server like MySQL and PostreSQL.

Flask provides the `Flask-SQLAlchemy` extension to help us manage our database. It is a Flask-friendly wrapper around the popular SQLAlchemy package. As an [Object Relational Mapper](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping), SQLAlchemy is a Python library that allows us to manage our database using classes, objects and methods instead of tables and SQL. The aim of an ORM is to translate these high level operations into database commands.

To install the `Flask-SQLAlchemy` extension, run the following command in the terminal:

```python
(comment_moderation)$ pip3 install flask-sqlalchemy
```

### Database Migrations

Continued use of a database may result in the need to modify the original database schema. We may need to add a new field in the database to now collect users' email addressing, data that originally did not exist. So, when the structure of the database changes, we need to update the database schema by _migrating_ to the modified schema.

We will use `flask-migrate` to perform this migration. Install it in the virtual environment by running the following command in the terminal:

```python
(comment_moderation)$ pip3 install flask-migrate
```


### Database Configuration

SQLite, being the most convinient choice in developing small applications, expects certain configurations from `Flask-SQLAlchemy`. We first provide the location of the database in the application through `SQLALCHEMY_DATABASE_URI` configuration variable. This variable will source its value from the `DATABASE_URL` environment variable.

`config.py: Database Configuration`
```python
import os
basedir = os.path.abspath(os.path.dirname(__file__)) # < ---- update


class Config(object):
    # Form
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'

    # Database
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(os.path.dirname(__file__), 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

If `DATABASE_URL` does not exist, then I have provided a fallback value where I am configuring a database called `app.db` in the top-level directory. The `SQLALCHEMY_TRACK_MODIFICATIONS` configuration option is set to `False` to disable a feature of Flask-SQLAlchemy that tracks every modifications to the database.

### Database Instance

The database will be referenced through a database instance. We will create a `db` variable that will be used to access the database.

`__init__.py: Database Instance`
```python
from flask import Flask
from flask_bootstrap import Bootstrap
from config import Config
from flask_sqlalchemy import SQLAlchemy # < ---- update
from flask_migrate import Migrate # < ---- update

app = Flask(__name__)
app.config.from_object(Config)

boostrap = Bootstrap(app)
db = SQLAlchemy(app) # < ---- update
migrate = Migrate(app, db) # < ---- update

from app import routes, errors, models # < ---- update

```

### Database Models

Our model will contain columns that will store a user's name, email address and comment. All these data are of type `VARCHAR` (in database jargon) and are limited to a specified number of characters. They are basically strings.

Let is create a new module called `models.py` that will contain our database models.

```python
(comment_moderation)$ touch  app/models.py
```

The translation of our database into code will look like this:

`models.py: Comment database Schema`
```python
from app import db
from datetime import datetime


class UserComment(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True)
    email = db.Column(db.String(120), index=True)
    content = db.Column(db.String(255))
    timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))

    def __repr__(self):
        return 'Comment: {}'.format(self.content)

```

I have added a `timestamp` column to the schema of the `Comment` model. This column will be used to sort the comments by their timestamps. Note how I carefully use `utc.now` and not `utc.now()`. I am passing the function itself and not the result of calling it.

### Database Migration

With every change we might make to the database schema, we would like the flexibility of applying the new changes and work with the modified schema. Also, if we change our mind about the database structure, we should be able to easily roll back to the previous version. As seemingly difficult as it is, we can use the `flask-migrate` extension to do this. `Alembic` maintains a history of the changes that have been made to the database in a _migrations repository_. 

To create a migrations repository, run the following command in the terminal:

```python
(comment_moderation)$ flask db init

# Output 
  Creating directory /home/harry/software_development/python/practice_projects/comment_moderation/migrations ...  done
  Creating directory /home/harry/software_development/python/practice_projects/comment_moderation/migrations/versions ...  done
  Generating /home/harry/software_development/python/practice_projects/comment_moderation/migrations/script.py.mako ...  done
  Generating /home/harry/software_development/python/practice_projects/comment_moderation/migrations/alembic.ini ...  done
  Generating /home/harry/software_development/python/practice_projects/comment_moderation/migrations/README ...  done
  Generating /home/harry/software_development/python/practice_projects/comment_moderation/migrations/env.py ...  done
  Please edit configuration/connection/logging settings in '/home/harry/software_development/python/practice_projects/comment_moderation/migrations/alembic.ini' before proceeding.
```

Remember that the `flask` command relies on the environment variable `FLASK_APP` to determine where our applciation is located. In the top-level directory, you should be able to see a _migrations repository_ which has a _versions_ subdirectory. This folder and all its files are now part of our applications, and should be committed our version control along with all other code.

To create our first migrations script, which will include the comments from our application users, run the following command in the terminal:

```python
(comment_moderation)$ flask db migrate -m "user comment table"

# Output

INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'user_comment'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_comment_email' on '['email']'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_comment_timestamp' on '['timestamp']'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_comment_username' on '['username']'
INFO  [alembic.autogenerate.compare] Detected removed index 'ix_comment_timestamp' on 'comment'
INFO  [alembic.autogenerate.compare] Detected removed table 'comment'
  Generating /home/harry/software_development/python/practice_projects/comment_moderation/migrations/versions/5fc3f28ab1bc_user_comment_table.py ...  done
```

The `-m` option is used to specify a message that will be included in the migration history. If you check the _versions_ subdirectory, you will notice that we have a new file called `XXX_user_comment_table.py`. This file contains two important functions: `upgrade()` and `downgrade()`. The `upgrade()` function will be called when the migration is applied. The `downgrade()` function will be called when the migration is rolled back.

To apply these changes to the database, run the following command in the terminal:

```python
(comment_moderation)$ flask db upgrade

# Output
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade -> 5fc3f28ab1bc, user comment table
```

You will add the migration script to source control and commit it. 

If at any one point you would like to roll back the changes, you can run the `flask db downgrade` command, though this is less likely in a production system.

## Database Relationships

At the moment, we, the creators, know that the comments are posted by users. However, our application does not know this yet. We will need to add a new model called `User`. This model will contain all user information such as username and email address.

`models.py: User database Schema`
```python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)

    def __repr__(self):
        return 'User: {}'.format(self.username)
```

A user of our application will be able to post comments. Our application will allow this user to post more than one comment. At the end of the day, when we query our database, we will find that one user can have many comments, and many comments can belong to one user.

In relational databases, this kind of relationship is referred to as _one to many_ relationship or _many to one_ relationship. We need to establish a link between our `Comment` model and the `User` model. Once a link has been established, our database can answer queries about this link.

Let us expand our models to include a link between the two.

`models.py: Database relationships`
```python
from app import db
from datetime import datetime


class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    comments = db.relationship('Comment', backref='author', lazy='dynamic') 
    # backref is used to establish a link between the two models

    def __repr__(self):
        return 'User: {}'.format(self.username)


class Comment(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.String(255))
    timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id')) # This is the link to the User model

    def __repr__(self):
        return 'Comment: {}'.format(self.content)

```

The `user_id` field is initialized to `user.id` as a foreign key. This means that it references and `id` value in the `User` model. It is an inconsistency that the foreign key is initialized in lower case letters whereas the model is actually upper case. 

In the `User` model, we have used the `db.relationship` function to establish a link to the `Comment` model. It is conviniently used on the 'one' side in a _one to many_ relationship to help answer the query of one user having many comments. It can also be placed in the 'many' side where many comments can be traced to one user.

 This function takes two arguments:

* `backref`: defines the field to be added to the many side of the relationship and it will be used to refer to the 'one' side. For example, `comment.author` will refer to the 'one' side of the relationship.

* `lazy`: determines how the related objects get loaded when querying through relationships

With all these updates, we are ready to create and apply the changes to the database. Run the following commands in the terminal:

```python
(comment_moderation)$ flask db migrate -m "user table"
(comment_moderation)$ flask db upgrade
```

## Update the Database

Since all user information is now stored in our `User` database which is linked to the `Comment` database, we can now query the database to display all comments. Every time a request to render our index page content is made, we will query the database to display all comments.

The first step is to update our database every time new data comes through the Comments Form. 

`routes.py: Update the database`
```python
from flask.helpers import url_for
from werkzeug.utils import redirect
from app import app, db
from flask import render_template, flash, redirect, url_for
from app.forms import CommentForm
from app.models import UserComment # < ----- update


@app.route('/', methods=['GET', 'POST'])
@app.route('/index', methods=['GET', 'POST'])
def index():
    form = CommentForm()
    if form.validate_on_submit():
        user = UserComment(
            username=form.username.data,
            email=form.email.data,
            content=form.comment.data
        )
        db.session.add(user)
        db.session.commit()
        flash('Your comment has been posted!')
        return redirect(url_for('index'))
    return render_template('index.html', form=form)

```

`form.validate_on_submit()` is used to validate the form. If the form is valid, the `user` object is created and added to the database. Otherwise, the index page will be displayed. I have added a flash message to notify the user that their comment has been posted. To see the message, we need to update our `base.html` template.

`base.html: Show flash message`
```html
<!-- Contents of all our pages will go here -->
{% block content %}
    <div class="container">

        {% with messages = get_flashed_messages() %}
            {% if messages %}
                {% for message in messages %}
                    <div class="alert alert-success" role="alert">
                        {{ message }}
                    </div>
                {% endfor %}
            {% endif %}
        {% endwith %}
        
        {% block app_context %}{% endblock %}
    </div>
{% endblock %}
```

Try post a comment. If all goes well, then you should be able to see the flash message.

![Flash Message](images/comment_moderation/flash_message.png)

