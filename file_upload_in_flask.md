# Handling File Upload in Flask

Most web application allow for file uploads. In this article, I will show you how you can implement that feature in a flask app.

### Flask Application Structure

![App structure](/images/file_upload_structure.png)

Create the application's structure as shown above using the `mkdir` and `touch` commands in your terminal. For example:

```python
$ mkdir my_new_project # this creates an empty directory in the current working directory
$ touch my_new_file # this creates an empty file in the current working directory
```

### Useful dependancies

Make sure to be working in a virtual environment before you begin the project. Create and activate your virtual environment as follows:

```python
$ mkvirtualenv venv # I am using virtualenvwrapper
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
(venv)$ pip3 install python-dotenv # load flask environment variables
```

#### What We Will Do

This article covers:

1. Accepting file uploads
2. File Submission in Flask
3. Securing uploaded files
4. Consuming uploaded files

**NOTE: The assumption is you have a basic understanding of how to create forms manually, as I will not be reviewing manual creation of forms, but rather focus on how to render forms that are styled using `flask-bootstrap`.**

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

Browse the file submission code [here](https://github.com/GitauHarrison/handling-file-uploads-in-flask/commit/3fb49d3f3c46aba655cee2aed76252da4223fefd).

### 3. File Upload Security

A rule of thumb when building web application is that data submitted by clients should never be trusted. Hence, the need for strict form validation. This data could maliciously be intended to crash the server by because the file is too large that the disk space in the server is completely filled, or the file is named in a manner that it tricks the server into rewriting system configuration files.

Basic steps we can take to protect our application would be:

* Limiting the size of files uploaded to the application's server
* Validating the names of the uploaded files
* Scanning the file content before submission

#### Limit File Size

Flask provides an option to limit the file size a client can upload. In `config.py`, we can add this configuration:

```python
MAX_CONTENT_LENGTH = 1024 * 1024 # equal to 1 MB
```
If you try to upload a file larger than 1 MB, the application will now refuse it. The server will return a `413` Request Entity Too Large error because the data value transmitted exceeds the capacity limit.

#### Validating Filenames

We can implement very simple form validation to check that the file extension used by clients are accepted by the application. `flask-wtf` provides validators such as `FileAllowed` and `FileRequired` when creating forms. Look at the example below:

```python
file = FileField('File', validators=[FileRequired()])
```

For our appliaction, we will list all allowed extension in the `config.py` file as seen below:

```python
UPLOAD_EXTENSIONS = ['.jpg', '.png', '.gif']
```

These are the current configurations:

`config.py: Securing file uploads`
```python
import os


class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
    MAX_CONTENT_LENGTH = 1024 * 1024
    UPLOAD_EXTENSIONS = ['.jpg', '.png', '.gif']

```

Let us implement this configiration in our `index()` method in `routes.py`:

```python
from app import app
from flask import render_template, url_for, request, redirect, abort
from app.forms import UploadForm
import os


@app.route('/', methods=['GET', 'POST'])
@app.route('/index', methods=['GET', 'POST'])
def index():
    form = UploadForm()
    if request.method == 'POST':
        # start of update
        uploaded_file = request.files['file']
        filename = uploaded_file.filename
        if filename != '':
            file_ext = os.path.splitext(filename)[1]
            if file_ext not in app.config['UPLOAD_EXTENSIONS']:
                abort(400)
                # end of update
            uploaded_file.save(uploaded_file.filename)
        return redirect(url_for('index'))
    return render_template('index.html', title='Home', form=form)

```

Files whose extensions are not configured will return a `400` Bad Request error when a client tries to upload them. The application checks for the file in the `FileField` and extracts the extension. If the file extension does not exist in the application configuration, flask aborts the upload are returns the 400 error.

Another way to secure files based on their names would be to use Werkzeug's [secure_filename](https://werkzeug.palletsprojects.com/en/1.0.x/utils/#werkzeug.utils.secure_filename) function. No matter how complicated or malicious a filename is, `secure_filename()` reduces it to a flat filename.

`routes.py: using secure_filename function`

```python
from app import app
from flask import render_template, url_for, request, redirect, abort
from app.forms import UploadForm
import os
from werkzeug.utils import secure_filename # <--------- new update


@app.route('/', methods=['GET', 'POST'])
@app.route('/index', methods=['GET', 'POST'])
def index():
    form = UploadForm()
    if request.method == 'POST':
        uploaded_file = request.files['file']
        filename = secure_filename(uploaded_file.filename)  # <--------- new update
        if filename != '':
            file_ext = os.path.splitext(filename)[1]
            if file_ext not in app.config['UPLOAD_EXTENSIONS']:
                abort(400)
            uploaded_file.save(os.path.join(app.config['UPLOAD_PATH'], filename))  # <--------- new update
        return redirect(url_for('index'))
    return render_template('index.html', title='Home', form=form)

```
Every time a new upload is made, we now want the files to be organized and stored in one location within the top-level directory when the `save()` method is called. Go ahead and create an `uploads` directory in the application's root directory. Then update your configration to allow for storage in the newly created folder, as follows:

`config.py: path to uploads folder`

```python
import os


class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
    MAX_CONTENT_LENGTH = 1024 * 1024
    UPLOAD_EXTENSIONS = ['.jpg', '.png', '.gif']
    UPLOAD_PATH = 'uploads'  # < ------- new update

```

#### Validate File Content

Say your application allows only for __image files__ to be uploaded. It is possible to first check the content of the uploaded file before accepting it.

Python provides the [imghdr](https://docs.python.org/3/library/imghdr.html) package to validate that the file's header is actually an image file.

`routes.py: validate file content`

```python
from app import app
from flask import render_template, url_for, request, redirect, abort
from app.forms import UploadForm
import os
import imghdr
from werkzeug.utils import secure_filename


# new update
def validate_image(stream):
    header = stream.read(512)
    stream.seek(0)
    format = imghdr.what(None, header)
    if not format:
        return None
    return '.' + (format if format != 'jpeg' else 'jpg')


@app.route('/', methods=['GET', 'POST'])
@app.route('/index', methods=['GET', 'POST'])
def index():
    form = UploadForm()
    if request.method == 'POST':
        uploaded_file = request.files['file']
        filename = secure_filename(uploaded_file.filename)
        if filename != '':
            file_ext = os.path.splitext(filename)[1]
            # updated
            if file_ext not in app.config['UPLOAD_EXTENSIONS'] or \
                    file_ext != validate_image(uploaded_file.stream):
                # end of update
                abort(400)
            uploaded_file.save(os.path.join(app.config['UPLOAD_PATH'], filename))  # <--------- new update
        return redirect(url_for('index'))
    return render_template('index.html', title='Home', form=form)

```

`validate_image()` function reads 512 bytes of data when a file is first uploaded, and quickly resets the pointer to 0 before the file is saved. 512 bytes is sufficient data to identify an image.

`imghdr.what()` function looks at the data stored in memory if the first argument is `None`, and then passed that data to the second argument. This function will primarily return the _image format_. If an unknown format is detected (the function supports a variety of image formats such as `jpeg`, `gif` etc), it returns a `None` value. 

If an image format is detected, the function will return the format. However, to make good use of the returned file format, we'd rather have our application return the file extension. We add `.` for all image formats except `jpeg`, which normally uses `jpg`.

Browse the complete code for securing files [here](https://github.com/GitauHarrison/handling-file-uploads-in-flask/commit/e27bcfb72514c1c6185acdbb1c5449d7ab752d63).

### Consuming Uploaded files

To be able to make use of the uploaded files, we need to access the _uploads_ folder, which is currently located at the top-level directory. Normally, all static files are organized in a static sub-folder inside the _app_ folder.

One advantage we will have working with the uploads folder outside the static sub-folder is that we can create a route specifically used to access the folder contents. Being able to work with a route directly allows for the use of `@login_required` to help protect the contents inside the folder.

Let us go ahead an update our `routes.py` file to access the uploads folder content:

`routes.py: access folder content`

```python
from flask import send_from_directory


@app.routes('/uploads/<filename>')
def upload(filename):
    return send_from_directory(app.config['UPLOAD_PATH'], filename)

```

Additionally, we need to send the file contents to our `index()` function  for rendering.

`routes.py: render content from uploads directory`

```python
@app.route('/', methods=['GET', 'POST'])
@app.route('/index', methods=['GET', 'POST'])
def index():
    form = UploadForm()
    files = os.listdir(app.config['UPLOAD_PATH'])  # <------- new
    if request.method == 'POST':
        uploaded_file = request.files['file']
        filename = secure_filename(uploaded_file.filename)
        if filename != '':
            file_ext = os.path.splitext(filename)[1]
            if file_ext not in app.config['UPLOAD_EXTENSIONS'] or \
                    file_ext != validate_image(uploaded_file.stream):
                abort(400)
            uploaded_file.save(os.path.join(app.config['UPLOAD_PATH'], filename))
        return redirect(url_for('index'))
    return render_template('index.html', title='Home', form=form, files=files)
```

To display the loaded files, we need to loop through all available content in the _uploads_ folder.

`index.html: display content in uploads folder`

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

    <!-- display content in uploads folder -->
    <div class="row">
        <div class="col-sm-12 col-md-4 col-lg-2">
            {% for file in files %}
                <img src="{{ url_for('upload', filename=file) }}" style="width: 64px;">
            {% endfor %}
        </div>
    </div>
    <!-- end of display -->
   
{% endblock %}
```

When you reload your page, you should be able to see tumbnails of the images from your _uploads_ directory.

### The Miracle of Dropzone.js

[dropzone.js](https://www.dropzonejs.com/) is a popular upload client. So far, we have been using default browser widget to do the uploading. This JS client has the ability to show upload progress when uploading file, a useful feedback especially when uploading large files.

To incorporate it in our flask form, we first need to load dropzone CSS and Javascript from a CDN in our `base.html` file:

`base.html: load dropzone CSS and JS`

```html
<!-- Load CSS, place after head block -->
{% block styles %}
    {{ super() }}
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/dropzone/5.7.1/min/dropzone.min.css">
{% endblock %}
<!-- End CSS -->

<!-- JS block is placed at the bottom -->
{% block scripts %}
    {{ super() }}    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/dropzone/5.7.1/min/dropzone.min.js"></script>
{% endblock %}
<!-- End JS -->

```
Then update `forms.py` file to render the `dropzone` class as a keyword argument to the `FileField`:

```python
class UploadForm(FlaskForm):
    file = FileField('Upload File', render_kw={"class": "dropzone"})
```

This is what you get:

![Dropzone.js](/images/unattractive_dropzone_js.png)

To get rid of the browser widget, we will resort to manually creating our form without using `flask-bootstrap` as shown below:

`index.html: manually create form`

```html
<div class="row">       
        <div class="col-sm-12 col-md-4 col-lg-8">
            <!-- {{ wtf.quick_form(form) }} -->
            <form action="{{ url_for('index') }}" class="dropzone" method="POST" enctype="multipart/form-data" novalidate>
                {{ form.hidden_tag() }}                        
            </form>
        </div>            
    </div><hr>
```
Note that the action attribute has been set to load from the `index()`  method URL. 

The enctype attribute in the `<form>` element is normally not included with forms that don't have files. This attribute defines how the browser should format the data before it is submitted to the server. 

`multipart/form-data` is a format that is required when at least one of the fields in the form is a file field.

With that, you can now drag and drop files into the form and it will automatically be downloaded to the _uploads_ folder, with indicators for both progression and status of the final upload(whether successful or not).

One thing you will not is that when the uploaded file fails either of our application's configurations setting, dropzone does make an effort to display a message. This message, however, is not easily readable.

![Dropzone Error Message](/images/dropzone_error_msg.png)

The 413 File too Large error message is generated by Flask when the payload exceeds the capacity limit. We can overide the default error message be creating our own custom error message.

`errors.py: create custom error message`

```python
from app import app


@app.errorhandler(413)
def file_too_large(error):
    return 'File is too large', 413

```

What about when the file upload process fails the other configurations other than too large a file? We need to update our `index()` function to accomodate this.

```python
if file_ext not in app.config['UPLOAD_EXTENSIONS'] or \
                    file_ext != validate_image(uploaded_file.stream):
                return 'Invalid image', 400
```
That's it! Your flask application can now safely consume file uploads.

You can browse [the completed projet on GitHub](https://github.com/GitauHarrison/handling-file-uploads-in-flask).