# Two-factor Authentication Using Twilio Verify in Flask

There are a couple of ways to use two-factor authentication in a flask application. In a [previous article](2fa_flask.md), I showed how you can enable mandatory two-factor authentication where users of an app have to key in a time-based one-time password that is sent to TOTP app. In this article, we will make two-factor authentication optional. Whenever a user enables this feature, an sms with entry code will be sent to their phone. They will use that code to authenticate themselves.

[Twilio Verify](https://www.twilio.com/verify) is a service that allows your application to send verification codes to users of your application through SMS or phone call.

![Twilio Verify](images/twilio_verify/twilio_verify.gif)

What we will do:

1. Create a flask app with user login features
2. Add a profile page
3. Integrate Twilio Verify

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
