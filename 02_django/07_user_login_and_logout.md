# User Login and Logout

By default, Django provides inbuilt methods to log a user into an account and then log them out. A typical workflow presumes that once a user has registered for an account, they should verify their identity first before accessing their account. To help protect their account, a user can optionally choose to log out. This article shows how to enable the user authentication feature in Django.

For your reference, if you would like to skip to a particular section in the entire Django tutorial, you can do so by clicking on any of the links below:

- [Django Overview](00_overview.md)
- [Getting Started With Django](01_getting_started.md)
- [Applications and routes](02_applications_and_routes.md)
- [Templates in Django](03_templates.md)
- [Admin Page](04_admin_page.md)
- [Databases and Migrations](05_database_and_migrations.md)
- [User Registration](06_user_registration.md)
- [User Login and Logout](07_user_login_and_logout.md) (this article)

### Table of Contents

This article is broken down into the following subsections:

- [Create A User App](#create-a-user-app)


## Add Login and Logout URLs

It makes sense to add the authentication feature in the `users` app more than anywhere else. Therefore, all changes to enable this feature will take place in the `users` app of our project. To begin, we shall update the project's `URLs` module with `login` and `logout` views.

```python
# blog_app/urls.py

from django.contrib.auth import views as auth_views

urlpatterns = [
    # ...
    path('login/', auth_views.LoginView.as_view(), name='login'),
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
    # ...
]
```

`LoginView` and `LogoutView` are class-based views (we shall discuss that in another article). Django uses them to handle the login and logout logic for us right out of the box. However, Django does not handle the templates related to these views for us.

![Login template error](/02_django/images/07_login_and_logout/login_template_error.png)

Above, I tried to access the URL http://127.0.0.1:8000/login/ and got the `TemplateDoesNotExist at /login/` error. Django expects a `login.html` file in a `registration` templates folder, but our blog is structured slightly differently. On one hand, the fact that Django does not handle the auth templates for us, is a good thing since we can do it ourselves. 

Notice that when we try to logout of the application by pasting http://127.0.0.1:8000/logout/ on a browser tab, we are redirected to the admin site's logout page.

![Admin logout page](/02_django/images/07_login_and_logout/logout_admin.png)

This is alright, but what we want is that regular users will have their own authentication system separate from that of the admininstrators of the application.


## Update Login and Logout Templates

Within the `users` templates, let us add two more templates:

```python
(venv) cd users/templates/users/ && touch login.html logout.html
```

This will create two empty files for us. To make our work easier, we can copy the content of `register.html` and paste it in `login.html` albeit with a few modifications as seen below.

```html
<!-- users/login.html -->

{% extends 'blog/base.html' %}
{% load crispy_forms_tags %}

{% block content %}
    <div class="content-section">
        <form action="" method="post" class="pb-3">
            {% csrf_token %}
            <fieldset class="form-group">
                <legend class="border-bottom mb-4">
                    Log in to your account.
                </legend>
                {{ form | crispy }}
            </fieldset>
            <div class="form-group">
                <button class="btn btn-outline-info" type="submit">Login</button>
            </div>
        </form>
        <div class="border-top pt-3">
            <small class="text-muted">
                New here? <a class="ml-2" href=" {% url 'register' %} ">Register</a>
            </small>
        </div>
    </div>
{% endblock %}
```

The `logout.html` file will not need to use a form, so we can do the following.

```html
<!-- users/logout.html -->

{% extends 'blog/base.html' %}

{% block content %}
    <div class="content-section">        
        <div class="pt-3">
            <p>{{ title }}</p>
            <small class="text-muted">
                You can <a href=" {% url 'login' %} ">log back in</a>.
            </small>
        </div>
    </div>
{% endblock %}
```

## Specify Templates Django Should Use

By default, Django looks for the login template in a `registration` folder. We have our `login.html` file in the `users` folder instead. So, we need to update our application's template configurations to point to our new location. First, we specify that the `LogiView` class should use `users/login.html` file as follows:

```python
# blog_app/urls.py

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(template_name='users/login.html'), name='login'),
    path('logout/', auth_views.LogoutView.as_view(template_name='users/logout.html'), name='logout'),
]
```

We have also specified the the `LogoutView` should use the `logout.html` file in the `users` subdirectory.

If we go to our browser, now the `TemplateDoesNotExist at /login/` error is gone. We are able to access both login and logout pages.

![Login and logout templates](/02_django/images/07_login_and_logout/login_logout_templates.gif)


## Handling Login Redirect

At this point, if we try to log an existing user into their account, we will get a `Page not found` error. 

![Page not found error](/02_django/images/07_login_and_logout/404_error.png)

This is because Django, by default, tries to redirect us to `accounts/profile` page, which does not exist in our application. What we want is that when a user logs into their account, they should instead be redirected to the home page. To fix this, we need to update our template configurations again.

```python
# blog_app/settings.py

LOGIN_REDIRECT_URL = 'blog-home'
```

This configuration has been added to the bottom of the `settings.py` file. Now, if we try to log a user into their account, they will be redirected to the home page. The `blog-home` is the name we gave the view function handling the login page.

![Login redirect](/02_django/images/07_login_and_logout/login_redirect.gif)

There isn't much visual feedback to tell us that we have logged in; we will fix that later. The blog right now works well. You will realize that when you try to access the admin site right now by pasting http://127.0.0.1:8000/admin/ on a browser, you will be redirected to the private admin page. The logout, from the admin site, is automatically handled for us such that when we click on the _logout_ link, we will be completely logged out of the application, and our custom logout page is used instead of the default admin logout page.

![Log admin out](/02_django/images/07_login_and_logout/log_admin_out.gif)


## Protecting Pages

Typical of many blog applications, restrictions are normally put on select pages to ensure that a user has to login before they can interact with certain aspects of the application. For example, if a user wants to make a post, they can be asked to log in first. Also, if this user wants to make some changes on their profile, they can be required to verify themselves before continuing with the changes.

At the start of this tutorial, we saw that Django by default redirects a user to the `accounts/profile` page once they log in. We made some changes that instead redirected a the user to the home page of the application. Now, we can add our own version of the profile page which shall display only the logged in user's username.

```python
# users/view.py

def profile(request):
    return render(request, 'users/profile.html')
```

With the view function in place, there are two things we need to do:
- Add this to the URL patterns in the main project

    ```python
    # blog_app/urls.py

    urlpatterns = [
        # ...
        path('profile/', user_views.profile, name='profile'),
        # ...
    ]
    ```

- Update the profile template to display the logged in user's username. 
    - First we create an empty file called `profile.html` as follows:
        ```python
        (venv)$ touch users/templates/users/profile.html
        ```

    - Then, we display the current user's username.

        ```html
        <!-- users/profile.html -->

        {% extends 'blog/base.html' %}

        {% block content %}
            <div class="content-section">        
                <div class="pt-3">
                    <p>Username: {{ user.username }}</p>
                </div>
            </div>
        {% endblock %}
        ```
- Display a link to the profile page once a user is logged in:
    ```html
    <ul class="nav navbar-nav navbar-right">
        {% if user.is_authenticated %}
            <li class="nav-item"><a class="nav-link" href=" {% url 'profile' %} ">Profile</a></li>
            <li class="nav-item"><a class="nav-link" href=" {% url 'logout' %} ">Logout</a></li>
        {% else %}
            <!-- previous links -->
        {% endif %}              
    </ul>
    ```

At this stage, the profile page is still accessible by an anonoymous user. However, it is intelligent enough to display a user's username only when they have logged in.

![Display user's username](/02_django/images/07_login_and_logout/unrestricted_profile_page2.gif)

Now, to effect page restrictions, we can do the following:

```python
# users/views.py

# ...
from django.contrib.auth.decorators import login_required


@login_required
def profile(request):
    return render(request, 'users/profile.html')
```

This should limit access to the profile page. Pasting http://127.0.0.1:8000/profile on the browser interestingly throws as another `Page not found` error.

![Profile page not found](/02_django/images/07_login_and_logout/profile_page_redirect_error2.png)

Django is trying to access the restricted profile page by asking that we log in from the URL `accounts/login` before continuing. This is somewhat similar to the faulty configuration we experienced earlier. To amend this, we need to update our application settings as follows:

```python
# blog_app/settings.py

LOGIN_REDIRECT_URL = 'blog-home'
LOGIN_URL = 'login'

```

This tells Django that the view function to be used to access the login page should be the one identified by the name `login`. Now, if we attempt to access the profile page using the URL http://127.0.0.1:8000/profile, we shall be redirected to http://127.0.0.1:8000/login/?next=/profile/ which is the login page pointing to the profile page once a user is successfully verified.

![Restricted profile page](/02_django/images/07_login_and_logout/restricted_profile_page.gif)


## Further Updates

Previously, we had configured our application to redirect a user to the home page once they register. We did this at the time because we did not have a login page ready. But now that the login page is ready to be used, we can update the redirect in the `register()` view function to point towards the login page.

```python
# users/views.py

def register(request):
    if request.method == 'POST':
        if form.is_valid():
            # ...
            messages.success(request, f'Account successfully created for {username}. Log in to continue')
            return redirect('login')
    else:
        # ...
```

Now, every time a new user registers for an account, they will be redirected to the login page for verification. 

The other update we should make is to link the text "Login" found in the navigation bar to the login page.

```html
<!-- blog/base.html -->

<li class="nav-item"><a class="nav-link" href=" {% url 'login' %} ">Login</a></li>
```

This makes it easy for a user to access the login page. All they need to do is click on this link in the navigation bar to access the login page instead of appending the word 'login' to the URL in the address bar.

The last change we need to make is to display the "Logout" link in the navigation bar once a user has already logged into their account. It makes sense to only show the _logout_ link to this user instead of both the _login_ and the _register_ links. 

```html
<!-- blog/base.html -->

<ul class="nav navbar-nav navbar-right">
    {% if user.is_authenticated %}
        <li class="nav-item"><a class="nav-link" href=" {% url 'logout' %} ">Logout</a></li>
    {% else %}
        <li class="nav-item"><a class="nav-link" href=" {% url 'login' %} ">Login</a></li>
        <li class="nav-item"><a class="nav-link" href=" {% url 'register' %} ">Register</a></li>
    {% endif %}              
</ul>
```

Using a conditional statement, we are able to determine the login status of a user and display the appropriate links to them. Now, once a user logs into their account, they will only see the _logout_ link.

![Logout link in navbar](/02_django/images/07_login_and_logout/logout_link.png)

