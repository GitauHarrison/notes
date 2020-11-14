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

