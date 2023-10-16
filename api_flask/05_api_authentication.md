# API Authentication In Flask

On a web browser, a user needs to provide some sort of authentication (AuthN) and Authorization (AuthZ) to access certain features of the application. When working with APIs, we can extend the same to all clients such that there needs to be some sort of identification so that the server knows what user the client is representing, and can verify if the requested action is allowed or not. In the article [Unique Resource Identifiers](/api_flask/04_unique_resource_identifiers.md), you may have noticed that all endpoints are open to any client.

Browse the completed code on [GitHub](https://github.com/GitauHarrison/api_in_flask/tree/v7.0.0-token-authentication).

For your reference, these are the topics in our discussion:

- [Overview](00_overview.md)
- [API Blueprint](01_api_blueprint.md)
- [Resource Representation](02_resource_representation.md)
- [Error Handling](03_error_handling.md)
- [Unique Resource Identifiers](04_unique_resource_identifiers.md)
- [API Authentication](05_api_authentication.md) (this article)
- [API-friendly Error Messages](06_api_friendly_error_messages.md)
- [API Testing Using Postman](07_api_testing_postman.md)


### Table Of Content

This article is broken down into the following subsections:

- [Overview](#overview)
- [Working With Tokens](#working-with-tokens)
- [Token Requests](#token-requests)
- [Protecting API Endpoints With Tokens](#protecting-api-endpoints-with-tokens)
- [Revoking Tokens](#revoking-tokens)



## Overview

The most obvious way to protect a route is to use the `@login_required` decorator. The problem with using it is that any unauthenticated client will be redirected to an HTML login page. When working with APIs, there is no such thing as HTML or login pages. The server should not assume that the client is a web browser, or that the client can handle redirects or render and process HTML login pages. Instead, if a client's request lacks or misses valid data, the server should refuse the connection and return a 401 status code. With a 401 status code, the client will know that it needs to ask the user for credentials (which is not the business of the server).


## Working With Tokens

We can use the token authentication scheme to help authenticate APIs. When a client needs to interact with an API, it needs to request a temporary token, authenticating with a _username_ and _password_. The client can then send API requests passing the token as authentication for as long as the token is valid. Once expired, a new token needs to be requested.

```python
# app/models.py: Support for tokens

import secrets
from datetime import datetime, timedelta
import os


class User(UserMixin, PaginatedAPIMixin, db.Model):
    # ...
    token = db.Column(db.String(32), index=True, unique=True)
    token_expiration = db.Column(db.DateTime)

    # ...
    def get_token(self, expires_in=120):
        now = datetime.utcnow()
        if self.token and self.token_expiration > now + timedelta(seconds=60):
            return self.token
        self.token = secrets.token_urlsafe(16)
        self.token_expiration = now + timedelta(seconds=expires_in)
        db.session.add(self)
        return self.token

    def revoke_token(self):
        self.token_expiration = datetime.utcnow() - timedelta(seconds=1)

    @staticmethod
    def check_token(token):
        user = User.query.filter_by(token=token).first()
        if user is None or user.token_expiration < datetime.utcnow():
            return None
        return user

```

The change above adds a unique `token` attribute to the `User` model which should be indexed to improve query performance. We also have a new column called `token_expiration` which contains the date and time a token expires. Tokens that remain valid for too long can become a security risk hence the need to have a `token_expiration` column.

The `get_token()` method generates a very random token for the user. Before a new token is created, the method first checks if there is an existing token and if the existing token has at least one minute to expire, in which case, it returns the token.

Intentionally, we have created a `revoke_token()` method to improve security. Should we want to revoke a token immediately instead of relying on the expiration feature only, we can do so by invoking this function.

The static `check_token()` method verifies a token before returning a user. Should the token be invalid or expired, it will return `None`.

Since these changes update the schema of the `User` model, we need to generate a new migration and upgrade the database:

```python
(venv)$ flask db migrate -m 'user tokens'
(venv)$ flask db upgrade
```

## Token Requests

The real power of APIs is that any client, be it a smartphone or a single-page application, can interact with a backend. Therefore, when developing APIs, we have to consider that our clients are not always going to be web browsers. When these specialized clients need to access API services, they begin by requesting a token (which is similar to logging in in web browsers). To implement the token authentication mechanism, we can use the [Flask-HTTPAuth](https://flask-httpauth.readthedocs.io/) package:

```python
(venv)$ pip3 install flask-httpauth
```

There are different API authentication mechanisms that Flask-HTTPAuth supports. In this article, we shall use the [HTTP Basic Authentication (BA)](https://en.wikipedia.org/wiki/Basic_access_authentication) mechanism in which the client sends the user credentials in a standard [Authorization](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) HTTP header (you may want to check out [Digest Access Authentication](https://en.wikipedia.org/wiki/Digest_access_authentication) too). For your information, the HTTP BA transaction requires a client to provide a user's username and password when making a request. The syntax of such an HTTP request header is going to be:

```http
Authorization: <auth-scheme> <authorization-parameters>
```

Where `<auth-scheme>` will be `Basic` (the mechanism - see [more mechanisms](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication#authentication_schemes)) and `<authorization-parameters>` is the [base64](https://en.wikipedia.org/wiki/Base64) encoded key-value pair of username:password (where `username:password` will be something such as `YWxhZGRpbjpvcGVuc2VzYW1l`). The request header will therefore be:

```http
Authorization: Basic YWxhZGRpbjpvcGVuc2VzYW1l
```

In the event the request is rejected, say because of invalid user credentials or because the token has expired, the server will return a **HTTP 401 Unauthorized** response to the client with the header `WWW-Authenticate: Basic realm="Authentication Required"`.

>Warning: Base64-encoding can easily be reversed to obtain the original name and password (the credentials are encoded but not encrypted), so Basic authentication is completely insecure. HTTPS is always recommended when using authentication, but is even more so when using Basic authentication.

To integrate with Flask-HTTPAuth, the application will need to provide two functions: (1) Define the logic used to check the username and password provided by the user and (2) Return the error response in case of an authentication failure. The two functions will be registered by Flask-HTTPAuth through decorators and automatically called by the extension as needed during the authentication workflow.

```python
# app/api/auth.py: Basic authentication support

from flask_httpauth import HTTPBasicAuth
from app.models import User
from app.api.errors import error_response


basic_auth = HTTPBasicAuth()


@basic_auth.verify_password
def verify_password(username, password):
    user = User.query.filter_by(username=username).first()
    if user and user.check_password(password):
        return user


@basic_auth.error_handler
def basic_auth_error(status):
    return error_response(status)

```

It is the `HTTPBasicAuth()` class from Flask-HTTPAuth that implements the basic authentication workflow. The two required functions are configured through the  `verify_password` and `error_handler` decorators respectively. If the username and password provided are valid, the respective user is returned, otherwise, we get `None`. The authenticated user will then be available as `basic_auth.current_user()`.

A standard 401 error response is going to be generated should a user's credentials be invalid. This status code will be passed to the `basic_auth_error()` method letting the clients know that they need to resend the request with valid credentials.

Now, with authentication support implemented, we can generate tokens that clients will need:

```python
# app/api/tokens.py: Generate token

from app.api.auth import basic_auth
from app import db
from flask import jsonify
from app.api import bp


@bp.route('/tokens', methods=['POST'])
@basic_auth.login_required
def get_token():
    token = basic_auth.current_user().get_token()
    db.session.commit()
    return jsonify("token": token)

```

The `get_token()` view function is decorated with `@basic_auth.login_required` decorator from `HTTPBasicAuth` instance which instructs Flask-HTTPAuth to verify a user based on the supplied credentials. Notice that the authenticated user is referred to by `basic_auth.current_user()`. We attach the `get_token()` method from the `User` model to this user to produce a token. 

If we try to generate a token without providing the correct credentials, we will get a 401 Unauthorized error.

```python
(venv)$ http POST http://localhost:5000/api/tokens

# Output
HTTP/1.1 401 UNAUTHORIZED
Connection: close
Content-Length: 30
Content-Type: application/json
Date: Mon, 16 Oct 2023 05:05:58 GMT
Server: Werkzeug/2.3.6 Python/3.8.10
Vary: Cookie
WWW-Authenticate: Basic realm="Authentication Required"

{
    "error": "Unauthorized"
}
```

The error response has been handled by the `basic_auth_error()` function we defined earlier. Let us try to pass in the correct credentials (my user `muthoni` has the password `muthoni123`):

```python
(venv)$ http --auth muthoni:muthoni123 POST http://localhost:5000/api/tokens

# Output
HTTP/1.1 200 OK
Connection: close
Content-Length: 40
Content-Type: application/json
Date: Mon, 16 Oct 2023 05:14:17 GMT
Server: Werkzeug/2.3.6 Python/3.8.10
Vary: Cookie

{
    "token": "S4ut_MoQzn6a8iY5yoLGGg"
}
```

The status code has changed to 200 which is the status code for a successful request, and the payload includes the token of the user.


## Protecting API Endpoints With Tokens

With token generation in place, we can now verify these tokens in our endpoints. How can token verification happen? Flask-HTTPAuth can handle this for us using the `HTTPTokenAuth` class.

```python
from flask_httpauth import HTTPTokenAuth


token_auth = HTTPTokenAuth()


# ...

@token_auth.verify_token
def verify_token(token):
    return User.check_token(token) if token else None


@token_auth.error_handler
def token_auth_error(status):
    return error_response(status)

```

Flask-HTTPAuth uses the `verify_token` decorated function to verify a token. Besides that, everything else is similar to the basic authentication. Our token verification function checks for the user who owns the token provided and returns it. Should the returned value be `None`, the request will be rejected.

To protect endpoints with tokens, we can use the `@token_auth.login_required` decorator as follows:

```python
# app/api/routes.py: Protect endpoints with token authentication

from app.api.auth import token_auth


@bp.route('/users/<int:id>', methods=['GET'])
@token_auth.login_required
def get_user(id):
    # ...


@bp.route('/users/<int:id>/my-posts', methods=['GET'])
@token_auth.login_required
def get_posts_by_author(id):
    # ,,,


@bp.route('/users', methods=['GET'])
@token_auth.login_required
def get_users():
    # ...


@bp.route('/users', methods=['POST'])
def create_user():
    # ...


@bp.route('/users/<int:id>', methods=['PUT'])
@token_auth.login_required
def update_user(id):
    if token_auth.current_user().id != id:
        abort(403)
    # ...
```

Notice that the view function `create_user()` does not require user authentication because this endpoint can be accessed anonymously. Also, note how the PUT request, which modifies a user, has an additional check that prevents a user from modifying another user's account. If the `id` of the requested user does not match that of the authenticated user, we return a **403 Forbidden** response which indicates that the user does not have permission to carry out the operation.

If we try to send a request to any of these endpoints, we will get a 401 Unauthorized error response. To gain access, we need to add an `Authorization` header to the request, with a user's token received from `api.get_token()` view function. Flask-HTTPAuth expects the token to be sent as a "Bearer" token which is not directly supported by HTTPie.

>Initially, you saw the use of the basic authentication mechanism where the authentication scheme was "Basic":
>
>Authorization: Basic < credentials >
>
> For basic authentication, a user's credentials are passed over a network as clear text (it is base64 encoded, but base64 is a reversible encoding), meaning it is not secure. Basic authentication is typically used in conjunction with HTTPS/TLS.
>
>With token authentication, the authentication mechanism expects "Bearer" where the syntax of the new request header becomes:
>
> Authorization: Bearer < token >
>
>Bearer authentication (also known as token authentication) has security tokens called "bearer" tokens. They are cryptic strings usually generated by the server in response to a login request. They are appropriately called "bearer" because they belong to the bearer of the token. It gives access to the bearer of the token.

For basic authentication using username and password, HTTPie provides the `--auth` option, but for tokens, the header needs to be explicitly provided as follows:

```python
(venv)$ http GET http://localhost:5000/api/users/2 "Authorization: Bearer S4ut_MoQzn6a8iY5yoLGGg"

# Output

HTTP/1.1 200 OK
Connection: close
Content-Length: 309
Content-Type: application/json
Date: Mon, 16 Oct 2023 06:26:24 GMT
Server: Werkzeug/2.3.6 Python/3.8.10
Vary: Cookie

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

```

I have logged in using `muthoni`'s `token` and I can see another user of `id=2`.


## Revoking Tokens

Clients can send a `DELETE` request to the server through the `api.revoke_token()` view function to invalidate their tokens.

```python
# app/api/tokens.py: Revoke tokens

from app.api.auth import token_auth


@bp.route('/tokens', methods=['DELETE'])
@token_auth.login_required
def revoke_token():
    token_auth.current_user().revoke_token()
    db.session.commit()
    return '', 204

```

The authentication for this route is token-based, and in fact, it is meant to delete the token sent in the `Authorization` header. Revocation simply resets the expiration date on the token. Once the date has been reset, we commit our changes so that it is written in the database. The resulting response does not have a body, so we can return an empty string and set its status code to **204 No Content**, which is the status code for a response without a body.

```python
(venv) http DELETE http:localhost:5000/api/tokens "Authorization: Bearer S4ut_MoQzn6a8iY5yoLGGg"

# Output
HTTP/1.1 204 NO CONTENT
Connection: close
Content-Type: text/html; charset=utf-8
Date: Mon, 16 Oct 2023 06:51:50 GMT
Server: Werkzeug/2.3.6 Python/3.8.10
Vary: Cookie



```
