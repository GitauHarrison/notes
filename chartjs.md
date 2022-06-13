# Visualize Data in Your Flask App Using ChartJS

You can tell very powerful stories using data. 

![ChartJS Demo](images/data_visualization/chartjs/chartjs_demo.png)

Should you want to 'see' or understand deeply the data generated in your application, there are a handful of libraries that can help. One of them is ChartJS, the focus of this article. [ChartJs](https://www.chartjs.org/docs/latest/) is a free JavaScript library for creating charts in the browser (HTML-based charts). It is very easy to use, though basic understanding of JavaScript is required.

## What We Will Do?

We will build a simple flask application for a class teacher to record the mean scores of all the subjects in a class throughout a 3-term year. Example data we will need may include the following:

* **Subjects**: Math, English, Science, History, and Computer Science
* **Mean Scores**: [70, 80, 90, 100, 95]
* **Term**: [1, 2, 3]

We will assume that the teacher has already calculated the mean scores of each subject, so we don't need to stress about it.

## Table of Contents

1. [Create a simple flask app](#create-a-simple-flask-app)
2. [Add web forms to the app](#add-web-forms-to-the-app)
3. [Add data to the app](#add-data-to-the-app)
4. Enable user login
5. Display the mean scores of each subject per term
6. Display the mean scores of each subject in a line chart during each term

### Create a simple flask app

I have already created a simple flask app for you. You can refer to it in [this Github repository](https://github.com/GitauHarrison/starting-a-flask-server). If you are new to flask and would like to start with the basics, check out the [starting-a-flask-server
](https://github.com/GitauHarrison/notes/blob/master/start_flask_server.md) tutorial.

### Add web forms to the app

Flask provides the [wtf](https://flask-wtf.readthedocs.io/en/latest/) library for creating web forms. This library is used to create forms that are used to collect data from a user, in our case, it will be the classroom teacher. To create a form, we will need to:

- Create a _forms_ module to define all the forms we will need (login, register and meanscore)
- Create templates for each form (login.html, register.html, meanscore.html)
- Create a _views_ module to render our forms

In the terminal, install `flask-wtf` in your virtual environment.

```python
(venv)$ pip install flask-wtf
```

#### Forms module

This module (`forms.py`) will contain all the forms we will need. It is located in the root folder of the application. We will use classes to define all the fields we want in the forms.

`forms.py: Define login and register forms`
```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField, TextAreaField, PasswordField, BooleanField
from wtforms.validators import DataRequired, Length, Email, Regexp, EqualTo


class LoginForm(FlaskForm):
    email = StringField('Email', validators=[DataRequired(), Length(1, 64), Email()])
    password = PasswordField('Password', validators=[DataRequired()])
    remember_me = BooleanField('Keep me logged in')
    submit = SubmitField('Log In')


class RegistrationForm(FlaskForm):
    email = StringField('Email', validators=[DataRequired(), Length(1, 64), Email()])
    username = StringField('Username', validators=[
        DataRequired(), Length(1, 64),
        Regexp('^[A-Za-z][A-Za-z0-9_.]*$', 0,
               'Usernames must have only letters, numbers, dots or '
               'underscores')])
    password = PasswordField('Password', validators=[
        DataRequired(), EqualTo('password2', message='Passwords must match.')])
    password2 = PasswordField('Confirm password', validators=[DataRequired()])
    submit = SubmitField('Register')
```

Here, we want to capture a teacher's email, username and their password to protect their account. A few arguments are used to validate a teacher's credentials. For example:

- `DataRequred` ensures that the field is not empty. 
- `Length` ensures that the field is at least 1 character long and at most 64 characters long. 
- `Email` ensures that the field has a valid email address. 
- `Regexp` ensures that the field only contains letters, numbers, dots or underscores. 
- `EqualTo` ensures that the password fields match.

`Email()` requires that we install `email-validator`, so remember to do so in the terminal (`pip3 install email-validator`).

![Web forms](images/data_visualization/chartjs/web_forms.gif)

For the web forms to work, we will need to configure our application such that we provide a SECRET_KEY to help protect our forms from CSRF attacks. 

`config.py: Configure the SECRET_KEY`
```python
import os


class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'hard to guess string'

```

This variable will be sourced from the environment. In the event the variable does not exist in the environment, we will use a hard-coded string. This is a safe fall-back to ensure that our application does not crash.

### Add data to the app

Just like web forms, we will also use classes to create a database model to store a user's data. `flask-sqlalchemy`, a flask-friendly wrapper to SQLAlchemy, will translate the classes objects and methods to tables and SQL. Intentionally, I will use the SQLite database since it does not require a server to run.

To install Flask-SQLAlchemy, run:

```python
(venv)$ pip install flask-sqlalchemy
```

Every time we create a new model, we need apply those changes to our database. Same goes to when we update the structure/schema of our database. This action is called `migrating` the database. Database migrations are easily handled by `flask-migrate`.

To install Flask-Migrate, run:

```python
(venv)$ pip install flask-migrate
```

We need to further configure the application to accommodate the database. This is done in the `config.py` file.

`config.py: Database configuration`
```python
import os

basedir = os.path.abspath(os.path.dirname(__file__))


class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'hard to guess string'
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(os.path.dirname(__file__), 'data.sqlite')
    SQLALCHEMY_TRACK_MODIFICATIONS = False

```

Once again, the database URL is sourced from the environment. Should the `DATABASE_URL` variable not exist, we default to using the file-based database called _data.sqlite_ which will be located in the root directory of the application. `SQLALCHEMY_TRACK_MODIFICATIONS` is set to `False` to prevent SQLAlchemy from tracking changes to the database. 

That is all for the configuration! We now should create instances of the database and the migrations in the app. This is normally done in `__init__.py`.

`app/__init__.py: Initialize the database and migrations`

```python
from flask import Flask
from flask_bootstrap import Bootstrap
from config import Config
from flask_migrate import Migrate # < --- new update
from flask_sqlalchemy import SQLAlchemy


app = Flask(__name__)
app.config.from_object(Config)


bootstrap = Bootstrap(app)
db = SQLAlchemy(app) # < --- new update
migrate = Migrate(app, db) # < --- new update

from app import routes, models
```

We can now create a `User` table in `models.py` within `app/` folder.

`app/models.py: Create the User table`
```python
from app import db
from werkzeug.security import generate_password_hash, check_password_hash


class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)
    email = db.Column(db.String(64), unique=True, index=True)
    password_hash = db.Column(db.String(128))

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        return check_password_hash(self.password_hash, password)
```

We have created three columns in our table. The columns will store every user's username, email and password. The `password_hash` column will store the hashed version of the user's password. This is an additional security measure to secure users' data in the event the database is compromised.

This is a new structure. We need to apply these changes and create a brand new database table. To do so, we will run the commands below:

```python
(venv)$ flask db init # creates migrations folder
(venv)$ flask db migrate -m "user table" # creates a new migration file
(venv)$ flask db upgrade # applies the new migration file
```

You should be able to see a new __data.sqlite__ file in the root directory of the application.