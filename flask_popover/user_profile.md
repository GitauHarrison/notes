# User Profile

For your reference, these are the sections included in this tutorial:

[Section 1: Create a simple web application](/flask_popover.md#create-a-simple-application)
[Section 2: Web Forms](web_forms.md)
[Sectin 3: Working with Databases](database.md)
[Section 4: User Login](user_login.md)
[Section 5: User Posts](user_posts.md)
[Section 6: Implement Popover](popover.md)

All we need to display in the user's profile page is their avatar, username and email. Since much of the work is already done, the only bit remaining is adding a user's avatar. This can be done in the `User` model.

`app/models.py`: User avatar
```python
# ...
from hashlib import md5


class User(UserMixin, db.Model):
    # ...

    def avatar(self, size):
        digest = md5(self.email.lower().encode('utf-8')).hexdigest()
        return f'https://www.gravatar.com/avatar/{digest}?d=identicon&s={size}'
```

I am using the [Gravatar](https://en.gravatar.com/) service to generate the user's avatar. The avatar is generated using the user's email address and the `md5` hash function. The `avatar` method takes a size parameter which is the size of the avatar. The avatar is then returned as a string.

We can create a profile page to display the user's information, and all their posts (discussed later in the tutorial).

```python
(venv) $ cd templates
(venv) $ touch user.html
```

Update this template with the following:

`app/templates/user.html`: User profile
```html
{% extends 'base.html' %}

{% block app_context %}
<div class="row">
    <div class="col-md-12">
        <h1>{{title}}</h1>
    </div>  
</div>
<div class="row">
    <div class="col-sm-3 text-center">
        <p>
            <img src="{{ user.avatar(70) }}" alt="Avatar" class="img-responsive">
        </p>
    </div>
    <div class="col-sm-9">
        <p>
            Username: {{ user.username }}
            Eail: {{ user.email }}
        </p>
        {% if user == current_user %}
            <p>
                <a href="#">Edit profile</a>
            </p>
        {% endif %}
    </div>
</div>
<!-- Posts will go here -->

<!-- End of posts -->
{% endblock %}
```
At the moment, the application is capable of displaying several users. What we want is that when a user's username is clicked, the current user is redirected to the user whose username has been clicked. In this event, we want to restrict the visibility of the _edit profile_ link to the current user only.

Let us render this page:

`app/routes.py`: User profile
```python
@app.route('/user/<username>')
@login_required
def user(username):
    user = User.query.filter_by(username=username).first()
    return render_template(
        'user.html',
        user=user,
        title='User Profile')
```

You should be able to see the user's profile page.

![User profile](/images/flask_popover/user_profile.png)

## More Intersting Profile

We can improve how the profile page looks like by adding a few useful user data. For example, we can add a user's bio information as well as the last time the user was active.

Since these are changes to a user's personal information, we will update the `User` model.

`app/models.py`: More interesting user profile
```python
# ...


class User(UserMixin, db.Model):
    # ...
    about_me = db.Column(db.String(140))
    last_seen = db.Column(db.DateTime, default=datetime.utcnow)

    # ...
```

These new fields will cause a change in the database schema. We will need to update the database to reflect these changes.

```python
(venv) $ flask db migrate -m 'more fields for user profile'
(venv) $ flask db upgrade
```

The application can get a user's last visit time.

`app/routes.py`: User's last visit time
```python
# ...
from datetime import datetime

@app.before_request
def before_request():
    if current_user.is_authenticated:
        current_user.last_seen = datetime.utcnow()
        db.session.commit()

# ...
```

The `@before_request` decorator is called before each request. It updates the user's last visit time and commits the changes to the database. This is extremely useful because it allows us to insert code that we want to execute before any view function is called.

We can then update the user's profile page with these new updates.

`app/templates/user.html`: User profile
```html
{% extends 'base.html' %}

{% block app_context %}
<div class="row">
    <div class="col-md-12">
        <h1>{{title}}</h1>
    </div>  
</div>
<div class="row">
    <div class="col-sm-3 text-center">
        <p>
            <img src="{{ user.avatar(70) }}" alt="Avatar" class="img-responsive">
        </p>
    </div>
    <div class="col-sm-9">
        <p>
            Username: {{ user.username }}
        </p> 
        <p>
            Email: {{ user.email }}
        </p> 
        <p>            
            Active: {{ user.is_active }}
        </p>        
        {% if user.about_me %}
            <p>
                {{ user.about_me }}
            </p>
        {% endif %}
        <p>
            {{ user.last_seen }}
        </p>
        {% if user == current_user %}
            <p>
                <a href="#">Edit profile</a>
            </p>
        {% endif %}
    </div>
</div>
<!-- Posts will go here -->

<!-- End of posts -->
{% endblock %}
```

![New profile](/images/flask_popover/new_profile.png)

## Format the timestamp

Currently, the timestamp is less readable. We can format the timestamp to make it more readable. [Moment.js](http://momentjs.com/) is a JavaScript library that provides a set of utility functions to format dates and times. To use it in flask, we need to install the `flask-moment` extension.

```python
(venv) $ pip3 install flask-moment
```
Register this package in the application instance:

`app/__init__.py`: Register the extension
```python
# ...
from flask_moment import Moment

# ...

app = Flask(__name__)

# ...

moment = Moment(app)
```

Then, in our base template, we will create a `scripts` block to include the `moment.js` library.

`app/templates/base.html`: Moment.js
```html
{% block scripts %}
    {{ super() }}
    {{ moment.include_moment() }}
{% endblock %}
```

To format our timestamp as seen in the user's profile page, we can simply update the dynamic `last_seen` variable.

```python
{{ moment(user.last_seen).format('LLL') }}
```

You should be able to see a more readable timestamp.

![Formatted timestamp](/images/flask_popover/formatted_timestamp.png)

## Edit User Profile

We can allow a user to update his profile. The user will have the option of updating his _username_ and _about me_ information.

`app/forms.py`: User profile form
```python
# ...
from wtforms.validators import ValidationError
from app.models import User


class EditProfileForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    about_me = StringField('About me', validators=[Length(min=0, max=140)])
    submit = SubmitField('Submit')

    def __init__(self, original_username, *args, **kwargs):
        super(EditProfileForm, self).__init__(*args, **kwargs)
        self.original_username = original_username

    def validate_username(self, username):
        if username.data != self.original_username:
            user = User.query.filter_by(username=self.username.data).first()
            if user is not None:
                raise ValidationError('Please use a different username.')

```
I have implemented some functionality in this form such that if the user tries to update their username to an existing username, then a validation error will be displayed in the form.

We can now create a template to display the _edit profile_ form.

```python
(venv) $ cd templates
(venv) $ touch edit_profile.html
```

Update the `edit_profile.html` template:

`app/templates/edit_profile.html`: Edit profile form
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

To render this form, we will create a new view function with relevant logic.

`app/routes.py`: Edit profile
```python
# ...


@app.route('/edit-profile', methods=['GET', 'POST'])
@login_required
def edit_profile():
    form = EditProfileForm(current_user.username)
    if form.validate_on_submit():
        current_user.username = form.username.data
        current_user.about_me = form.about_me.data
        db.session.commit()
        flash('Your changes have been saved.')
        return redirect(url_for('user', username=current_user.username))
    elif request.method == 'GET':
        form.username.data = current_user.username
        form.about_me.data = current_user.about_me
    return render_template(
        'edit_profile.html',
        title='Edit Profile',
        form=form)
```

The only difference when validating this form is that we check what method is being used. So, if the method being used to submit the form is _POST_, then we will update a user's _username_ and _about me_ information. If the method being used is _GET_, then we conveniently populate the form with the current user's _username_ and _about me_ information before making any changes.

Finally, we can update the link to edit the user's profile.

`app/templates/user.html`: Edit profile link

```html
{% if user == current_user %}
    <p>
        <a href="{{ url_for('edit_profile') }}">Edit profile</a>
    </p>
{% endif %}
```

![Edit profile form](/images/flask_popover/edit_profile_form.png)