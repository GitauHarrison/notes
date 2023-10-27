# Friendly API Error Messages

While creating [unique resource identifiers](/api_flask/04_unique_resource_identifiers.md#retrieve-a-user-and-a-post) for users and posts, we noticed that if we try to query the backend for a user who does not exist in the database, say the `id` of the last user is `3` and we query for `id=4`, the error response was a 404 Not Found error in HTML format. The nice thing about APIs is that such errors can be overridden with JSON versions in the API blueprint. However, some errors handled by Flask still go through the error handlers that are globally registered for the application, and they continue to be HTML.

Browse the completed code on [GitHub](https://github.com/GitauHarrison/api_in_flask/tree/v8.0.0-api-friendly-error-response).

For your reference, these are the topics in our discussion:

- [Overview](00_overview.md)
- [API Blueprint](01_api_blueprint.md)
- [Resource Representation](02_resource_representation.md)
- [Error Handling](03_error_handling.md)
- [Unique Resource Identifiers](04_unique_resource_identifiers.md)
- [API Authentication](05_api_authentication.md)
- [API-friendly Error Messages](06_api_friendly_error_messages.md) (this article)
- [API Testing Using Postman](07_api_testing_postman.md)
- [API Testing And Documentation Using API Fairy](08_api_testing_documentation_using_api_fairy.md)


The REST architecture guides that a client and server need to agree on the format that information will be transacted, which in the context of modern APIs is JSON. There is also the option to provide more formats through _content negotiations_ where the parties involved mention the formats they accept through a request's header.

The client needs to send an `Accept` header in the request, indicating the format they would prefer. The server then looks at the list and responds using the best format it supports from the list offered by the client.

What we are going to do now is modify the global application error handlers so that they use content negotiation to reply in HTML or JSON according to a client's preference. We can achieve this using `request.accept_mimetypes` object from Flask.

```python
# app/errors/handlers.py: Content negotiation for error response

from flask import request, render_template
from app import db
from app.errors import bp
from app.api.errors import error_response as api_error_response


def want_json_response():
    return request.accept_mimetypes['application/json'] >= request.accept_mimetypes['text/html']


@bp.app_errorhandler(404)
def not_found_error(error):
    if wants_json_response():
        return api_error_response(404)
    return render_template('errors/404.html', title='Page Not Found'), 404


@bp.app_errorhandler(500)
def internal_error(error):
    db.session.rollback()
    if want_json_response():
        return api_error_response(404)
    return render_template('errors/500.html', title='Unexpected Error'), 500

```

The `want_json_response()` helper function makes a comparison for the preference of JSON or HTML selected by a client in their list of preferred formats. If JSON rates higher than HTML, then a JSON response is returned. Otherwise, we return the original HTML responses based on templates. We import `error_response` from the API blueprint and rename them to something far clearer in this context, which is `api_error_response()`. Now, if we try to query a backend for a non-existent user or post, we will get an appropriate error response format.

```python
(venv)$ http GET http:localhost:5000/api/users/4 "Authorization: Bearer S4ut_MoQzn6a8iY5yoLGGg"

# Output

HTTP/1.1 401 UNAUTHORIZED
Connection: close
Content-Length: 30
Content-Type: application/json
Date: Wed, 18 Oct 2023 13:59:01 GMT
Server: Werkzeug/2.3.6 Python/3.8.10
Vary: Cookie
WWW-Authenticate: Bearer realm="Authentication Required"

{
    "error": "Unauthorized"
}
```