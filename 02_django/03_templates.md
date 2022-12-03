# Templates in Django

In this tutorial, we are going to learn how to return more complex HTML code. We will also learn how to pass Python variables to these templates. In the [application and routes article](02_applications_and_routes.md), we saw how to return basic text in a HTTP Response. Most templates, though, contain full HTML structures. 

### Table of Contents

If you wish to skip to a particular section within this tutorial, you can do so by clicking on any of these links:

- [Update File Structure](#update-file-structure)
- [Template Configurations](#templates-configurations)
- [Render Templates](#render-the-templates)
- [Working With Templates](#working-with-templates)
- [Template Inheritance](#template-inheritance)
- [Bootstrap Templates](#bootstrap-templates)
- [Adding A Navigation Bar](#adding-a-navigation-bar)

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

Notice that we are no longer using `HttpResponse`. Hence, I have discarded that import which was previously located at the top of the file.


## Working With Templates

Our final application, as seen in the introduction of this Django Series, contained blog posts. Users are allowed to post comments that can be seen by other users. In this section, we will learn how to add dummy data and pass it to the templates.

### Variables

Let us begin by adding some fake posts in the `views` module.

```python
# demo_project/blog_app/blog/views.py

from django.shortcuts import render

posts = [
    {
        'author': 'Harry',
        'title': 'Blog post 1',
        'content': 'Content 1',
        'timestamp': 'Dec 3, 2022'
    },
    {
        'author': 'Gitau',
        'title': 'Blog post 2',
        'content': 'Content 2',
        'timestamp': 'Dec 2, 2022'
    },
    {
        'author': 'Muthoni',
        'title': 'Blog post 3',
        'content': 'Content 3',
        'timestamp': 'Dec 1, 2022'
    }
]


def home(request):
    context = {
        'posts': posts
    }
    return render(request, 'blog/home.html', context)

```

Above, I have created dummy posts that I intend they be displayed in the `home.html` template. This is achieved by creating a dictionary in the `home()` view function whose key would be used to access the `posts` list.

The templating engine that Django usese is very similar to Flask's Jinja2. We can loop through the `posts` list to access individual data and display them on the home template.

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
    {% for post in posts %}
        <h1> {{ post.title }} </h1>
        <p> {{ post.author }} said on {{ post.timestamp }}</p>
        <p> {{ post.title }} </p>
    {% endfor %}
</body>
</html>
```

The `posts` variable that we are looping through using the `for` loop is the key of the `context` dictionary in the home view function. To access a particular item on a list, the templating engine allows us to use the double curly braces `{{  }}` for dynamic content (this is content that can easily change). For example, to print all the post titles, we can simple use `{{ post.title }}`. Notice that the `for` loop has the `endfor` block. We need to use it to ensure that the templating engine understand the scope of the loop.

On our browser, we can see that the three posts have been displayed.

![Posts](/02_django/images/03_templates/posts.png)


## Dynamic Titles

Within the `head` element of every HTML structure is the `title` element. As seen in our templates, these elements are used to display the title of each page seen at the top of each tab on a browser. For example, if you are on a _home_ page, then the title seen on the page's tab on a browser would be "Home". If you are on an _about_ page, then the title is likely to be "About". 

![Page titles](/02_django/images/03_templates/page_titles.png)

So far, we have been modifying the `title` element manually on each template. In the `home.html` file, we have the line `<title>Blog home</title>` while in the `about.html` we have `<title>Blog about</title>`. Imaging having several templates in an application. This will mean that we have to manually provide a title to each template, a repetitive task that quicky gets cumbersome. Thankfully, Django's templating engine allows us to pass in dynamic titles in each template. 

```html
<!-- demo_project/blog_app/blog/templates/blog/about.html -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    {% if title %}
        <title>{{ title }} - Django Blog</title>
    {% else %}
        <title>Django Blog</title>
    {% endif %}
</head>
<body>
    <h1>Blog About</h1>
</body>
</html>
```

Above, I have used the `if - else` statement to check if a title exists. If one exists, then the title `{{ title }} - Django Blog` shall be used. The dynamic `{{ title }}` attribute will be replaced by the actual title name. If there is no title given, then the application will default to using the text "Django Blog". This logic can also be used in the `home.html` file. 

To create an actual title, we need to update our `views` functions to pass in titles for our pages.

```python
# demo_project/blog_app/blog/views.py

# ...

def about(request):
    return render(request, 'blog/home.html', context, {'title': 'Home'})
```

Back to our browser, you will notice that the titles change dynamically depending on what page we are on. When there is no title for a page, then a default value is used.

![Dynamic titles](/02_django/images/03_templates/dynamic_titles2.png)


## Template Inheritance

Notice that a lot of code in our templates are repetitive. The only part in the templates that are unique to a page is the `body` content. To help make our work easier especially when dealing with multiple templates, we can use the concept of template inheritance. 

Template Inheritance is the where  the base structure of an application is defined in a parent template and the children inherit this structure without redefining it. Child templates instead focus on displaying content that is unique to them. 

To achieve this, we can modify the structure of our templates subfolder in the `app` blog to define a `base` template. Let us create a new file called `base.html`.

```python
(venv)$ cd templates/blog
(venv)$ touch base.html
```

Once created, we can update it with the common structure to be used by all templates.

```html
<!-- blog/templates/blog/base.html -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    {% if title %}
        <title>{{ title }} - Django Blog</title>
    {% else %}
        <title>Django Blog</title>
    {% endif %}
</head>
<body>
    {% block content %}{% endblock %}
</body>
</html>
```

The block `content` has been used to allow child templates to pass in their unique contents.

```html
<!-- blog/templates/blog/home.html -->

{% extends 'blog/base.html' %}

{% block content %}
    {% for post in posts %}
        <h1> {{ post.title }} </h1>
        <p> {{ post.author }} said on {{ post.timestamp }}</p>
        <p> {{ post.title }} </p>
    {% endfor %}
{% endblock %}
```

The child template makes use of the keyword `extends` to inherit the base template. Its content are then wrapped in the block `content` earlier defined in `base.html`. Just like it was done for the `for` loop, a block needs to be closed too. You can optionally choose to say `{% endblock %}` as seen above, or you can say `{% endblock content %}`. 

# Bootstrap Templates

[Bootstrap](https://getbootstrap.com/) is a powerful, feature-packed frontend kit that allows us to not only prototype but also to build to production quicky. If we navigate to the [Bootstrap documenation](https://getbootstrap.com/docs/5.2/getting-started/introduction/), we can see how to get started. We will copy the sample template and use it in our `base.html` file.

```html
<!-- blog/templates/blog/base.html -->

<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    {% if title %}
        <title>{{ title }} - Django Blog</title>
    {% else %}
        <title>Django Blog</title>
    {% endif %}

    <!-- Bootstrap CSS -->
    <link 
        href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/css/bootstrap.min.css" 
        rel="stylesheet" 
        integrity="sha384-rbsA2VBKQhggwzxH7pPCaAqO46MgnOM80zW1RWuH61DGLwZJEdK2Kadq2F9CUG65"
        crossorigin="anonymous">
  </head>
  <body>
    {% block content %}{% endblock %}
    <!-- Bootstrap JS Bundle, including Popper -->
    <script 
        src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/js/bootstrap.bundle.min.js" 
        integrity="sha384-kenU1KFdBIe4zVF0s0G1M5b4hcpxyD9F7jL+jjXkk+Q2h455rYXK/7HAuoJl+0I4" 
        crossorigin="anonymous"></script>
  </body>
</html>
```

This change allows us to use Bootstrap's classes to style our pages and make them more responsive. For example, if we want to a bit of padding and margin to all our page contents, we can wrap the block `content` with the `container` class.

```html
<!-- blog/templates/blog/base.html -->


<!-- Previous code -->

<body>
    <div class="container">
        {% block content %}{% endblock %}
    </div>
</body>
```

Now, if we reload any of our pages, we should be able to see that significt whitespaces have been created to the left and right of the children templates.

![Bootstrap templates](/02_django/images/03_templates/bootstrap_template.png)


## Adding A Navigation Bar

Bootstrap offers quite a handful of navbar snippets we can utilize to make a simple navigation bar. Sample navigation bars can be found [here](https://getbootstrap.com/docs/5.2/components/navbar/).