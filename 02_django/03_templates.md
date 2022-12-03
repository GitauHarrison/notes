# Templates in Django

In this tutorial, we are going to learn how to return more complex HTML code. We will also learn how to pass Python variables to these templates. In the [application and routes article](02_applications_and_routes.md), we saw how to return basic text in a HTTP Response. Most templates, though, contain full HTML structures. Below, we will learn how to return multiple HTML templates.

### Table of Contents

If you wish to skip to a particular section within this tutorial, you can do so by navigating to any of these links:

- [Update File Structure](#update-file-structure)
- [Template Configurations](#templates-configurations)
- [Render Templates](#render-the-templates)

## Update File Structure

By default, Django looks for a `templates` sub-directory in each of our installed apps. Within the `blog` app, let us go ahead and create a `templates` directory, which will contain all our HTML files.

```python
(venv)$ mkdir demo_project/blog/templates
```

Now, since Django will be looking in all applications for the `templates` subdirectory, it is a convention to create another sub-directory within the `blog` app's templates folder called `blog`. If we create anther application, then its templates subdirectory will contain another directory by its name to house all its HTML files. I hope it makes sense.

```python
(venv)$ mkdir demo_project/blog/templates/blog
```

Now, we can create the `home.html` and `about.html` files withing `templates/blog` folder.

```python
(venv)$ cd demo_project/blog/templates/blog
(venv)$ touch home.html about.html
```

Our current application structure looks like this:

```python
.
└── blog_app
    ├── blog
    │   ├── admin.py
    │   ├── apps.py
    │   ├── __init__.py
    │   ├── migrations
    │   │   └── __init__.py
    │   ├── models.py
    │   ├── __pycache__
    │   │   ├── __init__.cpython-38.pyc
    │   │   ├── urls.cpython-38.pyc
    │   │   └── views.cpython-38.pyc
    │   ├── templates
    │   │   └── blog
    │   │       ├── about.html
    │   │       └── home.html
    │   ├── tests.py
    │   ├── urls.py
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


## Templates Configurations

With individual files in place, we can add a bit of content to our HTML files. 

```html
<!-- demo_project/blog_app/blog/templates/blog/home.html -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Blog home</title>
</head>
<body>
    <h1>Blog Home</h1>
</body>
</html>
```

Now that we have this template ready, we need to add our `blog` application to the list of installed apps so that Django knows it needs to look there for the templates directory. To do this, it is recommended to add a our app configurations to our project's `settings.py` module. The app configurations is located inside `apps.py` module within the `blog` app.

![App configurations](/02_django/images/03_templates/app_config.png)

This module contains a class called `BlogConfig` which inherits from the `AppConfig` class. Copy the name `BlogConfig`. Navigate to the project's `settings.py` to access the list of installed apps. You may need to scroll down to see the list of installed apps.

![Installed Apps](/02_django/images/03_templates/installed_apps.gif)

Once there, update the list from the top as follows:

```python
# demo_project/blog_app/blog_app/settings.py

INSTALLED_APPS = [
    'blog.apps.BlogConfig',
    # ...
]
```

Every time we make an app, we should be in the habit of updating this list of installed apps because this allows Django to correctly search for our templates.


## Render The Templates

Rendering is a process used in web development that turns website code into the interactive pages users see when they visit a website. The term generally refers to the use of HTML, CSS, and JavaScript codes. The process is completed by a rendering engine, the software used by a web browser to render a web page.

To render our `home.html` template, we need to point our blog `views` to use this template. There are two ways we can render a template:

- Load the template in, and render it, then pass that to `HttpResponse`, but it needs a few more steps to achieve the outcome.

    ```python
    from django.template.loader import render_to_string
    rendered = render_to_string('my_template.html', {'foo': 'bar'})
    response = HttpResponse(rendered)
    ```
- Using the `render` shortcut, provided by Django by default. If you look at the top of `blog/views.py` you will see `from django.shortcuts import render`. Below, you will see me use this shortcut.

To render the `home.html` template, let us make the following changes to the `views` module:

```python
# demo_project/blog_app/blog/views.py

from django.shortcuts import render

# Create your views here.
def home(request):
    return render(request, 'blog/home.html')

```

The first argument of the `render()` function is always a `request` object, followed by the path to the `home.html` template. Notice that this path is a string. Despite the changes, it is important to remember that we still get an HTTP Response in the background. This is what you get whenever you query a server for a resource. The other response is normally an exception.

To make sure that everything is working, it is best practice to test every little change we make. In this case, let us pull up our browser to access the home page. Ensure that your server is running in the terminal. If not, execute this command:

```python
(venv)$ python manage.py runserver
```

Copy the URL http://127.0.0.1:8000/ and paste it in the browser. You may see the following page.

![Page Not found](/02_django/images/03_templates/404.png)

This is an issue with how we configured our project's URL patters. In the [previous article](/02_django/02_applications_and_routes.md#project-urls), we pointed out that if we would like the root URL to point to the the home page of our `blog` app, then we need to make the first argument of the `path()` function an empty string. Otherwise, we will need to append `/blog` to the root URL to access the home page of our blog.

As soon as you make the changes, the text "Blog Home" should show up. It looks exactly the same as before, but if you inspect the source code, you will notice that the file structure has changed.

![Blog home source code](/02_django/images/03_templates/blog_home_source_code.gif)

We can do the same for our `about.html` template. Back to our application on VS Code, let us update this file with a complete HTML structure as we did for the `home.html` file.

```html
<!-- demo_project/blog_app/blog/templates/blog/about.html -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Blog about</title>
</head>
<body>
    <h1>Blog About</h1>
</body>
</html>
```

The next step is to render this update template.

```python
# demo_project/blog_app/blog/view.py

from django.shortcuts import render

# Create your views here.
def home(request):
    return render(request, 'blog/home.html')


def about(request):
    return render(request, 'blog/about.html')

```

Notice that we are no longer using `HttpResponse`. Hence, I have discarded that import previously located at the top of the file.