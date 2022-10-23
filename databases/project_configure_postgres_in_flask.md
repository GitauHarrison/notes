# Create A Flask App Using PostgreSQL Database

You have now decided to use the PostgreSQL database to manage your data. Given all the features that come with it, as discussed in [Postgres Overview](00_postgresql_overview), you may be ready to start using it. Throughout this tutorial, I will show you how to create a password-based user authentication system that utilized the PostgreSQL database.

If you are new to PostgreSQL, I recommend that you start from these tutorials, in their order:

1. [Postgres Overview](databases/00_postgresql_overview)
2. [Install Postgres](databases/01_install_postgresql.md)
3. [Access PostgreSQL using `psql`](databases/access_postgresql/psql.md)
4. [Access PostgreSQL Using DBeaver](databases/access_postgresql/dbeaver.md)
5. [How to Secure PostgreSQL](databases/02_how_to_secure_postgresql.md)
6. [Getting Started With PostgreSQL](databases/03_getting_started_with_postgresql.md)


## Project Overview

The project we will create features a login page and registration page that allows users to create an account before accessing their profiles. We won't make the application more robust by adding extra features because the goal is to learn how to use the postgreSQL database ONLY.

![Postgres Demo Project App](/images/databases/postgres_demo_project/postgres_demo_app.gif)


## Project Structure

