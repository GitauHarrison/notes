# User Posts

For your reference, these are the sections included in this tutorial:

[Section 1: Create a simple web application](/flask_popover.md#create-a-simple-application)
[Section 2: Web Forms](web_forms.md)
[Sectin 3: Working with Databases](database.md)
[Section 4: User Login](user_login.md)
[Section 5: User Posts](user_posts.md)
[Section 6: Implement Popover](popover.md)

Let us create a `PostForm` in the index page to allow users to add a new post.

`app/forms.py`: Post form
```python
# ...
from wtforms import TextAreaField


class PostForm(FlaskForm):
    body = TextAreaField('Comment', validators=[DataRequired()])
    submit = SubmitField('Submit')
```

To display this form, we will update our `index.html` template.

`app/templates/index.html`: Post form
```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_context %}
<div class="row text-center">
    <div class="col-md-12">
        <h1>{{title}}</h1>
    </div>  
</div>
<div class="row">
    <div class="col-sm-12">
        {{ wtf.quick_form(form) }}
    </div>
</div>
{% endblock %}
```

The `index()` view function, render this form, will have the logic to show all the posts made in the application.

`app/routes.py`: Render post form
```python
# ... 
from app.forms import PostForm


@app.route('/', methods=['GET', 'POST'])
@app.route('/index', methods=['GET', 'POST'])
@login_required
def index():
    form = PostForm()
    if form.validate_on_submit():
        post = Post(body=form.body.data, author=current_user)
        db.session.add(post)
        db.session.commit()
        flash('Your comment has been published.')
        return redirect(url_for('index'))
    page = request.args.get('page', 1, type=int)
    posts = Post.query.order_by(Post.timestamp.desc()).paginate(
        page, app.config['POSTS_PER_PAGE'], False)
    next_url = url_for('index', page=posts.next_num) \
        if posts.has_next else None
    prev_url = url_for('index', page=posts.prev_num) \
        if posts.has_prev else None
    return render_template(
        'index.html',
        title='Posts',
        form=form,
        next_url=next_url,
        prev_url=prev_url,
        posts=posts.items)
```

Notice that we are using a new evironment variable `POSTS_PER_PAGE` to define the number of posts to be displayed per page. Since this configuration is lacking at the moment, we need to update the `config.py` file to include it.

`config.py`: Posts per page configuration
```python
# ...


class Config(object):
    # ...

    POSTS_PER_PAGE = int(os.environ.get('POSTS_PER_PAGE') or 10)
```

Let us provide a value for the `POST_PER_PAGE` variable:

`.env` file: Posts per page value
```
POSTS_PER_PAGE=3
```

You should be able to see the form displayed.

![Post form](/images/flask_popover/post_form.png)


## Display Posts

We need to define a template that can be reused throughtout the application to display all users' posts. Let us create a new template to display all posts.

```python
(venv) $ cd templates
(venv) $ touch _posts.html
```

This template is deliberately named with an underscore simply to make it clear that it is not meant to be used directly, rather it will be included in other templates. Let us update it with a table that defines the structure of a post.

`app/templates/_posts.html`: Posts table
```html
<table class="table table-hover">
    <tr valign='top'>
        <td width='70px'>
            <img src=" {{ post.author.avatar(36) }} ">
        </td>
        <td>
            <span class="user_popup">
                <a href="{{ url_for('user', username=post.author.username) }}">
                    {{ post.author.username }}
                </a>
            </span>
            said
            <span>
            {{ moment(post.timestamp).fromNow() }}:
            </span><br>
            <span>
                {{ post.body }}                    
            </span>
        </td>
    </tr>
</table>
```

I have used a table to display the posts. The table is defined with a `<table>` tag, and the `<tr>` tag defines a row in the table. The first column is the user's avatar, and the second column is the user's post (username as a link, timestamp and body).

Within the `index()` view function, we have already defined a `posts` variable that contains all the posts made in the application. We can use this variable to display all the posts made in the application. We can loop through this variable to retrieve all the posts and display them in a table.

`app/template/index.html`: Display all posts
```html
<!-- Previous code -->

<!-- Display all posts -->
<div class="row">
    <div class="col-sm-12">
        {% for post in posts %}
            {% include '_post.html' %}
        {% endfor %}
    </div>
<!-- End of displaying all posts -->
```

## Pagination

For testing purposes, we can set the `POSTS_PER_PAGE` to 3 to quickly see how pagination will work. Pagination allows us to display a limited number of posts per page. If the number of posts is more than 3 in our test example, then post 4, 5, and 6 will be displayed in page 2 of `/index`.

Within the `index()` view function you have seen me use the `paginate()` method to effect pagination. This method takes three arguments:

* `page`: The page number to be displayed.
* `per_page`(set as an environment variable): The number of posts to be displayed per page.
* `error_out`: If `True`, then the `paginate()` method will return `None` if the page number is out of range. If `False`, then the `paginate()` method will return an empty list if the page number is out of range.

We need to display buttons to navigate through the pages of posts. 

`app/templates/index.html`: Pagination buttons
```html
<!-- Previous code -->

<!-- Pagination Links -->
<nav aria-label="...">
    <ul class="pager">
        <li class="previous{% if not prev_url %} disabled{% endif %}">
            <a href="{{ prev_url or '#' }}">
                <span aria-hidden="true">&larr;</span> Newer Comments
            </a>
        </li>
        <li class="next{% if not next_url %} disabled{% endif %}">
            <a href="{{ next_url or '#' }}">
                Older Comments <span aria-hidden="true">&rarr;</span>
            </a>
        </li>
    </ul>
</nav>
<!-- End of post pagination -->
```
Hopefully, you can see the navigation buttons displayed.

![Pagination buttons](/images/flask_popover/pagination_buttons.png)

### User's Own Posts

All that needs to be done is to link the posts made to its author. We want a user to see only his own posts when he is in his profile page.

`app/routes.py`: User posts in his profile
```python
# ...


@app.route('/user/<username>')
@login_required
def user(username):
    user = User.query.filter_by(username=username).first_or_404()
    page = request.args.get('page', 1, type=int)
    posts = user.posts.order_by(Post.timestamp.desc()).paginate(
        page, app.config['POSTS_PER_PAGE'], False)
    next_url = url_for('user', username=user.username, page=posts.next_num) \
        if posts.has_next else None
    prev_url = url_for('index', page=posts.prev_num) \
        if posts.has_prev else None
    return render_template(
        'user.html',
        title='User Profile',
        username=username,
        user=user,
        next_url=next_url,
        prev_url=prev_url,
        posts=posts.items
    )
```

A user's posts are retrieved from the database by quering the `User` model using the `posts` relationship. We display them based on when they were created. 

Let us update the `user.html` template to display these posts with pagination included.

`app/templates/user.html`: User's own posts
```html
{% extends 'base.html' %}

{% block app_context %}
<!-- Previous code -->

<!-- Display all posts -->
<div class="row">
    <div class="col-sm-12">
        {% for post in posts %}
            {% include '_post.html' %}
        {% endfor %}
    </div>
<!-- End of displaying all posts -->

<!-- Pagination Links -->
<nav aria-label="...">
    <ul class="pager">
        <li class="previous{% if not prev_url %} disabled{% endif %}">
            <a href="{{ prev_url or '#' }}">
                <span aria-hidden="true">&larr;</span> Newer Comments
            </a>
        </li>
        <li class="next{% if not next_url %} disabled{% endif %}">
            <a href="{{ next_url or '#' }}">
                Older Comments <span aria-hidden="true">&rarr;</span>
            </a>
        </li>
    </ul>
</nav>
<!-- End of post pagination -->
{% endblock %}
```

## Testing Pagination

1. Create two browser windows
2. Create a new user from each browser window
3. Log them in
4. Add a post from each user in their separate browser windows

![Users Posts](/images/flask_popover/users_posts.png)

A user needs to reload his page to see the new posts. The user will be able to see another user's profile by clicking their username.