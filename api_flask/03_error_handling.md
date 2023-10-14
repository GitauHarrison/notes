# Handle API Errors In Flask

In the referenced [chat application](https://github.com/GitauHarrison/api_in_flask/blob/v2.0.0-blueprint/app/errors/handlers.py), you will notice that there is a mechanism to handle errors. Whenever the errors occur, the application will return error templates suitable for a web client. When an API returns an error, we need a "machine-friendly" type of error that the client (whatever it is) can interpret. 

Browse the completed code on [GitHub](https://github.com/GitauHarrison/api_in_flask/tree/v5.0.0.0-handle-api-errors).

For your reference, these are the topics in our discussion:

- [Overview](00_overview.md)
- [API Blueprint](01_api_blueprint.md)
- [Resource Representation](02_resource_representation.md)
- [Error Handling](03_error_handling.md) (this article)
- [Unique Resource Identifiers](04_unique_resource_identifiers.md)
- [API Authentication](05_api_authentication.md)
- [API-friendly Error Messages](06_api_friendly_error_messages.md)
- [API Testing Using Postman](07_api_testing_postman.md)


### Table Of Content

- [Error Representation](#error-representation)
- [Error Response](#error-response)
- [Bad Request Error](#bad-request-error)


## Error Representation

First, let us decide on how to represent an API error message.

```json
{
    "error": "short error description",
    "message": "error message (this is optional)"
}
```


## Error Response

Besides the error payload (what the error is), we can use status codes from the HTTP protocol to indicate the general class of the error.

```python
# app/api/errors.py: Generate error responses


from flask import jsonify
from werkzeug.http import HTTP_STATUS_CODES

def error_response(status_code, message=None):
    payload = {"error": HTTP_STATUS_CODE.get(status_code, "Unknown error")}
    if message:
        payload["message"] = message
    response = jsonify(payload)
    response.status_code = status_code
    return response

```

The `error_response()` method utilizes the HTTP_STATUS_CODE dictionary from Flask's Werkzeug to provide a short descriptive name for each HTTP status code. We have used these names for the `error` field in our error representation so now we can only worry about the status codes and the optional long description. `jsonify()` returns a Flask `Response` object with a default status code of 200, so after creating that response, we set the status code to the correct one for the error.


## Bad Request Error

The most commonly returned error in APIs is the **400 Bad Request** error. It indicates that the client sent a request with invalid data in it. Since it is so common, we can create a dedicated function that only requires a long descriptive message as an argument.

```python
# app/api/errors.py: Bad request response

def bad_request(message):
    return error_response(400, message)

```