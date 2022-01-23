# User Login

For your reference, these are the sections included in this tutorial:

[Section 1: Create a simple web application](/flask_popover.md#create-a-simple-application)
[Section 2: Web Forms](web_forms.md)
[Sectin 3: Working with Databases](database.md)
[Section 4: User Login](user_login.md)
[Section 5: User Posts](user_posts.md)
[Section 6: Implement Popover](popover.md)

A user can access their account upon registration by providing their username and password. It is very important that a user's password is kept secret at all times. Even the admin of the application should not be able to know the password of a user. It is recommended that we use a hashing algorithm to hash the password instead of storing it in plain text. A hash is a representation of a string (a user's password in this case) that is not easily decoded.

The entire hashing logic can be implemented in our `User` model.

`app/models.py`: Password hashing
```python
# ...
from werkzeug.security import generate_password_hash, check_password_hash


class User(db.Model):
    # ...

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        return check_password_hash(self.password_hash, password)
```
This methods provide a secure password verification without the application storing a user's password in its original form.

## Manage User Sessions

Flask's functionality is extended by the `flask-login` package which allows us to manage the state of a user. Before implementing the login functionality, we need to install this package:

```python
(venv) $ pip3 install flask-login
```
Then instantiate it in the application's instance, as you have seen in the previous section:

`app/__init__.py`: Login object
```python
# ...
from flask_loign import LoginManager

app = Flask(__name__)
app.config.from_object(Config)

# ...
login = LoginManager(app)
login.login_view = 'login'

# ...
```
Flask login can restrict access to certain pages to only logged in users. When this is implemented, I want a user to be redirected to the login page if they try to access a page that requires login. This is done by setting the `login_view` property of the `login` object to the name of the view function that handles the login, which in this case is `login`.


## Preparing the User Model

Since our user is found in the `User` model, we need to prepare it. Flask-login provides a mixin class called _UserMixin_ that includes generic implementations that are used by the `User` model.

`app/models.py`: Usermixin
```python
from flask_login import UserMixin


class User(UserMixin, db.Model):
    # ...
```

To keep track of a logged in user, we need to store the user's ID in the user's session. The application expects that we will configure a user loader function that can be called to load a user (by their ID).

`app/__init__.py`: User loader
```python
# ...
from app import login


@login.user_loader
def load_user(id):
    return User.query.get(int(id))
```

## Login Logic

With the `User` model ready, we can now implement the login functionality.

`app/routes.py`: User login
```python
# ...
from flask import render_template, flash, redirect, url_for
from flask_login import login_user, logout_user, current_user
from app.models import User


@app.route('/login', methods=['GET', 'POST'])
def login():
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.username.data).first()
        if user is None or not user.check_password(form.password.data):
            flash('Invalid username or password')
            return redirect(url_for('login'))
        login_user(user, remember=form.remember_me.data)
        flash(f'Welcome {user.username}')
        return redirect(url_for('index'))
    return render_template('login.html', title='Login', form=form)
```

First, I check if the user is already logged in. If so, I redirect them to the index page. Otherwise, I check if the data given through the form is valid. If the username entered does not exist in the database or the password entered does not match the password stored in the database, I flash an error message and redirect the user to the login page. However, if the user's data is accurate, I log them in and authorize access to the index page.


## Log the User Out

If user wants to logout, flask login provides a logout function that easily logs this user out of their account to protect their data.

`app/routes.py`: User logout
```python
# ...


@app.route('/logout')
def logout():
    logout_user()
    return redirect(url_for('login'))
```

To expose these links to the user, we need to update the base template's navigation bar.

```html
<div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">            
    <ul class="nav navbar-nav navbar-right">
        {% if current_user.is_authenticated %}
            <li><a href=" {{ url_for('logout') }} ">Logout</a></li>
        {% else %}
            <li><a href=" {{ url_for('login') }} ">Login</a></li>
        {% endif %}
    </ul>                       
</div>
```
I have used a conditional statement to check if the user is authenticated or not. You have already seen this in the `head` section of the base template. If the user is authenticated, I display the logout link. Otherwise, I display the login link.


## Flash Messages

To restrict a page, as mentioned earlier, `flask-login` provides a `login_required` decorator. This decorator is used to protect a view function from unauthorized and anonymous access. We can require a user to log in to their account before accessing the index page by adding this decorator.

`app/routes.py`: Login required
```python
# ...
from flask_login import login_required


@app.route('/')
@app.route('/index')
@login_required
def index():
    return render_template('index.html', title='Home')
```
For a better user experience, it would be nice if a message is displayed to the user telling them that they need to login first before accessing the index page. You have already seen me use the `flash()` method from flask when logging in a user. To do this, we will update our base template such that this messge is visible to all parent templates.

`app/templates/base.html`: Flash message
```html
<!-- Previous content -->

{% block content %}
    <div class="container">
        <!-- Flash message -->
        {% with messages = get_flashed_messages() %}
            {% if messages %}
                {% for message in messages %}
                    <div class="alert alert-success" role="alert">
                        {{ message }}
                    </div>
                {% endfor %}
            {% endif %}
        {% endwith %}
        <!-- End of flash message -->
        
        {% block app_context %}{% endblock %}
    </div>
{% endblock %}

<!-- Previous content -->
```
If you click on "Flask Popover" in the navigation bar, which is a link to the index page, a user will see a nice message telling them that they first need to login in before accessing the page. And appropriately enough, the user is automatically redirected to the login page.

Notice how the login URL is appended with _?next=%2Findex_ such that the new URL becomes http://127.0.0.1:5000/login?next=%2Findex. This, by itself, if good since the user can know that the next page they will be redirected to is the index page.

![Login required](images/flask_popover/login_required.png)

## User Registration

To create a user, we first need to register them. All that is left to do now it to add the registration logic in our routes.

`app/routes.py`: User registration
```python
from app import db


@app.route('/register', methods=['GET', 'POST'])
def register():
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    form = RegistrationForm()
    if form.validate_on_submit():
        user = User(
            username=form.username.data,
            email=form.email.data)
        user.set_password(form.password.data)
        db.session.add(user)
        db.session.commit()
        flash('Congratulations! Login to continue.')
        return redirect(url_for('login'))
    return render_template(
        'register.html',
        title='Register',
        form=form)
```
The user's data in appropriately stored in the database by adding him to the database session and committing the changes. The user is then redirected to the login page for authentication.

![Register user](/images/flask_popover/register_user.png)

When authenticated, the user will be welcomed to the index page.

![Welcome user](/images/flask_popover/welcome.png)