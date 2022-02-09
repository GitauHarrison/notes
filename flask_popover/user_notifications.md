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

`app/templates/send_message.html`: Send private message form
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

## Static Notification Badges

Creating a notification bagde to tell a user that there are new messages in their inbox is actually quite simple. Bootstrap provides a badge widget that we can take advantage of.

`app/templates/base.html`: Add badge to messages link
```html
<li>
    <a href="{{ url_for('messages') }}">
        Messages
        {% set new_messages = current_user.new_messages() %}
        {% if new_messages %}
            <span class="badge">{{ new_messages }}</span>
        {% endif %}
    </a>
</li>
```

By invoking the `new_messages()` helper method in the `User` model, we are able to store the number of unread messages in a new variable called `new_messages`. We then pass this variable's value to the badge widget.

![Static notification badge](/images/flask_popover/static_notification_badge.gif)

## Visibility of Notification Badges

A user has to click on any link within the application so that the bage can appear. This means that the badge can only be seen when the value of unread messages in non-zero. This does not provide the best user experience. What would be great is to have the badge be present all the time even if the value of `new_messages` is zero.

`app/templates/base.html`: Add visibility to badge
```html
<li>
    <a href="{{ url_for('messages') }}">
        Messages
        {% set new_messages = current_user.new_messages() %}
        {% if new_messages %}
            <span id="message_count" class="badge"
                style="visibility: {% if new_messages %} visible {% else %} hidden {% endif %};">
                {{ new_messages }}
            </span>
        {% endif %}
    </a>
</li>
```

Now, the badge will always be present, but the its visibility is conditional. If the value of `new_messages` is zero, the badge will be hidden, but if the value of `new_messages` is non-zero, the badge will be visible.

We can use JQuery to update the badge's visibility whenever the value of `new_messages` changes.

`app/templates/base.html`: Update badge visibility
```html
{% block scripts %}
    <script>
        // ...
        function set_message_count(n) {
            $(#message_count).text(n);
            $(#message_count).css('visibility', n ? 'visible': 'hidden');
        }
    </script>
{% endblock %}
```
## Deliver Notification Changes to Clients

There are two ways to deliver an update to the notification badges seen by the client. The changes will invoke `set_message_count()` to update the badge's visibility.

1. Using ajax
2. Using a websocket

### Using Ajax

The application (client) will send periodic asynchronous request to the server to check for new messages. The server will respond with a list of updates which the client can use to update various parts of the application such as the notification badge. All that needs to be done is to render a route that returns a JSON list of notifications. This approach, though, has a short-coming, in the sense that there is going to be a delay between the time a request is made and the time the response is received. If the client asks for an update every 5 seconds, notifications can be received 5 seconds late.

### Using a WebSocket

The most common method to implement server-initiated updates is by extending the flask server to support WebSocket connections besides HTTP. 

> WebSocket is a protocol that unlike HTTP, establishes a permanent connection between the server and the client. The server and the client can both send data to the other party at any time, without the other side asking for it. 

The advantage of using this approach is that there is no delay whenever an event of interest occurs. The client can receive notifications as soon as they occur.

If the kind of application being built requires near-zero latency, then the WebSocket approach is most suitable. In our case, we can use the first approach since it is much easier to setup compared to WebSocket.

## Prepare the Database for Notification Updates

We need to update the `User` model to work with user notifications.

`app/models.py`: Notification table
```python
# ...
import json
from time import time


class User(UserMixin, db.Model):
    # ...
    notifications = db.relationship(
        'Notification', backref='user', lazy='dynamic')


class Notification(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(128), index=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    timestamp = db.Column(db.Float, index=True, default=time)
    payload_json = db.Column(db.Text)

    def get_data(self):
        return json.loads(str(self.payload_json))
```

The payload will be different for each notification, so writing it as a JSON string is a good idea. The `get_data()` method will be used to deserialize and retrieve the payload of a notification.

