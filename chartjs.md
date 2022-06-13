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
4. [Enable user login](#enable-user-login)
5. [Improve user experience](#improve-user-experience)
6. Display the mean scores of each subject per term
7. Display the mean scores of each subject in a line chart during each term

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

### Add a user to the database

To update the database with new data, our application will use the registration form to create a new user. To do this, let us add a few logic to the `routes.py` file.

`app/routes.py: Register a new user`
```python
from app import db
from flask import url_for, flash, redirect, render_template
from app.models import User


@app.route('/register', methods=['GET', 'POST'])
def register():
    form = RegistrationForm()
    if form.validate_on_submit():
        user = User(email=form.email.data, username=form.username.data)
        user.set_password(form.password.data)
        db.session.add(user)
        db.session.commit()
        flash('Successfully registered! You can now log in.')
        return redirect(url_for('login'))
    return render_template('register.html', title='Register', form=form)
```

We store a user's information in a variable called `user`. Of interest her is to note that we do not pass a user's password data into this variable. Instead, we hash the password using the helper function `set_password()` as seen in `models.py`. Thereafter, we add the user to the database.

### Enable user login

After a user has registered, we redirect them to the login page. This is a good way to ensure that the user has successfully registered. To add this functionality, we will need to add some login logic to the `login()` view functions.

User login in flask is easily handled by `flask-login` package. We need to first install it and then create an instance of it in the app.

```python
(venv)$ pip install flask-login
```

`app/__init__.py: Initialize the login manager`
```python
# ...
# from flask_login import LoginManager

# ...
login = LoginManager()
```

Since our database has no clue about user sessions, we need to modily it slightly. 

`app/models.py: Modify the User table`
```python
# ...
from app import login
from flask_login import UserMixin


@login.user_loader
def load_user(id):
    return User.query.get(int(id))


class User(UserMixin, db.Model):
    # ...
```

We first ensure that a particular user is accessed using their _id_ and returned during the login process. Flask-Login provides certain properties and methods to work with a database user. For example, the _is_authenticated_ and _is_anonymous_. To implement these properties, the _UserMixin_ class from flask-login is used.

We can now update our `login()` view function to use the `login_user()` method from flask-login.

`app/routes.py: Login a user`

```python
from flask_login import login_user


@app.route('/login', methods=['GET', 'POST'])
def login():
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first()
        if user is None or not user.check_password(form.password.data):
            flash('Invalid username or password')
            return redirect(url_for('login'))
        login_user(user, remember=form.remember_me.data)
        return redirect(url_for('index'))
    return render_template('login.html', title='Login', form=form)
```

To show this change, it would be nice and necessary to change the _Login_ link in the navigation bar to _Logout_. This will give a user the chance to protect their account by logging out whenever they are done using it. We will implement this by first creating a new `Logout` view function.

`app/routes.py: Logout a user`

```python
@app.route('/logout')
def logout():
    logout_user()
    return redirect(url_for('login'))
```

We will then update our _base.html_ template to include a conditional statement to determine the state of the user.

`app/templates/base.html: Log user out`

```html
<! -- ... -- >

{% block navbar %}
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">            
            <ul class="nav navbar-nav navbar-right">
                {% if current_user.is_authenticated %}
                    <li><a href="{{ url_for('logout') }}">Logout</a></li>
                {% else %}
                    <li><a href=" {{ url_for('login') }} ">Login</a></li>
                {% endif %}
            </ul>                       
        </div>
{% endblock %}

<! -- ... -- >
```

A quick reload will reveal the state of a user. If the user is logged in, then the link will change to _logout_. Upon logout, the user will be redirected to the login page.

### Improve user experience

__Flash Message__

To improve the user experience, we will add a flash message every time a user registers or logs into their account. This can be achieved by adding the following to the _base.html_ file.

`app/templates/base.html: Flash message`

```html
<! -- ... -- >

{% block content %}
    <div class="container">
        {% with messages = get_flashed_messages() %}
            {% if messages %}
                {% for message in messages %}
                    <div class="alert alert-success">
                        {{ message }}
                    </div>
                {% endfor %}
            {% endif %}
        {% endwith %}
        
        {% block app_context %}{% endblock %}
    </div>
{% endblock %}

<! -- ... -- >
```

![User sessions](images/data_visualization/chartjs/user_sessions.gif)


__Duplicate registration__

To further improve a user's experience during registration, we will do the following:

`app/forms.py: Improve User Experience During Registration`

```python
from wtforms.validators import ValidationError
from app.models import User


def RegistrationForm(FlaskForm):
    # ...

    def validate_email(self, field):
        if User.query.filter_by(email=field.data).first():
            raise ValidationError('Email already registered.')
    

    def validate_username(self, field):
        if User.query.filter_by(username=field.data).first():
            raise ValidationError('Username already in use.')

```

Every time a new user tries to user an already existing email address or username, we will raise a validation error and provide useful information as to why the registration process does not work.

![Ux registration](images/data_visualization/chartjs/ux_registration.png)