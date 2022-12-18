# User Registration

During this tutorial, we are going to learn how to register a user. Our application currently features an admin site that is only accessible to select individuals. However, to ensure that any user can create an account and post comments, there needs to be a way to accommodate user registration.

For your reference, if you would like to skip to a particular section in the entire Django tutorial, you can do so by clicking on any of the links below:

- [Django Overview](00_overview.md)
- [Getting Started With Django](01_getting_started.md)
- [Applications and routes](02_applications_and_routes.md)
- [Templates in Django](03_templates.md)
- [Admin Page](04_admin_page.md)
- [Databases and Migrations](05_database_and_migrations.md)
- [User Registration](06_user_registration.md) (this article)
- [User Login and Logout](07_user_login_and_logout.md)

### Table of Contents

This article is broken down into the following subsections:

- [Create A User App](#create-a-user-app)
- [Create A Register View Function](#create-a-register-view-function)
- [Register Template](#register-template)
- [Update URL Patterns](#update-url-patterns)
- [Form Submission](#form-submission)
- [Handling Flash Messages](#handling-flash-messages)
- [Adding Form Fields](#adding-form-fields)
- [Improved Form Interface](#improved-form-interface)


## Create A User App

As a guiding principle on how to design the application, we can consider the user registration feature as a standalone app. The logic to handle user registration is going to have its own views, templates and forms, and it therefore makes sense to have it as its own app, just like the `blog` app. Should we want to make any changes to user registration, then we know where to go to within the application.

```python
(venv)$ python3 manage.py startapp users
```

The command above will create a new _users_ subfolder within the project's directory. Traditionally, whenever we create a new app, Django requires us to register it among the list of installed applications. In the project's `settings.py` file, let us add this new app to the list.

```python
# blog_app/settings.py

# ...

INSTALLED_APPS = [
    # ...
    'users.apps.UsersConfig',
    # ...
]
# ...

```

To find `UsersConfig()`, you will need to get into the `apps.py` file within the _users_ app.


## Create A Register View Function

A view function is used to load a particular resource. What we want is that whenever we click on a _register_ link on the application, then we shall be presented with a registration form. This registration form will be used to allow an anonymous user to create an account in the application. So let us begin by creating a view function called `register` to handle the registration logic.

```python
# users/views.py

from django.shortcuts import render
from django.contrib.auth.forms import UserCreationForm

# Create your views here.


def register(request):
    form = UserCreationForm()
    return render(request, 'users/register.html', {'form': form}, {'title': 'Register'})

```

Thankfully, Django has an inbuilt `UserCreationForm` that we can utilize to create a user registration form. As you saw with the admin site, the form comes ready to use, with all form validation features added. It has three fields: _username_, _password_ and _password2_. We begin by importing it from `django.contrib.auth.forms` before instantiating a `form` object. The `form` object is then passed to the `render()` function so that it can displayed on the templates.


## Register Template

At this point, we do not have the `register.html` file. We will need to create it. Tradtionally, all templates will be in the app's templates folder. To maintain Django convetion when handling templates, the `register.html` file will be in a subdirectory whose name is similar to the app's.

```python
(venv)$ mkdir -p users/templates/users && touch users/templates/users/register.html
```

With the template in place, we can extend the `base.html` file found in the `blog` app and use it to display a registration form.

```html
<!-- users/templates/users/register.html -->

{% extends 'blog/base.html' %}

{% block content %}
    <div class="content-section">
        <form action="" method="post">
            {% csrf_token %}
            <fieldset class="form-group">
                <legend class="border-bottom mb-4">
                    Create An Account Today
                </legend>
                {{ form }}
            </fieldset>
            <div class="form-group">
                <button class="btn btn-outline-info" type="submit">Register</button>
            </div>
        </form>
        <div class="border-top pt-3">
            <small class="text-muted">
                Already have an account? <a class="ml-2" href="#">Login</a>
            </small>
        </div>
    </div>
{% endblock content %}
```

The `POST` method is typically used whenever we want to submit some data from a form. Therefore, you can see me use the `method='post'` in the form. I have left `action=''` blank so that the current URL shall be used during the submission. All web forms are required to use `csrf_token` to protect against a nasty attack called [Cross-site Request Forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery). Without this token, our register form will not work. To display the form fields, we then pass `{{ form }}` dynamically. This, however, does not create a submit button for us, so we have to manually  create one for ourselves. Typicall of most web applications, a link _login_ is normally provided just in case a user already has an account.


## Update URL Patterns

We have the option to add a URL within the `users` app by creating a `urls.py` file as we did for the `blog` app. However, I will add the `register` URL directly to the project's list of URL patterns.

```python
# blog_app/urls.py

from django.contrib import admin
from django.urls import path, include
from users import views as user_views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('register/', user_views.register, name='register'),
    path('', include('blog.urls'))
]
```

To easily access the _register_ page, I will now update `base.html` to point to the `register()` view function.

```html
<!-- templates/blog/base.html -->

<li class="nav-item"><a class="nav-link" href=" {% url 'register' %} ">Register</a></li>
```

![Register page](/02_django/images/06_user_registration/register_page.gif)

As you can see above, the user registeration form is displayed. If we try to fill in the form, nothing happens to update our User model. That is so because we have not yet updated the logic to handle data passed into the form.


## Form Submission

GET and POST HTTP methods are used when working with forms. As mentioned earlier, the POST method is used to send data. GET, by contrast, is used to retrieve data and its related resources. In our `register()` view function, we can put a conditional statement to determine what method should be used to handle the register form's data.

```python
# users/views.py

def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
    else:
        form =  UserCreationForm()
    return render(request, 'users/register.html', {'form': form})
```

Whenever the application is ready to send a user's data, the POST method shall be used. The `UserCreationForm` uses the `request.POST` argument to store the information passed by the form. To retrieve the from, such as when a user clicks on the _register_ link, a blank form will be served instead.

To save a user's information, we first need to validate the data passed into the form.

```python
# users/views.py
from django.shortcuts import render, redirect


def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('blog-home')
    else:
        form =  UserCreationForm()
    return render(request, 'users/register.html', {'form': form})

```

Once Django has validated the data passed into the form, a call to `form.save()` function is all we need to do to update our `User` model with a new `user` object. We then redirect the user to the home page of the blog using the `redirect()` function from `django.shortcuts`.


## Handling Flash Messages

One way to improve a user's experience when using the application is by providing them with feedback about an action they have taken. For example, if a user successfully registers, then it would be nice to notify them that their account has been created prior to being redirected to the home page. 

Django provides support for flash messages for both anonymous and authenticated users. The messages framework allows us to temporarily store messages in one request and retrieve them for display in another request. Flash messages logic is enabled by default whenever we create a new app. If you check the `settings.py` file in the project's folder, you will see messages shown among the listed installed app.

```python
# blog_app/settings.py

INSTALLED_APPS = [
    # ...
    'django.contrib.messages',
]
```

Flash messages have levels, similar to Python's logging module. For example, we can issue a success flash message if a user's action is completed successfully. In the `register` view function, we can make the following changes:

```python
# users/views.py

# ...
from django.contrib import messages


def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            username = form.cleaned_data.get('username')
            messages.success(request, f'Account successfully created for {username}')
            return redirect('blog-home')
    else:
        form =  UserCreationForm()
    return render(request, 'users/register.html', {'form': form})
```

We begin by importing `messages` from `django.contrib`. Then, we create a success flash message and pass in a custom message that includes the user's username as captured from the registration form's dictionary data.

The last step is to display this flash message in the `base.html` template. We want to use the base template because it is the parent template and it allows the messages to be displayed on any of the child templates.

```html
<!-- blog/base.html -->

<main role="main" class="container">
    <div class="row">
        <div class="col-md-8">
          {% if messages %}
            {% for message in messages %}
              <div class="alert alert-{{ message.tags }}"> {{ message }} </div>
            {% endfor %}
          {% endif %}

          {% block content %}{% endblock %}

        </div>
        <div class="col-md-4">
          <!-- previous code -->
        </div>
    </div>
</main>
```

Now, we can register a new user using the registration form.

![Flash Message](/02_django/images/06_user_registration/flash_message.gif)


## Adding Form Fields

Our registration form currently allows us to provide only a username and password. We cannot add a user's email address. This is true even when we check the `Users` model in the admin site.

![Emailless Users](/02_django/images/06_user_registration/emailless_users.png)

To add an _email_ field, we can do so by creating a `forms` module in the `users` app and update it with the following.

```python
(venv) touch users/forms.py
```

This creates an empty `forms.py` file.

```python
# users/forms.py

from django import forms
from django.contrib.auth.models import User
from django.contrib.auth.forms import UserCreationForm


class UserRegistrationForm(UserCreationForm):
    email = forms.EmailField()

    class Meta:
        model = User
        fields = ['username', 'email', 'password1', 'password2']

```

The `UserRegistrationForm()` inherits the `UserCreationForm`. To add an email field, we instantiate one using `form.EmailField()`. I have used the class `Meta` to give a nested namespace for configurations and keeps the configurations in one place. The model that will be affected when saving user data will be the `User` model. Now, we can use `UserRegistrationForm` in the user views module instead of `UserCreationForm`.

```python
# users/views.py

from django.shortcuts import render, redirect
from .forms import UserRegistrationForm
from django.contrib import messages
# Create your views here.


def register(request):
    if request.method == 'POST':
        form = UserRegistrationForm(request.POST)
        if form.is_valid():
            form.save()
            username = form.cleaned_data.get('username')
            messages.success(request, f'Account successfully created for {username}')
            return redirect('blog-home')
    else:
        form =  UserRegistrationForm()
    return render(request, 'users/register.html', {'form': form})

```

You should be able to see an email field added to the registration form. At this stage we can register another user, and this time round they will have an email address added to their account.


## Improved Form Interface

You can agree with me that our registration form do not look presentable at all. As much as it works, it is not attractive to work with. Thankfully, there is a Python package that we can use to style Django forms, and the most popular among them is [Django Cripsy Forms](https://django-crispy-forms.readthedocs.io/en/latest/index.html). It provides some filters and tags that allow us to control the rendering behaviour of Django forms in a very elegant and DRY way without writing custom form templates. Django Crispy forms has support for multiple CSS frameworks such as [Bootstrap5](https://github.com/django-crispy-forms/crispy-bootstrap5), [Foundation](https://github.com/sveetch/crispy-forms-foundation), and [TailWind](https://github.com/django-crispy-forms/crispy-tailwind). I will be using Crispy-Bootstrap5 for demonstration.

To begin, we need to install the crispy plugin in our active virtual environment:

```python
(venv)$ pip3 install crispy-bootstrap5
```

Once installed, we need to tell Django that this is an installed app. Among the list of installed apps as seen in the project's `settings.py` file, let us add crispy.

```python
# blog_app/settings.py

INSTALLED_APPS = [
    "crispy_forms",
    "crispy_bootstrap5",
]
```

We then need to specify what CSS framework we want to use. On the same `settings.py` file, at the very bottom, we can add these lines:

```python
# blog_app/settings.py

CRISPY_ALLOWED_TEMPLATE_PACKS = 'bootstrap5"
CRISPY_TEMPLATE_PACK = "bootstrap5"
```

With the crispy forms configurations in place, let us load in the crispy form tags in our `register.html` file.

```html
<!-- templates/users/register.html -->

{% extends 'blog/base.html' %}
{% load crispy_forms_tags %}

{% block content %}
    <div class="content-section">
        <form action="" method="post">
            {% csrf_token %}
            <fieldset class="form-group">
                <legend class="border-bottom mb-4">
                    Create An Account Today
                </legend>
                {{ form | crispy }}
            </fieldset>
            <div class="form-group">
                <button class="btn btn-outline-info" type="submit">Register</button>
            </div>
        </form>
        <div class="border-top pt-3">
            <small class="text-muted">
                Already have an account? <a class="ml-2" href="#">Login</a>
            </small>
        </div>
    </div>
{% endblock %}
```

We begin by loading the crispy form tags at the top of the template then filter our form as `{{ form | crispy }}`. This does the styling magic for us and we should have a far better-looking form now.

![Crispy Form](/02_django/images/06_user_registration/crispy_form.png)

If we fill in some data, the form is able to capture any errors and show them in a more elegant way. Below, I have tried to register a user with a pre-existing name whose password fields do not match.

![Error cripsy forms](/02_django/images/06_user_registration/error_crispy_forms.png)