# Moderate User Comments in Flask

As an administrator of a website, one of the things you would really want to do is to moderate what users say publicly to each other. This is a very common task, and having it implemented in your Flask website can really help sanitize the contents. In this tutorial, I will show you how you can implement this feature.

## Create A Simple Flask Application

We will need a simple application to test the Flask server. I have created one already [here](https://github.com/GitauHarrison/starting-a-flask-server). We will build on it and add a comment moderation feature. If you would like to know how the application in the link was built, you can read the [start a flask server tutorial](start_flask_server.md).

## Application Configurations

To receive comments in our application, we need to provided forms that users can fill. The information that users pass through the forms will be used to populate our comments section. So, how can we make forms in Flask?

Flask provides the `flask-wtf` extension, a wrapper around the [WTForms](https://wtforms.readthedocs.io/en/3.0.x/) package. Let us install it in our virtual environment:

```python
(comment_moderation)$ pip3 install flask-wtf
```

As our application grows, certain configuratins will be needed. At the moment, we have had no need to use any configuration in our application. Most Flask applications expect certain configurations to be set. One such configuration is the `SECRET_KEY`. it is a variable whose value is used as a cryptographic key useful in the generation of signatures and tokens. Flask-WTF uses it to protect web forms against a nasty attack called [Cross Site Request Forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) or CSRF (pronounced as 'sea-surf'). This value, as the name suggests, is meant to be a secret.

If you look carefully, I have a module called `config` in the top-level directory. Following the principle of _separtion of concerns_, all the configurations that our application will need will be added here.

`config.py: Secret key configuration`
```python
import os


class Config(object):
    # Form
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'

```

I am sourcing the value of `SECRET_KEY` from the an environment variable. If the value is missing, I have provided a safe fallback by hardcoding a string. The environment variable is a secret and should not be added to version control. I will create a new file called `.env` which will have the actual value of `SECRET_KEY`. `.gitigonore` file we already have ignores this file from version control, so it is safe to create it in the top-level directory.

```python
(comment_moderation)$ touch .env
```

I will use the following method to generate the value of the SECRET_KEY:

```python
(comment_moderation)$ python3 -c 'import os; print(os.urandom(16))'

# Random Output
b'\x1eh\xfcIWC\x91\xd7\xb3\xfd\x02dK\xe0\xb5z'
```

Hard to guess, right? I will add this value to my environment variable.

`.env: Add secret configuration keys`
```python
SECRET_KEY=b'\x1eh\xfcIWC\x91\xd7\xb3\xfd\x02dK\xe0\xb5z'
```

With the variable set, we can not update our application instance to read and apply our configurations

`__init__.py: Regiser the config module in application instance`
```python
from flask import Flask
from flask_bootstrap import Bootstrap
from config import Config # < ------------ update

app = Flask(__name__)
app.config.from_object(Config) # < ------------ update

boostrap = Bootstrap(app)

from app import routes, errors

```

I have began by importing the `Config` class from the top-level directory. I have used the `app.config.from_object(Config)` to create an instance of our `Config` class. The lower name `config` comes from python to indicate the `config` module whereas the upper case is the actual `Config` class.

## Flask Web Forms

Flask-WTF uses Python classes to represent web forms. The form class defines the variable fields we would like to have in our forms.

### Define Comment Form

Let us begin by creating a simple comment form. The information we want from a user of our application are their username, their email addresses and the content they would like to share on our application. We will begin by creating an empty `forms` module in the `app/` sub-directory.

```python
(comment_moderation)$ touch app/forms.py 
```

Let is now create the class that defines all the fields we want in our comments form.

`forms.py: Create a user comment form`
```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField, TextAreaField
from wtforms.validators import DataRequired


class CommentForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    comment = TextAreaField('Comment', validators=[DataRequired()])
    submit = SubmitField('Post')

```

`flask-wtf` has a base class called `FlaskForm` which is needed. The variable fields are defined using `StringField`, `TextAreaField` and the `SubmitField` from `wtforms`. To ensure that the user does not submit empty data, we attach `DataRequired`. This will prevent them from clicking the `Post` button if any of the fields above are empty.

### Display Comment Form

The next step is to display this form in our templates. We will use the `index.html` file to display our comment form. 

`index.html`
```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %} <!-- Update -->

{% block app_context %}
    <div class="row">
        <div class="col-md-12">
            <h1>Comment Moderation</h1>
        </div>  
    </div>
    <div class="row">
        <div class="col-md-6">
            {{ wtf.quick_form(form) }} <!-- Update -->
        </div>
    </div>
{% endblock %}
```

I have used Bootstrap to quicky display my comment form, though you can manually disply the same form using HTML. 

With the form ready to be displayed, we will now update our `index()` view function to render it.

`routes.py: Render the comments form`

```python
from app import app
from flask import render_template
from app.forms import CommentForm # < ---- update


@app.route('/')
@app.route('/index')
def index():
    form = CommentForm() # < ---- update
    return render_template('index.html', form=form) # < ---- update

```

Start your flask server in the terminal by running the command `flask run`. Navigate to http://127.0.0.1:5000/ to see the comments form displayed.

![Comment Moderation Form](images/comment_moderation/comment_moderation_form.png)