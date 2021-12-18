# Load Multiple Users in a Flask Application

Imagine you have a database table of students and another database table of teachers. Let us imagine again that the application we are interested in building should be able to load all the students and all the teachers, separately. How do you go about it? 

I struggled with this problem for quite a while during my initial days of learning Flask. I knew how to load all students, and say, all their posts. However, when the need to create a teachers table came up, I fambled, and failed. Several months down the line, I revisited this concept with the  courage that I could figure it out. And this tutorial is the outcome of that effort.

## What We Will Do

1. Create a simple flask application
2. Add students to the application
3. Add teachers to the application
4. Load all students and teachers

## Getting Started

I have already created a simple application. You can browse [this GitHub repository](https://github.com/GitauHarrison/starting-a-flask-server) to learn more. We will build on it moving forward.

## Working with Web Forms

Web forms are used to collect user information from a web application. We are going to create two types of users: students and teachers. This means that we will need to create a registration form for each user type. Upon successful registration, each user should be able to log into their own account.

Flask provides us with the `flask-wtf` package that allows for the creation of webforms. We will need to install it in our virtual environment.

```python
(venv) $ pip3 install flask-wtf
```

### Create a Registration Form

Following the principle of _separation of concerns_, we will create a `forms` module with will hold all the forms our application will need. Inside `app/` subfolder, create an empty form called `forms.py`.

```python
(venv) $ mkdir app/forms.py
```

We will begin by creating a registration form. This form will collect the same information from both students and teachers.

`forms.py: Create a Registration Form`
```python

from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, SubmitField
from wtforms.validators import DataRequired, Length, Email, EqualTo


class RegisterForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired(), Length(min=3, max=20)])
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField('Password', validators=[DataRequired()])
    confirm_password = PasswordField('Confirm Password', validators=[DataRequired(), EqualTo('password')])
    submit = SubmitField('Register')
```

`Flask-wtf` provides several fields and validators that we have used to create the registration form. The first is the `StringField` which is used to collect a string of characters. The second is the `PasswordField` which is used to collect a string of characters. The third is the `EqualTo` validator which is used to compare the value of the password field to the value of the confirm password field. `DataRequired` ensures that the field needs to be field, otherwise clicking the submit button won't work. The last field is the `SubmitField` which is used to submit the form.

### Display the Registration Form

Within the `app/templates`, we will create a `register.html` file. This file will contain the registration form. I will use `flask-bootstrap` to create a quick form.

```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_context %}
<div class="row">
    <div class="col-sm12">
        <h1>{{ title }}</h1>
        <p>
            {{ wtf.quick_form(form) }}
        </p>
    </div>
</div>
{% endblock %}
```

In a single line of code, we are able to display the registration form. We are using the `wtf.quick_form` macro to create the form. The `wtf.quick_form` function takes in a form as an argument. We will use the `form` variable to store the form. The good thing here is that our form has all the styles needed to look good, thanks to the Bootstrap framework.

### Render the Registration Form

To see this form, we will need to create a route that will render the registration form. First, we will create a registration route that handles the registration of a student. Then  we will create a route that handles the registration of a teacher.

`routes.py: Create a Registration Route`
```python
# ...
from app.forms import RegisterForm


# ...
@app.route('/register/student', methods=['GET', 'POST'])
def register_student():
    form = RegisterForm()
    return render_template(
        'register.html',
        title='Register Student',
        form=form
        )
```

To easily access this view function, we will need to update our navigation bar in `base.html` to include a link to student registration.

`base.html: Link to Student Registration`
```html
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
            <a class="navbar-brand" href=" # ">Users</a>
        </div>
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">            
            <ul class="nav navbar-nav navbar-right">
                <li><a href=" {{ url_for('register_student') }} ">Register Student</a></li>
            </ul>                       
        </div>
    </div>
</nav>
{% endblock %}
```

If you try to access the registration page at this point, you will be greeted with a `RuntimeError` because we have not provided a secret key. All webforms require a secret key to prevent cross site scripting attacks. We will need to create a secret key in `config.py`.

`config.py: Create a Secret Key`
```python
import os


class Config(object):
    # Form
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'

```
We are sourcing the value of the `SECRET_KEY` from an environment variable. If the variable is not set, we will use the string `you-will-never-guess`. 

The next step will be to register this configuration module within the application instance.

`__init__.py: Register the Configuration Module`
```python
# ...
from config import Config

# ...
app.config.from_object(Config)

# ...
```

To add this enviroment variable, we will do so in the `.env` file. Create this file in the top-level directory of our application.

```python
(venv) $ touch .env
```

Update the `.env` file with the following content:

```text
SECRET_KEY=b'\xa4\x87\xd9%U\xa5@$\x1frs\x125\x9f\xe4:'
```

As the name suggests, this value should be secret, and hard to guess. I was able to obtain this value by running the following command in the terminal:

```python
(venv) $ python3 -c "import os; print(os.urandom(16))"
```

Now, reloading the application will display the registration form.

![Student Registration Form](images/load_multiple_users/register_student.png)

To register a teacher, we will need to create a route that handles the registration of a teacher. We will use the same template as the student registration route. Why so? Can't we use the same route if the template is the same? That will be discussed in subsequent sections.

`route.py: Teacher Registration Route`
```python
# ...
@app.route('/register/teacher', methods=['GET', 'POST'])
def register_teacher():
    form = RegisterForm()
    return render_template(
        'register.html',
        title='Register Teacher',
        form=form
        )
```

Add a link in the navigation bar to the teacher registration page.

`base.html: Add a Link to Teacher Registration`
```html
<!-- Previous code -->
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
            <a class="navbar-brand" href=" # ">Users</a>
        </div>
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">            
            <ul class="nav navbar-nav navbar-right">
                <li><a href=" {{ url_for('register_student') }} ">Register Student</a></li>
                <li><a href=" {{ url_for('register_teacher') }} ">Register Teacher</a></li>
            </ul>                       
        </div>
    </div>
</nav>
{% endblock %}
```