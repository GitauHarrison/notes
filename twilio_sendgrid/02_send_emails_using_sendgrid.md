# Send Emails Using Twilio SendGrid

Setting up an email service for your application is always difficult. You can choose to set everything up yourself or use a third-party email service. The most ideal approach, all factors considered, is always to favor the simplicity of a dedicated email service. 

In the tutorial [Email Support In Flask](/email_support_in_flask.md), we went over how you can use the Flask-Mail extension to set an email infrastructure for your application. In this article, you will learn how to integrate Twilio SendGrid into a similar application. 

## Requiremets

- Python 3.6 and above
- A free [Twilio SendGrid account](/twilio_sendgrid/01_create_acccount.md)
- Familiarity with Flask


## SendGrid Configuration

Once you have created an account, you need to generate an API key. To do so, from your dashboard, click "Settings > API Key".

![Settings API key](/images/sendgrid/send_emails/settings_api.png)

Then, click on the blue "Create API" button.

![Create API Key](/images/sendgrid/send_emails/create_api_key.png)

Fill in the details and select "Full Access" for permissions. This will allow you to perform all email-sending functions.

![Name API Key](/images/sendgrid/send_emails/name_api_key.png)

You will be redirected to another page containing your API Key. Copy the key in your clipboard.

![API Key](/images/sendgrid/send_emails/api_key.png)

This key should be a secret. I am showing it to you now, but I will discard it in no time. Once copied, click "Done".

## Updating Email Keys

In the project built in [Email Support In Flask](/email_support_in_flask.md#add-email-server-details), we set up our email configurations as follows:

```python
# config.py: Email configuration

# ...


class Config(object):
    # ...

    # Email configurations
    MAIL_SERVER = os.environ.get('MAIL_SERVER')
    MAIL_PORT = int(os.environ.get('MAIL_PORT') or 25)
    MAIL_USE_TLS = os.environ.get('MAIL_USE_TLS') is not None
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')

```

The only change we are going to make is the `MAIL_USERNAME` and `MAIL_PASSWORD`. We are also going to add `MAIL_DEFAULT_SENDER`.

```python
# config.py: SendGrid configuration

# ...


class Config(object):
    # ...

    # Email configurations
    MAIL_SERVER = os.environ.get('MAIL_SERVER')
    MAIL_PORT = int(os.environ.get('MAIL_PORT') or 25)
    MAIL_USE_TLS = os.environ.get('MAIL_USE_TLS') is not None
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('SENDGRID_API_KEY')
    MAIL_DEFAULT_SENDER = os.environ.get('MAIL_DEFAULT_SENDER')
```

The values of these variables are found in `.env` file as follows:

```python
# .env: Values of environment variables

MAIL_SERVER=smtp.sendgrid.net
MAIL_PORT=587
MAIL_USE_TLS=True
MAIL_USERNAME=apikey
SENDGRID_API_KEY=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
MAIL_DEFAULT_SENDER=name@email.com
```

While working with SendGrid, the mail server will always be `smtp.sendgrid.net`, the port `587` (or `25` if you prefer) and `TLS` must be enabled. For the username, you have to use `apikey`. Your password will be your API Key we just copied earlier. SendGrid will require you to authenticate an email that will be used in the "From" field. This email is what is referred to as `MAIL_DEFAULT_SENDER`. 

To create a `MAIL_DEFAULT_SENDER`, click on "Settings > Sender Authentication".

![Sender authentication](/images/sendgrid/send_emails/sender_authentication.png)

Click on "Verify A Single Sender".

![Verify A Single Sender](/images/sendgrid/send_emails/verify_a_single_sender.png)

You will be redirected to a new page where you can create your sender. Once done, click "Create".

![Create new sender](/images/sendgrid/send_emails/create_new_sender.png)

This sender's email address, once verified, should be used as the `MAIL_DEFAULT_SENDER` value in `.env`.


## Sending Email

To illustrate how to send an email in your Flask app, we will use the terminal. Activate the shell by running `flask shell`:

```python
(venv)$ flask shell
```

And now, to send an email, we can do the following:

```python
>>> from flask_mail import Message
>>> from app import mail
>>> msg = Message("[Test] My Subject", recipients=["myemail@example.com"])
>>> msg.body = "This is a text body"
>>> msg.html = "<p>This is an HTML body</p>"
>>> mail.send(msg)
```

That is it! Check your inbox for a new mail. Hope everything went well. If you would like to learn how to use a graphical user interface to send an email using SendGrid, learn how to do that [here](/email_support_in_flask.md#sending-password-reset-email).