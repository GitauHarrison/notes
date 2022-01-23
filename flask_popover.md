# Add Popovers To Your Flask App

On Twitter, you often see that when you hover your mouse over a user's username, the person's profile is displayed in a tiny popover. This is true for Facebook and LinkedIn as well. In this tutorial, we will learn how to add popovers to our Flask app.

![Flask popover](/images/flask_popover/flask_popovers.gif)

You can find the completed project on this [flask popovers](https://github.com/GitauHarrison/flask-popovers) github repository

## What We Will Do

1. Create a simple web application
2. Display user comments
3. Add user profile
4. Integrate popovers on username links

## Create a Simple Application

I have already created a simple application that we will further build on. You can see follow through by checking the [start flask server](start_flask_server.md) tutorial.

## Project Requirements

We will take advantage of the following packages to build this application:

* Python3 for programming
* Flask framework
* Flask-WTF for web form creation
* Flask-Bootstrap for styling (already used)
* Flask-SQLAlchemy for database integration
* Flask-Migrate for database migrations
* Flask-Moment for displaying timestamps
* Python-dotenv for loading environment variables
* Email validator for email validation
* Flask-Login for user authentication

## Web Forms

Web forms are one of the most basic building blocks of web applications. They are used to collect data from the user and send it to the server. We will begin by install the following package:

```python
(venv) $ pip install flask-wtf
```

The forms we intend to create will be used to "register" and "login" users. Let us do so within the `/app` subdirectory.

```python
(venv) $ cd app
(venv) $ touch forms.py # Create an empty file
```

### Configure Environment Variables

At this point, the application is very basic and we do not have to worry about configurations. However, to work with web forms, Flask expects the `SECRET_KEY` variable to be set. We will set it to a random string.

`config.py`: Secret key for protection against CSRF
```python
import os


class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
```

I have set the value of `SECRET_KEY` to be sourced from an existing environment variable. If this variable does not exist,then I provide a safe fallback value. This is to ensure that the application does not crash.

To set the value of `SECRET_KEY`, you can run the following command to generate a random value:

```python
(venv) $ python -c "import os; print(os.urandom(24))"
```

Paste the result of the above command into the `.env` file, located in the top-level directory. If you do not have a `.env` file, you can create one by running the command `touch .env`.

`.env` file: Environment variables
```txt
SECRET_KEY=b'\x11\xc40\x02ax\x1ed\x88\xc9-\xb0\xb1\xe9\xde\xef'
```
We will then need to tell our application to load the environment variables. This is done in the `__init__.py` file.

`__init__.py`: Register the configration object
```python
from flask import Flask
from config import Config # < --- new


app = Flask(__name__)
app.config.from_object(Config) # < --- new


from app import routes
```

The application still needs help to load this environment variable, even though the configuration is registered in the application instance. This is done by installing the `python-dotenv` package.

```python
(venv) $ pip3 install python-dotenv
```

### Create Forms

Following the principle of _separation of concerns_, we will create a forms module that will define all the forms we want. Create an empty `forms.py` file in the `app` subdirectory.

```python
(venv) $ cd app
(venv) $ touch forms.py
```
Ensure that you are in the `app` subdirectory before creating the file.

#### Registration Form


`forms.py`: User registration form
```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, SubmitField
from wtforms.validators import DataRequired, Email, Length, EqualTo

class RegistrationForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField(
        'Password', validators=[DataRequired(), Length(min=8, max=30)])
    confirm_password = PasswordField(
        'Confirm Password',
        validators=[
            DataRequired(), Length(min=8, max=30), EqualTo('password')
            ])
    submit = SubmitField('Submit')
```
I have used the `RegistrationForm()` class to define all the fields we want to use in our form. The `username`, `email`, `password` and `confirm_password` fields are required before submission. The `password` fields must be at least 8 characters long. The `confirm_password` field must match the `password` field.

With the `email` field in place, we have used `Email()` to validate the data entered by a user. Flask expects `email-validator` package to be installed. To install it, run the following command:

```python
(venv) $ pip3 install email-validator
```

To display this form, we will do so in `register.html` file. Our application currently has all templates in the `templates` subdirectory. Create an empty `register.html` file in the `templates` subdirectory.

```python
(venv) $ cd templates
(venv) $ touch register.html
```

Flask-Bootstrap provides for a quick method to create and display formatted forms using the `quick_form()` function.

`register.html`: User registration form
```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_context %}
<div class="row">
    <div class="col-md-12 text-center">
        <h1>{{title}}</h1>
    </div>  
</div>
<div class="row">
    <div class="col-sm-4">
        <!-- Empty column -->
    </div>
    <div class="col-sm-4 my-form">
        {{ wtf.quick_form(form) }}
    </div>
    <div class="col-sm-4">
        <!-- Empty column -->
    </div>
</div>
{% endblock %}
```

With the registration template ready for display, we will create a new route to render the registration form.

`routes.py`: Register route
```python
from app.forms import RegistrationForm
from flask import render_template

# ...


@app.route('/register')
def register():
    form = RegistrationForm()
    return render_template(
        'register.html',
        title='Register',
        form=form)

```

The `register()` view function is used to render the registration form. The object `form` is passed to the `quick_form()` method, without which the form won't be displayed.

Navigate to the URL `http://localhost:5000/register` to see the registration form.

![Registration Form](/images/flask_popover/registration_form.png)


#### Login Form

To authenticate users, all we will want a user to do is enter their username and password. We will create a login form to handle this.

`forms.py`: User login form
```python
# ...
from wtforms import BooleanField


class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField(
        'Password', validators=[DataRequired(), Length(min=8, max=30)])
    remember = BooleanField('Remember Me')
    submit = SubmitField('Submit')
```

Again, we need to create a `login.html` file to display the login form.

```python
(venv) $ cd templates
(venv) $ touch login.html
```
Update the `login.html` file with the following:

```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_context %}
<div class="row">
    <div class="col-md-12 text-center">
        <h1>{{title}}</h1>
    </div>  
</div>
<div class="row">
    <div class="col-sm-4">
        <!-- Empty column -->
    </div>
    <div class="col-sm-4 my-form">
        {{ wtf.quick_form(form) }}
    </div>
    <div class="col-sm-4">
        <!-- Empty column -->
    </div>
</div>
{% endblock %}
```
Notice how similar the design of the login template is to that of the registration template. To render it, we will create another view function to handle the login form.

`routes.py`: Login route
```python
# ...
from app.forms import LoginForm


@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    return render_template(
        'login.html',
        title='Login',
        form=form)
```

Navigate to the URL `http://localhost:5000/login` to see the login form.

![Login Form](/images/flask_popover/login_form.png)




