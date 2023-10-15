# Implement An API Blueprint In Flask

Now that we have a solid understanding of the REST API architecture, it is time to start implementing one on a project. In the [Overview](00_overview.md) article, you saw the end project, fully built with API developed for it. However, to help you learn how it was built, we will start from the bottom. I have already created a [basic chat application](https://github.com/GitauHarrison/api_in_flask/tree/v1.0.0-basic-app) that you can refer to as you go through this article. To compare your work with the article's final code, refer to [v3.0.0-api-blueprint](https://github.com/GitauHarrison/api_in_flask/tree/v3.0.0-api-blueprint) release on GitHub.

For your reference, these are the topics in our discussion:

- [Overview](00_overview.md)
- [API Blueprint](01_api_blueprint.md) (this article)
- [Resource Representation](02_resource_representation.md)
- [Error Handling](03_error_handling.md)
- [Unique Resource Identifiers](04_unique_resource_identifiers.md)
- [API Authentication](05_api_authentication.md)
- [API-friendly Error Messages](06_api_friendly_error_messages.md)
- [API Testing Using Postman](07_api_testing_postman.md)

### Table Of Content

This article is broken down into the following subsections:

- [Overview](#overview)
- [Create An API Blueprint](#create-an-api-blueprint)
    - [Users Resources](#users-resources)
    - [Posts Resources](#posts-resources)
    - [Errors Module](#errors-module)
    - [Tokens Module](#tokens-module)
    - [Register API Blueprint With The Factory Function](#register-api-blueprint-with-the-factory-function)


## Overview

In the linked basic chat application (_v1.0.0-basic-app_ above), no blueprints have been used. A blueprint is a logical structure that represents a subset of an application. For example, instead of bundling everything in `routes.py`, we can choose to separate the logic into **auth** for authentication-related features or **errors** for error-related features. Check out the [_v2.0.0-blueprint](https://github.com/GitauHarrison/api_in_flask/tree/v2.0.0-blueprint) release. Why use a blueprint? Blueprints are fantastic for larger applications. Should I ever want to scale this application, a new project structure utilizing blueprints would be suitable. 


## Create An API Blueprint

To create an _api_ blueprint, let us start by creating an empty folder called _api_ in the _app/_ sub-directory.

```python
(venv)$ mkdir app/api
# I am running this command from the application's root directory
```

Create a new file called `__init__.py` in the new directory:

```python
(venv)$ touch  app/api/__init__.py
```

Update this new file with the following initialization:

```python
# app/__init__.py: Initialize the API blueprint

from flask import Blueprint

bp = Blueprint('api', __name__)

from app.api import users, errors, tokens, posts
```

The last line requires us to create the four files (also known as modules -- they do not exist yet). They are imported at the bottom to avoid circular dependency errors.

```python
(venv)$ cd app
(venv)app$ touch users errors tokens posts
```

The gist of our API is going to be in the __app/api/users.py__ and __app/api/posts.py__ modules.


### Users Resources

To begin with the `_app/api/users.py_` module, these are the endpoints we are going to create and work with:

| HTTP Method | Resource URL | Description |
| ----------- | ------------ | ----------- |
| GET | /api/users/user_id | Return a user based on their `id` |
| GET | /api/users | Return a collection of users |
| POST | /api/users | Create a new user account |
| PUT | /api/users/user_id| Modify a user of `id` |

The API endpoints for the _users_ are going to be as follows (skeleton):

```python
# app/api/users.py: User API resources

from app.api import bp


@bp.route('/users/<int:id>', methods=['GET'])
def get_user(id):
    pass


@bp.route('/users', methods=['GET'])
def get_users():
    pass


@bp.route('/users', methods=['POST'])
def create_user():
    pass


@bp.route('/users/<int:id>/', methods=['PUT'])
def update_user(id):
    pass
```


### Posts Resources

The endpoints that we are going to work with on the **app/api/posts.py** module are as follows:

| HTTP Method | Resource URL | Description |
| ----------- | ------------ | ----------- |
| GET | /api/posts/post_id | Return a post based on its `id` |
| GET | /api/posts | Return a collection of posts |
| POST | /api/posts | Create a new post |
| PUT | /api/posts/post_id| Modify the post of `id` |

The API endpoints for the _posts_ are going to be as follows (skeleton):

```python
# app/api/posts.py: Posts API resources

from app.api import bp


@bp.route('/posts/<int:id>', methods=['GET'])
def get_post(id):
    pass


@bp.route('/posts', methods=['GET'])
def get_posts():
    pass


@bp.route('/posts', methods=['POST'])
def create_post():
    pass


@bp.route('/posts/<int:id>', methods=['PUT'])
def update_post(id):
    pass


@bp.route('/posts/<int:id>', methods=['DELETE'])
def update_post(id):
    pass
```

Something about the posts APIs is that the posts have to be linked to a user. For example, in the POST method (while working with posts APIs), we need to ensure that a post is made by a logged-in user identified by their `id`. This post will be associated with them. Similarly, when interacting with a post (say to update or delete one), a logged-in user can only do so to posts they have authored.


### Errors Module

We can define a few helper functions to deal with error responses. To begin, let us add the following:

```python
# app/api/errors.py: Deal with error responses

def bad_request():
    pass
```

### Tokens Module

This is where the authentication subsystem is going to be defined. We will provide an alternative way for clients who are not web browsers to log in.

```python
# app/api/tokens.py: API authentication module

def get_token():
    pass


def revoke_token():
    pass

```

### Register API Blueprint With The Factory Function

Finally, to complete the setup process, we can now register the API blueprint with the application factory function:

```python
# app/__init__.py: Register API blueprint with factory function


def create_app(config_class=Config):
    app = Flask(__name__)

    # ...

    from app.api import bp as api_bp
    app.register_blueprint(api_bp, url_prefix='/api')

    # ...

```
