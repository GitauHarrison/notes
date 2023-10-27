# Unique Resource Identifiers In Flask APIs

This article is dedicated to coding the endpoints used to work with the `User` and `Post` JSON [respresentations](/api_flask/02_resource_representation.md). If you recall the [Uniform Resource](/api_flask/00_overview.md#unique-resource-identifiers) principle of the REST architecture, user (and now posts) endpoints are used to access the individual resources.

Browse the completed code on [GitHub](https://github.com/GitauHarrison/api_in_flask/tree/v6.0.0-user-endpoints).

For your reference, these are the topics in our discussion:

- [Overview](00_overview.md)
- [API Blueprint](01_api_blueprint.md)
- [Resource Representation](02_resource_representation.md)
- [Error Handling](03_error_handling.md)
- [Unique Resource Identifiers](04_unique_resource_identifiers.md) (this article)
- [API Authentication](05_api_authentication.md)
- [API-friendly Error Messages](06_api_friendly_error_messages.md)
- [API Testing Using Postman](07_api_testing_postman.md)
- [API Testing And Documentation Using API Fairy](08_api_testing_documentation_using_api_fairy.md)


### Table Of Content

This article is broken down into the following subsections:

- [Retrieve A User And A Post](#retrieve-a-user-and-a-post)
  - [Test The API Route On The Browser](#test-the-api-route-on-the-browser)
  - [Test The API Route On The Terminal (Commandline)](#test-the-api-route-on-the-terminal-commandline)
- [Retrieving Collections Of Users And Posts](#retrieving-collections-of-users-and-posts)
- [Creating Entities](#creating-entities)
  - [Register A New User](#register-a-new-user)
  - [Make A Post](#make-a-post)
- [Editing Entities](#editing-entities)
- [Delete Entities](#delete-entities)



## Retrieve A User And A Post

Let us begin with the request to retrieve a single user and a single post given their `id`:

```python

# app/api/users.py: Return a user

from flask import jsonify
from app.models import User


@bp.route('/users/<int:id>', methods=['GET'])
def get_user(id):
    return jsonify(User.query.get_or_404(id).to_dict())
```

The view function `get_user()` receives a dynamic `id` argument that is used to return a `User` object if it exists. The variant `get_or_404()` method will abort the requests and return a 404 error to the client instead of `None` if the object does not exist. The advantage of `get_or_404()` over `get()` is that it removes the need to check the result of the query, thereby simplifying the logic in view functions.

You will notice that if you provide a large `id` value, the 404 error will be returned, but in HTML format. [Later](/api_flask/06_api_friendly_error_messages.md), we will learn how to return the 404 error in JSON format.

We begin by first getting a Python dictionary representation of the selected `User` object and then use Flask's `jsonify()` function to convert the dictionary to JSON format to return to the client. The same can be applied to a `Post` object.

```python
# app/api/posts.py: Return a post

from flask import jsonify
from app.models import Post


@bp.route('/posts/<int:id>', methods=['GET'])
def get_post(id):
    return jsonify(Post.query.get_or_404(id).to_dict())

```

### Test The API Route On The Browser

To see the output of our first API route, type (or copy and paste) the following URLs in your browser:

```python
# -------
# User endpoint
# -------

http://localhost:5000/api/users/1

# Output
{
  "_links": {
    "avatar": "https://www.gravatar.com/avatar/3f4360b2a748228ba4f745a3ebd428dc?d=identicon&s=128",
    "my_posts": "/api/users/1/my-posts",
    "self": "/api/users/1"
  },
  "about_me": null,
  "id": 1,
  "last_seen": "2023-10-15T01:01:16.120362Z",
  "post_count": 2,
  "username": "harry"
}

# -------
# Post endpoint
# -------

http://localhost:5000/api/posts/1

# Output
{
  "_links": {
    "to_post_author": "/api/users/1",
    "to_this_post": "/api/posts/1"
  },
  "author": {
    "about_author": null,
    "id": 1,
    "post_count": 2,
    "username": "harry"
  },
  "body": "Hi, I am new here",
  "id": 1,
  "timestamp": "2023-10-13T04:40:57.718678Z",
  "title": "Hello"
}
```

The user endpoint returns all the fields we defined in our `to_dict()` method (I have added the `my_posts` link that returns a list of all the actual posts authored (not the post count) by the user identified by the `id`). The view function rendering the URL _/api/users/1/my-posts_ is as follows:

```python
# app/api/users.py: List of posts authored by a user

@bp.route('/users/<int:id>/my-posts', methods=['GET'])
def get_posts_by_author(id):
  return jsonify(
    [
      {'title': post.title, 'body': post.body} \
        for post in User.query.get_or_404(id).posts.all()
    ]
  )
```

If you navigate to the endpoint http://localhost:5000/api/users/1/my-posts, you will see the list of posts by the selected user. In the example below, the user _harry_ whose `id` is `1` has two posts:

```json
[
  {
    "body": "Hi, I am new here",
    "title": "Hello"
  },
  {
    "body": "@gitau This is a simple chat app",
    "title": "Simple chat app"
  }
]
```

### Test The API Route On The Terminal (Commandline)

It is also possible to get the JSON responses seen above on your terminal. Let us begin by installing [HTTPie](https://httpie.io/), a command-line HTTP client written in Python that makes it easy to send API requests:

```python
(venv)$ pip3 install httpie
```

Now, if we want to request information about a user through the command-line, we can do so by specifying the type of request associated with an endpoint.

```python
# User
(venv)$ http GET http://localhost:5000/api/users/1

# Output
http: error: ConnectionError: HTTPConnectionPool(host='localhost', port=5000): \
Max retries exceeded with url: /api/users/1 (Caused by NewConnectionError('<urllib3.connection.HTTPConnection '
'object at 0x7fe6753ffc40>: Failed to establish a new connection: [Errno 111] Connection refused')) \
while doing a GET request to URL: http://localhost:5000/api/users/1
```

The error above indicates that the Flask server is not running. Make sure that you are running your application's server (`flask run`), then try again.

```python
# User response
(venv)$ http GET http://localhost:5000/api/users/1

# Output
HTTP/1.1 200 OK
Connection: close
Content-Length: 309
Content-Type: application/json
Date: Sun, 15 Oct 2023 01:47:21 GMT
Server: Werkzeug/2.3.6 Python/3.8.10
Vary: Cookie

{
    "_links": {
        "avatar": "https://www.gravatar.com/avatar/3f4360b2a748228ba4f745a3ebd428dc?d=identicon&s=128",
        "my_posts": "/api/users/1/my-posts",
        "self": "/api/users/1"
    },
    "about_me": null,
    "id": 1,
    "last_seen": "2023-10-15T01:01:16.120362Z",
    "post_count": 2,
    "username": "harry"
}

# Try to get a post response too
```

## Retrieving Collections Of Users And Posts

In the article [Resource Representations](/api_flask/02_resource_representation.md#representing-collections-of-users), we created the `to_collection_dict()` that had a generic `PaginatedAPIMixin` class. Now, we can rely on this class to return a collection of users and posts.

```python
# app/api/users.py: Return collection of users

from flask import request


@bp.route('/users', methods=['GET'])
def get_users():
  page = request.args.get('page', 1, type=int)
  per_page = min(request.args.get('per_page', 10, type=int), 100)
  data = User.to_collection_dict(User.query, page, per_page, 'api.get_users')
  return jsonify(data)

```

We begin by extracting the `page` and `per_page` query strings from the request, providing 1 and 10 as the defaults. We cap the `per_page` at 100 to mitigate any performance problems when the client requests extremely large data. To return a list of users, we get the collection dictionary by passing in all the relevant arguments needed in the representation. Once that is done, we can test this endpoint on the command-line as follows:

```python
# Return a list of users
(venv)$ http GET http://localhost:5000/api/users

# Output
HTTP/1.1 200 OK
Connection: close
Content-Length: 931
Content-Type: application/json
Date: Sun, 15 Oct 2023 02:31:52 GMT
Server: Werkzeug/2.3.6 Python/3.8.10
Vary: Cookie

{
    "_links": {
        "next": null,
        "prev": null,
        "self": "/api/users?page=1&per_page=10"
    },
    "_meta": {
        "page": 1,
        "per_page": 10,
        "total_items": 2,
        "total_pages": 1
    },
    "items": [
        {
            "_links": {
                "avatar": "https://www.gravatar.com/avatar/3f4360b2a748228ba4f745a3ebd428dc?d=identicon&s=128",
                "my_posts": "/api/users/1/my-posts",
                "self": "/api/users/1"
            },
            "about_me": null,
            "id": 1,
            "last_seen": "2023-10-15T01:01:16.120362Z",
            "post_count": 2,
            "username": "harry"
        },
        {
            "_links": {
                "avatar": "https://www.gravatar.com/avatar/fecca06940d3dc7dcc37a62cf773bf27?d=identicon&s=128",
                "my_posts": "/api/users/2/my-posts",
                "self": "/api/users/2"
            },
            "about_me": null,
            "id": 2,
            "last_seen": "2023-10-15T00:43:05.494829Z",
            "post_count": 1,
            "username": "gitau"
        }
    ]
}
```

The same can also be applied to the `Post` model so that we get a list of posts from the request.

```python
# app/api/posts.py: Return collection of posts

from flask import request


@bp.route('/posts', methods=['GET'])
def get_posts():
  page = request.args.get('page', 1, type=int)
  per_page = min(request.args.get('per_page', 10, type=int), 100)
  data = Post.to_collection_dict(Post.query, page, per_page, 'api.get_posts')
  return jsonify(data)

```

## Creating Entities

There are two resources we are going to create: (1) user and (2) post. The HTTP method used to create entities is `POST`.

### Register A New User

To register a user, we are going to use the `POST` request of the view function `create_user()`. This request is going to accept a user representation in JSON format from the client.

```python
# app/api/users.py: Create a user

from app.api.errors import bad_request
from flask import url_for
from app import db


@bp.route('/users', methods=['POST'])
def create_user():
    data = request.get_json() or {}
    if 'username' not in data or 'email' not in data or 'password' not in data:
        return bad_request('must include username, emal and password fields')
    if User.query.filter_by(usename=data['username']).first():
        return bad_request('please use a different username')
    if User.query.filter_by(email=data['email']).first():
        return bad_request('please use a different email address')
    user = User()
    user.from_dict(data, new_user=True)
    db.session.add(user)
    db.session.commit()
    response = jsonify(user.to_dict())
    response.status_code = 201
    response.headers['Location'] = url_for('api.get_user', id=user.id)
    return response

```

Flask provides the `get_json()` method to extract JSON from a request and return a Python dictionary or `None` if JSON data is not found in the request. In the event `None` is returned, this will cause an error in the API. To ensure that we always get a dictionary, we provide the fallback `{}` such that the new data expression is `get_json() or {}`.

To register a new user, we make sure we got all the information by checking if the mandatory three fields have been included (the `about_me` field is not mandatory). Also, we make sure that the data being submitted does not exist in the `User` table (meaning they are being used by another user). Should any of those checks fail, we provide an informative error message to the client. 

Once all validations are passed, we can now register a new user. The argument `new_user` is set to True to allow for the addition of the password which originally was not part of the user representation.

We, then, need to return this user's response so the `to_dict()` returns the payload. The status code for a payload that creates a new resource or entity is 201. The HTTP protocol requires that a 201 response includes a `Location` header that is set to the URL of the new resource.

Below, let us see how we can register a new user from the command-line using HTTPie:

```python
# Remember to remove the \ (it has been used to visually break the line)

(venv)$ http POST http://localhost:5000/api/users username=muthoni \
  password=muthoni123 email=muthoni@email.com \
  "about_me=I am learning what APIs are."

# Output
HTTP/1.1 201 CREATED
Connection: close
Content-Length: 337
Content-Type: application/json
Date: Sun, 15 Oct 2023 03:27:13 GMT
Location: /api/users/3
Server: Werkzeug/2.3.6 Python/3.8.10
Vary: Cookie

{
    "_links": {
        "avatar": "https://www.gravatar.com/avatar/0073cf58337d9c43bb12909e60929e2a?d=identicon&s=128",
        "my_posts": "/api/users/3/my-posts",
        "self": "/api/users/3"
    },
    "about_me": "I am learning what APIs are.",
    "id": 3,
    "last_seen": "2023-10-15T03:27:13.823094Z",
    "post_count": 0,
    "username": "muthoni"
}
```


### Make A Post

Posts can be made by registered users. To make a post, we need to associate that post with an existing user in the database.

```python
# app/api/posts.py: Make a post

from app import db
from app.api.errors import bad_request


@bp.route('/posts/user/<int:id>', methods=['POST'])
def create_post(id):
  data = request.get_json() or {}
  user = User.query.get_or_404(id)
  if 'title' not in data or 'body' not in data:
      return bad_request('title and body fields must be included.')
  post = Post(title=data['title'], body=data['body'], author=user)
  db.session.add(post)
  db.session.commit()
  response = jsonify(post.to_dict())
  response.status_code = 201
  response.headers['Location'] = url_for('api.get_post', id=post.id)
  return response
```

Create a sample post in your terminal as follows:

```python
# Remember to remove the \ (it has been used to visually break the line)

(venv)$ http POST http://localhost:5000/api/posts/user/3 \
  "title=Make A Post" "body=Get the URL arguments to complete a POST request"

# Output
{
    "_links": {
        "to_post_author": "/api/users/3",
        "to_this_post": "/api/posts/4"
    },
    "author": {
        "about_author": "I am learning what APIs are.",
        "id": 3,
        "post_count": 1,
        "username": "muthoni"
    },
    "body": "Get the URL arguments to complete a POST request",
    "id": 4,
    "timestamp": "2023-10-15T03:51:26.552996Z",
    "title": "Make A Post"
}
```

## Editing Entities

Let us begin by modifying the information of an existing user:

```python
# app/api/users.py: Edit a user


@bp.route('/users/<int:id>', methods=['PUT'])
def update_user(id):
    user = User.query.get_or_404(id)
    data = request.get_json() or {}
    if 'username' in data and data['username'] != User.query.filter_by(username=data['username']).first():
        return bad_request('please a different username')
    if 'email' in data and data['email'] != User.query.filter_by(emal=data['email']).first():
        return bad_request('please use a different email address.')
    user.from_dict(data, new_user=False)
    db.session.commit()
    return jsonify(user.to_dict())

```

Here is an example that edits the `about_me` field with HTTPie:

```python
(venv)$ http PUT http://localhost:5000/users/1 "about_me=Hello, I am Harry"

# Output
HTTP/1.1 200 OK
Connection: close
Content-Length: 324
Content-Type: application/json
Date: Sun, 15 Oct 2023 04:29:27 GMT
Server: Werkzeug/2.3.6 Python/3.8.10
Vary: Cookie

{
    "_links": {
        "avatar": "https://www.gravatar.com/avatar/3f4360b2a748228ba4f745a3ebd428dc?d=identicon&s=128",
        "my_posts": "/api/users/1/my-posts",
        "self": "/api/users/1"
    },
    "about_me": "Hello, I am Harry",
    "id": 1,
    "last_seen": "2023-10-15T01:01:16.120362Z",
    "post_count": 3,
    "username": "harry"
}

# You will notice that the about_me field has changed from null to "Hello, I am Harry"
```

A post can also be modified:

```python
# app/api/posts.py: Modify a post


# Update post view function
@bp.route('/posts/<int:id>', methods=['PUT'])
def update_post(id):
    post = Post.query.get_or_404(id)
    data = request.get_json() or {}
    post.from_dict(data)
    db.session.commit()
    return jsonify(post.to_dict())


# Edit using HTTPie
(venv)$ http PUT http://localhost:5000/api/posts/1 "title=Edited title"

# Output
HTTP/1.1 200 OK
Connection: close
Content-Length: 385
Content-Type: application/json
Date: Sun, 15 Oct 2023 04:35:54 GMT
Server: Werkzeug/2.3.6 Python/3.8.10
Vary: Cookie

{
    "_links": {
        "to_other_posts_by_author": "/api/users/1/my-posts",
        "to_post_author": "/api/users/1",
        "to_this_post": "/api/posts/1"
    },
    "author": {
        "about_author": "Hello, I am Harry",
        "id": 1,
        "post_count": 3,
        "username": "harry"
    },
    "body": "Hi, I am new here",
    "id": 1,
    "timestamp": "2023-10-13T04:40:57.718678Z",
    "title": "Edited title"
}
```

## Delete Entities

Now that you have learnt how to modify the information of an existing resource, it has been left to the reader to try **delete** an existing resource.
