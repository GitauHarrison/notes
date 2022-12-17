# Applications in Django

The thinking behind Django is that we have a website project, which we have already created in [the previous tutorial](01_getting_started.md), and have multiple apps each doing its own thing. For example, we can have a blog section of the website, which is an app on its own, then we can have a store section which is an app as well. So, a single project can contain multiple apps. In this tutorial, we are going to add a blog app to our project.

For your reference, if you would like to skip to a particular section in the entire Django tutorial, you can do so by clicking on any of the links below:

- [Django Overview](00_overview.md)
- [Getting Started With Django](01_getting_started.md)
- [Applications and routes](02_applications_and_routes.md) (this article)
- [Templates in Django](03_templates.md)
- [Admin Page](04_admin_page.md)
- [Databases and Migrations](05_database_and_migrations.md) 


### Table of Contents

This article is broken down into the following subsections:

- [Views](#views)
- [Application URLs](#application-urls)
- [Project URLs](#project-urls)
- [Additional routes](#additional-routes)




## Create An App

Ensure you are currently at the top-level directory of our project, where `manage.py` is located. From the terminal, run the following command to create a blog app:

```python
(venv)$ python3 manage.py startapp blog
```

You will notice that Django automatically creates a full app for us called `blog` with its own structure. The `tree` command can reveal this new structure:

```python
(venv)$ tree

# Output

.
├── blog
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── blog_app
│   ├── asgi.py
│   ├── __init__.py
│   ├── __pycache__
│   │   ├── __init__.cpython-38.pyc
│   │   ├── settings.cpython-38.pyc
│   │   ├── urls.cpython-38.pyc
│   │   └── wsgi.cpython-38.pyc
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── db.sqlite3
└── manage.py
```

It might be quite intimidating to know that we have a lot of project files to work with right out of the box but let's start with a few of them to get the hang of it.

### Views

We shall begin by making changes to the `views` module of our `blog` app. What we want to do is to create a view function that will handle the traffic from the home page.

```python
# blog/views.py

from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.
def home(request):
    return HttpResponse('<h1>Blog home!</h1>')

```

By default, Django already imports `render` for us. This function is used to combine a template with a context dictionary and returns a HttpResponse object with the rendered text. It is a shortcut to the long function `django.template.loader.render_to_string`. 

At this point, though, we are not going to use the `render` function. I have used the `HttpResponse` function to display a simple message whenever a user visits the home page. This function is used in Django to return a text response. Typically, every view function takes a `request` argument.

### URL Patterns

URL patterns are used to match requested URLs. Django runs through all available patterns in order and retrieves the first one that matches a requested URL. Once one of the URL patterns matches, Django imports and calls the given view function.

### Application URLs

Within our `blog` app directory, let us create a new module called `urls.py` which is similar to what we have in our root project folder. 

```python
(venv)$ touch blog/urls.py
```

Let us update this module with the following code:

```python

# demo_project/blog/urls.py

from django.urls import path
from . import views


urlpatterns = [
    path('', views.home, name='blog-home'),
]

```

Here, Django queries the `urls` module to look for the variable `urlpatterns`, which is a sequence. It runs through each item in the sequence in order (there is only one at the moment) and stops at the first one that matches the requested URL. The view function `home` is called from the `views` module. I have used the `.` (dot) to denote that both the `views` and the `urls` modules are located in the current app folder called `blog`. The first argument is an empty string to show that it is the home URL. I have also passed the `name` argument to assign a name to this pattern. Intentionally, I have named it `blog-home` instead of `home` because as the project grows, I am most likely to have multiple `home` functions for each app.

### Project URLs

To complete the setup process, we also need to update the `urls` module in the root project folder. This updated URL configuration will tell Django that upon request, the project should serve the requested resource found in the specified application.

```python

# demo_project/blog_app/urls.py

from django.contrib import admin
from django.urls import path, include


urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),
]

```

Django will check all URL patterns in `blog_app.urls.py` to find anyone that matches a request. Since we want to access the home page in our `blog` app, the request http://127.0.0.1:8000/blog will tell Django that it needs to serve the blog app. Thereafter, the application will redirect any further request to `blog.urls.py` for more processing.

As a side note, URLs in Django do not come with a preceding slash. All URLs have a forward slash and it is therefore redundant to prepend them. However, Django URLs end with a slash. 

To see the changes, let us start our Django server by running:

```python
(venv)$ python3 manage.py runserver
```

Paste the URL http://127.0.0.1:8000 on the browser to access the application. 

![Blog App](/02_django/images/02_application_and_routes/access_blog_app.gif)

Notice that no page is found once we paste the localhost link on our browser. This is because there is no matching URL for http://127.0.0.1:8000/. Appending `blog` to the URL serves as the text "Blog home!". If we want the root of our website to be the home page, we need to leave the path to the blog's home page empty. So, instead of `path('blog/' include('blog.urls'))` we will have `path('', include('blog.urls'))`.


### Additional Routes

To get a better grasp of how URLs work in Django, let us add one more view function to our `blog` app. Within the `views` module in the `blog` app, let us add a function that will handle the logic for the "About" page. 

```python
# demo_project/blog/views.py

# ...

def about(request):
    return HttpResponse('<h1>Blog about page!</h1>')
```

Once the function is defined, we need to update the `blog.urls.py` file to point all requests for the about page to the `about()` view function.

```python
# demo_project/blog/urls.py

# ...

urlpatterns = [
    # ...
    path('about/', views.about, name='blog-about'),
]

```

That is it. When we go to the link http://127.0.0.1:8000/blog/about/ on our browser, we should be able to see the text "Blog about page".