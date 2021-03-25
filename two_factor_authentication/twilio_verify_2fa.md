# Two-factor Authentication Using Twilio Verify in Flask

There are a couple of ways to use two-factor authentication in a flask application. In [another article](2fa_flask.md), I showed how you can enable mandatory two-factor authentication where users of an app have to key in a time-based one-time password that is sent to a TOTP app in their smartphone. In this article, we will make two-factor authentication optional. Whenever a user enables this feature, a code will be sent to their phone through sms. They will use that code to authenticate themselves.

[Twilio Verify](https://www.twilio.com/verify) is a service that allows your application to send verification codes to users of your application through SMS or phone call.

![Twilio Verify](/images/twilio_verify/twilio_verify.gif)

What we will do:

1. Create a flask app with user login features
2. Add a profile page
3. Integrate Twilio Verify

Kindly note that you will need these before you proceed:

* A working phone number
* A Twilio account. Create a [free account](https://www.twilio.com/try-twilio?promo=WNPWrR) now.
* Python 3.5+

### Service Setup

Once you have an account, 
* Navigate to [Twilio Console](https://www.twilio.com/console). 
* From the far-left menu bar, click "All Products and Services"
* Find and click on [Verify](https://www.twilio.com/console/verify/services)
* Create a new service by clicking on the blue button

![Create Service](/images/twilio_verify/create_service.png)

* Give it a friendly name and click "Create"
* You will be given a _Service ID_

![Service ID](/images/twilio_verify/service_id.png)

I have shown you the _Service ID_. However, this should be secret. I am not worried about it because I will discard it in no time.

* Besides the _Service ID_, you will also need your _Twilio Account SID_ and _Auth Token_ which are found in the [Twilio Console](https://www.twilio.com/console).

Copy all these somewhere because you will need them for your project.

### Flask App with User Login

This project assumes you know a bit about Flask and Python. If you are new, you will need to start [here](personal_blog.md).

#### Project Structure

We will use this structure:

```python
project_folder
    |------- .flaskenv
    |------- .env
    |------- .env.template
    |------- config.py
    |------- twilio_verify.py
    |------- requirements.txt
    |------- .gitignore
    |------- app/
              |------- __init__.py
              |------- routes.py
              |------- models.py
              |------- forms.py
              |------- errors.py
              |------- email.py
              |------- static/
                        |------- images/
                        |------- css/styles.css
              |------- templates/
                        |------- base.html
                        |------- home.html
                        |------- login.html
                        |------- register.html
                        |------- 404.html
                        |------- 500.html
                        |------- reset_password_request.html
                        |------- reset_password.html
                        |------- user.html
                        |------- verify_2fa.html
                        |------- edit_profile.html
                        |------- enable_2fa.html
                        |------- disable_2fa.html
                        |------- email/
                                   |----- reset_password.html
                                   |----- reset_password.txt

```

You can create this structure using the `mkdir` and `touch` terminal commands:

```python
$ mkdir project_folder # creates an empty directory called project_folder
$ touch project_folder/config.py # creates an empty config.py file inside project_folder
```

Once you have completed creating this project structure, move into _project_folder_:

```python
$ cd project_folder
```

#### Create Virtual Environment

Virtual environments allow you to isolate the project requirements from that of your Operating System. You need to create and activate it:

```python
$ mkvirtualenv twilio_verify 
```

I have used the `virtualenvwrapper` to manage my virtualenv workflow. If you do not know what it is, learn more [here](virtualenvwrapper_setup.md).

This application will use these dependencies:

* [flask](https://flask.palletsprojects.com/en/1.1.x/)
* [flask-sqlalchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/)
* [flask-boostrap](https://pythonhosted.org/Flask-Bootstrap/)
* [flask-login](https://flask-login.readthedocs.io/en/latest/)
* [flask-wtf](https://flask-wtf.readthedocs.io/en/stable/)
* [flask-mail](https://pythonhosted.org/Flask-Mail/)
* [flask-migrate](https://flask-migrate.readthedocs.io/en/latest/)
* [python-dotenv](https://pypi.org/project/python-dotenv/)
* [pyjwt](https://pyjwt.readthedocs.io/en/stable/)
* [pyngrok](https://pypi.org/project/pyngrok/)
* [email-validator](https://pypi.org/project/email-validator/)

To install all of them at once, run:

```python
(twilio_verify)$ pip3 install flask flask-sqlalchemy flask-bootstrap # Add all the other dependencies within this same line
```

To work with Twilio Verify, you will also need to install the Twilio Helper Library for Python:

```python
(twilio_verify)$ pip3 install "twilio>=6.17.0"
```

From your root directory (project_folder), update your `requirements.txt` to contain all the installed dependencies:

```python
(twilio_verify)$ pip3 freeze > requirements.txt
```

#### Build Initial Project

Let us make sure that the structure shown at the beginning of the article works by building a minimalist application:

`__init__.py: Create application instance`
```python
from flask import Flask

app = Flask(__name__)

from app import routes, models, errors
```

We have created an instance of our flask application

`routes.py: Handle app routeing`

```python
from app import app


@app.route('/', methods=['GET', 'POST'])
@app.route('/home', methods=['GET', 'POST'])
def home():
    return 'This is a test!'

```

The application should eventually display "This is a test".

`app.py: Create entry point to the applicaiton`

```python
from app import app

```

Flask expects this file at the top-level directory.

`.flaskenv: Flask Environment Variables`

```python
FLASK_APP=app.py
FLASK_ENV=development
FLASK_DEBUG=True
```

Flask will use these variables to fire up our server. It will be a development server with Flask's hot auto-reload enabled.

We can now fire up our Flask server from the terminal:

```python
(twilio_verify)$ flask run
```

You should see this:

![Test](/images/twilio_verify/test.png)

#### Database Configuration

Our application will allow new users to register and current users to login. Let us implement this now. This information will be hosted by our SQLite database. 

`config.py: Database configuration`

```python
import os

basedir = os.path.abspath(os.path.dirname(__file__))

class Config(object):
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

Configure a path to our database and disable a feature of Flask-sqlalchemy that we do not need, which is to signal the application every time a change is about to be made in the database.

`__init__.py: Initialize database`

```python
# ...
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import 
from config import Config

app = Flask(__name__)
app.config.from_object(Config)
db = SQLAlchemy(app)
migrate = Migrate(app, db)

# ...
```
`flask-sqlalchemy` is an extension that provides a Flask-friendly wrapper to the popular [SQLAlchemy](http://www.sqlalchemy.org/) package. This package is an ORM which allows applications to manage a database using high-level entities such as classes, objects and methods instead of tables and SQL.

`flask-migrate` will allow us to make changes to our database by handling migratrion.

`models.py: Database schema to for a user`

```python
from app import db

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))

    def __repr__(self):
        return '<User: {}>'.formart(self.username)
```

With the database schema and configuration set up, let us apply those changes:

```python
(twilio_verify)$ flask db init

# Output

Creating directory /home/harry/verify_twilio/migrations ...  done
  Creating directory /home/verify_twilio/migrations/versions ...  done
  Generating /home/harry/verify_twilio/migrations/script.py.mako ...  done
  Generating /home/harry/verify_twilio/migrations/alembic.ini ...  done
  Generating /home/harry/verify_twilio/migrations/README ...  done
  Generating /home/harry/verify_twilio/migrations/env.py ...  done
  Please edit configuration/connection/logging settings in
  '/home/harry/verify_twilio/migrations/alembic.ini' before proceeding.
```

A _migratitons_ repository will be created in the top-level directory. In it, there is a _versions_ sub-folder which will soon hold all the changes we make to our database.

```python
(twilio_verify)$ flask db migrate -m 'user table'

# Output

INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'user'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_email' on '['email']'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_username' on '['username']'
  Generating
  /home/harry/software_development/python/practice_projects/verify_twilio/migrations/versions/9d3452db7add_user_table.py ...  done
```

A migration script called `...user_table.py` has been created in _versions_ sub-folder.

To apply the changes we have made, we will run:

```python
(twilio_verify)$ flask db upgrade

# Output

INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 9d3452db7add, user table
```

These are the steps we will follow every time we want to make changes to our database.

#### User Login

User login will involve finding an existing user in the database and retrieving that information. Flask provides `flask-login` which is responsible for handling all user login needs. 

Right of the back, it is not recommended to store a user's password in the database. Rather, a long representation of itself which is hard to guess is often used. This is called password hashing. [Werkzeug](http://werkzeug.pocoo.org/) provides `generate_password_hash` and `check_password_hash` to handle password hashing.

We will update our models to accommodate this feature.

`models.py: Password hashing`

```python
# ...
from flask_login import UserMixin
from werkzeug.security import generate_password_hash, check_password_hash

class User(UserMixin, db.Model):
    # ...

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        return check_password_hash(self.password_hash, password)
```

Keywords such as `is_authenticated`, `is_active`, `is_anonymous`, `get_d` are normally used to work with a user's login session. To implement them, we need to import the _mixin_ class from `flask-login` and pass it to our `User` model. Thereafter, we add password hashing.

However, `flask-login` knows nothing about a user. We need to help it know which user has been connected to the application by configuring a user loader function that can be called to load a user using a user's given ID.

`models.py: User loader`

```python
# ...
from app import login


@login.user_loader
def load_user(id):
    return User.query.get(int(id))

```

To complete the login view function, we will create a route called `/login`.

`routes.py: User login`

```python
# ...
from flask import render_template, redirect, url_for, flash, request
from werkzeug.urls import url_parse
from flask_login import login_user, current_user
from app.forms import LoginForm

@app.route('/login', methods=['GET', 'POST'])
def login():
    if current_user.is_authenticated:
        return redirect(url_for('login'))
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.username.data).first()
        if user is None or not user.check_password(form.password.data):
            flash('Invalid username or password')
            return redirect(url_for('login'))
        login_user(user, remember=form.remember_me.data)
        next_page = request.args.get('next')
        if not next_page or url_parse(next_page).netloc != '':
            next_page = url_for('home')
        return redirect(next_page)
    return render_template('login.html',
                           title='Login',
                           form=form
                           )
```

We need to create the `LoginForm`

```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, SubmitField, BooleanField
from wtforms.validators import DataRequired, Email, EqualTo, ValidationError


class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    remember_me = BooleanField('Remember Me')
    submit = SubmitField('Login')

```

Flask expects that we set a `SECRET_KEY` which will be used to protect our web forms against a nasty attack called  [CSRF](https://en.wikipedia.org/wiki/Cross-site_request_forgery). This should be done in the config file.

`config.py: Set SECRET_KEY`

```python
# ...

class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'extremely-difficult-to-guess'
    # ...
```

Here is the login template. We will use `flask-bootstrap` to quickly create one.

`login.html: Login template`

```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_content %}
    <div class="row">
        <div class="col-md-4">
            <h1>Login</h1>
        </div>
    </div>
    <div class="row">
        <div class="col-md-4">
            {{ wtf.quick_form(form) }}
        </div>
    </div>
    <div class="row">
        <div class="col-md-4">
            <p>New Here? <a href="{{ url_for('register') }}">Register Here</a></p>
            <p>Forgot Password? <a href="#">Reset Here</a></p>
        </div>
    </div>
{% endblock %}

```

We are importing the base template using the keyword `extends` but it does not exist yet. Let us update it below:

`base.html: Base template`

```html
{% extends 'bootstrap/base.html' %}

<!-- Title Section -->
{% block title %}
    {% if title %}
        2fa | {{ title }}
    {% else %}
        Flask Auth
    {% endif %}
{% endblock %}

<!-- Head Section -->
{% block head %}
    {{ super() }}
    <!-- Add your own image -->
    <link rel="icon" type="image/png" href="{{url_for('static', filename = 'images/<choice-image.ext>')}}">
    <link rel="preconnect" href="https://fonts.gstatic.com">
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300&display=swap" rel="stylesheet">
{% endblock %}

<!-- Link Styles -->
{% block styles %}
    {{ super() }}
    <link type="text/css" rel="stylesheet" href="{{ url_for('static', filename = 'css/styles.css') }}">
{% endblock %}

<!-- Navbar Section -->
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
            <a class="navbar-brand" href="{{ url_for('home') }}">Twilio Verify</a>
        </div>
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">            
            <ul class="nav navbar-nav navbar-right">  
                {% if current_user.is_anonymous %}
                    <li><a href=" {{ url_for('login') }} ">Login</a></li>
                {% else %}                                      
                <li><a href=" {{ url_for('logout') }} ">Logout</a></li>
                {% endif %}
            </ul>                       
        </div>
    </div>
</nav>
{% endblock %}

<!-- Main Content Goes Here -->
{% block content %}
    <div class="container">
        {% with messages = get_flashed_messages() %}
            {% if messages %}
                {% for message in messages %}
                    <div class="alert alert-warning" role="alert"> {{ message }} </div>
                {% endfor %}
            {% endif %}
        {% endwith %}

        {% block app_content %}
        
        {% endblock %}
    </div>
{% endblock %}
```

To log a user out, we will create a separate route:

`route.py: Log out a user`

```python
# ...
from flask_login import logout_user


@app.route('/logout')
def logout():
    logout_user()
    return redirect(url_for('home'))

```

To require users to login in order to access the home page, we will use `@login_required` decorator. But first, we need our application to know what view function handles logins.

`__init__.py: Register login required view function`

```python
# ...
from flask_login import LoginManager
from flask_botstrap import Bootstrap

login = LoginManager(app)
login.login_view = 'login'
bootstrap = Bootstrap(app)

```

Pass the `@login_required` decorator to the home page.

`routes.py: Require login to access home page`

```python
from flask_login import login_required

@app.route('/', methods=['GET', 'POST'])
@app.route('/home', methods=['GET', 'POST'])
@login_required
def home():
    return render_template('home.html',
                           title='Home',
                           )
```
![Login](/images/twilio_verify/login_required.png)

#### User Registration
 
Next, we well create a route that handles user registration:

`routes.py: User registration`

```python
# ...
from app import db
from app.forms import RegisterForm


@app.route('/register', methods=['GET', 'POST'])
def register():
    if current_user.is_authenticated:
        return redirect(url_for('home'))
    form = RegisterForm()
    if form.validate_on_submit():
        user = User(username=form.username.data, email=form.email.data)
        user.set_password(form.password.data)
        db.session.add(user)
        db.session.commit()
        flash('You have successfully registerd. Login to continue')
        return redirect(url_for('login'))
    return render_template('register.html', 
                           title='Register',
                           form=form
                           )

```

Here is the registration form:

`forms.py: Create registration form`

```python
# ...
from app.models import User


class RegisterForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField('Password', validators=[DataRequired()])
    confirm_password = PasswordField('Confirm Password',
                                     validators=[DataRequired(),
                                                 EqualTo('password')])
    submit = SubmitField('Register')

    def validate_username(self, username):
        user = User.query.filter_by(username=username.data).first()
        if user is not None:
            raise ValidationError('Please use a different username')

    def validate_email(self, email):
        user = User.query.filter_by(email=email.data).first()
        if user is not None:
            raise ValidationError('Please use a different email address')

```

The register template will look like this:

`register.html: Display registration template`

```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_content %}
    <div class="row">
        <div class="col-md-4">
            <h1>Register</h1>
        </div>
    </div>
    <div class="row">
        <div class="col-md-4">
            {{ wtf.quick_form(form) }}
        </div>
    </div>
{% endblock %}

```
![Register](/images/twilio_verify/register.png)

After a successful registration and login, a user will be redirected to the home page. This is how the home page will look like:

`home.html: Display home page`

```html
{% extends 'base.html' %}

{% block app_content %}
    <div class="row">
        <div class="col-md-4">
            <h1>Welcome home</h1>
        </div>
    </div>
{% endblock %}
```

![Home Page](/images/twilio_verify/home.png)

### User Profile Page

![User Profile](/images/twilio_verify/better_user_profile.png)

First thing, we will provide a link to a user's profile once they are logged in.

`base.html: Profile page link`

```html
<!---Previous code-->
            <ul class="nav navbar-nav navbar-right">  
                {% if current_user.is_anonymous %}
                    <li><a href=" {{ url_for('login') }} ">Login</a></li>
                {% else %}
                    <li><a href=" {{ url_for('user', username=current_user.username) }} ">Profile</a></li>
                    <li><a href=" {{ url_for('logout') }} ">Logout</a></li>
                {% endif %}
            </ul>
<!---Previous code-->
```

To make the profile page more interesting, we will add:

* Last seen time
* About Me 
* User Avatar

#### User Avatar

We will use the [Gravatar](http://gravatar.com/) service to provide user images. It is actually simple to use. To request an image for a given user, an email is required. 

```python
# Gravatar format

https://www.gravatar.com/avatar/<hash>

# hash is the md5 hash of a user's email address
```

Example on a Python shell:

```python
>>> from hashlib import md5
>>> 'https://www.gravatar.com/avatar/' + md5(b'harry@email.com').hexdigest()
'https://www.gravatar.com/avatar/3f4360b2a748228ba4f745a3ebd428dc'
```
If you follow that link, you will see a default gravatar image:

![Default Gravatar Image](/images/twilio_verify/default_gravatar.png)

 Learn more from the [gravatar documentation](https://gravatar.com/site/implement/images).

Since we want to add a custom avatar to an existing user, we will update our `User` model to accommodate this.

`models.py: Add user avatar`

```python
# ...
from hashlib import md5

class User(UserMixin, db.Model):
    # ...
    def avatar(self, size):
        digest = md5(self.email.lower().encode('utf-8')).hexdigest()
        return 'https://www.gravatar.com/avatar/{}?d=identicon&s={}'.format(
            digest, size)
```

Update the user template:

`user.html`

```html
{% extends "base.html" %}

{% block content %}
    <table>
        <tr valign="top">
            <td><img src="{{ user.avatar(128) }}"></td>
            <td><h1>User: {{ user.username }}</h1></td>
        </tr>
    </table>
    <hr>
{% endblock %}

```

Create a view function that maps the `user/<username>` route.

`routes.py: Map user profile view function`

```python
# ...


@app.route('/user/<username>')
def user(username):
    user = User.query.filter_by(username=username).first_or_404()
    return render_template('user.html',
                           title='Profile'
                           )

```

#### Last Seen and About Me

Let's extend out database to support this new feature:

`models.py: Add new fields`

```python
# ...
from flask import datetime


class User(UserMixin, db.Model):
    # ...
    about_me = db.Column(db.String(140))
    last_seen = db.Column(db.DateTime, default=datetime.utcnow)
```

Since we have made changes to our database, we need to apply them:

```python
(twilio_verify)$ flask db migrate -m 'date and about me fields in user model'
(twilio_verify)$ flask db upgrade
```

We can now reflect those changes in our `user.html` template:

`user.html: More interesting profile`

```html
{% extends "base.html" %}

{% block content %}
<div class="row">
    <div class="col-md-4">
        <table class="table table-hover">
            <tr valign="top">
                <td width="70px">
                    <img src="{{ user.avatar(128) }}">
                </td>
                <td>
                    <h1>User: {{ user.username }}</h1>
                    {% if user.about_me %}<p>{{ user.about_me }}</p>{% endif %}
                    {% if user.last_seen %}<p>Last seen on: {{ user.last_seen }}</p>{% endif %}
                </td>
            </tr>
        </table>
        <hr>
    </div>
</div>    
{% endblock %}
```

However, when you try reloading your profile page, nothing will show because we have not recorded the last time a user was seen nor edit the `about_me` content (it is currently empty).

`routes.py: Records user's last seen time`

```python
# ...
from datetime import datetime


@app.before_request
def before_request():
    if current_user.is_authenticated:
        current_user.last_seen = datetime.utcnow()
        db.session.commit()

```

Editing the `about_me` section will involve the use of a form. We need to create one.

`form.html: Edit about me`

```python
# ...
from wtforms import TextAreaField
from wtforms.validators import Length

class EditProfileForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    about_me = TextAreaField('About me', validators=[Length(min=0, max=140)])
    submit = SubmitField('Submit')

```

Display edit form:

`edit_profile.html: Display edit profile form`

```python
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_content %}
    <div class="row">
        <div class="col-md-4">
            <h1>Edit Profile</h1>
        </div>
    </div>
    <div class="row">
        <div class="col-md-4">
            {{ wtf.quick_form(form) }}
        </div>
    </div>
{% endblock %}
```

Add a view function that binds everything together:

`routes.py: Edit profile view function`

```python
# ...
from app.forms import EditProfileForm


@app.route('edit_profile', methods=['GET', 'POST'])
@login_required
def edit_profile():
    form = EditProfileForm()
    if form.validate_on_submit():
        current_user.username = form.username.data
        current_user.about_me = form.about_me.data
        db.session.commit()
        flash('Your profile has been updated')
        return redirect(url_for('user', username=current_user.username))
    elif request.method == 'GET':
        form.username.data = current_user.username
        form.about_me.data = current_user.about_me
    return render_template('edit_profile.html',
                           title='Edit Profile',
                           form=form
                           )

```

Add a link to update profile in the `user.html` template:

`user.html: Add edit link`

```python
{% if user == current_user %}
    <p><a href="{{ url_for('edit_profile') }}">Edit Profile</a></p>
{% endif %}
```

![Edit Profile](/images/twilio_verify/edit_profile.png)

### Integrate Twilio Verify

Implementation of Twilio Verify works in two steps:

1. Verification code is sent to your phone
2. That code is verified by the application


#### Send Verification Code

Hopefully you have your _Twilio Account SID_, _Twilio Auth Token_ and _Service SID_. To demonstrate how the Twilio Verify API works, we will run the commads below in our Python shell:

```python
>>> from twilio.rest import Client
>>> client = Client('<your_Twilio_Account_SID>', '<your_Twilio_Auth_Token>')
>>> verify = client.verify.services('<your_Twilio_Verify_Service_SID>')
>>> verify.verifications.create(to='<your_active_phone_number>', channel='sms')
```

Check your phone. You should receive an text message notification.

#### Verify Code Received

This is done on the flask application with another call into the Twilio Verify API.

```python
>>> result = verify.verification_checks.create(to='<your_phone_number>', code='123456')
>>> result.status
'pending'
>>> result = verify.verification_checks.create(to='<your_phone_number>', code='507296')
>>> result.status
'approved'
```

#### Add Twilio Credentials

We will add the three Twilio credentials to our configuration file

`config.py: Twilio configurations`

```python
# ...


class Config(object):
    # ...
    TWILIO_ACCOUNT_SID = os.environ.get('TWILIO_ACCOUNT_SID')
    TWILIO_AUTH_TOKEN = os.environ.get('TWILIO_ACCOUNT_TOKEN')
    TWILIO_VERIFY_SERVICE_ID = os.environ.get('TWILIO_VERIFY_SERVICE_ID')

```

These keys are imported from environment variables. They are also private and should not be committed to version control. For this reason, we will add the actual keys to `.env` file which is located in the top-level directory.

`.env: Add actual values to our credetials`

```python
TWILIO_ACCOUNT_SID='your_twilio_account_sid'
TWILIO_AUTH_TOKEN='your_twilio_auth_token'
TWILIO_VERIFY_SERVICE_ID='your_twilio_verify_service_sid'
```

To load these environment variables, we will use `python-dotenv` package which imports environment variables from `.env` file.

`config.py: Load environment variables`

```python
# ...
from dotenv import load_dotenv

basedir = os.path.abspath(os.path.dirname(__file__))
load_dotenv(basedir, '.env')


class Config(object):
    # ...

```

To ignore these sensitive files from version control, we need to update our `.gitignore` file in the top-level directory by explicitly adding files we want ignored. Check what files to ignore [here](https://github.com/github/gitignore/blob/master/Python.gitignore).

If another person tried to clone this project repository, say from [GitHub](https://github.com), they will not get the `.env` file. We can provide a template of our `.env` file to help guide future users:

`.env-template: Environment variables template`

```python
# Exclude the actual values

TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_VERIFY_SERVICE_ID=
```

#### Access Twilio Verify API

We will handle the access to Twilio Verify API in a separate module called `twilio_verify_api.py` in the _app_ folder. Let us create this module:

```python
# make sure you are in the right working directory

(twilio_verify)$ touch twilio_verify_api.py
```

This module will:

* Verify the client trying to access the API
* Request for a verification token
* Verify the requested token

`twilio_verify_api.py: Access Twilio Verify API`

```python
from app import app
from twilio.rest import Client, TwilioException


def _get_twilio_verify_client():
    return Client(
        app.config['TWILIO_ACCOUNT_SID'],
        app.config['TWILIO_AUTH_TOKEN']).verify.services(
            app.config['TWILIO_VERIFY_SERVICE_ID'])


def request_verification_token(phone):
    verify = _get_twilio_verify_client()
    try:
        verify.verifications.create(to=phone, channel='sms')
    except TwilioException:
        verify.verifications.create(to=phone, channel='call')


def check_verification_token(phone, token):
    verify = _get_twilio_verify_client()
    try:
        result = verify.verification_checks.create(to=phone, code=token)
    except TwilioException:
        return False
    return result.status == 'approved'

```

`_get_twilio_verify_client()` function checks for the client based on the Twilio credentials used earlier and returns a `verify` object. Using this object, we use `verifications.create(to=phone, channel='sms')` to attempt seninding a verification code via sms. If this fails, we make a phone call instead. `check_verification_token()` function uses the `verify` object to check the verification token sent to the phone and returns `True` if the token is valid or `False` if it is invalid.


Make sure to import this module in the application package:

`__init__.py: Import Twilio Verify module`

```python
# ...

from app import twilio_verify_api

```

#### Add Phone Number To User Database

Since we will be sending verification codes to a user's phone number, it needs to be stored in the database so that the tokens can be sent out every time they log in.

`config.py: Add user phone number`

```python
# ...


class User(UserMixin, PaginatedAPIMixin, db.Model):
    # ...
    verification_phone = db.Column(db.String(16))

    def two_factor_enabled(self):
        return self.verification_phone is not None

```

The `two_factor_enabled` helper method checks whether the `verification_phone` attribute exists or not.

We need to apply this change to our database by running the commands:

```python
(twilio_verify)$ flask db migrate -m 'Add phone number field'

# Output

INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added column 'user.verification_phone'
  Generating /home/verify_twilio/migrations/versions/49f34ca11d82_add_phone_number_field.py ...  done
```

```python
(twilio_verify)$ flask db upgrade

# Output
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade 50b197ac3799 -> 49f34ca11d82, Add phone number field
```

### Links to Enable/Disable Two-factor Authentication

We can display the option to enable or disable two factor authentication on the user's _profile_ page.

`user.html: Links to enable/disable 2fa`

```html
{# app/templates/user.html #}
{% extends "base.html" %}

{% block app_content %}
    <table class="table table-hover">
        <tr>
            <td width="256px"><img src="{{ user.avatar(256) }}"></td>
            <td>
                ...
                ...
                {% if not user.two_factor_enabled() %}
                    <p>
                        <a href="#">Enable two-factor authentication</a>
                    </p>
                {% else %}
                    <p>
                        <a href="#">Disable two-factor authentication</a>
                    </p>
                {% endif %}
            </td>
        </tr>
    </table>
```

![2fa Link](/images/twilio_verify/2fa_link.png)

#### Route to Handle Two-factor Authentication

To access this feature from their profile, a user has to provide their phone number. We will therefore need a form to recieve a user's phone number. We will use the [`phonenumbers`](https://pypi.org/project/phonenumbers/) package to validate the phone number given by the user. Install it in your application and update `requirements.txt`.

```python
(twilio_verify)$ pip3 install phonenumbers
(twilio_verify)$ pip3 freeze > requirements.txt
```

The main object that the library deals with is a `PhoneNumber` object. 

```python
# E164 format

>>> import phonenumbers
>>> kenya = phonenumbers.parse("+254720227267", 'KEN')
>>> print(kenya)
Country Code: 254 National Number: 720227267

# Invalid Country Code
>>> kenya = phonenumbers.parse("0720227267", 'KEN')
Traceback (most recent call last):
  File "<console>", line 1, in <module>
  File "/home/harry/.virtualenvs/test_verify/lib/python3.8/site-packages/phonenumbers/phonenumberutil.py", line 2850, in parse
    raise NumberParseException(NumberParseException.INVALID_COUNTRY_CODE,
phonenumbers.phonenumberutil.NumberParseException: (0) Missing or invalid default region.
```

You can create this from a string representing a phone number using the `parse` function, but you also need to specify the country that the phone number is being dialled from (unless the number is in _E.164_ format, which is globally unique).


`forms.py: Provide phone number`

```python
# ...
import phonenumbers


class Enable2faForm(FlaskForm):
    verification_phone = StringField('Phone', validators=[DataRequired()])
    submit = SubmitField('Enalble 2fa')

    def validate_verification_number(self, verification_phone):
        try:
            p = phonenumbers.parse(verification_phone.data)
            if not phonenumbers.is_valid_number(p):
                raise ValueError
        except (phonenumbers.phonenumberutil.NumberParseException, ValueError):
            raise ValidationError('Invalid phone number')
 
```

When the user clicks on the _enable two-factor authentication_ link, they will be redirected to this form. Only after this initial verification is complete will their account be updated to use two-factor authentication.

`routes.py: Update account with 2fa`

```python
# ...
from app.forms import Enable2faForm


@app.route('/enable_2fa', methods=['GET', 'POST'])
@login_required
def enable_2fa():
    form = Enable2faForm()
    if form.validate_on_submit():
        session['phone'] = form.verification_phone.data
        request_verification_token(session['phone'])
        return redirect(url_for('verify_2fa'))
    return render_template('enable_2fa.html',
                           form=form,
                           title='Enable 2fa'
                           )

```

Display the phone form

`enable_2fa.html: Dispaly phone verification phone`

```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_content %}
    <div class="row">
        <div class="col-md-4">
            <h1>Enable Two-factor Authentication</h1>
        </div>
    </div>
    <div class="row">
        <div class="col-md-4">
            {{ wtf.quick_form(form) }}
        </div>
    </div>
{% endblock %}

```

![Enable 2fa form](/images/twilio_verify/enable_2fa_form.png)

To improve usability of the phone number form, considering that the users might be from different regions around the globe where different phone number formats are used, we will use the [`intl-tel-input`](https://github.com/jackocnr/intl-tel-input) JavaScript library.

`base.html: Add Javascript code to handle international numbers`

```html
<!--Add intl tel input package CSS-->
{% block styles %}
    {{ super() }}
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/intl-tel-input/16.0.4/css/intlTelInput.css">
{% endblock %}


<!--scripts block goes to the bottom-->
{% block scripts %}
    {{ super() }}
    <script src="https://cdnjs.cloudflare.com/ajax/libs/intl-tel-input/16.0.4/js/intlTelInput-jquery.min.js"></script>
    <script>
        $("#verification_phone").css({position: 'absolute', top: '-9999px', left: '-9999px'});
        $("#verification_phone").parent().append('<div><input type="tel" id="_verification_phone"></div>');
        $("#_verification_phone").intlTelInput({
            separateDialCode: true,
            utilsScript: "https://cdnjs.cloudflare.com/ajax/libs/intl-tel-input/16.0.4/js/utils.js",
        });
        $("#_verification_phone").intlTelInput("setNumber", $('#verification_phone').val());
        $('#_verification_phone').blur(function() {
            $('#verification_phone').val($('#_verification_phone').intlTelInput("getNumber"));
        });
    </script>
{% endblock %}
```

![Improved Phone Form UI](/images/twilio_verify/better_phone_form.png)

Country codes are shown; the phone number placeholder information is also displayed to help guide the user when they provide their phone numbers.

#### Expand Login Logic

The login logic needs to be expanded to accomodate users with two-factor authentication enabled. 

`routes.py: Consider 2fa logic when loggin in`

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if current_user.is_authenticated:
        return redirect(url_for('login'))
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.username.data).first()
        if user is None or not user.check_password(form.password.data):
            flash('Invalid username or password')
            return redirect(url_for('login'))
        next_page = request.args.get('next')
        if not next_page or url_parse(next_page).netloc != '':
            next_page = url_for('home')
        if user.two_factor_enabled():
            request_verification_token(user.verification_phone)
            session['username'] = user.username
            session['phone'] = user.verification_phone
            return redirect(url_for('verify_2fa',
                                    next=next_page,
                                    remember='1' if form.remember_me.data else '0'
                                    )
                            )
        login_user(user, remember=form.remember_me.data)
        return redirect(next_page)
    return render_template('login.html',
                           title='Login',
                           form=form
                           )
```

We have postponed the login attempt to cater for users who have two-factor authentication enabled. For those users, they will be redirected to the `verify_2fa` view function as soon as they receive their verification tokens.

The “remember me” flag from the login form and the "next page" are added to the redirect URL as query string arguments, while the username and phone number for the user are stored in the user session to protect them against tampering.

#### Token Verification Route

The user needs to provide the token sent to their phone. This token will be captured by the application through a token verification form.

`forms.py: Confirm token form`

```python
# ...


class Confirm2faForm(FlaskForm):
    token = StringField('Token')
    submit = SubmitField('Verify')

```

Dispaly this form to the user to make it accessible

`verify_2fa.html: Display token verification form`

```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_content %}
    <div class="row">
        <div class="col-md-4">
            <h1>Enter Token Received</h1>
        </div>
    </div>
    <div class="row">
        <div class="col-md-4">
            {{ wtf.quick_form(form) }}
        </div>
    </div>
{% endblock %}

```

Once the token submission is made, `verify_2fa` function will handle it.

`routes.py: Verify token`

```python
# ...
from app.forms import Confirm2faForm


@app.route('/verify_2fa', methods['GET', 'POST'])
def verify_2fa():
    form = Confirm2faForm()
    if form.validate_on_submit():
        phone = session['phone']
        if check_verification_token(phone, form.token.data):
            del session['phone']
            if current_user.is_authenticated:
                current_user.verification_phone = phone
                db.session.commit()
                flash('You have enabled two-factor authentication')
            else:
                username = session['username']
                del session['username']
                user = User.query.filter_by(username=username).first()
                next_page = request.args.get('next')
                remember = request.args.get('remember', '0') == '1'
                login_user(user, remember=remember)
                return redirect(next_page)
        form.token.errors.append('Invalid token')
    return render_template('verify_2fa.html',
                           form=form,
                           title='Verify Token'
                           )

```

When a user requests for two-factor authentication, the phone number will be added to the database. In case of a normal login attempt, this route must log the user into the system, by invoking `login_user()` function using the data previously stored in the user session and the query string.

We have determined two states of a user:

* The current user is logged in and is trying to request for two-factor authentication
* The user is not logged in and is trying to log in

#### Disable Two-factor Authentication

In the user's profile page, once two-factor authentication is enabled, the _disable two-factor authentication_ link is displayed. When this link is clicked, a disable two-factor authentication button is displayed. We need to create this button form.

`forms.py: Disable button`

```python
# ...


class Disable2faForm(FlaskForm):
    submit = SubmitField('Disable 2fa')

```

The view function that handles this action is `disable_2fa`.

`routes.py: Disable 2fa logic`

```python
# ...
from app.forms import Disable2faForm


@app.route('/disable_2fa', methods=['GET', 'POST'])
@login_required
def disable_2fa():
    form = Disable2faForm()
    if form.validate_on_submit():
        current_user.verification_phone = None
        db.session.commit()
        flash('You have disabled two-factor authentication')
        return redirect('user', username=current_user.username)
    return render_template('disable_2fa.html',
                           form=form,
                           title='Disable 2fa'
                           )
```

We can now display this form to the user

`disable.html: Display Disable 2fa button`

```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_content %}
    <div class="row">
        <div class="col-md-4">
            <h1>Disable Two-factor Authentication</h1>
        </div>
    </div>
    <div class="row">
        <div class="col-md-4">
            {{ wtf.quick_form(form) }}
        </div>
    </div>
{% endblock %}

```

That's it!


### Additional Concepts

This application has both _email_ and _error_ modules. So far, we have not handled any of them. You can try to implement them in the application.

* If a user forgets their account's password, a _Reset Password_ link is provided in the _Login_ page
* If a non-existent page is requested from the server, the application should show a nicer looking error page with redirection to the home page rather than the scary and possibly too-revealing flask debug error page.