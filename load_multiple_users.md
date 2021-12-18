# Load Users in Different Database Tables in a Flask Application

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

## Working With a Database

Once a student or a teacher has registered we will need to store that information in a database. This will allow the application to remember the user's information during their subsequent visit.

Let us begin by installing two packages that will allow us to work with a database.

```python
(venv) $ pip3 install flask-sqlalchemy flask-migrate
```
`Flask-sqlalchemy` is a popular Object Relational Mapper (ORM) for SqlAlchemy. Instead of using raw SQL commands to interact with the database, this package uses high-level classes, objects and methods.

`Flask-migrate`, on the other hand, is used to manage the database. Should there be any change to the database tables, then `flask-migrate` is responsible for creating and managing these migrations.

### Register Database Packages

Just like the other packages, ensure that register them in the application instance.

`__init__.py: Register the Database Packages`
```python
# ...
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate


# ...
db = SQLAlchemy(app)
migrate = Migrate(app, db)

# ...
from app import models # <--- update
```

### Configure the Database

Before we can create our database, let us create some configurations that will be needed by the application in order to work with our database of choice, SQLite.

`config.py: Create the Database Configuration`
```python
import os

basedir = os.path.abspath(os.path.dirname(__file__))


class Config(object):
    # Database
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or\
        'sqlite:///' + os.path.join(basedir, 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```
SQLite is perfect for the needs of our application at the moment. It if convinient to use with applications that are small and simple. However, if we were to use this application with a larger scale application, we would want to use a database that is more suitable and robust for larger scale applications.

### Create the Database
Now we can create a `models.py` file in the `app` sub-directory to define our database.

```python
(venv) $ touch app/models.py
```
`models.py: Define the Database Model`
```python

from app import db


class Student(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))

    def __repr__(self):
        return f'Student: {self.username}'

```

We have defined a `Student` model with four columns. The first column is always an `id` of type Integer. The other three columns are strings. It is recommended to not store a user's password in its original form. Rather, it should be stored as a hash, a representation of itself. This is an additional security measure in the event that the database is compromised and the users' information is expose.

### Apply the Changes Made to the Database

Since we have just defined a new model, we need to apply this change by creating a migration script for the student. On our terminal, let us run the following commands:

```python
(venv) $ flask db init

# Output
Creating directory /home/harry/software_development/python/current_projects/load_multiple_users/migrations ...  done
  Creating directory /home/harry/software_development/python/current_projects/load_multiple_users/migrations/versions
  ...  done
  Generating /home/harry/software_development/python/current_projects/load_multiple_users/migrations/script.py.mako ...  done
  Generating /home/harry/software_development/python/current_projects/load_multiple_users/migrations/alembic.ini ...  done
  Generating /home/harry/software_development/python/current_projects/load_multiple_users/migrations/README ...  done
  Generating /home/harry/software_development/python/current_projects/load_multiple_users/migrations/env.py ...  done
  Please edit configuration/connection/logging settings in
  '/home/harry/software_development/python/current_projects/load_multiple_users/migrations/alembic.ini' before
  proceeding.
```

This command will create a _migrations_ folder in the application's top-level directory. All database migrations scripts will be stored in this folder. 

Next is to create a migration script for the student. This script will contain all the details of the student table.

```python
(venv) $ flask db migrate -m "student table"

# Output
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'student'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_student_email' on '['email']'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_student_username' on '['username']'
  Generating /home/harry/software_development/python/current_projects/load_multiple_users/migrations/versions/92f4003438
  7a_students_table.py ...  done
```

You will notice that the file `app.db`, defined in our configurations, is created in the top-levle directory. Inspecting the _migrations_ folder, you will see a new sub-folder called _versions_. This folder will contain all the migration scripts that we will create. You will notice that a script called _92f4003438
  7a_students_table.py_ has been added. The number _92f4003438
  7a_ will be different in your case.

To apply these changes, we will need to run the following command:

```python
(venv) $ flask db upgrade

# Output
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 92f40034387a, students table
(load_multiple_users) harry@harry:~/software_development/python/current_projects/lo
```

**Remember that for you to use the command `flask`, you need to be in the top-level directory of the application.**

We need to do the same for the teacher table.

`models.py: Teacher Model`
```python
# ...

class Teacher(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))

    def __repr__(self):
        return f'Teacher: {self.username}'
```

Since this is a change to our database, we need to create a migration script for the teacher table.

```python
(venv) $ flask db migrate -m "teacher table"


# Output
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'teacher'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_teacher_email' on '['email']'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_teacher_username' on '['username']'
  Generating /home/harry/software_development/python/current_projects/load_multiple_users/migrations/versions/07e7001b07
  bd_teachers_table.py ...  done
```

Then apply the changes to the database:

```python
(venv) $ flask db upgrade


# Output
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade 92f40034387a -> 07e7001b07bd, teachers table
```

That's it! We have now created a database with two tables, `Student` and `Teacher`.