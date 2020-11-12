The completed project used in this article can be referenced [here](https://github.com/GitauHarrison/personal-blog-tutorial-project/commit/b388b9099738c53b44650ef8e99a4364a923964a).

In the previous chapter, you learnt how to work with flask templates. In this chapter, I will show you how you can create flask web forms. Web forms are an integral part of a web application. We will use web forms to allow visitors of our blog to post comments on each article.

### Introduction to WTForms

[WTForms](https://wtforms.readthedocs.io/en/2.3.x/) is a Python library that we will use to help render and validate forms. Flask has an extension called [Flask-WTForms](https://pythonhosted.org/Flask-WTF/) which help integrate Flask and WTForms. This the the first flask extension that we will use, among many more. Let us go ahead and install it in our virtual environment:

```python
$ pip3 install flask-wtf
```
Previously, we created a `config.py` file in the application's root directory. This file will contain all the configurations we need for our applications. The very first configuration we will need is called the SECRET_KEY. This configuration variable will be used by Flask_WTform to help secure our forms against a nasty attack called [CSRF](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (pronounced as 'sea-surf'). 

In our `config.py` file, let us update it to include this new security variable.

config.py: Add configuration variable
```python
import os

class Config(object):
    SECRET_KEY=os.environ.get('SECRET_KEY') or 'you-cannot-guess-this'
```

As the name suggests, this should be secret, and should only be known to trusted maintainers of the application. The value of the SECRET_KEY is set to two, separated by the `or` operator. The first value is obtained from an envrionment variable called SECRET_KEY. If this value does not exist, then the application will use the value to the right of the `or` operator. This value can be anything. I have used `you-cannot-guess-this` for now. 

At the moment, since we are developing the application, security concerns are low, hence the values used for the SECRET_KEY are sufficient. But upon deployment, I will set a hard-to-guess value for the SECRET_KEY to maximize on the application's security. 

With the config file in place, we need to register this configuration with the application. The application needs to read and apply it.

app/__init__.py: Apply flask configuration
```python
from flask import Flask
from config import Config

app = Flask(__name__)
app.config.from_object(Config)

from app import routes
```

Notice that I am importing Flask (capital 'F') from flask (lower case 'f'). flask (with a lower case is the package name from which we get Flask). Similarly, I am importing Config(with uppercase 'C') from config (lower case 'c'). config (with a lower case) is the module name available in our root directory (the `config.py` file). Config (with uppercase 'C') is the class name used in our config module. This may seem a bit confusing at first, but over time it will start to make sense.

### Comments Form

Flask_WTF uses classes to represent web forms. A form class basically defines the fields we will use in a form.

Let us create a `forms.py` file in our app subfolder that we will use to define our form.

```python
touch app/forms.py
```

app/forms.py: Define the comments form
```python
from flask_wtf import FlaskForm
from wtforms import StringField, TextAreaField, SubmitField
from wtforms.validators import DataRequired, Email

class CommentForm(FlaskForm):
    username = StringField('Username', validators = [DataRequired()])
    email = StringField('Email', validators = [DataRequired(), Email()])
    comment = TextAreaField('Comment', validators = [DataRequired()])
    submit = SubmitField('Post')
```
Flask extensions conventionally use the format `flask_<extension-name>` during imports. Flask_WTF uses `flask_wtf` from which we import `FlaskForm`. All the four fields in our form are imported directly from the Flask-WTF package.

The `validators` argument is optional. We have used `DataRequired` to ensure that whenever a user wishes to post a comment, then they have to fill in each field that contain that value. 

### Form Template

We have now defined how our form will be. The next step is to render this form in a comments template. 

```python
touch app/templates/comments.html
```
app/templates/comments.html: Render Comments form

```html
{% extends 'base.html' %}

{% block content %}
    <h1>Comments</h1>
    <form action="" method="POST" novalidate>
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}
            {{ form.username(size=32) }}
        </p>
        <p>
            {{ form.email.label }}
            {{ form.email(size=32) }}
        </p>
        <p>
            {{ form.comment.label }}
            {{ form.comment(size=64) }}
        </p>
        <p>
            {{ form.submit() }}
        </p>
    </form>
{% endblock %}
```
Here, you can see that I am reusing the `base` template. The reason for this is so that we can inherit all the features, styles and layout that define our application. You will see me do this all the templates.

The HTML `<form>` element is used to render web forms. I have used the `action` attribute to tell the browser which URL to submit the form when the submit button is clicked. When the value is set to empty, the URL to be used will be the one in the address bar, the one that rendered the form. 

The HTTP method I want to use to submit the form data is `POST`. The default HTTP method to send the form is a `GET` request. but using the `POST` method provides for a better user experience such that the form data is submitted in the body of the form. When you use the `GET` method, the form fields are added to the URL, making it cluttered. 

The `novalidate` tells the web browser not to apply any validation to the form fields.This now allows Flask to handle all form validation. 

`form.hidden_tag()` is used to protect against CSRF attacks. It generates a hidden field that includes a token that enhance form protection. To protect a web form, all you have to do is to ensure you have the SECRET_KEY configured and the `form.hidden_tag()` used in the form. Flask_WTF does the rest for you.

`{{ form.<field_name>.label }}` is used to display the label of a field while `{{ form.<field_name>(size=32) }}` displays the actual field. 