We will keep it simple, though we will employ the concept of [_separation of concerns_](https://en.wikipedia.org/wiki/Separation_of_concerns) while building the application. This principle allows us to bundle related tasks into modules instead of having the entire application in a single file. Below is the project structure:

```python
project_folder
    | --- config.py
    | --- main.py
    | --- .flaskenv
    | --- .env
    | --- .gitignore
    | --- requirements.txt
    | --- app/
            | --- __init__.py
            | --- forms.py
            | --- routes.py
            | --- models.py
            | --- templates/
                    | --- base.html
                    | --- profile.html
                    | --- login.html
                    | --- register.html
            | --- static/css/main.css
```

## Setting Things Up

Before we can start building the project, it is always advisable to set things up. We will need to create and activate a virtual environment.

### Step 1: Navigate To Your Project Directory

You have to make sure you are in your current project folder before issuing further commands. To change the directory, run:

```python
$ cd project_folder
```

![Change Directory](/images/databases/postgres_demo_project/change_directory.png)


### Step 2: Create A Virtual Environment

A virtual environment is a copy of your Operating System, used to isolate the needs of one application from another and keep your system clean and clutter-free. In Python, this can be done using the command below:

```python
$ python3 -m venv venv
```

![Create venv](/images/databases/postgres_demo_project/create_venv.png)

The first `venv` is the command used to create a virtual environment. The second `venv` is the name of the virtual environment. 


### Step 3: Fix Failing Command

Notice that the terminal returns an 'error' telling us we need to fix the failing command. It provides us with the actual command we need to use:

```python
$ sudo apt install python3.8-venv
```

![Install venv](/images/databases/postgres_demo_project/install_venv.png)

### Step 4: Re-create The Virtual Environment

To ensure that the virtual environment is fully created, we need to re-run the previous command in our terminal:

```python
$ python3 -m venv venv
```

### Step 5: Activate Virtual Environment

At this stage, we can now activate our virtual environment. Run this command in the terminal:

```python
$ source venv/bin/activate
```

![Activate Venv](/images/databases/postgres_demo_project/activate_venv.png)

You will see `(venv)` at the beginning of your terminal, meaning the virtual environment is activated. 


## Building The Application

The first task we will do is to fire up the flask server. It does not work magically. We have to set things up. If you would like to refer to the completed project, you can find it in the repository [configure flask to use postgresql](https://github.com/GitauHarrison/configure-flask-to-use-postgresql).

### Step 1: Install Project Dependencies

To create the demo app, we need to install the following extensions and packages within our active virtual environment. Run these commands:

```python
(venv)$ pip3 install flask flask-sqlalchemy flask-migrate flask-bootstrap flask-wtf flask-login python-dotenv psycopg2
```

Notice that we are installing `psycopg2`, an extension needed when working with PostgreSQL. On many distros, the development headers needed for compiling against libraries are not installed by default. For psycopg2 on Ubuntu you'll need the python3 and PostgreSQL headers:

```python
# Install necessary headers
(venv)$ sudo apt install python3-dev libpq-dev

# Then install psycopg2 in your active virtual environment
(venv) pip3 install psycopg2
```

### Step 2: Update Project Requirements File

Should you want to share your project with anyone else, it is recommended you list all the project dependencies. `requirements.txt` (as the name suggests) is used to show what dependencies have been used to build an application. To update it, run:

```python
(venv)$ pip3 freeze > requirements.txt
```

You need to be in the top-level directory where this file is located before running this command. You will notice that it will be populated with all the installed project dependencies.

### Step 3: Register Extensions in Application Instance

After the installation of the extensions, we need to register some of them within the application instance. The file `__init__.py` is used to create an application instance.

```python
# app/__init__.py

from flask import Flask
from flask_bootstrap import Bootstrap
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_login import LoginManager
from config import Config


app = Flask(__name__)
app.config.from_object(Config)


bootstrap = Bootstrap(app)
db = SQLAlchemy(app)
migrate = Migrate(app, db)
login = LoginManager(app)
login.login_view = 'login'


from app import routes, models

```

### Step 4: Configure The Application

Besides the installed extensions, notice that we have linked the application instance to the configurations file in `__init__.py`. What we need to do now is to set up all the needed configurations. This will be done in the `config.py` file.

```python
# config.py

import os

basedir = os.path.abspath(os.path.dirname(__file__))


class Config(object):
    """All application configurations"""

    # Secret key
    SECRET_KEY = os.environ.get('SECRET_KEY')

    # Database configurations
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False

```

We are sourcing the PostgreSQL database file from an environment variable called `DATABASE_URL`. If this file does not exist (at this point it doesn't), then the application will fall back to a file-based database called `app.db` to be found in the top-level directory of the application.

The `SECRET_KEY` environment variable is required by the application to protect the login and registration forms from a nasty attack called [Cross-site Request Forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery).


### Step 5: Create Environment Variables

Environment variables are used to hide sensitive data you do not want to be exposed to anyone else except the maintainer(s) of the application. Otherwise, we would have passed the actual value of these keys in the `config.py` file. For example, we would done `SECRET_KEY = '1234' `.

The reason the format of the PostgreSQL database file is hidden is because it also contains sensitive data we do not want to expose as you will see in the section [PostgreSQL URL Format](#postgresql-url-format).

Environment variables are created in a hidden text file called `.env`, as seen in the project structure. The dot (`.`) means "hidden". Let us update this file with the following information"

```python
#.env

SECRET_KEY=harry
DATABASE_URL=postgresql://muthoni:muthoni@localhost:5432/demo_postgres_app
```

### Step 6: PostgreSQL URL Format

The following format is used to access PostgreSQL

```python
DATABASE_URL=[engine]://[user name]:[user password]@[host]:[port]/[database]
```


### Step 7: Add View Functinos

Now that we have the configurations properly set, we can create view functions to handle specific endpoints. This is done in the `routes.py` file.

```python
# app/routes.py

from app import app, db
from flask import render_template, url_for, redirect, flash
from app.forms import LoginForm, RegistrationForm
from flask_login import login_user, logout_user, current_user, login_required
from app.models import User


@app.route('/')
@app.route('/profile')
@login_required
def profile():
    """Profile page"""
    return render_template('profile.html', title='Profile')


@app.route('/login', methods=['GET', 'POST'])
def login():
    """Login URL"""
    if current_user.is_authenticated:
        return redirect(url_for('profile'))
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.username.data).first()
        if user is None or not user.check_password(form.password.data):
            flash('Invalid username or password')
            return redirect(url_for('login'))
        login_user(user, remember=form.remember_me.data)
        flash('Welcome')
        return redirect(url_for('profile'))
    return render_template('login.html', title='Login', form=form)


@app.route('/register', methods=['GET', 'POST'])
def register():
    """Registration URL"""
    if current_user.is_authenticated:
        return redirect(url_for('profile'))
    form = RegistrationForm()
    if form.validate_on_submit():
        user = User(username=form.username.data)
        user.set_password(form.password.data)
        db.session.add(user)
        db.session.commit()
        flash('Registered successfully. Please log in to continue)
        return redirect(url_for('login'))
    return render_template('register.html', title='Register', form=form)


@app.route('/logout')
@login_required
def logout():
    """Used to log out a user"""
    logout_user()
    return redirect(url_for('login'))

```

The view functions our application needs are:

- `login()`
- `register()`
- `profile()`
- `logout()`

The package we installed earlier called `flask-login` is used to the state of users of the application. The states may include when the user is logged in or when they are logged out. This package is also useful to load a particular user from the database.


### Step 8: Create User Model

If you have noticed, both the `login()` and `register()` view functions use the class `User` from `app.models` to update the database file. However, at this point, this `User` schema does not exist. So we need to create it. A schema is used to define how a database table will look, and what information goes where.

```python
# app/models.py

from app import db, login
from flask_login import UserMixin
from werkzeug.security import generate_password_hash, check_password_hash


@login.user_loader
def load_user(id):
    """Load user by their ID"""
    return User.query.get(int(id))



class User(db.Model, UserMixin):
    """User table"""
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(32), unique=True, index=True)
    password_hash = db.Column(db.String(128))

    def __repr__(self):
        return f'User: {self.username}'

    def set_password(self, password):
        """Hash user password befor storage"""
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        """Confirms a user's password"""
        return check_password_hash(self.password_hash, password)

```

We begin by loading a user from the database using the `flask-login` package. This package also has the attribute `UserMixin` that helps the application to know if the user is authenticated, anonymous or active.

As a security precaution, a user's password is never stored in its original form. Instead, it is hashed into some cryptographic value using the methods `generate_password_hash`,  and `check_password_hash` before being stored. This is to help secure user data in the unfortunate event the database becomes compromised through an attack.


### Step 9: Create The Database

What we have done above was simply to define the structure of our database. The next step to start using the database would be to create it. Run these commands in your terminal to create the application's database:

```python
(venv)$ flask db init                      # Creates a migrations folder in the top-level directory
(venv)$ flask db migrate -m 'user table'   # Creates user migration script
(vemv)$ flask db upgrade                   # Applies the changes
```

### Step 10: Define Forms

The two forms used by our application are the login form and the registration form. This is easily done with the help of `flask-wtf` package we installed earlier. Let us update `forms.py` to define our form fields.

```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField, PasswordField, BooleanField
from wtforms.validators import DataRequired, Length, EqualTo


class LoginForm(FlaskForm):
    """Login Form"""
    username = StringField('Username', validators=[DataRequired(), Length(1, 64)])
    password = PasswordField('Password', validators=[DataRequired()])
    remember_me = BooleanField('Keep me logged in')
    submit = SubmitField('Log In')


class RegistrationForm(FlaskForm):
    """Registration Form"""
    username = StringField('Username', validators=[DataRequired(), Length(1, 64)])
    password = PasswordField('Password', validators=[DataRequired()])
    confirm_password = PasswordField(
        'Confirm Password', validators=[DataRequired(), EqualTo('password')])
    submit = SubmitField('Register')

```

### Step 11: Update Templates

Templates are used to display information. We will begin by defining some reusable elements within our templates in the `base.html` file.

```html
<!-- app/templates/base.html -->

{% extends 'bootstrap/base.html' %}

<!-- Link all style files here -->
{% block head %}
    {{  super() }}
    <link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='css/main.css') }}">
{% endblock %}

<!-- The title of our application is defined here -->
{% block title %}
    {% if title %}
        {{ title }} - Postgres
    {% else %}
        Using Postgres
    {% endif %}
{% endblock %}

<!-- This is the navbar -->
{% block navbar %}
<nav class="navbar navbar-default">
    <div class="container">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand" href=" {{ url_for('profile') }} ">Postgres Demo</a>
        </div>
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">            
            <ul class="nav navbar-nav navbar-right">                
                {% if current_user.is_authenticated %}
                    <li><a href=" {{ url_for('logout') }} ">Logout</a></li>
                {% else %}
                    <li><a href=" {{ url_for('login') }} ">Login</a></li>
                {% endif %}
            </ul>                       
        </div>
    </div>
</nav>
{% endblock %}


{% block content %}
<div class="container">
    <!-- Flash messages -->
    {% with messages = get_flashed_messages() %}
        {% if messages %}
            <div class="alert alert-info">
                {% for message in messages %}
                    {{ message }}
                {% endfor %}
            </div>
        {% endif %}
    {% endwith %}
    
    <!-- Contents of all our pages will go here -->
    {% block app_context %}{% endblock %}
</div>
{% endblock %}


<!-- All scripts will go here -->
{% block scripts %}
    {{  super() }}
    
{% endblock %}
```

We are using Bootstrap to enforce responsiveness and cross-browser compatibility. The flask package that easily integrates Bootstrap is `flask-bootstrap`.


### Step 12: Define Child Templates

The base template defines reusable elements such as the title, the navbar, the styles, and the flash messages. The elements can be inherited by other templates (ie profile, login, and register) using the keyword `extends`. The Jinja templating engine allows for inheritance using this keyword.

```html
<!-- app/templates/profile.html -->

{% extends 'base.html' %}

{% block app_context %}
<div class="row">
    <div class="col-md-12">
        <h1>Profile</h1>
        <p>Username: {{ current_user.username }}</p>
    </div>  
</div>
{% endblock %}


<!-- app/templates/login.html -->

{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_context %}
    <div class="row">
        <div class="col-md-4">
            <!-- Emtpy column -->
        </div>
        <div class="col-md-4">
            {{ wtf.quick_form(form) }}
            <p>
                New here? <a href="{{ url_for('register') }}">Register</a>.
            </p>
        </div>
        <div class="col-md-4">
            <!-- Empty column -->
        </div>
    </div>
{% endblock %}


<!-- app/templates/register.html -->

{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_context %}
    <div class="row">
        <div class="col-md-4">
            <!-- Empty column -->
        </div>
        <div class="col-md-4">
            {{ wtf.quick_form(form) }}
        </div>
        <div class="col-md-4">
            <!-- Empty column -->
        </div>
    </div>
{% endblock %}

```

Flask-bootstrap allows for the integration of our forms using the macro `wtf`. All form features such as styling and security are automatically included, relieving us of this responsibility.


### Step 13: Ignore Both The Sensitive and The Unnecessary Files

In the event you want to push your code to a version control platform such as GitHub, you will need to ignore certain files. You can file Python-specific files to ignore in [this link](https://github.com/github/gitignore/blob/main/Python.gitignore).

Paste the contents of the files and folders you want to ignore in the hidden file `.gitignore` within the top-level directory of the project.

```python
# .gitignore

.env
venv/
```
The file `.env` and the folder `venv` will be ignored by the version control system.

### Step 14: Create An Entry Point

There needs to be an entry point to the application. Flask will use this entry point to access our code. Intentionally, we can update the `main.py` file to act as an entry point.

```python
# main.py

from app import app
from app.models import User
from app import db


@app.shell_context_processor
def make_shell_context():
    return dict(db=db, User=User)

```

We are importing our application's instance before passing the values of our database as well as the `User` model to the flask shell. The Flask shell allows us to access the application's context from the terminal. This is normally achieved by simply running `(venv)$ flask shell` in the terminal.


### Step 15: Update Environment Variables

Flask expects certain environment variables before starting the flask server. Instead of pasting each of these commands in the terminal one at a time, we can update our `.flaskenv` file.

```python
# .flaskenv

FLASK_APP=main.py
FLASK_ENV=development
FLASK_DEBUG=True
```

We are telling flask that the application's entry point is the file called `main.py`. Since we are still developing it, we set the environment to `development`. `FLASK_DEBUG=True` allows flask to 'catch' bugs for us so that it is easier for us to fix them during development.

### Step 16: Start The Flask Server

Believe it or not, we are done with our application. All we need to do now is to start the flask server. Run this in the terminal:

```python
(venv)$ flask run

# Output

* Serving Flask app 'main.py'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 485-335-726
```

Paste the URL http://127.0.0.1:5000 on your favorite browser to see the application.

![Final Application](/images/databases/postgres_demo_project/final_app.png)


## Configure PostgreSQL Database

Our Application is now functional, but before we can start populating our PostgreSQL database, let us set up a few things.

### Step 1: Access PostgreSQL from PSQL

PSQL is a terminal front-end tool used to access the PostgreSQL database. If you have reviewed the lesson [Access PostgreSQL using `psql`](databases/access_postgresql/psql.md), this will be quite easy. If not, I recommend that you start there.

Log into PostgreSQL as the Postgres user by running this command in the terminal:

```sql
(venv)$ sudo -u postgres psql

# Output
psql (14.5 (Ubuntu 14.5-1.pgdg20.04+1))
Type "help" for help.

postgres=# 
```

You can split your VS Code terminal into two so that one window runs the flask server, while the other allows you to access the PostgreSQL database. Alternatively, you can stop the flask server by pressing "Ctrl + C" before running the command above.

### Step 2: Create a New User

Notice that the prompt on the terminal has changed to `postgres`. We can issue our first postgres command to create another user called `muthoni` by running:

```sql
postgres=# CREATE USER muthoni SUPERUSER;
CREATE ROLE
```

### Step 3: List all Users

You can list all the application users by running:

```sql
postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 harry     | Superuser                                                  | {}
 muthoni   | Superuser                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 taste     |                                                            | {}

postgres=# 
```

You can see that `muthoni` is now a super user. A super user bypasses all database checks, so you should be VERY CAREFULL when doing this.


### Step 4: Add User Password

To help secure user `muthoni`, we can assign her a password. 

```sql
postgres=# ALTER USER muthoni WITH PASSWORD 'muthoni';
ALTER ROLE
```

Now, every time `muthoni` tries to access the postgres database, she will be required to identify herself by providing the password "muthoni".


### Step 5: Create A DATABASE

At this stage, we can create a database and make `muthoni` an owner.

```sql
postgres=# CREATE DATABASE demo_postgres_app WITH OWNER muthoni;
```

Remember our `DATABASE_URL` environment variable earlier?

```python
# .env

DATABASE_URL=postgresql://muthoni:muthoni@localhost:5432/demo_postgres_app
```

Be sure to update it accordingly if your user and database are named differently. Refer to this [postgreSQL format](#step-6-postgresql-url-format).

### Step 6: See All Your Databases

To see a list of all your databases, you can run this command:

```sql
postgres=# \l

# Output
                                      List of databases
       Name        |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-------------------+----------+----------+-------------+-------------+-----------------------
 demo_postgres_app | muthoni  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 newsletter_app    | harry    | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres          | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres_test     | taste    | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/taste            +
                   |          |          |             |             | taste=CTc/taste
 template0         | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                   |          |          |             |             | postgres=CTc/postgres
 template1         | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                   |          |          |             |             | postgres=CTc/postgres
(6 rows)
```

You can see that `muthoni` is the owner of `demo_postgres_app` database.

And, that is it! Your Flask Application is now ready to use the PostgreSQL database. 