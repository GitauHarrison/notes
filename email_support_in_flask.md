# Email Support In Flask

Many circumstances might require you to add email support to your application. In this article, the scope of these circumstances is going to be limited to sending password reset emails to users whenever they forget their passwords. Whenever users forget their passwords, they can click a "Forgot password" link, and the application will send them a custom reset link to their inbox from where they can take action.

The completed project can be found on [GitHub](https://github.com/GitauHarrison/how-to-add-email-support-in-a-flask-app/). 


## Configure Flask-Mail

You probably know that Flask uses extensions to support its philosophy of being lean and extensible, meaning, as a developer, you get to choose what extension you want to use to support your development work. In the context of sending password reset emails, we shall be using the [Flask-Mail](https://pythonhosted.org/Flask-Mail/) package. Kindly note that the assumption here is that you already know how to work with Flask. If this is your first time, I recommend you start [here](start_flask_server.md).

Let us begin by installing the Flask-Mail package in an active virtual environment:

```python
(venv)$ pip3 install flask-mail
```

We will need to configure the extension from the `app.config` object:

```python
# app/__init__.py: Flask-Mail instance

# ...
from flask_mail import Mail


app = Flask(__name__)


# ...
mail = Mail(app)

#...
```

## Add Email Server Details

For us to send an email, we need to provide all the configuration variables that are required such as an email server and port, a flag to enable encrypted connections, and option username and passwords. 

```python
# config.py: Email configuration variable

# ...

class Config(object):
    # ...
    MAIL_SERVER = os.environ.get('MAIL_SERVER')
    MAIL_PORT = int(os.environ.get('MAIL_PORT') or 25)
    MAIL_USE_TLS = os.environ.get('MAIL_USE_TLS') is not None
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')            # <--- your email address
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')            # <--- your app password for less secure apps
    ADMINS = ['your-email@example.com']                        # <--- admin email for error logs (optional)
```

All these are configuration variables. They point to the `.env` file which contains the actual values. This file should be a secret and you should not push it to version control since it contains very sensitive data such as passwords to your account. You can add it to your `.gitignore` file. If you do not have this file, create one in the project's root directory:

```python
(venv)$ touch .env
```

Then provide the values for each. For example:

```txt
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USE_TLS=True
MAIL_USERNAME=myemail@example.com
MAIL_PASSWORD=sfgs afav asdfa adfa
ADMINS=[adminemail@example.com]
```

There are a handful of email servers out there. In our illustration, we are going to use GMAIL. Google recently made changes to the [requirements needed by less secure apps](https://support.google.com/accounts/answer/185833?hl=en&visit_id=638152967033191399-217831286&p=app_passwords_sa&rd=1) to access the GMAIL server. Be sure to create and use an app password, and not your GMAIL account password. If you would like to use a real email server and do not want to complicate yourself with configuring GMAIL, a nice alternative is [SendGrid](twilio_sendgrid/00_overview.md). It allows you to send a maximum of 100 emails per day in a free account.


## Using Flask-Mail

To quickly test our email set up, we can resort to using the Python shell. Fire it up using `flask shell` and let us run the following commands:

```python
(venv)$ flask shell


>>> from flask_mail import Message
>>> from app import mail
>>> msg = Message("[Test] My Subject", sender=app.config["ADMINS"][0], recipients=["myemail@example.com"])
>>> msg.body = "This is a text body"
>>> msg.html = "<p>This is an HTML body</p>"
>>> mail.send(msg)
```

If you check the inbox of `recipients`, you will notice that indeed there is a new email. The sender of this email is the first item in `ADMINS`. To fully craft an email, we need to provide a subject and the email body. Our `subject` is ""[Test] My Subject" while the `body` of the email contains two formats: (1) Text and (2) HTML. Should there be no content for HTML, the email server will default to using the text copy of the body.


## Defining A Simple Email Framework

As wonderful as the example above is, it will become tedious whenever we want to send multiple emails. Rather than having to do this every single time, we can define a simple method that accepts all the needed email fields. This email framework will exist in a standalone module called `app/email.py`.

```python
# app/email.py

from flask_mail import Message
from app import app


def send_email(subject, sender, recipients, text_body, html_body):
    msg = Message(subject, sender=sender, recipients=recipients)
    msg.body = text_body
    msg.html = html_body
    mail.send(msg)

```

## Request A Password Reset

Users of the application have the option of requesting a password reset. This can be achieved by providing them a link that redirects them to a form where they can input their email addresses. These email addresses will be used to receive the password reset link.

```html
<!-- app/templates/login.html: Link to request password reset -->

<p>
    Forgot your password? 
    <a href="{{ url_for('request_password_reset') }}">Reset here</a>
</p>
```

Once the "Forgot your password?" link is clicked, it will take them to this form:

```python
# app/forms.py: Define the request password form

# ...
class RequestPasswordResetForm(FlaskForm):
    email = StringField('Email', validators=[DataRequired(), Email()])
    submit = SubmitField('Request Password Reset')
```

To display the form in HTML, we will need to create and update `request_password_reset.html` template as follows:

```html
<!-- app/templates/request_password_reset.html: Request Password Reset Template -->

{% extends "base.html" %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block content %}
    <h1>Reset Password</h1>
    {{ wtf.quick_form(form) }}
{% endblock %}
```

The view function handling this request is as follows:

```python
# app/routes.py: View function used to request password reset


from app.forms import RequestPasswordResetForm
from app.email import send_password_reset_email

@app.route('/reset_password_request', methods=['GET', 'POST'])
def reset_password_request():
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    form = ResetPasswordRequestForm()
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first()
        if user:
            send_password_reset_email(user)
        flash('Check your email for the instructions to reset your password')
        return redirect(url_for('login'))
    return render_template(
        'reset_password_request.html', title='Reset Password', form=form)
```

Should the user be logged in to their account, there is no need to reset their password. In this case, we return them to the index page. If they are not logged in, we begin by validating their information to ensure that indeed they are users of our application in which case `send_password_reset_email()` sends them a reset email. This helper method does not currently exist; we shall create it later in a [subsequent section](#sending-password-reset-email) below.


## Generating Password Reset Tokens

One of the things we need to pay attention to when sending links that have the potential to alter a user's account is security. For example, we have to make sure that the password reset link is sent to a user who truly owns the account. Additionally, we can ensure that we provide a time window during which this user needs to update their account. After this window, the link sent to them will expire and become unusable at which point they will need to generate another one.

The links are going to be provisioned with a token, and this token will be validated before allowing the password change, as proof that the user that requested the email has access to the email address on the account. A very popular token standard for this type of process is the JSON Web Token or JWT. Let us see how it works:

```python
(venv)$ flask shell


>>> import jwt
>>> token = jwt.encode({"password": "testpassword123"}, "secret", algorithm="HS256")
>>> token
'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJwYXNzd29yZCI6InRlc3RwYXNzd29yZDEyMyJ9.DoE0AM2MVJtq33Ojo2mfbIvplC51wv8_25SA-YXb804'

>>> jwt.decode(token, "secret", algorithms=["HS256"])
{"password": "testpassword123"}
```
The dictionary `{"password": "testpassword123"}` is a sample payload written into the token. The string "secret" has been used to make the token secure and is used to create the cryptographic signature. Later, you will see us replace the string "secret" with the value of `SECRET_KEY` as this is a lot more secretive and difficult to guess. The `algorithm` argument specifies how the token is to be generated.

The output of the token seems difficult to guess, but the truth is it can be decoded easily by anyone. To test, copy the value of the token in the [JWT Debugger](https://jwt.io/#debugger-io) and see its contents. This can be worrisome, but there is a way around it. To make the token secure, we need to sign it such that should it be tampered with, it becomes invalid. We can also provide an expiration time after which the user will need to generate another token.

Since tokens belong to a user, the most appropriate place to make these changes is the `User` model:

```python
# app/models.py: Token generation

# ...
from time import time
from app import app
import jwt


class User(UserMixn,  db.Model):
    # ...


    def get_reset_password_token(self, expires_in=600):
        return jwt.decode(
            {'reset_password': self.id, 'exp': time() + expires_in},
            app.config['SECRET_KEY'],
            algorithm='HS256'
        )

    @staticmethod
    def verify_reset_password_token(token):
        try:
            id = jwt.decode(
                token,
                app.config['SECRET_KEY'],
                algorithms=['HS256'])['reset_password']
        except:
            return
        return User.query.get(id)

```

The `get_reset_password_token()` returns a JWT token which is decoded and verified by `verify_reset_password_token()` static method. If the token cannot be validated or is expired, we return `None`, otherwise, we return the user.


## Sending Password Reset Email

Earlier, you so we used `send_password_reset_email()` in the view function used to request a password reset. Below, you can see the generic `send_email()` framework to send a reset link.

```python
# app/email.py: Send reset email


from flask import render_template,
from app import app


# ...


def send_password_reset_email(user):
    token = user.get_reset_password_token()
    send_email(
        '[Test] Reset Your Password',
        sender=app.config['ADMINS'][0],
        recipients=[user.email],
        text_body=render_template(
            'email/reset_password.txt', user=user, token=token),
        html_body=render_template(
            'email/reset_password.html', user=user, token=token))
```

The flexibility that this function provides is that we can create custom messages in the templates.

```python
# app/templates/email/reset_password.txt: Text email for password reset


Dear {{ user.username }},

To reset your password click on the following link:

{{ url_for('reset_password', token=token, _external=True) }}

If you have not requested a password reset simply ignore this message.

Sincerely,

Email Support Team
```

A nicer HTML version is as follows:

```html
<!-- app/templates/email/reset_password.txt: HTML email for password reset -->

<p>Dear {{ user.username }},</p>
<p>
    To reset your password
    <a href="{{ url_for('reset_password', token=token, _external=True) }}">
        click here
    </a>.
</p>
<p>Alternatively, you can paste the following link in your browser's address bar:</p>
<p>{{ url_for('reset_password', token=token, _external=True) }}</p>
<p>If you have not requested a password reset simply ignore this message.</p>
<p>Sincerely,</p>
<p>Email Support Team</p>
```

The `reset_password()` view function is currently not defined, but we will do so below. When `_external=True` is passed as an argument, complete URLs are generated either using localhost or by an application's domain name.


## Resetting A User's Password

Once a user has clicked on the link received in their inbox, they will be redirected to a form with two password input fields, one to create a new password, and another to confirm the new password. This form will be rendered by the `reset_password()` view function:

```python
# app/routes.py: Reset password view function

from app.forms import ResetPasswordForm


@app.route('/reset_password/<token>', methods=['GET', 'POST'])
def reset_password(token):
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    user = User.verify_reset_password_token(token)
    if not user:
        return redirect(url_for('index'))
    form = ResetPasswordForm()
    if form.validate_on_submit():
        user.set_password(form.password.data)
        db.session.commit()
        flash('Your password has been reset.')
        return redirect(url_for('login'))
    return render_template('reset_password.html', form=form)

```

The token passed in the URL is used to identify a user. Notice how we are passing this token to the `verify_reset_password_token()` method to get the user. Once found, we use the `set_password()` helper method to update the user's password and commit the changes to the database. Here is the `ResetPasswordForm` form definition:

```python
# app/forms.py: Reset password form

class ResetPasswordForm(FlaskForm):
    password = PasswordField('Password', validators=[DataRequired()])
    confirm_password = PasswordField(
        'Confirm Password', validators=[DataRequired(), EqualTo('password')])
    submit = SubmitField('Password Reset')

```

This form will be displayed as follows:

```html
<!-- app/templates/reset_password.html: Password reset form template. -->

{% extends "base.html" %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block content %}
    <h1>Reset Your Password</h1>
    {{ wtf.quick_form(form) }}
{% endblock %}
```

That is it! The password reset feature is complete; you can try it out.


## Asynchronous Emails

One thing you probably noticed when trying out the sending of emails is that the application temporarily comes to a standstill until the email has been processed. During this time, there is not much you can do. The reason for this is that the email processing is happening in the foreground where everything works linearly. The best approach would be to send this task to the background since it slows down our application. This sending it to the background is sometimes referred to as _asynchronous_ processing of emails. What this means is that while email processing continues, we are free to continue interacting with other parts of the application at the same time.

Python has support for asynchronous tasks. It provides the `threading` and `multiprocessing` modules for this. Starting the email processing as a background thread is a lot less resource-intensive than starting a brand-new process, so we can use that approach.

```python
# app.emal.py: Asynchronous email processing

# ...

def send_asyn_email(app, msg):
    with app.app_context():
        mail.send(msg)


def send_email(subject, sender, recipients, text_body, html_body):
    msg = Message(subject, sender=sender, recipients=recipients)
    msg.body = text_body
    msg.html = html_body
    Thread(target=send_async_email, args=(app, msg)).start()

```

The send_async_email function now runs in a background thread, invoked via the Thread class in the last line of send_email(). With this change, the sending of the email will run in the thread, and when the process completes the thread will end and clean itself up. If you have configured a real email server, you will notice a speed improvement when you press the submit button on the password reset request form.


When working threads remember to add the _contexts_ of the application. Flask has two types of contexts, the _application_context_ and the _request_context_. Most of the time, the contexts are usually managed by the framework, but when we send a task to a thread, we may need to manually create them, hence you see me pass in the application's context as `app`.

The `mail.send()` method needs to access the configuration values of the email server from the object `app.config`. This is the reason why it needs to know what the application is. The application context is created through `app.app_context()`.