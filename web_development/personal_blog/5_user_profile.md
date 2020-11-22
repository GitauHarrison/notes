The completed project used in this article can be referenced [here](https://github.com/GitauHarrison/personal-blog-tutorial-project/commit/0f32d10287e51e85ad519c1a4cff8b84dd0b6cda).

In this chapter, I will show you how you can create a profile for every user who decides to leave a comment or a question in our blog. We will keep it very simple. Whenever a user posts a commment, we want the post to look something like this:

![User Comment](/images/user_comment.png)

Things you can see from this comment:
* Avatar
* Time a comment was posted
* The comment itself

Our blog is set to English, as a default language. But say a visitor of our site decides to post a comment in a non-English language, like Swahili, our blog should allow this comment, and additionally, provide live language translation. 

![Non English Comment](/images/non-English_Comment.png)

With a non-English comment, a translate link is provided. Another visitor can click on the translate link to know what that comment means. I want the blog to be able to support language translation of non-English comments

In sections below, we will look at three things:
1. How to set up  a user's profile
2. Dates and Time
3. Live language translation

### User Profile
You certainly will agree that the addition of an avatar makes the user comment look more interesting. I have used the [Gravatar](https://en.gravatar.com/) service to help generate the avatars. 

It is very simple to use Gravatar images. To create an image for a user, we will use their email address. The format is '_https://www.gravatar.com/avatar/<_hash_>_' where <_hash_> is the md5 hash of the user email.

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
The method `avatar` returns an identicon image that can be set to a any size. Gravatar has several image options you can pick from. To note a few, there is 404, monsterid, robohash among others. You can learn more on how to implement gravatar images [here](https://en.gravatar.com/site/implement/images/).

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
I have opted to use a `table` to display user comment. If you do not know how to work with tables, learn more [here](https://www.w3schools.com/html/html_tables.asp). 

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
The `for` loop goes over all the available data in our database. `posts` is a variable that will store the results of the queries we will be making to our  database. The implementation of this will be done in our `flask_webforms` view function as seen in a subsequent section below.


If you recall from [chapter 4](4_working_with_database.md), we created a relationship between the _User_ model and the _Comment_ model. Every time a comment is posted, it will be referrenced to the user who is stored in our database. With each loop, the variable `post` stores new data. We use this `post` variable to get a user's `username` `avatar` and `body` of the comment from the database. 

So now, if you visit the URL mapped to the `flask_webforms` view function and post several comments, you will see formated user comments as seen below:

![User Comments Displayed](/images/user_comments_displayed.png)

### Dates and Time
Our _Comment_ model has a `timestamp` field. Above, we have seen how we can display a user's `username` and comment `body`. What we need to do now is display the time a comment was made in a user post. 

##### Understanding Dates and Time in Python

Python has a [datetime](https://docs.python.org/3/library/datetime.html#module-datetime) module which supplies classes for manipulating dates and times. We will work with a few examples to understand how `datetime` works. Run your Python interpreter by typing `python3` in your terminal.

```python
>>> import datetime
>>> datetime_object = datetime.datetime.now()
>>> print(datetime_object)
#Output
2020-11-22 05:45:58.883645
```
We have imported the datetime module. We then use one of the classes in `datetime` module called `datetime` and add `now()` method to create a datetime object containing the current local date and time.

If we want to get only the date, then we will do:

```python
# do not close the interpreter, if you do, you will need to import datetime again
# Below, I am not importing datetime because my interpreter session is still active
>>> datetime_object = datetime.date.today()
>>> print(datetime_object)
#Output
2020-11-22
```
We have used `today()` method defined in the `date` class to get a `date` object containing the current local date.

```python
>>> d = datetime.date(2020, 11, 22)
>>> print (d)
# Output
2020-11-22
```
`date()` is a constructor of the `date` class. The constructor takes three arguments: year, month, day.

To be more specific in retrieving either the year, the month or the day, we will do:

```python
>>> from datetime import date

>>> today = date.today()
>>> print('Current year: ', today.year)
# Output
Current year:  2020
>>> print('Current month: ', today.month)
# Output
Current month:  11
>>> print('Current day: ', today.day)
# Output
Current day:  22
```

Let us now see how we can get the time object.

```python
>>> from datetime import time

>>> t = time()
>>> print('t = ', t)
# Output
>>> t =  00:00:00

q = time(6, 7, 30)
print('q = ', q)
# Output
q =  06:07:30
```

You get the idea. Using the `time` class, you can create a time object.

```python
>>> from datetime import datetime,date
>>> t1 = date(year = 2020, month = 11, day = 22)
>>> t2 = date(year = 2019, month = 10, day = 13)
>>> t3 = t1 - t2
>>> print('t3 = ', t3)
# Output
t3 =  406 days, 0:00:00
```

It is also possible to get the difference between two dates and times as seen above.

```python
>>> from datetime import timedelta
>>> t1 = timedelta(weeks = 4, days = 5, hours = 10, seconds = 10)
>>> t2 = timedelta(days = 7, hours = 2, minutes = 21, seconds = 34)
>>> t3 = t1 - t2
>>> print('t3 = ', t3)
# Output
t3 =  26 days, 7:38:36
```
We have created two `timedelta` objects and printed their difference. For more authoritative information on the Python `datetime` module, check the [python documentation](https://docs.python.org/3/library/datetime.html).

At this point, you are fairly familiar with how to use time and date in Python. But there is one more thing I would like to discuss: time zones. We know that their is an obvious possibility that the visitors of our blog may be living in different time zones. We want out blog to be able to generate consistent timestamps despite of the fact that users of our blog live in different time zones. 

To understand the effect of timezone, I will show you how we are affected by it below:

```python
>>> from datetime import datetime
>>> str(datetime.now())
# Output
'2020-11-22 06:25:15.366203'
```

If another user who is an a different time zone from mine runs the codes above, the output will be different. This is due to the concept to time zones. So how do we ensure consistency in our time?

```python
>>> str(datetime.utcnow())
# Output
'2020-11-22 03:27:03.055821'
```

Regardless of timezones, any user who runs the code above will get the same output. `datetime.utcnow()` will always return the same time, regardless of location. UTC is the most used uniform timezone and is supported in the `datetime` class. You saw me implement this in [chapter 4](4_working_with_database.md) while creating fields for the _Comment_ model. This was to ensure consistency in our time as used in the blog. 

While standardizing timestamps to UTC makes a lot of sense from the server's point of view, this creates a usability problem for users. I am in Nairobi and the current time here is `06:25:15`. However, UTC returns `03:27:03`. This means that I have to do the calculation of my actual time. Every visitor of our blog will have to do the same. Below, i will address this challenge while still maintaining a standardized time in our server.

