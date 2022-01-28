# Add User Notifications To Your Flask App

For your reference, these are the sections included in this tutorial:

1. [Section 1: Create a simple web application](/flask_popover.md#create-a-simple-application)
2. [Section 2: Web Forms](web_forms.md)
3. [Sectin 3: Working with Databases](database.md)
4. [Section 4: User Login](user_login.md)
5. [Section 5: User Posts](user_posts.md)
6. [Section 6: Implement Popover](popover.md)
7. [Section 7: User Notifications](user_notifications.md)

It is faily common in web applications to notify users of important events. In the case of social applications, web applications often notify users of new posts, new comments or private messages, usually showing a little badge with a number. 

![Notifications](/images/flask_popover/notifications.gif)

We are going to extend our application to include a private messaging system that allows users to send and recieve messages. Then we will learn how we can implement notifications to users.

## Support for Private Messages

We need to extend our database to support private messages. We will add a new table called `Messages` to our database.

`app/models.py`: Message table
```python
class Message(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    sender_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    recipient_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    body = db.Column(db.String(140))
    timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)

    def __repr__(self):
        return f'Message: {self.body}'
```

Other than the body of the message, this table contains two columns that that define the sender and recipient of the message. We will need to update the `User` model to include a `messages` relationship.

`app/models.py`: Private messages relationship
```python
class User(UserMixin, db.Model):
    # ...
    messages_sent = db.relationship(
        'Message',
        foreign_keys='Message.sender_id',
        backref='author',
        lazy='dynamic')
    messages_received = db.relationship(
        'Message',
        foreign_keys='Message.recipient_id',
        backref='recipient',
        lazy='dynamic')
    last_message_read_time = db.Column(db.DateTime)
```
The `User` model now has a `messages_sent` and `messages_received` relationship. The `messages_sent` relationship is a `dynamic` relationship that returns a query object. This query object will return all messages sent by a user while `messages_received` will return all messages received by a user. The `last_message_read_time` column is used to track when a user last read their messages, found in the messages page.

I am going to add a helper method to the `User` model to get the number of unread messages a user has.

`app/models.py`: Get number of unread messages
```python
class User(UserMixin, db.Model):
    # ...
    def new_messages(self):
        last_read_time = self.last_message_read_time or datetime(1900, 1, 1)
        return Message.query.filter_by(recipient=self).filter(
            Message.timestamp > self.last_read_time).count()
```

Since these are changes that will affect the schema of the `User` model, we will need to update the database to reflect these changes.

```python
(venv) $ flask db migrate -m "private messages"
(venv) $ flask db upgrade
```

## Sending Private Messages

Next, we will create a form that a user can use to send another user a private message.

`app/forms.py`: Send private message form
```python
class MessageForm(FlaskForm):
    message = TextAreaField('Message', validators=[
        DataRequired(), Length(min=0, max=140)])
    submit = SubmitField('Submit')
```

To display these private message form, we will need to create a `send_message` template.

```html
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block app_context %}
    <div class="row">
        <div class="col-md-12 text-center">
            <h1>Send a private message to {{ recipient }}</h1>
        </div>  
    </div>
    <div class="row">
        <div class="col-sm-4">
            <!-- Empty column -->
        </div>
        <div class="col-sm-4 my-form">
            {{ wtf.quick_form(form) }}
        </div>
        <div class="col-sm-4">
            <!-- Empty column -->
        </div>
    </div>
{% endblock %}
```
We will then create a new route to handle the form submission.

`app/routes.py`: Send private message route
```python
# ...
from app.forms import MessageForm


@app.route('/send-message/<recipient>', methods=['GET', 'POST'])
@login_required
def send_message(recipient):
    user = User.query.filter_by(username=recipient).first_or_404()
        form = MessageForm()
        if form.validate_on_submit():
            msg = Message(
                author=current_user,
                recipient=user,
                body=form.body.data)
            db.session.add(msg)
            db.session.commit()
            flash('Your message has been sent.')
            return redirect(url_for('user', username=recipient))
        return render_template(
            'send_message.html',
            title='Send Message',
            form=form,
            recipient=recipient)

```

To make it easy for a user to send a private message, we will need to display a link to this form on the user's profile page. This can be extended to the popover template too.


```html
{% if user != current_user %}
    <p>
        <a href="{{ url_for('send_message', recipient=user.username) }}">
            Send private message
        </a>
    </p>
{% endif %}
```

## View Private Messages

To complete the private messaging system, we need to create a new route to render the messages page.

`app/routes.py`: All messages route
```python
@app.route('/messages')
@login_required
def messages():
    current_user.last_message_read_time = datetime.utcnow()
    db.session.commit()
    page = request.args.get('page', 1, type=int)
    messages = current_user.messages_received.order_by(
        Messages.timestamp.desc()).paginate(
            page, app.config['POSTS_PER_PAGE'], False)
    next_url = url_for('messages', page=messages.next_num) \
        if messages.has_next else None
    prev_url = url_for('messages', page=messages.next_num) \
        if messages.has_prev else None
    return render_template(
        'messages.html',
        messages=messages.items,
        next_url=next_url,
        prev_url=prev_url)
```

Once a user has clicked on the link to the messages page, the first thing we want to do is to update the `last_message_read_time` column of the current user. This basically marks the messages send to this user as read. We then query the `Messages` table for all messages received by the current user and order them by the timestamp in descending order, and paginate the results.

The next obvious thing is to display all these messages in a template.

`app/templates/messages.html`: Messages template
```html
{% extends 'base.html' %}

{% block app_context %}
    <div class="row text-center">
        <div class="col-md-12">
            <h1>{{title}}</h1>
        </div>  
    </div>
    <!-- Display all posts -->
    <div class="row">
        <div class="col-sm-12">
            {% for post in messages %}
                {% include '_post.html' %}
            {% endfor %}
        </div>
    </div>
    <!-- End of displaying all posts -->

    <div class="row">
        <div class="col-sm-12">
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
        </div>
    </div>
{% endblock %}
```

Notice how I am looping through the messages and displaying them.

To make it easier for a user to read their private messages, we will need to add a link, preferably in the navigation bar, to the messages page.

`app/templates/base.html`: Link to messages page
```html
{% if current_user.is_authenticated %}
    <li>
        <a href="{{ url_for('send_message') }}">
            Messages
        </a>
    </li>
{% endif %}
```
Create two users in two browser windows and send a private message from one to the other. Hopefully, you can see the message appear in the messages page of the recipient.

![Private messaging system](/images/flask_popover/private_messaging_system.gif)