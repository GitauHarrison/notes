# Enable Two-factor Authentication using Twilio Authy's Push Notification in Your Flask App

![Enable 2fa push notification](/images/twilio_authy/push_notification.gif)

To better understand what push notification is, we will compare it with other forms of two-factor authentication.

* The first method is the mandatory time-based two-factor authentication where a user has to key in a token from a TOTP app in their smartphone before being granted access to their account. I have talked about in [this article](two_factor_authentication/2fa_flask.md). If a user loses his smartphone, for instance, and therefore cannot access the TOPT app, he has no other way to access his account. This method is especially used in the banking sector to safeguard user's accounts.
* The second method is where a numeric code is sent to a user's current phone number either as an SMS or a phone call. The numeric code is specific to this user who will then add that token when he attempts log in to his account. I have also discussed it in [this other article](two_factor_authentication/twilio_verify_2fa.md). 

Both methods above require a user's active participation in the authentication process. With push notification, instead of using a TOTP app or a numeric code being sent to a user's current phone number, a notification is sent to an app, say [Twilio Authy](https://authy.com/), and the user simply has to click "Accept" or "Deny" to complete the log in process or stop such an attempt.

![Twilio Authy Notiifcation](/images/twilio_authy/authy_notification.png)

## Getting Started

To understand how a project can add optional two-factor authentication using push notification, let us look at [this elaborate design](https://www.figma.com/proto/yup5WjYWXFWS2z2Yrwj84d/Twilio-2fa-Push-Notifcation?node-id=1%3A323&scaling=min-zoom&page-id=0%3A1). 

* A new user registers for an account.
* He navigate to the profile page where the find a link to enable 2fa.
* Clicking this link in the profile page navigates him to a button that he can finally click to enable 2fa. This might logically sound repetitive but it gives the user a chance to decide whether they really want to enable 2fa.
* Clicking the "Enable 2fa" button presents a QR Code page that they need to scan using Twilio Authy app to finally enable 2fa.
* He is redirected to the _Profile_ page where the link now becomes _Disable two-factor authentication_.
* When he logs out at this stage, he will be required to press either "Approve" or "Deny" in the next attempt to access their account.
* An approved status response logs the user back to his account.
* He can choose to disable 2fa by clicking the _Disable two-factor authentication_ link in the _Profile_ page.
* A _Disable_ button will be shown to initiate the disabling process.
* He is redirected to the _Profile_ page after disabling 2fa

## Implementing Push 2FA

Now that we understand we can create the project, let us start implementing two-factor authentication.

### Project Structure

We are going to create an application that shows only how to implement two-factor authentication. We will begin by creating a simple structure for our application.

```
twilio_authy_project
  | ---- config.py
  | ---- my_authy_app.py
  | ---- requirements.txt
  | ---- .gitignore
  | ---- .flaskenv
  | ---- .env
  | ---- .env-template
  | ---- app/
          | --- __init__.py
          | --- email.py
          | --- models.py
          | --- auth/
                | --- __init__.py
                | --- authy.py
                | --- email.py
                | --- forms.py
                | --- routes.py
         | --- errors/
                | --- __init__.py
                | --- handlers.py
         | --- main/
                | --- __init__.py
                | --- forms.py
                | --- routes.py
         | --- static/
                | --- css/
                       | --- styles.css
         | --- templates/
                | --- base.html
                | --- home.html
                | --- edit_profile.html
                | --- user.html
                | --- auth/
                       | --- check_2fa.html
                       | --- disable_2fa.html
                       | --- enable_2fa.html
                       | --- enable_2fa_qr.html
                       | --- login.html
                       | --- register.html
                       | --- reset_password_request.html
                       | --- reset_password.html
                | --- email/
                       | --- reset_password.html
                       | --- reset_password.txt
                | --- errors/
                       | --- 404.html
                       | --- 500.html
```                
Use the commands `mkdir` and `touch` in your terminal to create this structure:

```python
# Create the project's directory
$ mkdir twilio_authy_project 

# Create the config file inside the project's folder
$ touch twilio_authy_project/config.py

# Complete the structure above
```

The assumption is that you have a basic understanding of Python and Flask. If not, consider this reference links as you build this project:

* New? [Start here](https://github.com/GitauHarrison/notes/blob/master/web_development/personal_blog/personal_blog.md).
* If you would like to use `virtualenvwrapper` to manage your workflow, use [this guide](https://github.com/GitauHarrison/notes/blob/master/virtualenvwrapper_setup.md) to set up your machine. Otherwise, manually create and activate your virtual environments.

### Project Dependencies

This project will require several dependencies. These are:

* flask (framework)
* Twilio Authy API (2fa)
* flask-wtf (web forms)
* flask-bootstrap (styling)
* flask-login (user sessions)
* flask-sqlalchemy (database)
* flask-migrate (database management)
* flask-moment (pretty time display)
* pyjwt (token generation)
* pyngrok (localhost testing)
* email-validator 
* qrcode

Ensure you install these dependencies in an activate virtual environment. I will create one called 'authy' using `mkvirtualenv` command from [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/):

```python
$ mkvirtualenv authy # created and activated at the same time
```

First, let us include the [Authy client for Python](https://pypi.org/project/authy/):

```python
(authy)$ pip3 install "authy>=2.2.5"
```

The client library for Python must be version 2.2.5 or higher. From the application design, you have noticed that we use a QR Code image. So the application needs to generate tokens and QR Code. We will use `pyjwt` and `qrcode` packages.

```python
(authy)$ pip3 install pyjwt qrcode
```

Finally, we need to install all the other packages:

```python
(authy)$ pip3 install flask flask-sqlalchemy # and the rest
```