With these new changes, we need to apply them to our database.

```python
(venv) $ flask db migrate -m 'notifications'
(venv) $ flask db upgrade
```

Since we are not going to create a form to add notifications, we can create a helper method within `Notifications` table to update a user's notifications.

`app/models.py`: Add notifications to the database
```python
class Notification(db.Model):
    # ...

    def add_notification(self, name, data):
        self.notifications.filter_by(name=name).delete()
        n = Notification(name=name, payload_json=json.dumps(data), user=self)
        db.session.add(n)
        return n
```

The first thing we want to do is to delete the name of the previous notification as soon as a new update has come. This is done by filtering the notifications by the name and deleting them. We can give a notification a name, say "unread_message_count". If the database has a notification by this name with a value of, say 3, and the message count goes to 4, then we need to replace the old notification with the new one. The `add_notification()` method will create a new notification and add it to the database.

## Update the Client to Use the New Notification Model

The next logical step would be to update the user's notifications when two events take place:

1. When a user is sending a message (this creates the need for a notification)
2. When a user is reading a message (this removes the need for a notification)

We need to update the `send_message()` view function to add a notification whenever a user sends a message.

`app/routes.py`: Create a notification
```python
def send_message(recipient):
    # ...
    if form.validate_on_sbumit():
        # ...
        user.add_notification('unread_message_count', user.new_messages())
        db.session.commit()
```

The event of reading a message is handled by the `messages()` view function. Here, we need to reset the notification to zero.

`app/routes.py`: Reset notification
```python
def messages():
    # ...
    current_user.last_message_read_time = datetime.utcnow()
    current_user.add_notification('unread_message_count', 0)
    db.session.commit()
    # ...
```

The client can retrieve the notifications of a logged in user by making an asynchronous request to the server.

`app/routes.py`: Retrieve notifications
```python
from app.models import Notification
from flask import jsonify


@app.route('/notifications')
@login_required
def notifications():
    since = request.args.get('since', 0.0, type=float)
    notifications = current_user.notifications.filter_by(
        Notifications.timestamp > since).order_by(
            Notification.timestamp.asc())
    return jsonify(
        [
            {
                'name': n.name,
                'data': n.get_data(),
                'timestamp': n.timestamp
            }for n in notifications
        ]
    )
```

Each JSON payload is a list of notification for the user. Each notification is a dictionary with the following keys:

* `name`: The name of the notification
* `data`: The payload of the notification
* `timestamp`: The timestamp of the notification


## Poll the Server for Updates

As mentioned earlier, we  are going to make asynchronous request to the server to get the notifications. The client will need to periodically make this request.

`app/templates/base.html`: Add a polling script
```html
<script>
    // ...
    {% if current_user.is_authenticated %}
        $(function () {
            var since = 0;
            setInterval(function() {
                $.ajax("{{ url_for('notifications') }}?since=" + since).done(
                    function(notifications) {
                        for (var i = 0; i < notifications.length; i++) {
                            if (notifications[i].name == 'unread_message_count')
                                set_message+count(notifications[i].data);
                            since = notifications[i].timestamp;
                        }
                    }
                );
            }, 5000);
        });
    {% endif %}
</script>
```

The `setInterval()` function works similarly to the `setTimeout()` function. The only difference is that the function is called repeatedly. The first argument is the function to be called, and the second argument is the time in milliseconds between each call.

The function being called issues an Ajax request to the server to retrieve the notifications. By iterating through the list of notifications, if a notification with the name `unread_message_count` is received, the client will update the message count, visible in the badge.

What is worth paying attention to is the `since` argument. Unfortunately, Flask's `url_for()` function only runs once, meaning that this argument will remain the same. We have set it to 0 hence its initial value will be _/notifications?since=0_. But we need this argument to be dynamic such that as soon as a user receives a new notification, it should update according to its timestamp. This ensures that there are no duplicate notifications. The `since` argument is the timestamp of the last notification received.