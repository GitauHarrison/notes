# Resource Representations

The first aspect we need to decide on when implementing an API is the representation of its resources. In this chapter, the API we are building is going to work with users and posts. So, we will need to decide what the representation of user resources and post resources are going to be. 

Browse the completed code on [GitHub](https://github.com/GitauHarrison/api_in_flask/tree/v4.0.0-resource-representations).

For your reference, these are the topics in our discussion:

- [Overview](00_overview.md)
- [API Blueprint](01_api_blueprint.md)
- [Resource Representation](02_resource_representation.md) (this article)
- [Error Handling](03_error_handling.md)
- [Unique Resource Identifiers](04_unique_resource_identifiers.md)
- [API Authentication](05_api_authentication.md)
- [API-friendly Error Messages](06_api_friendly_error_messages.md)
- [API Testing Using Postman](07_api_testing_postman.md)

### Table Of Content

- [Overview](#overview)
- [Serialization and Deserialization Setup Of The User Object](#serialization-and-deserialization-setup-of-the-user-object)
- [Representing Collections Of Users](#representing-collections-of-users)
    - [Understanding Python Mixin](#understanding-python-mixin)
    - [PaginatedAPIMixin Class](#paginatedapimixin-class)
- [Serialization and Deserialization Setup Of The Post Object](#serialization-and-deserialization-setup-of-the-post-object)
- [Representing Collections Of Posts](#representing-collections-of-posts)


## Overview

Let us look at this sample JSON representation of user data:

```json
{
    "id": 1,
    "username": "harry",
    "email": "harry@email.com",
    "last_seen": "2023-10-13T13:31:27Z",
    "about_me": "Hey, I am enjoying APIs",
    "post_count": 3,
    "_links": {
        "self": "/api/users/1",
        "avatar": "https://www.gravatar.com/avatar/..."
    }
}
```

If you look closely at the `User`model of our sample chat application, you will notice that most fields are directly coming from the table's columns. The missing part is the `password` field which will only be used when registering a new user. Also, remember that passwords are not stored in their raw form, but rather as a hash, and therefore it is not returned.

The `email` field is special. This is because we do not want to expose our users' email addresses. It will only be returned, in our case, when a user requests for their entry but not when they retrieve entries from other users.

The field `post_count` is a virtual field, in that it does not exist in the `User` table. It is going to be provided to a client as a convenience. This virtual aspect of resource representation allows us the freedom to not necessarily match how the actual resource is defined in the server.

The `_links` section represents the hypermedia principle of REST. If you recall, this principle states that a client can discover new resources by traversing the relationships (more like clickable links in HTML pages). We have defined the current resource (the returned user whose `id` in the URL matches that of the resource) and their avatar. Later, we can link a user to their posts and, therefore, add another field in the `_links` section to list all the posts by this user of `id=1`.


## Serialization and Deserialization Setup Of The User Object


JSON is text or string. It is especially ideal when working with a string or strings that you want to convert to an object. Such an object will be written as a string using the JavaScript Object Notation. The process of converting an object to a string is called **serialization** whereas the inverse (string -> object) is called **deserialization**.

The nice thing about the JSON format is that it can easily be converted to a Python dictionary or list. Python provides the standard library `json` to take care of serialization and deserialization. The first thing that we are going to do is to retrieve a user object as a Python dictionary.

```python
# app/models.py: Return a user object as s Python dictionary

from flask import url_for

class User(UserMixin, db.Model):
    # ..

    def to_dict(self, include_email=False):
        data = {
            'id': self.id,
            'username': self.username,
            'last_seen': self.last_seen.isoformat() + 'Z',
            'about_me': self.about_me,
            'post_count': self.posts.count(),
            '_links': {
                'self': url_for('api.get_user', id=self.id),
                'avatar': self.avatar(128)
            }
        }
        if include_email:
            data['email'] = self.email
        return data
```

The `include_email` flag has been used to determine if that field should be included in the representation or not. It will be shown to a user who requests their data.

For the date and time fields, we have used the [ISO 8601]() format which Python's `datetime` can generate using the `isoformat()` method. But because we are using native `datetime` objects that are UTC and do not have timezone recorded in their state, we have added the `Z` at the end, which is ISO 8601's timezone code for UTC.

The `to_dict()` method converts a user object to a Python dictionary, which will then be converted to JSON. We can also parse through a client's representation in a request and convert it to a `User` object.

```python
# app/models.py: Convert client representation to a user object

class User(UserMixin, db.Model):
    # ...

    def from_dict(self, data, new_user=False):
        for field in ['username', 'email', 'about_me']:
            if field in data:
                setattr(self, field, data[field])
            if new_user and 'password' in data:
                self.set_password(data['password'])

```

We begin by looping through any of the fields that a client may set, which are `username`, `email` and `about_me`. We then check if there is a value provided for each field in the `data` argument. If there is, then we set the new value in the corresponding attribute using `setattr()`.

There is no `password` field in the object, hence it is treated specially. The `new_user` argument determines if this is a new user registration at which point a `password` is included. The `set_password()` method has been used to convert a user's password into a hash before it is stored in the database.


## Representing Collections Of Users

The representation shown above returns a single resource. What about when we want to see a list of users and everything associated with them? Let us look at the sample representation of a collection of users:

```json
{
    "items": [
        { ... },
        { ... }
    ],
    "_meta": {
        "page": 1,
        "per_page": 10,
        "total_pages": 20,
        "total_items": 30
    },
    "_links": {
        "self": "http://localhost:5000/api/users?page=1",
        "next": "http://localhost:5000/api/users?page=2",
        "prev": null
    }
}
```

Above, `items` is a list of user resources, each defined in the previous section. The `_meta` section includes metadata for the collection that the client might find useful in presenting pagination controls to the user. The `_links` section defines relevant links including a link to the collection itself, and the previous and next page links, also to help the client paginate the listing.


### Understanding Python Mixin

A mixin class provides a way to extend the functionality of classes without the need for traditional inheritance. It offers a specific set of behaviors or functionality that can be easily incorporated into other classes. It focuses on providing additional features that can be combined with multiple classes. Let us see the example below:

```python
# mixin.py

class EatMixin:
    def favourite_meal(self):
        print('I like pork and rice.')


class Human:
    print('These are my favourite meals: \n')


class Child(Human, EatMixin):
    def school(self, school, meal):
        print('At ', school, 'we are served ', meal)


class Parent(Human, EatMixin):
    def work(self, company, meal):
        print('At ', company, 'we eat ', meal)


child = Child()
child.school('Nairobi Homeschool', 'Fries and Soda')

worker = Parent()
worker.work('United Nations, Nairobi', 'beans and potatoes')

# -----
# Output
# -----

These are my favorite meals: 

At  Nairobi Homeschool we are served  Fries and Soda
At  United Nations, Nairobi we eat  beans and potatoes
```

We have defined a mixin class called `EatMixin` that provides the `favourite_meal()` method. The base `Human` class, together with the `EatMixin` class, are inherited by `Child` and `Parent` classes, giving them the ability to tell us what their favorite meals are at school and work. The mixin class here serves to provide additional behavior in other classes.


### PaginatedAPIMixin Class

In the context of our API, we can define a custom and very generic mixin class called `PaginatedAPIMixin` to help extend the behavior of the `User` class (and later the `Post` class) to include pagination controls.


```python
# app/models.py: Paginated representation of the mixin class

class PaginatedAPIMixin(object):
    @staticmethod
    def to_collection_dict(query, page, per_page, endpoint, **kwargs):
        resources = query.paginate(page=page, per_page=per_page, error_out=False)
        data = {
            "items": [item.to_dict() for item in resources.items],
            "_meta": {
                "page": page,
                "per_page": per_page,
                "total_pages": resources.pages,
                "total_items": resources.total
            },
            "_links": {
                "self": url_for(endpoint, page=page, per_page=per_page, **kwargs),
                "next": url_for(endpoint, page=page + 1, per_page=per_page, **kwargs) if resources.has_next else None,
                "prev": url_for(endpoint, page=page - 1, per_page=per_page, **kwargs) if resources.has_prev else None
            }
        }
        return data

```

The implementation above uses the `paginate()` method of the query object to obtain a page worth of items. To keep them generic, the links make use of `endpoint` rather than something such as `url_for('api.get_users', id=id)` to ensure that they are dependent on the particular collection of resources. And now that many routes have arguments, we capture those by providing the additional keyword argument `kwargs`, and pass them to the URL.

Finally, we need to add the mixin class to the `User` model:

```python
# app/models.py: Add mixin class

class User(PaginatedAPIMixin, UserMixin, db.Model):
    # ...
```

We are not going to have any routes that require a client to return a list of users, therefore, there is no need to have a reverse direction for collections. Should you need one, then you can try to create your implementation.


## Serialization and Deserialization Setup Of The Post Object

Just like we did with the `User` object, we also need to extend the functionality that helps us to work with the Python representation of the `Post` object. If you recall, we began by converting a `User` object to a Python dictionary, which would later be converted to JSON. We also considered the reverse direction where we receive a user request and need to convert that into a `User` object. We need to do the same for the `Post` object too.

```python
# app/models.py: Python representation of the Post object

class Post(db.Model):
    # ...

    # From object to Python representation
    def to_dict(self):
        data = {
            "id": self.id,
            "title": self.title,
            "body": self.body,
            "timestamp": self.timestamp.isoformat() + 'Z',
            "author": {
                "id": self.author.id,
                "username": self.author.username,
                "about_author": self.author.about_me
            },
            "_links": {
                "to_this_post": url_for('api.get_post', id=self.id),
                "to_post_author": url_for('api.get_user', username=self.author.username)
            }
        }
        return data
    

    # From Python representation to object
    def from_dict(self, data):
        for field in ["title", "body", "author_id"]:
            if field in data:
                setattr(self, field, data[field])

```


## Representing Collections Of Posts

The `PaginatedAPIMIxin` is so generic that it allows us to implement the same class on the `Post` model. To get a paginated response, we will need to return a list of post resources and the relevant pagination controls such as the current page of posts, the number of posts on a page, the total number of post pages, the total number of posts and the links to the next or previous pages. Ensure you add the `PaginatedAPIMIxin` to the `Post` model as follows:

```python
# app/models.py: Representing a paginated response of a collection of post objects

class Post(PaginatedAPIMIxin, db.Model):
    # ...

```
