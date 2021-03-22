# Two-factor Authentication Using Twilio Verify in Flask

There are a couple of ways to use two-factor authentication in a flask application. In a [previous article](2fa_flask.md), I showed how you can enable mandatory two-factor authentication where users of an app have to key in a time-based one-time password that is sent to TOTP app. In this article, we will make two-factor authentication optional. Whenever a user enables this feature, an sms with entry code will be sent to their phone. They will use that code to authenticate themselves.

[Twilio Verify](https://www.twilio.com/verify) is a service that allows your application to send verification codes to users of your application through SMS or phone call.

![Twilio Verify](images/twilio_verify/twilio_verify.gif)

What we will do:

1. Create a flask app with user login features
2. Add a profile page
3. Integrate Twilio Verify

Kindly note that you will need these before you proceed:

* A working phone number
* A Twilio account. Create a [free account](https://www.twilio.com/try-twilio?promo=WNPWrR) now.
* Python 3.5+

### Flask App with User Login

#### Project Structure

We will use this structure:

```python
project_folder
    |------- .flaskenv
    |------- .env
    |------- .env.template
    |------- config.py
    |------- twilio_verify.py
    |------- requirements.txt
    |------- .gitignore
    |------- app/
              |------- __init__.py
              |------- routes.py
              |------- models.py
              |------- forms.py
              |------- errors.py
              |------- email.py
              |------- static/
                        |------- images/
                        |------- css/styles.css
              |------- templates/
                        |------- base.html
                        |------- home.html
                        |------- login.html
                        |------- register.html
                        |------- 404.html
                        |------- 500.html
                        |------- reset_password_request.html
                        |------- reset_password.html
                        |------- user.html
                        |------- verify_2fa.html
                        |------- email/
                                   |----- reset_password.html
                                   |----- reset_password.txt

```

You can create this structure using the `mkdir` and `touch` terminal commands:

```python
$ mkdir project_folder # creates an empty directory called project_folder
$ touch project_folder/config.py # creates an empty config.py file inside project_folder
```

Once you have completed this project structure, move into project_folder:

```python
$ cd project_folder
```

#### Create Virtual Environment

Virtual environments allow you to isolate your project requirements from that of the Operating System. You need to create an activate it:

```python
$ mkvirtualenv twilio_verify 
```

I have used the `virtualenvwrapper` to manage my virtualenv workflow. If you do not know what it is, learn more [here](virtualenvwrapper_setup.md).

This application will use these dependencies:

* [flask](https://flask.palletsprojects.com/en/1.1.x/)
* [flask-sqlalchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/)
* [flask-boostrap](https://pythonhosted.org/Flask-Bootstrap/)
* [flask-login](https://flask-login.readthedocs.io/en/latest/)
* [flask-wtf](https://flask-wtf.readthedocs.io/en/stable/)
* [flask-mail](https://pythonhosted.org/Flask-Mail/)
* [flask-migrate](https://flask-migrate.readthedocs.io/en/latest/)
* [python-dotenv](https://pypi.org/project/python-dotenv/)
* [pyjwt](https://pyjwt.readthedocs.io/en/stable/)
* [pyngrok](https://pypi.org/project/pyngrok/)
* [pyqrcode](https://pypi.org/project/PyQRCode/)
* [email-validator](https://pypi.org/project/email-validator/)

To install all of them at once, run:

```python
(twilio_verify)$ pip3 install flask flask-sqlalchemy flask-bootstrap # Add all the other dependencies within this same line
```

To work with Twilio Verify, you will also need to install it:

```python
(twilio_verify)$ pip3 install "twilio>=6.17.0"
```

From your root directory (project_folder), update your `requirements.txt` to contain all the installed dependencies:

```python
(twilio_verify)$ pip3 freeze > requirements.txt
```