# Working with Rich Text

If you have been to [Stack Overflow](https://stackoverflow.com/), and at one point asked or answered a question, you have probaly noticed that the form used allows for markdown editing, which has a nice preview at the bottom.

![SO md preview](images/so_md_preview.png)

You can achieve this same feature in your Flask web app. Below, you will learn how to integrate markdown preview on the client side and also how to handle rich text in the server.

## Create Your Form

Since we want to have the ability to edit a post or comment in markdown, we will need to create a Flask form. If you are not aware of how to do this, check out how you can [create one here](recaptcha.md#create-a-simple-web-form).

![Comment form](/images/comment_form.png)

## Rich Text Client Preview

> `Rich` is a Python library for writing rich text (with color and style) to the terminal, and for displaying advanced content such as tables, markdown, and syntax highlighted code.

With our form set up, there is no way that you can style your comment before it is posted. What we want to show users as they are typing their comments is something like this: 

![Rich Text Preview](images/rich_text_preview.png)

There is the `flask-pagedown` extension we can use to enable client side markdown preview. Let us go ahead and install it:

```python
$ pip3 install flask-pagedown
```

The `flask-pagedown` extension needs to be registered in our application instance:

`app/__init__.py`: Register pagedown extension
```python
from flask_pagedown import PageDown

app = Flask(__name__)
pagedown = PageDown(app)
```

The Editor is supported through two Javascript files. To include these files in your HTML document, you will need to call `pagedown.html_head()` from inside the `<head>` element of your page:

`app/templates/base.html`: Include pagedown in template
```html
{% block head %}
    {{ super() }}
    {{ pagedown.html_head() }}
{% endblock %}
```
The Javascript files are loaded from a CDN, the files do not need to be hosted by your application.

## Update Form with `PageDownField`

The extention exports a `PagDownField` which is very similar to and works exactly as `TextAreaField`:

`app/forms.py`: Create form
```python
from flask_wtf import FlaskForm
from flask_pagedown.fields import PageDownField #<---------------New
from wtforms import SubmitField

class CommentForm(FlaskForm):
    comment = PageDownField('Comment', validators=[DataRequired()]) #<----------Edited
    recaptcha = RecaptchaField('Captcha')
    submit = SubmitField('Post')
```
That's it! You should be able to have a client-side comment preview right below the Comment box.

![Markdown preview](images/markdown_preview.gif)

## Handling Rich Text in the Server

If you have been following the tutorial on how to add [Google reCaptcha](recaptcha.md) to a web form, you will notice that the application does not work with a database yet. For this tutorial, however, we need to update the application to begin working with one.

First, we need to install the `flask-sqlalchemy` extension:

```python
(venv)$ pip3 install flask-sqlalchemy
```

Then, we need to add the `db` variable to our application instance:


`app/__init__.py`: Add db variable
```python
# ...
from flask_sqlalchemy import SQLAlchemy

# ...
db = SQLAlchemy(app)
```

The application expects certain configuration variables to be set in the `config.py` file. Add the following to the `config` module:


`config.py`: Add db configuration
```python
# ...

basedir = os.path.abspath(os.path.dirname(__file__))


class Config(object):
    # ...

    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

## Create a Model


The model we would like to create will be a `Comment` model. The model will have the following fields:


`app/models.py`: Comment model
```python
from app import db


class Comment(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    body = db.Column(db.Text)
    timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)

    def __repr__(self):
        return f'Comment: {self.body}'
```

With the `Comment` model defined, we can now create the database by applying the changes we have just made. In your terminal, run the commands below:

```python
(venv)$ flask db init
(venv)$ flask db migrate -m 'comment table'
(venv)$ flask db upgrade
```
You will notice that a _migrations_ folder will be created as soon as you run `flask db init`. This folder will hold all the migration scripts that the application will be using. `flask db migrate` creates a new migration script, and `flask db upgrade` applies the changes made.

Note that for you to use the `flask` command, you need to have initialized the environment variable `FLASK_APP` to point to the `test.py` file.

When the form is submitted, only the raw Markdown text is sent with the POST request; the HTML preview that is shown on the page is discarded. Sending the generated HTML preview is a security risk as an attacker can easily construct HTML sequences which don't match the markdown source and submit them. To avoid any risks, only the Markdown source text is submitted, and once in the server it is converted again to HTML using Markdown, a Python Markdown-to-HTML converter.

There are two more extensions that we can use to help us achieve this. Go ahead and install them:

```python
$ pip3 install markdown bleach
```

_`bleach` allows us to sanitize the resulting HTML to ensure that only a few HTML tags are allowed._

Comment conversion can be done in the `_comments.html` subtemplate, but it is inefficient to do it here as all comments will have to be converted every time they are rendered to a page. We get rid of this repetition by making the coversion when the data is in our database.

The HTML code for the rendered blog post is cached in a new field added to the Comment model that the template can access directly. The original Markdown source is also kept in the database in case the post needs to be edited.

## Update the Comment Table

First, we need to add a `html` field to the table. We can do this by adding the following to the `Comment` model:

`app/models.py`: Add html field
```python
import bleach
from markdown import markdown


class Comment(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    body = db.Column(db.String(140))
    body_html = db.Column(db.String(140)) # <----------------------------------- new
    timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))

    # new function
    @staticmethod
    def on_changed_body(target, value, oldvalue, initiator):
        allowed_tags = ['a', 'abbr', 'acronym', 'b', 'blockquote', 'code',
        'em', 'i', 'li', 'ol', 'pre', 'strong', 'ul',
        'h1', 'h2', 'h3', 'p']
        target.body_html = bleach.linkify(bleach.clean(
        markdown(value, output_format='html'),
        tags=allowed_tags, strip=True))

    def __repr__(self):
        return '<Post {}>'.format(self.body)

db.event.listen(Comment.body, 'set', Comment.on_changed_body) # <------------------new
```

The `on_changed_body()` function is registered as a listener of SQLAlchemy’s `“set”` event for body , which means that it will be automatically invoked whenever the body field is set to a new value. The handler function renders the HTML version of the body and stores it in `body_html` , effectively making the conversion of the Markdown text to HTML fully automatic.

The actual conversion is done in 3 steps:
* `markdown()` function does an initial conversion to HTML. The result is passed to `clean()` function with a list of approved HTML tags
* `clean()` function removes any tags that are not in the whitelist.
* `linkify()` function from `bleach` converts any URLs written in plain text into proper `<a>` links. Automatic link generation is not officially in the Markdown specification, but is a very convenient feature. On the client side, PageDown supports this feature as an optional extension, so linkify() matches that functionality on the server.

After these changes, we need to apply them to our database:

```python
$ flask db migrate -m 'Add new field to Comment table'
$ flask db upgrade
```
The route handling this request needs to be updated to retrieve all the comments in the `Comment` model.

`app/routes.py`: Retrieve all comments
```python
# ... 
@app.route('/', methods=['GET', 'POST'])
def index():
    form = CommentForm()
    if form.validate_on_submit():
        comment = Comment(body=form.body.data)
        db.session.add(comment)
        db.session.commit()
        flash('Your comment has been published.')
        return redirect(url_for('index'))
    page = request.args.get('page', 1, type=int)
    posts = Comment.query.order_by(Comment.timestamp.desc()).paginate(
        page, app.config['POSTS_PER_PAGE'], False)
    next_url = url_for('index', page=posts.next_num) \
        if posts.has_next else None
    prev_url = url_for('index', page=posts.prev_num) \
        if posts.has_prev else None
    return render_template(
        'index.html',
        title='Home',
        posts=posts.items,
        next_url=next_url,
        prev_url=prev_url
    )
```

With our database updated, we will now replace `comment.body` with `comment.body_html` in the template when available:

`app/templates/index.html`: Use HTML version in post body
```html
<div class="row"> 
    <div class='col-sm-12'>
        {% for post in posts %}             
            {% if post.body_html %}
                {{ post.body_html | safe }}
            {% else %}
                {{ post.body }}     
            {% endif %}   
        {% endfor %}
    </div>
</div>
<!-- Pagination of comments -->
<nav aria-label="...">
    <ul class="pager">
        <li class="previous{% if not prev_url %} disabled{% endif %}">
            <a href="{{ prev_url or '#' }}">
                <span aria-hidden="true">&#60;</span> Newer posts
            </a>
        </li>
        <li class="next{% if not next_url %} disabled{% endif %}">
            <a href="{{ next_url or '#' }}">
                Older posts <span aria-hidden="true">&#62;</span>
            </a>
        </li>
    </ul>
</nav>
<!-- End of pagination of comments -->
```

The ` | safe` suffix when rendering the HTML body is there to tell Jinja2 not to escape the HTML elements. Jinja2 escapes all template variables by default as a security measure, but the Markdown-generated HTML was generated by the server, so it is safe to render directly as HTML.

Reload your page and try to post a new comment with markdown syntax:

![Enabled Markdown Styling](images/enabled_md_styling.png)

## Update Project Dependencies

Ensure you update your project requirements:
```python
$ pip3 freeze > requirements.txt
```