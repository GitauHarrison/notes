# Database and Migrations

This section is dedicated to learning how to create our own models. In the previous lesson on [how to access the admin page](04_admin_page.md), we saw how we can create a superuser by running default and inbuilt terminal commands that automatically create the Admin and Auth models for us. Here, we will learn to create our own models to replace the dummy posts data we are currently using in the `blog` app.

For your reference, if you would like to skip to a particular section in the entire Django tutorial, you can do so by clicking on any of the links below:

- [Django Overview](00_overview.md)
- [Getting Started With Django](01_getting_started.md)
- [Applications and routes](02_applications_and_routes.md)
- [Templates in Django](03_templates.md)
- [Admin Page](04_admin_page.md)
- [Databases and Migrations](05_database_and_migrations.md)  (this article)
- [User Registration](06_user_registration.md)
- [User Login and Logout](07_user_login_and_logout.md)

### Table of Contents

This article is broken down into the following subsections:

- [Create A New Model](#create-a-new-model)
- [Create Migrations](#create-migrations)
- [Apply The Migrations](#apply-the-migrations)
- [Update The Database Interactively](#update-the-database-interactively)
- [Update The Admin GUI With Changes](#update-the-admin-gui-with-changes)
- [Use Real Post](#use-real-posts)


## Create A New Model

Django has an inbuilt Object Relational Mapper (ORM) that we can use to create a model. What an ORM does it it creates database objects using classes and methods instead of using raw SQL. The job of an ORM is to translate the high-level operations into database commands. The nice thing about Django is that we have the freedom to choose what database we want to use. We can utilize the simplicity of SQLite during development then configure our application to use PostgreSQL on production. 

To begin, let us define a simple `Post` model. Django already has created a `models.py` file in the `blog` app. This module shall be used to create custom models for the application. We do not have to worry about creating a `User` model, nor an authentication system because, if you noticed from [the previous article](04_admin_page.md), this was handled by Django for us. We did not have to do anything to create a user.

```python
# demo_project/blog_app/models.py

from django.db import models
from django.utils import timezone
from django.contrib.auth.models import User


class Post(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    timestamp = models.DateTimeField(default=timezone.now)
    author = models.ForeignKey(User, on_delete=models.CASCADE)

    def __repr__(self):
        return f'Post: {self.title}'
```

If you can recall the dummy posts we created in the article [templates](03_templates.md), each post had a _title_, a _content_ field, a _timestamp_ and an _author_. The `Post` model uses the same fields so that we do not have edit the `home.html` file again. However, if you would like to add a few more fields to the model, feel freed to do so, but remember to update the _home_ template.

A model is a single, definitive source of information about the data we will have. It contains all the essential fields and behaviours of the data we will soon be storing. Each model is a Python class that subclasses `django.db.models.Model`. The `models.py` module came with this import (`from django.db import models`) immediately we created the `blog` app.

Each field in a model has a field class type. The column type tells the database what kind of data to store. For example, the column `CharField` will store data of type `VARCHAR`, which are basically characters. We can optionally pass in some arguments  such as `max_length` to limit the amount of data a column can have. In the `Post` model, for example, we limit the size of a title to 100 characters.

The `DateTimeField` is used to represent Python's `datetime.datetime` instance, that is, the date and the time. There are various ways we can customize this field with arguments. As seen above, I have set that whenever a user posts something, the post's timestamp will use Django's utility timezone by default. Django stores datetime information in UTC in the database using time-zone-aware datetime objects internally that are automatically translated to the end user's zone in templates and forms. This allows a user to see the time he or she posted something depending on their time zone. You can read more on Django's handling of timezones [here](https://docs.djangoproject.com/en/4.1/topics/i18n/timezones/). 

We can also use the `auto_now=True` argument with the `DateTimeField`. This argument will auto-update a timestamp depending on when the post was last saved. The field will automatically be updated when calling `Model.save()`. A field with this argument is not updated when making edits to other fields in other ways.

`DateTimeField.auto_now_add` can be used to automatically set the field to now, the current date, when the object is first created. So, even if we set a default value when creating an object, it will be ignored.

Notice that we do not define a primary key field for the model. Contrary to Flask where we have to explicitly create a primary key field for each model, Django will do this for us in case we do not specify one. 

The `author` column is a bit different. This field is used to create a relationship between the `Post` table and the `User` table. The first argument is the Parent model, the source of one-to-many relationship. This is so because a user can author many posts, and a post can have only one user. So, from the `User` model direction, the relationship is a one-to-many whereas from the `Post` model direction it is a many-to-one relationship. 

The `on_delete` argument accepts [SQLAlchemy's Cascade](https://docs.sqlalchemy.org/en/20/orm/cascades.html) which is used to configure the behaviour of relationships. A cascade, in basic English terms, is a small waterfall that has several stages down a steep rocky slope. In the context of a database, one model can be related to another or many others. Operations performed on a "parent" object affects all children. In our blog application, if a user is deleted, that means that all their posts will lack a parent, which in turn cause a database error. To ensure that such an error does not occur, SQLAlchemy provides this feature (`on_delete=CASCADE`) to ensure that when a parent is deleted, all its children are also deleted. This saves us valuable time since we don't have to define functions that focus on deleting children data when a parent is deleted. Note that is does not affect a parent when a child is deleted. If we delete a post made by a user, the post's user will not be deleted as well. `on_delete` is usually configured on the many-to-one side fo the relationship where we have the `FOREIGN_KEY` contraint. At the ORM level, the direction is reversed. SQLAlchemy handles the deletion of 'child' objects relative the 'parent' from the 'parent' side. An overview of how the parent and child models would be defined (from an SQL standard) is as follows:

```python
class Parent(db.Model):
    db.Column(db.Integer, primary_key=True)
    children = db.relationship("Child", passive_deletes=True)


class Child(db.Model):
    db.Column(db.Integer, primary_key=True)
    children = db.ForeignKey("Parent", ondelete=CASCADE)

```

`passive_deletes=True` indicates that unloaded child items should not be loaded during a delete operation on the parent. Normally, when a parent item is deleted, all child items are loaded so that they can either be marked as deleted, or have their foreign key to the parent set to NULL. Marking this flag as True usually implies an ON DELETE <CASCADE|SET NULL> rule is in place which will handle updating/deleting child rows on the database side.

This model does not exhaust the field types available for use. You can refer to the [Model field documentation](https://docs.djangoproject.com/en/4.1/ref/models/fields/) to learn more. 


## Create Migrations

With the `Post` model in place, we need to now create this migration. Run this command on the terminal:

```python
(venv) python3 manage.py makemigrations

# Output

Migrations for 'blog':
  blog/migrations/0001_initial.py
    - Create model Post
```

A file called `0001_initial.py` will be created in the _blog/migrations_ directory. We can click on this file on your editor to see what it entials. We do not have to modify anything here, but it is good to understand what happened with the migrations. If we would like to see an SQL version of this file, we can run this:

```python
(venv) python3 manage.py slqmigrate blog 0001

# Output

BEGIN;
--
-- Create model Post
--
CREATE TABLE "blog_post" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "title" varchar(100) NOT NULL, "content" text NOT NULL, "timestamp" datetime NOT NULL, "author_id" integer NOT NULL REFERENCES "auth_user" ("id") DEFERRABLE INITIALLY DEFERRED);
CREATE INDEX "blog_post_author_id_dd7a8485" ON "blog_post" ("author_id");
COMMIT;
```

The beauty of using ORMs is that we do not really need to learn SQL commands such as the one shown above. They perform these high-level operations for us.


## Apply the Migrations

The next step is to apply our migrations to create the actual `Post` table in our database.

```python
(venv) $ python3 manage.py migrate

# Output

Operations to perform:
  Apply all migrations: admin, auth, blog, contenttypes, sessions
Running migrations:
  Applying blog.0001_initial... OK
```

This step now creats the database tables. As you can see, the `Post` model changes have been accepted. The `migrate` command is so powerful that we do not have to worry about how to apply any database changes without the need to delete the database or tables and make new ones. It simply upgrades our database live, without losing data.


## Update The Database Interactively

Django provides a shell access to our application from the terminal. Shell access allows us to test our database features even before we complete building the application. To invoke the Python shell in the context of Django, we can run this command in the terminal:

```python
(venv)$ python3 manage.py shell

# Output
Python 3.8.10 (default, Nov 14 2022, 12:59:47) 
[GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>>
```

Our quest right now is to update the `Post` model with data. A post will be made by a user. To do this interactively, we can run the following commands in the shell prompt:

```python
>>> from blog.models import Post
>>> from django.contrib.auth.models import User
>>> users = User.objects.all()
>>> for u in users:
...     u.id, u.username, u.email
... 
(1, 'harry', 'harry@email.com')
(4, 'gitau', '')
>>>
```

The application currently has two users. The `User.objects.all()` queries the database to retrieve all application users. We then loop through this list of users to access individual user's data. Django calls this list of users a `QuerySet` which is a list of objects. To make `harry` post a comment in the `Post` table, we can do the following:

```python
>>> user1 = User.objects.first()
>>> post = Post(title='Post 1', content='This is post 1', author=user1)
>>> post.save()
```

The `save()` function is used to update the `Post` model with the new post data. We have the option to now list the available posts found in the `Post` model.

```python
>>> posts = Post.objects.all()
>>> for p in posts:
...     p.title, p.content, p.author.username
... 
('Post 1', 'This is post 1', 'harry')
>>> 
```

An alternative way to create posts in the `Post` model is by using the `post_set.create()` function as follows:


```python
>>> user1_posts = user1.post_set.create(title='Second post', content='This is another post')
>>> posts.objects.all()
>>> posts = Post.objects.all()
>>> for p in posts:
...     p.title, p.author.username
... 
('Post 1', 'harry')
('Second post', 'harry')
>>> 
```

This function automatically adds the new post object directly to the database without calling the `save()` function.


## Update The Admin GUI With Changes

So far, the `Post` model has only two posts by user `harry`. The interesting thing here is that the `admin` site on http://127.0.0.1:8000/admin/ does not list the `Post` model.

![No Post Model in Admin Site](/02_django/images/05_migrations/no_post_model.png)

This outcome does not reflect the changes we have made thus far. To ensure that we have access to the `Post` model from the Admin Graphical User Interface (GUI), we need to update the `blog/admin.py` file with the following:

```python
# demo_project/blog_app/blog/admin.py

from django.contrib import admin
from .models import Post

admin.site.register(Post)
```

This module is used to register all our models. Above, I have imported the `Post` model from the same directory the `admin.py` module is found, denoted by the dot (`.`), and passed it as an argument to register it on the admin site. Now, if we refresh the admin site, we should be able to see the `Post` model with all its data.

![Posts Admin Site](/02_django/images/05_migrations/posts_admin_site.gif)


## Use Real Posts

The `views` module in the `blog` app utilizes dummy posts to populate our `home` template. Since we now have our own posts, we can get rid of the dummy posts and replace it with real ones.

```python
# demo_project/blog_app/blog/views.py

from django.shortcuts import render
from .models import Post


# Create your views here.
def home(request):
    context = {
        'posts': Post.objects.all()                  # < --- update
    }
    return render(request, 'blog/home.html', context)


def about(request):
    title = {
        'title': 'About'
    }
    return render(request, 'blog/about.html', title)

```

I used the same fields in my database as those in the dummy posts data, so I do not have to change anything in my home template. A simple refresh should show the two new posts.

![Real posts](/02_django/images/05_migrations/real_posts.png)

This is wonderful. Everything seems to be working well up until now. The last modification I would like to make on the application is on the timestamp. Notice that the timestamp has too much information which in the content of the app I want is not necessary. I would like the timestamp to only show the day, month, and year when a post is made. If you would like to accommodate the actual hour a post was made, then the current format of the timestamp should be sufficient.

```html
<!-- templates/blog/home.html -->

<small class="text-muted">{{ post.timestamp | date:"d M, Y" }}</small>
```

Django provides several date formats we can use as seen [here](https://docs.djangoproject.com/en/4.1/ref/templates/builtins/#date). If the value is a datetime object, for example, the result of `datetime.datetime.now()`, the output above would be "16 Dec, 2022". The format passed can be one of the predefined ones `DATE_FORMAT`, `DATETIME_FORMAT`, `SHORT_DATE_FORMAT` or `SHORT_DATETIME_FORMAT`, or a custom format that uses the format specifiers shown in the table above. The predefine formats listed may vary depending on the current locale.