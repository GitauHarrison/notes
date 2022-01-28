# Working With Web Forms

For your reference, these are the sections included in this tutorial:

1. [Section 1: Create a simple web application](/flask_popover.md#create-a-simple-application)
2. [Section 2: Web Forms](web_forms.md)
3. [Sectin 3: Working with Databases](database.md)
4. [Section 4: User Login](user_login.md)
5. [Section 5: User Posts](user_posts.md)
6. [Section 6: Implement Popover](popover.md)
7. [Section 7: User Notifications](user_notifications.md)

Web forms are one of the most basic building blocks of web applications. They are used to collect data from a user and send it to the server. We will begin by installing the following package:

```python
(venv) $ pip install flask-wtf
```
This package is a simple integration of [Flask](https://flask.palletsprojects.com/en/2.0.x/) and [WTForms](https://wtforms.readthedocs.io/en/3.0.x/), including CSRF, file upload, and reCAPTCHA.


## Configure Environment Variables

At this point(if you are running the application locally), the application is very basic and we do not have to worry about configurations. However, to work with web forms, Flask expects the `SECRET_KEY` variable to be set. We will set it to a random string.

`config.py`: Secret key for protection against CSRF attacks
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

`app/__init__.py`: Register the configuration object
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
That's it! You should be able to run the application locally.

## Create Forms

Following the principle of _separation of concerns_, we will create a forms module that will define all the forms we want. Create an empty `forms.py` file in the `app` subdirectory.

```python
(venv) $ cd app
(venv) $ touch forms.py
```
Ensure that you are in the `app` subdirectory before creating the file.

### Registration Form

In the forms module, we can define the structure of the registration form. 

`app/forms.py`: User registration form
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

With the `email` field in place, we have used `Email()` to validate the data entered by a user. Flask expects `email-validator` package to be installed for this validation to work. To install it, run the following command:

```python
(venv) $ pip3 install email-validator
```

To display this form, we will do so in `register.html` file. Our application currently has all templates in the `templates` subdirectory. Create an empty `register.html` file in the `templates` subdirectory.

```python
(venv) $ cd templates
(venv) $ touch register.html
```

Flask-Bootstrap provides for a quick method to create and display formatted forms using the `quick_form()` function.

`app/templates/register.html`: User registration form
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

`app/routes.py`: Register route
```python
from app.forms import RegistrationForm
from flask import render_template

# ...


@app.route('/register', methods=['GET', 'POST'])
def register():
    form = RegistrationForm()
    return render_template(
        'register.html',
        title='Register',
        form=form)

```

The `register()` view function is used to render the registration form. The `form` object is passed to the `quick_form()` method, without which the form won't be displayed.

Navigate to the URL `http://localhost:5000/register` to see the registration form.

![Registration Form](/images/flask_popover/registration_form.png)


### Login Form

To authenticate users, all we will want a user to do is enter their username and password. We will create a login form where they will fill in their credentials.

`app/forms.py`: User login form
```python
# ...
from wtforms import BooleanField


class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField(
        'Password', validators=[DataRequired(), Length(min=8, max=30)])
    remember_me = BooleanField('Remember Me')
    submit = SubmitField('Submit')
```

Again, we need to create a `login.html` file to display the login form.

```python
(venv) $ cd templates # skip this if you are in templates/
(venv) $ touch login.html
```
Update the `login.html` file with the following:

`app/templates/login.html`: User login form
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

`app/routes.py`: Login route
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

To improve a user's experience when navigating through the application, let us attach a link to the registration page within the login form.

`app/templates/login.html`: Login form with link to registration page
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
        <p>
            New here?
            <a href="{{ url_for('register') }}">Register</a>
        </p>
    </div>
    <div class="col-sm-4">
        <!-- Empty column -->
    </div>
</div>
{% endblock %}
```
Now, instead of appending "register" to the URL, a user can simply click the "Register" link to sign up.

![Login with register link](/images/flask_popover/login_with_register_link.png)