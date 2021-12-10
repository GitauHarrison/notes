# Add Uploaded Files to Your Flask Database

It is common to see applications that allow users to add and update their avatars. This, obviously, is not the only way users can upload images to an application. During this article, I will show you how you can allow users to upload images to their accounts. These images will be accessed by quering the database used by the application.

## What We Will Do

1. Create a simple application
3. Allow users to upload images to their accounts
4. Display all users and their images in the application

## Create a Simple Application

I have already created simple flask application. You can find it in [this repository](https://github.com/GitauHarrison/starting-a-flask-server). Working with Flask is fairly simple and straightforward. If you are interested in learning how to create a simple flask application, you can learn [here](https://github.com/GitauHarrison/notes/blob/master/start_flask_server.md).

## Working with Web Forms

Forms allow an application to collect information from a user. The information is then used to create a new object in the database. More often than not, this is a very common feature of web applications.

Flask provides us a simple way to create web forms. To begin, we will first need to install the [Flask-WTF](https://pypi.python.org/pypi/Flask-WTF) package in our virtual environment.

```python
(file_upload) $ pip install flask-wtf
```

Following the _principle of separation of concerns_, I will go ahead and create a `forms` modile in our application instance. This module will contain the form classes that we will use to collect information from the user. I will begin by creating an empty `forms.py` file.

```python
(file_upload) $ touch app/forms.py
```

### SECRET_KEY Token

Flask expects a `SECRET_KEY` variable to be defined in the `app.config` file. This is used to generate the `csrf_token` for the form. The key is necessary to prevent cross-site request forgery. I will create a `SECRET_KEY` variable in the `app.config` file.

`config.py: Add secret key`
```python
import os


class Config(object):
    # Form
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
```

The `SECRET_KEY` variable is first sourced from an environment variable. If the environment variable is not defined, then the `SECRET_KEY` is set to `you-will-never-guess`. This is a fallback value to ensure that our application does not crash.

As the name suggests, the value of this variable is secret. When committing the application to version control, you should not commit the `SECRET_KEY` value. The `.env` file is normally used to store sensitive information such as keys and passwords. This file should be created in the top-level directory.

```python
(file_upload) $ touch .env
```

Update the `.env` file with the following information:

```python
SECRET_KEY=b'\x97]\xe8\x86\xc6"\xc2\xc9\xea<\\+ \x04~@'
```

The value of the `SECRET_KEY` variable should be as difficult to guess as possible. I was able to generate this value using the following command in the terminal:

```python
(file_upload)$ python3 -c 'import os; print(os.urandom(16))'
```

The application has no idea of this configuration, so we need to tell it by registering it in the application instance.

`__init__.py: Register secret key`
```python
from flask import Flask
from flask_bootstrap import Bootstrap
from config import Config

app = Flask(__name__)
app.config.from_object(Config)

boostrap = Bootstrap(app)

from app import routes, errors

```

### Define Form

With all the necessary web forms configuration set, I will then create a `UserForm` class in the `forms` module. It will define all the fields that we will need in our form.

`forms.py: UserForm`
```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField, TextAreaField
from wtforms.validators import DataRequired, Length, Email
from flask_wtf.file import FileField, FileAllowed, FileRequired


class UserForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired(), Length(min=2, max=20)])
    email = StringField('Email', validators=[DataRequired(), Email()])
    about_me = TextAreaField('About Me', validators=[DataRequired()])
    avatar = FileField('Avatar', validators=[FileRequired(), FileAllowed(['jpg', 'png'], 'Images only!')])
    submit = SubmitField('Update')

```

The most crucial field in the form is the `avatar` field. This field is defined using the `FileField` class. This field will allow a user to upload an image. The `FileAllowed` validator will ensure that the file uploaded is an image. If any other file type is uploaded, the form will refuse to accept that data and display the message "Images only!".

### Display Form

Our `index.html` file will be used to display the form I have defined above. At the moment, this application makes use of `flask-bootstrap`. I will use this package to quickly display a rather nice looking form.

`index.html: Display Form`
```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_context %}
    <div class="row">
        <div class="col-md-12">
            <h1> {{ title }} </h1>
    </div>
    <div class="row">
        <div class="col-md-6">
            {{ wtf.quick_form(form) }}
        </div>
    </div>
    <hr>
    <div class="row">
        <div class="col-sm-12">
            <table class="table table-striped">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Username</th>
                        <th>Email</th>
                        <th>About Me</th>
                        <th>Avatar</th>
                    </tr>
                </thead>
                <tbody>
                     <tr>
                        <td>#</td>
                        <td>#</td>
                        <td>#</td>
                        <td>#</td>
                        <td>#</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
{% endblock %}
```

I will want to show all the users in the database in a table. Later on, the `body` section of the table will be populated with the data from the database. Meanwhile, I have used `#` as a placeholder for the data.

The `form` variable in `wtf.quick_form()` will be defined in the view function that will render this page.

### Render Form

I will make use of the existing `index()` view function to render the form. 

`index.html: Render Form`
```python
from app import app
from flask import render_template
from app.forms import UserForm


@app.route('/', methods=['GET', 'POST'])
@app.route('/index', methods=['GET', 'POST'])
def index():
    form = UserForm()    
    return render_template(
                           'index.html',
                           form=form,
                           title='User Form',
                           )
```

You should be able to see this:

![Home Page](images/file_uploads/home_page_form.png)