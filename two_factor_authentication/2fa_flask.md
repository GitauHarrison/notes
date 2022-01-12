# Enable Time-based Two-factor Authentication in Flask

![TOTP Demo](/images/2fa_flask/totp_demo.gif)

## Overview

Two-factor authentication is basically a method that requires a user of an application to provide two forms of identification before being allowed to use the application. These two methods may include:

1. Regular password
2. One-time token

In the event that the user's account is compromised, then an attacker will find it a bit hard to access the user's token, which is different every time, say within 30 minute intervals.

This application requires all users to use two-factor authentication.

## Project Requirements

1. A smartphone 
2. Access to Google Playstore or App Store

Search for either of these two TOTP apps:

* [FreeotOTP Authenticator](https://play.google.com/store/apps/details?id=org.fedorahosted.freeotp&hl=en)
* [Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=en)

You are not limited to these two. You can use any other if you like. Why these apps? We will need them to scan the application's QR Code when we try to register a new user. The apps will consistently generate 30-seconds one-time passwords for us.


## Testing

If you would like to test this project out, consider checking the [hosted application](https://simple-2fa.herokuapp.com/) or [test it locally](https://github.com/GitauHarrison/how-to-implement-time-based-two-factor-auth-in-flask).

To make the project a bit more complete, I have added features such as _password resets_ and _email verification_, though they have nothing to do with what we will do in this article.


## General Rules About Passwords

1. Never store a password in the database; rather store its hash
2. Always use secure `HTTP` to transmit passwords
3. Compare a user's password against its hash in the database when authenticating a user


## Create project structure

This is the structure we will follow:

```python
project_folder
    |------- .flaskenv
    |------- .env
    |------- .env.template
    |------- config.py
    |------- app.py
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
                        |------- two_factor_setup.html
                        |------- email/
                                   |----- reset_password.html
                                   |----- reset_password.txt

```

You can create this structure using the `mkdir` and `touch` terminal commands:

```python
$ mkdir project_folder # creates an empty directory called project_folder
$ touch project_folder/config.py # creates an empty config.py file inside project_folder
```

Once you have completed this project structure, move into `project_folder`:

```python
$ cd project_folder
```

Make it a `git` repository by running:

```python
$ git init
```

We will use `git` to host our application in both GitHub and Heroku. If you have not set up `git` in your computer, learn how to do that [here](install_git.md). `git` allows you to work with both of GitHub's `ssh` and `http`. Find out what they are and how to use them [here](github_ssh.md).

## Activate A Virtual Environment

While working on multiple prjects, it is recommended that you isolate the requirements of each project from that running in your Operating System. Virtual environments are for this purpose; they create a virtual environment that allows you to work on multiple and independent projects.

To create and activate your virtual environment, run:

```python
$ mkvirtualenv totp
```

The `mkvirtualenv` command creates and activates the virtual environment called `totp` for you. Here, I have used `virtualenvwrapper` to help me manage my workflow. Virtualenvwrapper has many more commands that makes it easier to work with virtual environments. Follow [this guide](virtualenvwrapper_setup.md) to learn how to set up your machine to use `virtualenvwrapper`.

## Application Dependencies

This project will use several dependencies:

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
* [pyqrcode](https://pypi.org/project/PyQRCode/)
* [email-validator](https://pypi.org/project/email-validator/)

To install all of them at once, run:

```python
(totp)$ pip3 install flask flask-sqlalchemy flask-bootstrap # Add all the other dependencies within this same line
```

From your root directory (project_folder), update your `requirements.txt` to contain all the installed dependencies:

```python
(totp)$ pip3 freeze > requirements.txt
```

## Build Project

Let's begin by creating an instance of our application:

### Application Instance

`__init__.py: Application Instance`

```python
from flask import Flask
from flask_bootstrap import Bootstrap
from flask_login import LoginManager
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_mail import Mail
from config import Config

app = Flask(__name__)
app.config.from_object(Config)
bootstrap = Bootstrap(app)
login = LoginManager(app)
login.login_view = 'login'
db = SQLAlchemy(app)
migrate = Migrate(app, db)
mail = Mail(app)


def start_ngrok():
    from pyngrok import ngrok

    url = ngrok.connect(5000)
    print('* Tunnel: ', url)


if app.config.get("ENV") == "development" and app.config["START_NGROK"]:
    start_ngrok()

from app import routes, models, errors

```

I have created an instance of the flask application. Additionally, I have instantiated dependencies we installed in the previous section. 

`ngrok` is a tool that will provide us with free public URLs that redirect to our service. This will be especially useful when we want to test our application, which is currently running on `localhost`, on other devices, say a smartphone or another computer. For security reasons, our computers are configured with firewalls, and are therefore inaccessible. If our application is running locally, then we need to find a way to by-pass this. This is where `ngork` excels at. What I have done here is I have set up `ngrok` to run as soon as we fire up our flask server.

Our other modules are imported at the botton of the application instance to avoid the issue of circular dependencies.

### Application Configuration

From the application instance section above, you have seen `app.config["START_NGROK"]`. Here, we are referring to the data of `START_NGROK` variable which is found in the config file. All of the configurations needed to run our applications will be found in the `config.py` file. 

`config.py: Add Application Configurations`
```python
import os
from dotenv import load_dotenv

basedir = os.path.abspath(os.path.dirname(__file__))
load_dotenv(basedir, '.env')


class Config(object):
    # Form security
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'

    # Database configuration
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False

    # Localhost testing
    START_NGROK = os.environ.get('START_NGROK') is not None and \
        os.environ.get('WERKZEUG_RUN_MAIN') is not 'true'

    # Email configurations
    MAIL_SERVER = os.environ.get('MAIL_SERVER')
    MAIL_PORT = int(os.environ.get('MAIL_PORT') or 25)
    MAIL_USE_TLS = os.environ.get('MAIL_USE_TLS') is not None
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')
    ADMINS = os.environ.get('ADMINS')

```
Our configurations are set up as secret environment variables. The `os.environ` is used to fetch our environment variables. This method of obtaining environment variables is extremely safe since the actual values of the environment variables are preserved, especially from version control.

We use Python's `dotenv` package to load our enviroment variables.

`.env` file is used to store our actual environment variables. The `.`(dot) preceding the file name means that it is hidden. If you try to run the command `ls` in your root directory you will notice that the files starting with the `.` do not show up. However, to list every file, including the hidden files, you can run `ls -a`.

Let us update `.env` file with our secret configurations:

`.env: Environment Configurations`
```python
SECRET_KEY='<put-here-you-hard-to-guess-key>'
MAIL_SERVER='smtp.gmail.com'
MAIL_PORT='587'
MAIL_USE_TLS='True'
MAIL_USERNAME='<your-email-address>'
MAIL_PASSWORD='<password-to-the-email-above>'
ADMINS=['<another-email-address-that-will-be-used-as-admin>']
```

This file should never be commited to version control because it contains sensitive data such as passwords and other secret keys. To provide guidance to people who would like to clone your project,say from [GitHub](https://github.com), the `.env.template` (a text-based file) is used as a guide, where no values are put.

`.env.template: Show .env template`

```python
SECRET_KEY=
MAIL_SERVER=
MAIL_PORT=
MAIL_USE_TLS=
MAIL_USERNAME=
MAIL_PASSWORD=
ADMINS=
```
We will need to register our application in the top-level file `app.py`. This will be the entry point to our application.

`app.py: Application Entry Point`

```python
from app import app
```

Flask expects certain variables to be set before the server can be fired up. These variables need to stored in the `.flaskenv` file in the top-level directory. They are the flask environment variables.

`.flaskenv: Add Flask Environment Variables`

```python
FLASK_APP=app.py
FLASK_ENV=development
FLASK_DEBUG=True
START_NGROK=1
```

Our application is in development so we have set our flask environment to development. Every time we make changes to the application, flask's debugger actively detects those changes and loads them immediately due to the setting `FLASK_DEBUG=True`.

## Working with Database

This application involves adding and loading a user from a database. we will now set up a database table called `User` to handle a user's data.

`models.py: Create database model`

```python
from app import db, login, app
from werkzeug.security import generate_password_hash, check_password_hash
from flask_login import UserMixin


class User(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    first_name = db.Column(db.String(64), index=True, unique=True)
    last_name = db.Column(db.String(64), index=True, unique=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))

    def __repr__(self):
        return '<User {}>'.format(self.username)

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        return check_password_hash(self.password_hash, password)


@login.user_loader
def load_user(id):
    return User.query.get(int(id))

```

Our database contains five entries. These will be populated using a web form, seen in sections below. As I had mentioned earlier, it is safe practice to store a user's password as a hash rather than the actual password. Above, I am using `generate_password_hash` and `check_password_hash` from [Werkzeug](http://werkzeug.pocoo.org/) to handle all the hashing needs of the application.

Look at this example run from a Python interpreter:

```python
>>> from werkzeug.security import generate_password_hash
>>> hash = generate_password_hash('foobar')
>>> hash
'pbkdf2:sha256:50000$vT9fkZM8$04dfa8f6a9c0s676acf7e788a1b5b3c35e217c78dc04539d295f011jfaosf96adfs68asf9a'
```

The password `foobar` is transformed into a long encoded string which makes it hard for anyone who gains access to the password hash to use it. Hashing the same password mutliple times gives a different result, making it even harder to know if two users have the same password.

To verify a user's password, we use `check_password_hash`:

```python
>>> from werkzeug.security import check_password_hash
>>> check_password_hash(hash, 'foobar')
True
>>> check_password_hash(hash, 'trippin')
False
```

In an actual application, this is how it will look like:

```python
>>> u = User(username='rahima', email='rahima@email.com')
>>> u.set_password('rahima')
>>> u.check_password('nassir')
False
>>> u.check_password('rahima')
True
```

## Web Forms

A user registers for an account by providing their names, username and password.

![Register](/images/2fa_flask/register.png)

Other forms we will need include the _Password Request_, _Password Reset_ and _Login_. Flask-wtf allows for form creation as seen below:

`forms.py: Capture User Input`

```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, SubmitField, BooleanField
from wtforms.validators import Email, DataRequired, EqualTo, ValidationError
from app.models import User


class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    remember_me = BooleanField('Remember Me')
    submit = SubmitField('Login')


class RegistrationForm(FlaskForm):
    first_name = StringField('First Name', validators=[DataRequired()])
    last_name = StringField('Last Name', validators=[DataRequired()])
    username = StringField('Choose a Username', validators=[DataRequired()])
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField('Password', validators=[DataRequired()])
    confirm_password = PasswordField('Confirm Password',
                                     validators=[DataRequired(),
                                                 EqualTo('password')
                                                 ]
                                     )
    submit = SubmitField('Register')

    def validate_username(self, username):
        user = User.query.filter_by(username=username.data).first()
        if user is not None:
            raise ValidationError('Please use a different username')

    def validate_email(self, email):
        user = User.query.filter_by(email=email.data).first()
        if user is not None:
            raise ValidationError('Please use a different email address')


class ResetPasswordRequestForm(FlaskForm):
    email = StringField('Email', validators=[DataRequired(), Email()])
    submit = SubmitField('Request Password Reset')


class ResetPasswordForm(FlaskForm):
    password = PasswordField('Password', validators=[DataRequired()])
    password2 = PasswordField('Repeat Password',
                              validators=[DataRequired(), EqualTo('password')]
                              )
    submit = SubmitField('Request Password Reset')

```

## Handling User Data

We will update our `routes.py` file with functions that handle each data group from the user.

`routes.py: Handle User Data`

```python
from flask import render_template, redirect, url_for, flash, request
from werkzeug.urls import url_parse
from flask_login import login_user, logout_user, current_user, login_required
from app import db, app
from app.models import User
from app.forms import LoginForm, RegistrationForm, ResetPasswordRequestForm,\
    ResetPasswordForm
from app.email import send_password_reset_email


@app.route('/')
@app.route('/home')
@login_required
def home():
    return render_template('home.html', title='Home')


@app.route('/login', methods=['GET', 'POST'])
def login():
    if current_user.is_authenticated:
        return redirect(url_for('home'))
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


@app.route('/logout')
def logout():
    logout_user()
    return redirect(url_for('login'))


@app.route('/register', methods=['GET', 'POST'])
def register():
    if current_user.is_authenticated:
        return redirect(url_for('home'))
    form = RegistrationForm()
    if form.validate_on_submit():
        user = User(first_name=form.first_name.data,
                    last_name=form.last_name.data,
                    username=form.username.data,
                    email=form.email.data
                    )
        user.set_password(form.password.data)
        db.session.add(user)
        db.session.commit()
        flash('Registration successful. Login to continue')
        return redirect(url_for('login'))
    return render_template('register.html',
                           title='Register',
                           form=form
                           )


@app.route('/reset_password_request', methods=['GET', 'POST'])
def reset_password_request():
    if current_user.is_authenticated:
        return redirect(url_for('home'))
    form = ResetPasswordRequestForm()
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first()
        if user:
            send_password_reset_email(user)
        flash('Check your email for the instructions to reset your password')
        return redirect(url_for('login'))
    return render_template('reset_password_request.html',
                           title='Reset Password',
                           form=form
                           )


@app.route('/reset_password/<token>', methods=['GET', 'POST'])
def reset_password(token):
    if current_user.is_authenticated:
        return redirect(url_for('home'))
    user = User.verify_reset_password_token(token)
    if not user:
        return redirect(url_for('home'))
    form = ResetPasswordForm()
    if form.validate_on_submit():
        user.set_password(form.password.data)
        db.session.commit()
        flash('Your password has been reset.')
        return redirect(url_for('login'))
    return render_template('reset_password.html', form=form)

```

Every time a user is authenticated, they are redirected to the home page. Otherwise, the application checks and verifies the information from the database before issueing appropriate redirects.

At this point, we need to complete setting up email support for our applicaiton. Our _route_ module has imported functions from the _email_ module such as `send_password_reset_email` which does not exist yet.

## Email Support

This is where we use `flask-mail`, already installed and imported at the beginning of this article. We will use it to allow users to request for password resets.

The _Login_ page will have a link to request for a password reset.

![Login Page](/images/2fa_flask/login.png)

`email.py: Add Email Support`

```python
from flask_mail import Message
from app import mail, app
from flask import render_template
from threading import Thread


def send_async_email(app, msg):
    with app.app_context():
        mail.send(msg)


def send_email(subject, sender, recipients, text_body, html_body):
    msg = Message(subject, sender=sender, recipients=recipients)
    msg.body = text_body
    msg.html = html_body
    Thread(target=send_async_email, args=(app, msg)).start()


def send_password_reset_email(user):
    token = user.get_reset_password_token()
    send_email('[2fa] Reset Your Password',
               sender=app.config['ADMINS'][0],
               recipients=[user.email],
               text_body=render_template('email/reset_password.txt',
                                         user=user, token=token),
               html_body=render_template('email/reset_password.html',
                                         user=user, token=token))

```

When the _ResetPasswordRequest_ form is validated (seen in the _routes_ module), the user is searched for in the database using the email they provided in the form. 

![Request Password Reset Form](/images/2fa_flask/request_password_reset.png)

`send_password_reset_email()` function is used to send a password reset email. Email to be sent is found in the _email_ templates folder. A flash message is shown regardless of whether the user is known or unknown. This is to prevent any user from figuring out who is or is not a member.

The password reset links will have a secure token in them. To generate these tokens, we will use [JSON Web Tokens](https://jwt.io/), already installed as `pyjwt`. The tokens belong to the users, hence we will need to update our _User_ model to generate and validate these tokens.

`models.py: Generate and Verify User Tokens`

```python
# Previous imports
from time import time
import jwt

class User(UserMixin, db.Model):
    # ...

    def get_reset_password_token(self, expires_in=600):
        return jwt.encode(
            {'reset_password': self.id, 'exp': time() + expires_in},
            app.config['SECRET_KEY'], algorithm='HS256')

    @staticmethod
    def verify_reset_password_token(token):
        try:
            id = jwt.decode(token, app.config['SECRET_KEY'],
                            algorithms=['HS256'])['reset_password']
        except:
            return
        return User.query.get(id)
```

The `get_reset_password_token()` function returns a JWT token as a string, which is generated directly by the `jwt.encode()` function. Upon token verification, if the token is valid, then the value of the `reset_password` key from the token's payload is the ID of the user, so we can load the user and return it.

## Template Pages

Note how we are using flask's `render_template` to return the `html` files that display our pages.

These are the files' content:

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
            <a class="navbar-brand" href="{{ url_for('home') }}">2fa</a>
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
If the user is anonymous, a link to log into the application is provided in the navbar. If the user is authenticated, a logout link is shown.

`login.html: Login Page`

```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_content %}
        <div class="row">
            <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12 text-center">
                <h1>Login</h1>                                              
            </div>
        </div>
        <div class="row">
            <div class="col-sm-12 col-md-4 col-lg-4 col-xl-4">                
                              
            </div>
            <div class="col-sm-12 col-md-4 col-lg-4 col-xl-4 my-form">                                   
                {{ wtf.quick_form(form) }}
                <p>
                    Forgot your Password? <span><a href="{{ url_for('reset_password_request') }}">Reset Here</a></span>.
                </p>
                <p>
                    New here? <span><a href="{{ url_for('register') }}">Register Here!</a></span>.
                </p>                                            
            </div>
            <div class="col-sm-12 col-md-4 col-lg-4 col-xl-4">                
                           
            </div>
        </div>
        

{% endblock %}
```

The _Login_ page contains links to the _Register_ and _Request Password Reset_ forms.

`register.html: Captures User's Data`

```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_content %}
    <div class="row">
        <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12 text-center">
            <h1>Register</h1>                                              
        </div>
    </div>
    <div class="row">
        <div class="col-sm-12 col-md-4 col-lg-4 col-xl-4">                
                          
        </div>
        <div class="col-sm-12 col-md-4 col-lg-4 col-xl-4 my-form">                                  
            {{ wtf.quick_form(form) }}                                                       
        </div>
        <div class="col-sm-12 col-md-4 col-lg-4 col-xl-4">                
                       
        </div>
    </div>
{% endblock %}
```

If registration is successful, the user will be redirected to the home page of the app.


`home.html: Display home page`

```html
{% extends 'base.html' %}

{% block app_content %}
        <div class="row">
            <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12 text-center">
                <!-- title -->
                <h1>Time-based OTP</h1>
            </div>
            <div class="col-sm-12 col-md-3 col-lg-3 col-xl-3 ">
                <!-- Empty div -->
            </div>
            <div class="col-sm-12 col-md-6 col-lg-6 col-xl-6 ">
                <p>
                    To successfully test this application, you will need to do the following:
                    <ul>
                        <li>Have a Smartphone</li>
                        <li>Have access to PlayStore or App Store </li>
                    </ul>
                </p>
                <p>
                    You can choose either of these applications to help with the QR Code scanning:
                    <ul>
                        <li><a href="https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=en_US&gl=US">Google Authenticator</a></li>
                        <li><a href="https://play.google.com/store/apps/details?id=org.fedorahosted.freeotp&hl=en_US&gl=US">Free OTP</a></li>
                    </ul>
                    Or any other you may like!
                </p>
            </div>
            <div class="col-sm-12 col-md-3 col-lg-3 col-xl-3 ">
                <!-- Empty div -->
            </div>
        </div>

        <!-- Add your own image -->
        <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12 text-center">
            <img class="img-fluid" style="max-width: 100%; height: auto;" src="{{ url_for('static', filename='images/<your-demo-image.png>') }}">
        </div>           
{% endblock %}
```
Provide for password resetting as seen below:

`reset_password_request.html: User can request for a password reset`

```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_content %}
    <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12 text-center">
        <h1>Request Reset Password</h1>        
    </div>
    <div class="row">
        <div class="col-sm-12 col-md-4 col-lg-4 col-xl-4">                
            <!-- Empty div -->
        </div>
        <div class="col-sm-12 col-md-4 col-lg-4 col-xl-4 my-form">                                  
            {{ wtf.quick_form(form) }}                                                       
        </div>
        <div class="col-sm-12 col-md-4 col-lg-4 col-xl-4">                
            <!-- Empty div -->
        </div>
    </div>
{% endblock %}
```
A user's registration email is needed to send a _password reset_ email. If this user exists, the email is sent, otherwise, no email is sent.

`reset_password.html: Form to set new password`

```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_content %}
    <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12 text-center">
        <h1>Reset Password</h1>        
    </div>
    <div class="row">
        <div class="col-sm-12 col-md-4 col-lg-4 col-xl-4">                
            <!-- Empty div -->
        </div>
        <div class="col-sm-12 col-md-4 col-lg-4 col-xl-4 my-form">                                  
            {{ wtf.quick_form(form) }}                                                       
        </div>
        <div class="col-sm-12 col-md-4 col-lg-4 col-xl-4">                
            <!-- Empty div -->
        </div>
    </div>
{% endblock %}
```

This new password will be added to the database to be used by the user in subsequent logins.

`emails/reset_password.html: Email template`

```html
<p>Dear {{ user.username }},</p>
<p>
    To reset your password
    <a href="{{ url_for('reset_password', token=token, _external=True) }}">
        click here
    </a>.
</p>
<p>Alternatively, you can paste the following link in your browser's address bar:</p>
<p>{{ url_for('reset_password', token=token, _external=True) }}</p>
<p>If you have not requested a password reset simply ignore this message.</p>
<p>Sincerely,</p>
<p>The 2fa Team</p>
```

This is how the email sent to the user will look like.

`emails/reset_password.txt: Email template`

```txt
Dear {{ user.username }},

To reset your password click on the following link:

{{ url_for('reset_password', token=token, _external=True) }}

If you have not requested a password reset simply ignore this message.

Sincerely,

The 2fa Team
```

A text version of the email template.

## Error Handling

Currently, the _home_ route is accessed through `/home`. What happens when you try to access the _home_ route through `/home/`? Obviously, this will throw a 404 error because we do not have a page like that. Instead of our application displaying a rather scary and possibly too revealing an error message, we can catch it and redirect a user to an existing page. We need to update our _errors_ module to handle these eventualities.

`errors.py: Handle Errors`

```python
from app import app, db
from flask import render_template


@app.errorhandler(404)
def not_found_error(error):
    return render_template('404.html'), 404


@app.errorhandler(500)
def internal_error(error):
    db.session.rollback()
    return render_template('500.html'), 500
```

You probably have already noticed that we imported the _errors_ module at the botton of `__init__.py` file.

`404.html: Page Not Found`

```html
{% extends 'base.html' %}

{% block app_content %}
    <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12 text-center">
        <h1>Page not Found</h1>
        <p>
            The requested page cannot be found. Click <a href="{{ url_for('home') }}">here</a> to return to the home page.
        </p>
    </div>
{% endblock %}
```

If a requested page is not found, this page will be displayed.

`500.html: Internal Error`

```html
{% extends 'base.html' %}

{% block app_content %}
    <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12 text-center">
        <h1>An unexpected internal error</h1>
        <p>
            An unexpected internal error has occurred. The administrator has been notified. Sorry for the inconvinience.
        </p>
        <p>
            Click <a href="{{ url_for('home') }}">here</a> to return to the home page.
        </p>
    </div>
{% endblock %}
```
In case there is an error with the server, then this `500.html` template will be displayed. It would be necessary to notfy the application's admin about the error. Since we have already set up our email feature, we can take advantage of it and update the admin through email. Let us update `__init__.py` file to not only send out error emails but to also log these error messages.

`__init__.py: Log errors and send our error emails`

```python
# Previous imports
import logging
from logging.handlers import SMTPHandler, RotatingFileHandler
import os

# Previous code

if not app.debug and not app.testing:
    if app.config['MAIL_SERVER']:
        auth = None
        if app.config['MAIL_USERNAME'] or app.config['MAIL_PASSWORD']:
            auth = (app.config['MAIL_USERNAME'],
                    app.config['MAIL_PASSWORD']
                    )
        secure = None
        if app.config['MAIL_USE_TLS']:
            secure = ()
        mail_handler = SMTPHandler(
            mailhost=(app.config['MAIL_SERVER'], app.config['MAIL_PORT']),
            fromaddr='noreply@' + app.config['MAIL_SERVER'],
            toaddrs=app.config['ADMINS'],
            subject='Flask 2fa Failure',
            credentials=auth, secure=secure
        )
        mail_handler.setLevel(logging.ERROR)
        app.logger.addHandler(mail_handler)
    else:
        if not os.path.exists('logs'):
            os.mkdir('logs')
        file_handler = RotatingFileHandler(
            'logs/2fa_flask.log',
            maxBytes=10240,
            backupCount=10
        )
        file_handler.setFormatter(logging.Formatter(
            '%(asctime)s %(levelname)s: %(message)s [in %(pathname)s:%(lineno)d]'
        ))
        file_handler.setLevel(logging.INFO)
        app.logger.addHandler(file_handler)
        app.logger.setLevel(logging.INFO)
        app.logger.info('2fa Testing')

# previous imports
```

These error templates provide a polite message with a gently redirect link to the home page.

## Creating Migration Repository

We will use SQLite database due to its convinience working with small applications. Flask-migrate becomes very handy at this stage.

We have created a database structure (or schema) for our application already. To apply these changes, we need to create a migration script which will be stored in the _migrations_ sub-directory.

We will follow this order to apply any changes we make to our schema:

1. `flask db init`
2. `flask db migrate -m 'name-your-migration-script'`
3. `flask db upgrade`

```python
(totp)$ flask db init 

# Output
Creating directory /home/me/project_foldermigrations ... done
  Creating directory /home/me/project_folder/migrations/versions ... done
  Generating /home/me/project_folder/migrations/alembic.ini ... done
  Generating /home/me/project_folder/migrations/env.py ... done
  Generating /home/me/project_folder/migrations/README ... done
  Generating /home/me/project_folder/migrations/script.py.mako ... done
  Please edit configuration/connection/logging settings in
  '/home/me/project_folder/migrations/alembic.ini' before proceeding.
```
This command creates a _migrations_ directory which contains a _versions_ sub-directory.

```python
(totp)$ flask db migrate -m 'user table'

# Output
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'user'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_email' on '['email']'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_username' on '['username']'
  Generating /home/me/project_folder/migrations/versions/e517276bb1c2_users_table.py ... done
```
This command creates a `user table` which maps to the schema of the database model.

```python
(totp)$ flask db upgrade

# Output
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> e517276bb1c2, users table
```

This command applys the changes we have made. `flask db upgrade` does not make any changes to the database.

_Every time we make changes to our database schema, we will be following the order of these commands to apply those changes._

## Run the Application

With the application setup complete (we have not yet added two-factor authentication), we can test its functionality. On your terminal, run:

```python
(totp)$ flask run
```

You should be able to access localhost on http://127.0.0.1:5000/. Click this link or paste it on a new browser tab to see the output.

## Two Factor Authentication

I will use [`onetimepass`](https://github.com/tadeck/onetimepass/) package. There are other packages that implement TOTP algorithms. A simple search on [pypi](https://pypi.python.org/pypi?%3Aaction=search&term=totp&submit=search) will reveal them.

### Update the User Model

We will need to add a new field called `otp_secret` to our User model. This field will store the shared secret that the TOTP algorithm uses as input.

`models.py: Add token`

```python
import os
import base64
import onetimepass

class User(UserMixin, db.Model):
    # previous code
    otp_secret = db.Column(db.String(16))

    def __init__(self, **kwargs):
        super(User, self).__init__(**kwargs)
        if self.otp_secret is None:
            # generate a random secret
            self.otp_secret = base64.b32encode(os.urandom(10)).decode('utf-8')

    # previous

    def get_totp_uri(self):
        return 'otpauth://totp/2FA-Demo:{0}?secret={1}&issuer=2FA-Demo' \
            .format(self.username, self.otp_secret)

    def verify_totp(self, token):
        return onetimepass.valid_totp(token, self.otp_secret)
```

The `otp_secret` is encoded as a [base32](http://en.wikipedia.org/wiki/Base32) string, which makes it a printable string with 16 characters. 

The `get_totp_uri()` function returns an authentication URI. The secret token is shared with the smartphone through this URI. The URI is rendered as a QRCode which you will need to scan with your phone.

This is how that URI looks like:

```python
otpauth://<protocol>/<service-name>:<user-account>?secret=<shared-secret>&issuer=<service-name>
```

* `<protocol>`: can be `totp`
* `<service-name>`: is the name of the service or application that  a user is authenticating to
* `<user-account>`: anything that identifies the user account. It can be the user's username or email address

* `<shared-secret>`: the code used to seed the token generator algorithm
* `<issuer>`: normally set to the service name

The `verify_totp()` function takes a token as input, and validates using the support provided by the onetimepass package.

Since we have made some changes to the database schema, we need to generate and apply a new migration script. Run the commands below in their order:

```python
(totp)$ flask db migrate -m 'add totp field'
(totp)$ flask db upgrade
```

### Authentication Logic

An application can implement two-factor authentication in two ways:
1. Making it optional
2. Making it mandatory

When two-factor authentication is optional, the application can provide a use with a link to either enable or disable this feature. Other applications can make two-factor authentication part of the user registration process, such that before the user is successfully registered, then they have to set up two-factor authentication. Subsequent access to the account will always require additional one-time passwords.

We will make use of the mandatory two-factor authentication by incorporating it right after a user has registered.

![QRCode](/images/2fa_flask/qrcode.png)

Upon registration, the user is redirected to another route before being sent to the _Login_ page.

`routes.py: 2fa Redirect`

```python
@app.route('/register', methods=['GET', 'POST'])
def register():
    # ...
    form = RegisterForm()
    if form.validate_on_submit():
        # ...

        # redirect to the two-factor auth page, passing username in session
        session['username'] = user.username
        return redirect(url_for('two_factor_setup'))
    return render_template('register.html', form=form)
```

Before being redirected, the user's `username` is passed in the session so the QRCode knows what user is registering.

`routes.py: Set Up 2fa`

```python
@app.route('/twofactor')
def two_factor_setup():
    if 'username' not in session:
        return redirect(url_for('index'))
    user = User.query.filter_by(username=session['username']).first()
    if user is None:
        return redirect(url_for('index'))
    # since this page contains the sensitive qrcode, make sure the browser
    # does not cache it
    return render_template('two-factor-setup.html'), 200, {
        'Cache-Control': 'no-cache, no-store, must-revalidate',
        'Pragma': 'no-cache',
        'Expires': '0'}
```

The `two_factor_setup` validates whether there is a username in the user's session. If it exists, all it does is to render a two-factor setup page. The page tells the browser not to do any caching so as to prevent an attacker from getting access to the QR code which has time-based tokens .

### Display the QRCode Page

The page includes a reference the the QRcode image, but the url to this image is not the usual image link. 

`two-factor-setup.html: Display QR code image`

```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_content %}
    <div class="row">
        <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12 text-center">
            <h1>2fa Authentication SetUp</h1>
            <p>You are almost done! Please start FreeOTP on your smartphone and scan the following QR Code with it:</p>
            <p>
                <img id="qrcode" src="{{ url_for('qrcode') }}">
            </p>
            <a href="{{ url_for('login') }}"><button type="button" class="btn btn-light">Login</button></a>
        </div>
    </div>    
{% endblock %}
```

The image is dynamically generated using `url_for` function from `flask`. This is because the QRCode image needs to be generated specifically for each user so a flask route is invoked to do the work.

`route.py: Return image data`

```python
# Previous imports
from flask import abort
from io import BytesIO
import pyqrcode

@app.route('/qrcode')
def qrcode():
    if 'username' not in session:
        abort(404)
    user = User.query.filter_by(username=session['username']).first()
    if user is None:
        abort(404)

    # remove username from session for added security
    del session['username']

    # render qrcode from FreeOTP
    url = pyqrcode.create(user.get_totp_uri())
    stream = BytesIO()
    url.svg(stream, scale=5)
    return stream.getvalue(), 200, {
        'Content-Type': 'image/svg+xml',
        'Cache-Control': 'no-cache, no-store, must-revalidate',
        'Pragma': 'no-cache',
        'Expires': '0'
    }
```

Rather than return a template as is the case with all other routes, the `qrcode` route returns an image data. Again, we check whether the `username` is in the session. If any of those validations fail, then we return a 404 error. 

If the user is valid, we remove it from the session because once the user has requested for the QR Code, we make sure that this image cannot be requested again. This means that if the user fails to scan the QR code in this occassion, then the account will become inaccessible.

The information stored in the QR codes is the URL that contains TOTP data. This is what the TOTP smartphones expect. `pyqrcode` simply renders the TOTP URL as an SVG image, saved in a `StringIO` stream in memory.

We ensure that the browser does not cache the page by including some headers.

### Login Token

We now need to include a token field in our _Login_ page.

`forms.py: Token field in Login page`

```python
class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    token = StringField('Token', validators=[DataRequired()])
    remember_me = BooleanField('Remember Me')
    submit = SubmitField('Login')
```

When this _Login_ data is submitted, we need to modify our `login()` function to allow for an additional validation check:

`routes.py: Tokekn authentication`

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    # ...
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.username.data).first()
        if user is None or not user.check_password(form.password.data) or \
                not user.verify_totp(form.token.data):
            flash('Invalid username, password or token.')
            return redirect(url_for('login'))

        # log user in
        # ...
    return render_template('login.html', form=form)
```

We have added `user.verify_totp()` to check tokens using the `onetimepass` package.

That's it! You now have a mandatory two-factor authentication set up in your flask application.

## Deployment

Heroku is a free Platform-as-a-Service which you can use to deploy your application. I have previously covered [how to deploy an app to Heroku](deploy_to_heroku.md) in another article. The method is exactly the same, with a few modifications to suite this app's features.

Note that before you can commit any changes to a `git` repository, it is necessary that you do not commit files which contain sensitive data. In our case, we have `.env` file. `git` provides a file called `.gitignore` to which we can add any files that we wish to exclude from version control. Find out [here](https://github.com/github/gitignore) what files you can ignore from version control. Ensure `.env` file is added to `.gitignore` before adding and commiting your application files.

In summary, this is what you will need to do to deploy your app to Heroku:

1. Login to heroku on your terminal (`$ heroku login`)
2. Ensure your application is in a git repository (`$ git remote -v`)
3. Create a Heroku application (`$ heroku apps:create totp-app`)
4. Create Postgres database (`$ heroku addons:add heroku-postgresql:hobby-dev`)
5. Login errors to STDOUT
6. Set environment variables in Heroku (`$ heroku config:set LOG_TO_STDOUT=1 # Add the others`)
7. Install `gunicorn` and `psycopg2` (`$ pip3 install gunicorn psycopg2`)
8. Add `Procfile` to root directory and update it (`$ touch Procfile`)
9. Commit your changes (`$ git commit -m '<your-commit-message>'`)
10. A remote Heroku repository (`$ heroku git:remote -a <your-heroku-app-name>`)
11. Push to Heroku (`$ git push heroku master`)


## Optional Two-factor Authentication in Flask

Want to see how to implement optional two-factor authentication in a flask app? Read more [here](twilio_verify_2fa.md).