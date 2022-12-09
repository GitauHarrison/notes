# Unit Testing in Flask

In this article, I will be using the sample project [Configure Flask To Use PostgreSQL](https://github.com/GitauHarrison/configure-flask-to-use-postgresql) to discus how to write unit tests in a Flask web application. The concepts, though, are universal and can be applied to other frameworks as well.

![Unit testing in flask](/unit_testing/images/unit_testing_in_flask.gif)

Before we can go any further, these are the other sections in the unit testing reviews. You can click on any of the following links to access additional content:

1. [Unit Testing Overview](/unit_testing/unit_testing_overview.md)
2. [Unit Testing In Python](/unit_testing/unit_testing_in_python.md)
3. [Unit Testing In Flask](/unit_testing/unit_testing_in_flask.md) (this article)

If you would like to follow along using this application, follow [these guidelines](https://github.com/GitauHarrison/configure-flask-to-use-postgresql#testing-the-application-locally) to get started. At this point in the setup process, let us also install the following packages which we shall use to create the testing infrastructure.

```python
(venv)$ pip3 install pytest pytest-cov
```

If you haven't already checked out the other reviews on unit testing, I recommend you go back to them to learn more, then continue with this. As I write this review, there is no `test` module in the application's root directory. I will be adding one and updating it as we go along. By the time you clone the repository, you may find that it has a test module, but that is because I added it by the end of this tutorial.

The application features the following:

- [x] A user can register for an account
- [x] A registered user's data is stored in a database
- [x] The user's data is retrieved to allow them to log in
- [x] Once logged in, they are redirected to the profile page
- [x] Useful feedback is provided to the user along the way
- [x] The profile page can only be accessed by a logged in user
- [x] The profile page displays the logged in user's username
- [x] The profile page features a _post_ form
- [x] The user can make a post, which would be displayed below the form
- [x] The logged in user can log themselves out of their accounts
- [x] Once logged out, they are redirected to the login page

It is important to know what parts the application you want to build will have so that you can adequately plan for them by creating tests that need to be passed before accepting any changes. Python includes a very useful `unittest` package that makes it easy to write and execute unit tests. In this example, I will be using an "enhanced" test runner called `pytest`.

## Test Application Instance

Let us begin by creating a `test` module in the top-level directory. This file will define all our test cases for the application.

```python
(venv)$ touch test_web_app.py
```

The first test case we are going to look at is to determine the application's instance. A flask application's instance of the project in review is in the variable `app` as seen in `app/__init__.py`. You've seen that `app` is called in almost all other modules, for example, `routes.py` needs this flask instance to create endpoints. If we are going to test every aspect of the application, chances are we will need the flask instance in each test. This will make our work tedious, so we are going start by defining common initialization and destruction tasks in a single place.

```python
# test_web_app.py

from app import app
import unittest


class TestWebApp(unittest.TestCase):
    def setUp(self):
        self.app = app
        self.app_ctxt = self.app.app_context()
        self.app_ctxt.push()

    def tearDown(self):
        self.app_ctxt.pop()
        self.app = None
        self.app_ctxt = None

    def test_app(self):
        assert self.app is not None
        assert app == self.app

```

The methods `setUp()` and `tearDown()` are automatically invoked before and after each test, and are therefore mandatory to have. It makes sense to have them in one place since they are common logic that will apply to all tests in the class. 

The `setUp()` method has been used to create an application instance, which is stored in the `app_ctxt` attribute. This attribute is then used to create an application context, often needed in Flask. 

If you are wondering what an application context is, it used to keep track of the application-level data during requests, commandline activities or other tasks. A Flask application is most likely going to have a `config` module, for example, whose values are needed everywhere in the application, such as the views and CLI commands. 

To ensure that we have access to such, an application context is used. When using blueprints and factory functions, the `current_app` proxy from flask is used to access the application context rather than having to import the `app` instance all the time. Flask automatically pushes an application context when handling requests. All functionality that run during a request, such as views, error handling et cetera, will have access to `current_app`, a proxy to a Flask context. Below is a slight modification of testing an application instance when using a factory function. 

```python
# test_web_app.py

import unittest
from app import create_app
from flask import current_app


class TestWebApp(unittest.TestCase):
    def setUp(self):
        self.app = create_app()
        self.app_ctxt = self.app.app_context()
        self.app_ctxt.push()

    def tearDown(self):
        self.app_ctxt.pop()
        self.app = None
        self.app_ctxt = None

    def test_app(self):
        assert self.app is not None
        assert current_app == self.app
```

The `test_app()` method in the `test`module ensures that `self.app` has been defined, and also either `app` or `current_app` has been set to the application, which should happen when the context is pushed.

The `tearDown()` method, which runs after every test, is used to undo anything that was done in `setUp()`. So, a pop is implemented on the application context, and resets the two attributes back to `None` so the class returns to its original clean state.

In the terminal, we can run the test and see the coverage.

```python
(venv)$ pytest --cov=test_web_app --cov-report=term-missing --cov-branch

# Output
========================== test session starts ===========================
platform linux -- Python 3.8.10, pytest-7.2.0, pluggy-1.0.0
rootdir: /home/harry/connecting_to_postgresql
plugins: cov-4.0.0
collected 1 item

test_web_app.py .                                                  [100%]

---------- coverage: platform linux, python 3.8.10-final-0 -----------
Name              Stmts   Miss Branch BrPart  Cover   Missing
-------------------------------------------------------------
test_web_app.py      14      0      0      0   100%
-------------------------------------------------------------
TOTAL                14      0      0      0   100%


=========================== 1 passed in 0.44s ============================
```

## Test Database

The application in review makes use of a database to store user information. It is really up to you to determine if you want to test the database or not. The first thing I normally do when testing a database is to use an actual test database. This allows me to access all database features without modifying the production one. 

The way to do this is to use an in-memory database. Since I am using `sqlalchemy`, an in-memory database URL would be `sqlite://` without providing the database name.

```python
# test_web_app.py

import os
os.environ['DATABASE_URL'] = 'sqlite://'

from app import app, db
import unittest


class TestWebApp(unittest.TestCase):
    def setUp(self):
        self.app = app
        self.app_ctxt = self.app.app_context()
        self.app_ctxt.push()
        db.create_all() # < --- update

    def tearDown(self):
        db.drop_all() # < --- update
        self.app_ctxt.pop()
        self.app = None
        self.app_ctxt = None

    def test_app(self):
        assert self.app is not None
        assert app == self.app
```

Intentionally, I set the in-memory database at the global scope just before all other imports to ensure that by the time the `config` module is imported, this variable is already set correctly. In the `setUp()` method, a call to initialize the in-memory database with empty tables for all models defined in the application is achieved using `db.create_all()`. For this to work, an application context is required, so it needes to be called after the application context has been pushed. It may be technically unnecessary to destroy all tables by calling `db.drop_all()` but anyways, let us just have it there.

## Working With a Test Client

To test requests, a web server is typically used. The communication between a web application and a web server is enabled by a Web Server Gateway Interface (WSGI) such as Gunicorn. Following a test-driven approach, where testing comes before building, there is actually no server to use to test such requests. Thankfully, we can use something called a "test client" to inject fake requests into the application for the purposes of testing. This would look like a real request has been sent but there will be no server involved.

```python
import os
os.environ['DATABASE_URL'] = 'sqlite://'
os.environ['SECRET_KEY'] = 'harry'

from app import app, db
import unittest


class TestWebApp(unittest.TestCase):
    def setUp(self):
        self.app = app
        self.app_ctxt = self.app.app_context()
        self.app_ctxt.push()
        db.create_all()
        self.client = self.app.test_client()

    def tearDown(self):
        db.drop_all()
        self.app_ctxt.pop()
        self.app = None
        self.app_ctxt = None
        self.client = None

    def test_app(self):
        assert self.app is not None
        assert app == self.app

    def test_profile_page_redirect(self):
        response = self.client.get('/', follow_redirects=True)
        assert response.status_code == 200
        assert response.request.path == '/login'

```

The `test_profile_page_redirect()` function sends a GET request to the top-level URL of the application. I have used the flag `follow_redirects` to set that the test client automatically handles the redirect responses. In the application, anonymous access to the profile page automatically redirects the user to the login page. Details of the originating requests is retrieved using the `response.request` attribute.

One thing to note is that the application in review does not use a factory function. As a result, we do not have access to the `config` module as is normally the case when `create_app` is used. This forces us to explicitly add all our configuration variables globally before the test is run and executed.


## Testing HTML-Specific Content

To give context on why you would consider testing for HTML-specific content, the profile page displays the current user's username. It is possible that given this definite output, we can test if indeed the profile page displays a logged in user's username. In another context, we can verify if a redirect to the login page actually displayed the login form by testing for specific HTML content such as the input fields or the login button. It may not make sense to go all out to test every aspect of a template. A "spot-check" would be sufficient.

```python
# test_web_app.py

import os
os.environ['DATABASE_URL'] = 'sqlite://'
os.environ['SECRET_KEY'] = 'harry'

from app import app, db
import unittest


class TestWebApp(unittest.TestCase):
    
    # ...

    def test_login_form(self):
        response = self.client.get('/login')
        assert response.status_code == 200
        html = response.get_data(as_text=True)

        assert 'name="username"' in html
        assert 'name="password"' in html
        assert 'name="remember_me"' in html
        assert 'name="submit"' in html

```

Above, I am making sure the GET response returns a 200 status code. Then, I extract the HTML response object from `response.get_data()`. By default, the returned object will be of type `bytes`, but as a matter of convinience, I request for its conversion to text. To check if a particular field is present in the form, I use the attribute `name="field-name"`. If you are familiar with HTML, you know that an input field has the format `<input ... name="field-name">`. So, I am utilizing the `name` attribute to verify that indeed the field is present in the login template.

