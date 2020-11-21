The completed project used in this article can be referenced [here](https://github.com/GitauHarrison/personal-blog-tutorial-project/commit/3cc77ab6b79c45ad0bb61b1e1abb20990e9aacbe).

In this chapter, I will show you how you can create a profile for every user who decides to leave a comment or a question in our blog. We will keep it very simple. Whenever a user posts a commment, we want the post to look something like this:

![User Comment](/images/user_comment.png)

Things you can see from this comment:
* Avatar
* Time a comment was posted
* The comment itself

Our blog is set to English, as a default language. But say a visitor of our site decides to post a comment in a non-English language, like Swahili, our blog should allow this comment, and additionally, provide live language translation. 

![Non English Comment](/images/non-English_Comment.png)

With a non-English comment, a translate link is provided. Another visitor can click on the translate link to know what that comment means.

Now we have:
* Language translation of non-English comments

In sections below, we will look at three things:
1. How to set up  a user's profile
2. Dates and Time
3. Live language translation

### User Profile
You certainly will agree that the addition of an avatar makes the user comment look more interesting. I have used the [Gravatar](https://en.gravatar.com/) service to help generate the avatars. 

It is very simple to use Gravatar images. To create an image for a user, we will use their email address. The format is as below:
'_https://www.gravatar.com/avatar/<_hash_>_' where <_hash_> is the md5 hash of the user email.

```python
>>> from hashlib import md5
>>> 'https://www.gravatar.com/avatar/' + md5(b'harry@email.com').hexdigest()
# Output
'https://www.gravatar.com/avatar/3f4360b2a748228ba4f745a3ebd428dc'
```

Avatars are associated with a user. We will update our _User_ model to include their avatars.

app/models.py: User avatar
```python
from hashlib import md5

class User(db.Model):
# ...
def avatar(self, size):
    digest = md5(self.email.lower().encode('utf-8')).hexdigest()
    return 'https://www.gravatar.com/avatar/{}?d=identicon&s={}'.format(digest, size)
```
The method `avatar` returns an identicon image that can be set to a any size. Gravatar has several image options you can pick from. To note a few, there is 404, monsterid, robohash among others. You can learn more on how to implement images [here](https://en.gravatar.com/site/implement/images/).

To generate an md5 hash, we first convert the email to lower case using the `.lower()` function since Gravatar requires this. Then we enccode md5 hash in bytes because md5 support in Python works in bytes and not strings. 

The next step would be to insert and display the avatar in our templates. 

app/templates/_user_comments.html: Comment format
```python
<table>
    <tr valign="top">
        <td>
            <img src="{{ post.author.avatar(36) }}">
        </td>
        <td>
            {{ post.author.username }} said <br>
            {{ post.body }}
        </td>
    </tr>
</table>
<hr>
```
I have opted to use a `table` to display user comment. Learn how to create tables [here](https://www.w3schools.com/html/html_tables.asp). 

I have used `_user_comments.html` as a sub-template because, like before, I intend to include it in another template. For this reason, let us include it in our `flask_webforms.html` file.

app/templates/flask_webforms.html
```html
{% extends 'base.html' %}

{% block content %}
<!--Previous code-->

    <!-- Displaying User comments -->
    {% for post in posts %}
        {% include '_user_comments.html' %}
    {% endfor %}
        <!-- End of User Comments -->

<!--Article Form-->
 {% include 'comments.html' %}

{% endblock %}
```
I am using the `for` loop to go over all the available data in our database. `posts` is a variable that will store the results of the queries we will be making to our  database. The implementation of this will be done in our `flask_webforms` view function as seen in a subsequent section below.


If you recall from [chapter 4](4_working_with_database.md), we created a relationship between the _User_ model and the _Comment_ model. Every time a comment is posted, it will be referrenced to the user who is stored in our database. We use `post`, which stores the database query results, to get a user's `username` `avatar` and `body` of the comment from the database. 

So now, if you visit the URL mapped to the `flask_webforms` view function and post several comments, you will see formated user comments as seen below:

![User Comments Displayed](/images/user_comments_displayed.png)


