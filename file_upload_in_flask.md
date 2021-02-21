# Handling File Upload in Flask

Most web application allow for file uploads. In this article, I will show you how you can implement that feature in a flask app.

### Flask Application Structure

![App structure](/images/file_upload_structure.png)

### Useful dependancies

Make sure to be working in a virtual environment before you begin the project. Create and activate your virtual environment as follows:

```python
$ mkvirtualenv venv #I am using virtualenvwrapper
```
Learn how to set up and use virtual environment wrapper [here](virtualenvwrapper_setup.md).

Then, install `flask`:

```python
$ pip3 install flask
```

Since the application will make use of forms, install `flask-wtf`:

```python
(venv)$ pip3 install flask-wtf
```

Other useful dependancies can be:

```python
(venv)$ pip3 install flask-bootstrap # cross-site responsiveness
(venv)$ pip3 install python-dotenv # load flask server quickly
```

#### What We Will Do

This article covers:

1. Accepting file uploads
2. File Submissions in Flask
3. Securing uploaded files
4. Consuming uploaded files

**NOTE: The assumption is you have a basic understanding of how to create forms manually, as I will not be reviewing manual creation of files, but rather focus on how to render forms that are styled using `flask-bootstrap`.**

### 1. Acccepting File Uploads

Let us set up our application to display a flask web form as follows:

`__init__.py: create an instance of the application`
```python
from flask import Flask
from flask_bootstrap import Bootstrap

app = Flask(__name__)
bootstrap = Bootstrap(app)

from app import routes, errors

```

Here, we initialize the application by first creating an instance of the `flask` app and other useful dependancies.

`base.html: template for our base style`
```html
{% extends 'bootstrap/base.html' %}

{% block title %}
    {% if title %}
        {{ title }} - File Upload
    {% else %}
        Welcome to Flask
    {% endif %}

{% endblock %}

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
                <a class="navbar-brand" href="{{ url_for('index') }}">File Upload</a>
            </div>
            <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
                <ul class="nav navbar-nav">
                    <li><a href="{{ url_for('index') }}">Home</a></li>                    
                </ul>                
            </div>
        </div>
    </nav>
{% endblock %}

{% block content %}
    <div class="container">
        <!-- You can add more features to your app here, such as message flashing -->
        {% block app_content %}{% endblock %}
    </div>
{% endblock %}

```

Our `base` template inherits all the styles from the `flask-bootstrap`'s `base.html` template. Noteworthy, we use the block called `app_content` to hold the details of our application.

`forms.py: create form`

```python
from flask_wtf import FlaskForm
from flask_wtf.file import FileField
from wtforms import SubmitField


class UploadForm(FlaskForm):
    file = FileField('Upload File', render_kw={"class": "dropzone"})
    submit = SubmitField('Submit')

```

Since we are working with file uploads, `flask-wtf` provides `FileField` which specifically handles forms that allow for file inputs. Instead of importing `FileField` from `wtforms` as is normally the case when working with string and boolean input data, file uploads requires that we import  `FileField` from `flask_wtf.file` as seen above.

`index.html: display upload form`

```html
{% extends "base.html" %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_content %}
    <div class="row">
        <div class="col-sm-12 col-md-4 col-lg-2">
            <!-- empty -->
        </div>
        <div class="col-sm-12 col-md-4 col-lg-8">
            {{ wtf.quick_form(form) }}             
        </div>
        <div class="col-sm-12 col-md-4 col-lg-2">
             <!-- empty -->
        </div>        
    </div><hr>
   
{% endblock %}
```

`flask-bootstrap` allows us to quickly create forms using `{{ wtf.quick_form(form) }}`. This not only saves time but also adds default form styles from _Bootstrap_.

`config.py: register application configuration`

```python
import os


class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'

```

Since we are creating a web form which is most likely to make changes in our application, the first thing we want to do is to ensure that the web form is secure against [CSRF](https://owasp.org/www-community/attacks/csrf) attacks. A secret key is needed to ensure for the security of our form. 

Once this secret key is configured, we need to register it in our application instance. Update `__init__.py` file as follows:

```python
from flask import Flask
from flask_bootstrap import Bootstrap
from config import Config  # <----------- new

app = Flask(__name__)
app.config.from_object(Config)  # <----------- new
bootstrap = Bootstrap(app)

from app import routes, errors

```

`routes.py: create view function used to render our form`

```python
from app import app
from flask import render_template, url_for, request, redirect
from app.forms import UploadForm


@app.route('/', methods=['GET', 'POST'])
@app.route('/index', methods=['GET', 'POST'])
def index():
    form = UploadForm()        
    return render_template('index.html', title='Home', form=form)

```

`file_upload.py: define flask application instance`

```python
from app import app
```

With the application structure and content now complete, we can run our script by first importing it in our top-level Python script.

Flask requires that we set a few environment variables which need to run when the application is first fired up. Based on the principle of _separation of concerns_, as has been the case throughout the creation of the application, we will initialize Flask environment variables in `.flaskenv` file as seen below:

`flaskenv: initialize Flask environment variables`

```
FLASK_APP=file_upload.py
FLASK_ENV=development
FLASK_DEBUG=True
```

Believe it or not, the application is complete! Let us fire up our flask server by running the command below in our terminal:

```python
(venv)$ flask run
```
You should be able to see this:

![Accepting File Uploads](/images/accepting_file_uploads.png)
Browse the compelete code [here](https://github.com/GitauHarrison/handling-file-uploads-in-flask/commit/40a9928c1c978b7fcf878cb33dd3a0b9874a77f0).

### 2. File Submissions in Flask

Form fileds in `Flask` are generally accessed used using `request.form` dictionary. For file fields, `request.files` is used.

We need to update the `index()` function in `routes.py` to access and save files as seen below:

`routes.py: submit a file`

```python
from app import app
from flask import render_template, url_for, request, redirect
from app.forms import UploadForm


@app.route('/', methods=['GET', 'POST'])
@app.route('/index', methods=['GET', 'POST'])
def index():
    form = UploadForm()
    if request.method == 'POST':
        uploaded_file = request.files['file']
        if uploaded_file.filename != '':
            uploaded_file.save(uploaded_file.filename)
        return redirect(url_for('index'))
    return render_template('index.html', title='Home', form=form)

```
We want to send our form only when the method used is `POST`. We store a file object in the variable called `uploaded_file` which uses `request.files` dictionary to access the data in the `FileField`. If that file object is not empty, then we call the `save()` function to store that data in a local disk in the application's top-level directory. After each submission, we display the home page by redirecting the URL to the `index()` function.

Reload your page and choose a file by clicking the _Choose File_ button. Hit the _Submit_ button and you will notice that a file has been saved in you top-level directory.

Browse the file submission code [here]().