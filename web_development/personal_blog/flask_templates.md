In the previous chapter, you learnt how to display a string on the web browser. During this chapter, I will show you how to work with HTML templates that help create a more elaborate and dynamic web application.

# Understanding Templates

Our application structure from [chapter 1](hello_world.md) was necessary to help us separate the application layout/presentation from the logic. Templates help us achieve this kind of separation. Flask templates are found in the _templates_ subfolder within the application instance.

### Working with Templates

Below, I am going to create a template for the home page of our blog.

app/static/templates/home.html: Home Page

```html
<html>
    <head>
        <title>
            Gitau Harrison | {{ title }}
        </title>  
    </head>
    <body>
        <h1>
            Available Updates
        </h1>
    </body>
</html>
```

This is a very standard HTML page. It basically contains `<head>` and `<body>` tags enclosed in `<html>`. The `<head>` has a dynamic content enclosed in `{{ ... }}`. The `title` is a placeholder variable which will be known at runtime.

With the simple presentation in place, we will now update our `home` view function.

app/routes.py: Render the home template
```python
from flask import render_template
from flask import app

@app.route('/')
@app.route('/home')
def home():
    return render_template('home.html', title = 'Home')
```

Flask provides the `render_template` function which allows us to render (or to show) HTML templates. About, we have returned the `home.html` template. Additionally, we have added a title to the function such that the placeholder value we saw previously will change to indicate 'Home'. Jinja2 is responsible of substituting the placeholder value with the actual dynamic content.

We can make the title of our blog a bit more interesting. Jinja2 provides support for control statements to be used in templates. Let us update our home page to include a control statement.

app/templates/home.html: Conditional statemenst in template
```html
<html>
    <head>
        {% if title %}
            <title>
                Gitau Harrison | {{ title }}
            </title> 
        {% else %}
            <title>
                Welcome to my Personal Blog
            </title> 
        {% endif %}         
    </head>
    <body>
        <h1>
            Available Updates
        </h1>
    </body>
</html>
```
The templates is slightly  smarter to know when a title has been provided or not. If there is a title in the view function, then it will use it within it's head, otherwise, it will resort to displaying _Welcome to My Personal Blog_.