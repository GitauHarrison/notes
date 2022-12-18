# Django Admin

One of the beautiful things about Django is that it comes with an already built admin page for us. Unlike Flask where you have to create one from scratch, Django comes ready with an admin interface able to read metadata from our models to provide quick, model-centric interface where trusted users can manage content on our site.

For your reference, if you would like to skip to a particular section in the entire Django tutorial, you can do so by clicking on any of the links below:

- [Django Overview](00_overview.md)
- [Getting Started With Django](01_getting_started.md)
- [Applications and routes](02_applications_and_routes.md)
- [Templates in Django](03_templates.md)
- [Admin Page](04_admin_page.md)  (this article)
- [Databases and Migrations](05_database_and_migrations.md)
- [User Registration](06_user_registration.md)
- [User Login and Logout](07_user_login_and_logout.md)

### Table of Contents

This article is broken down into the following subsections:

- [Access Admin Page](#access-admin-page)
- [Create A Superuser](#create-a-superuser)
- [Add Another Admin](#admin-another-admin)


## Access Admin Page

From our previous lessons, we have learnt that to access the `blog` app, all we need to do is to append the word "blog" to the end of the localhost URL as follows: http://127.0.0.1:8000/blog. If you had set the root URL to redirect to the `blog` app then you definitely do not need to append `/blog`. You can leave the localhost URL as http://127.0.0.1:8000/. If you look closely at the top-level project's `url.py` file, you will notice that we have defined one of the URL patterns as `/admin`.

```python
# demo_project/blog_style_app/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls'))
]
```

To access the admin page, therefore, we need to append the word "/admin" to the localhost URL as follows: http://127.0.0.1:8000/admin. This will redirect as to the admin login page.

![Admin Login Page](/02_django/images/04_admin_page/admin_login_page.png)


## Create Admin Model

We cannot log in an admin because at this stage none exists. To get started, we will need to create some migrations from our project's terminal as follows. Note that I am running this command from the top-level project's directory with an active virtual environment:

```python
(venv) python3 manage.py migrate

# Output

Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying sessions.0001_initial... OK
```

## Create A Superuser

Our initial migration was created successfully, and now that Auth table should now exist. To create a `superuser`, we will need to run this other command:

```python
(venv)$ python3 manage.py createsuperuser
```

This is an interactive command that guides us to create a superuser by providing a username and their password.

![Create superuser](/02_django/images/04_admin_page/create_superuser.gif)

The admin `harry` was successfully created, as seen in the image above. To access the admin page using this user's credentials, we will need to restart our server first (if you do not have one running already) to access the admin page.

```python
(venv)$ python3 manage.py runserver
```

Access the admin page by pasting the URL http://127.0.0.1:8000/admin on a new page. Login as `harry`.

![Log in as an admin](/02_django/images/04_admin_page/login_as_admin.gif)

Once logged in, you will notice that there is a link to the application's `Users`. If you click on this, `harry` will be listed as the sole user at the moment. So far, this user has only an email address. If we would like to update his details, then we can do so by clicking on his name as seen in the image above.

Django, by default, offers password hashing right out of the box. It does not store a user's raw password, but rather hashes it to create a representation to preserve the integrity of user data in the event the database is compromised. Since `harry` is a superuser, you will notice that he has been assigned all three available permissions, that is, `active`, `Staff status` and `Superuser status`.


## Admin Another Admin

From the dashboard, we can add other admins. This can be achieved by clicking the `ADD USER` button located on the top-right corner of the admin page.

![Add another admin user](/02_django/images/04_admin_page/add_admin.png)

Provide a username and a password, then click "Save". This new user will be added successfully with a permission status of `active` enabled by default. To allow this new user to log into the admin site, you will need to enable the `Staff status` permission. 

At the very bottom of this new user's details page, you may optionally delete him.

![Delete new admin](/02_django/images/04_admin_page/delete_admin.png).