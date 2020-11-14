The completed project used in this article can be referenced [here](https://github.com/GitauHarrison/personal-blog-tutorial-project/commit/3cc77ab6b79c45ad0bb61b1e1abb20990e9aacbe).

This chapter will focus on how we will go about creating a simple profile for our blog users. You have probably noticed that whenever a user posts a comment, his or her comment has an avatar and the user's username. These credentials are displayed alongside the actual user comment.

![User Profile](/images/user_profile.png)

### User Avatar

We will use [Gravatar](https://en.gravatar.com/) service to provide images for all our users. It is very simple to implement this service. To request an image for a given user, the URL format _https://www.gravatar.com/avatar/<hash>_, where `<hash>` is the md5 hash user email address.

```python
>>> from hashlib improt md5
>>> 'https://www.gravatar.com/avatar/' + md5(b'harry@email.com').hexdigest()

# Output
'https://www.gravatar.com/avatar/3f4360b2a748228ba4f745a3ebd428dc'
```
The returned image has a default size of 80X80 pixels. This size can, however, be changed by adding an `s` argument tothe URL's query string. This is what I mean:
_https://www.gravatar.com/avatar/3f4360b2a748228ba4f745a3ebd428dc?s=78_.

You can further customize the URL to include another type of image for your avatar. [Gravatar](https://en.gravatar.com/site/implement/images/) has these images:
* 404: do not load any image if none is associated with the email hash, instead return an HTTP 404 (File Not Found) response
* mp: (mystery-person) a simple, cartoon-style silhouetted outline of a person (does not vary by email hash)
* identicon: a geometric pattern based on an email hash
* monsterid: a generated 'monster' with different colors, faces, etc
* wavatar: generated faces with differing features and backgrounds
* retro: awesome generated, 8-bit arcade-style pixelated faces
* robohash: a generated robot with different colors, faces, etc
* blank: a transparent PNG image (border added to HTML below for demonstration purposes)

We can add an image of choice, say, the identicon, by passing the `d` argument to our query string. These are for users who do not have an avatar registered with the service.

Some sites can block the Gravatar service due to tracking concerns. So, if you do not see any image, maybe it is because of an extension that blocks the service.

Since avatars are associated with a user, we will add a logic to the _User_ model.

```python
from hashlib import md5

class User(UserMixin, db.Model):
    #...
    def avatar(self, size):
        digest = md5(self.email.lower().encode('utf-8')).hexdigest()
        return 'https://www.gravatar.com/avatar/{}?d=identicon&s={}'.format(digest, size)
```

To generate an md5 hash, we convert a user's email to lower case. md5 support in Python is in bytes and not strings, hence we encode the string as bytes before passing it on the hash function. Learn more from their [documentation](https://en.gravatar.com/site/implement/images).

### Display User Avatar

As earlier mentioned, the user avatar will be displayed alongside a comment. We will create a sub-template that will be used to display this information.

app/templates/_user_comments.html: Display User comments
```html
<table>
        <tr valign="top">
            <td><img src="{{ post.author.avatar(36) }}"></td>
            <td>{{ post.author.username }} says:<br>{{ post.body }}</td>
        </tr>
</table>
```
I have named this template using the `_` just to help us know what template is a sub-template. A sub-template will included in another template. In a previous chapter, we created a template called `flashed_messages.html`. Let us rename it to `_flashed_messages.html`. Make user to also rename its inclusion in `flask_webforms.html`.

We will include this sub-template in our articles. Our current article file is  `flask_webforms.html`. Let us add user comments:

app/templates/flask_webforms.html: Add user comments
```html
{% extends 'base.html' %}

{% block content %}
    <h1>Flask Webforms</h1>
    <p>
        Lorem Ipsum is simply dummy text of the printing and typesetting industry. 
        Lorem Ipsum has been the industry's standard dummy text ever since the 1500s,
    </p>
    <h3>Working with Flask Web Forms</h3>
    <p>
        Contrary to popular belief, Lorem Ipsum is not simply random text. 
        It has roots in a piece of classical Latin literature from 45 BC, 
        making it over 2000 years old. Richard McClintock, a Latin professor 
        at Hampden-Sydney College in Virginia, looked up one of the more obscure 
        Latin words, consectetur, from a Lorem Ipsum passage, and going through 
        the cites of the word in classical literature, discovered the undoubtable source.
    </p><hr>
    {% include '_flashed_messages.html' %}

    <!-- User Comments Come Before the Form -->
    {% for post in posts %}
            {% include '_comments.html' %}
        {% endfor %}
        <!-- End of User Comments -->

    {% include 'comments.html' %}
    
{% endblock %}
```

We have used the `for` loop to get all the available comments. 