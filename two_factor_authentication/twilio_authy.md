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

## Configure Authy Service

Now that we understand we can create the project, let us start implementing two-factor authentication.

Let us configure the Authy service:

* If you do not have an account, [create a free one](www.twilio.com/referral/WNPWrR) now. It is free with no hidden expectations. You will only need to provide your phone number for authentication
* From [Twilio Console](https://www.twilio.com/console), click "All Products and Services"
* Find and click [Authy](https://www.twilio.com/console/authy/applications)
* Click the blue button to create an application
* Provide a FRIENDLY NAME
![Provide friendly name](/images/twilio_authy/authy_app_name.png)

## Project Structure

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

## Project Dependencies

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

## Project Requirements

Ensure that you have:

* A smartphone
* [Twilio Authy](https://authy.com/) app downloaded and installed in your phone

## How to Generate Registration QR Code

QR Codes are graphical encodings of a short text, typically a URL that connects your application to the authentication app. Authy app expects QR Codes with this URL:

```python
authy//:account?token={JWT}
```

Where the `token` has the following payload:

```python
{
    'iss': '{authy_app_name}',
    'iat': {issue date},
    'exp': {expiration date},
    'context': {
        'custom_user_id': '{custom_user_id}',
        'authy_app_id': '{app_authy_id}'
    }
}
```
* `iss` (issuer): must be the Authy application name you defined in the [Twilio console](https://www.twilio.com/console/authy/applications)
* `iat` (issued at) and `exp` (expiration) are integers
* `customer_user_id` identifies the user of our application, typically the primary key assigned to a user in the database
* `authy_app_id` must be set to the identifier assigned by Authy to our application. To find this identifier, click on your Authy app name and find [Settings](https://www.twilio.com/console/authy/applications/388832/settings). 

![Authy App ID](/images/twilio_authy/authy_app_id.png)

* `authy_app_id` is APPLICATION ID

We will use the details in the "Properties" section of the Authy app we have created to generate a `token`. 

`app/auth/authy.py: Generate a registration JWT for a user`

```python
import time
import jwt
from flask import current_app


def get_authy_registration_jwt(user_id, expires_in=5*60):
       now = time.time()
       payload = {
              'iss': current_app.config['AUTHY_APP_NAME'],
              'iat': now,
              'exp': now + expires_in,
              'context': {
                     'custom_user_id': str(user_id),
                     'authy_app_id': current_app.config['AUTHY_APP_ID']
              },

       }
       return jwt.encode('payload',
                         currrent_app.config['AUTHY_PRODUCTION_API_KEY'])
```

We have a JWT. We need to generate the QR Code for Authy using the [`qrcode`](https://github.com/lincolnloop/python-qrcode) package:

`app/auth/authy.py: Generate QR Code`

```python
from io import BytesIO
import qrcode
import qrcode.image.svg


def get_authy_qrcode(jwt):
       qr = qrcode.make('authy://account?token=' + jwt,
       image_factory=qrcode.image.svg.SvgImage)
       stream = BytesIO()
       qr.save(stream)
       return stream.getvalue()
```

`get_authy_qrcode()` function generates a URL in the format expected by the Authy app. JWT is passed as an argument and it creates a QR Code.

SVG image format is used to save the QR Code, which is written to a byte stream and the stream contents returned.

## Scan the QR Code

The application needs to know whether a user has scanned the QR Code. There are two ways that can be used to check whether the QR Code has been scanned

* Webhooks
* Polling

We will make use of polling in this article. Even though this method is less efficient, it has the advantage of allowing the application to run from the localhost without the need to set up a domain and a secure SSL.

`app/templates/enable_2fa_qr.html: Polling implementation`

```python
{% block scripts %}
    {{ super() }}
    {{ moment.include_moment() }}
    <!-- Polling implementation -->
    <script>
        function check_registration() {
            $.ajax("{{ url_for('auth.enable_2fa_poll') }}").done(function(data) {
                if (data == 'pending') {
                    setTimeout(check_registration, 5000);
                }
                else {
                    window.location = "{{ url_for('main.user', username=current_user.username) }}";
                }
            });
        }
        setTimeout(check_registration, 5000);
    </script>
{% endblock %}
```
This piece of code is added on the template where the QR Code image is displayed. A request is sent to the route `enable_2fa_poll` by the function `check_registration()`. We have set the polling timeout to be 5 seconds after the page loads. In the case where the QR Code has not been scanned, the response status is going to be "pending". The application will keep polling after every 5 seconds untill the QR Code has been scanned, in which case we redirect the user to his profile page.

## Checking a User's Registration Status

We will create a new function that will handle this.

`app/auth/authy.py: Check registraion status`

```python
from authy.api import AuthyApiClient

def get_registration_status(user_id):
    authy_api = AuthyApiClient(current_app.config['AUTHY_PRODUCTION_API_KEY'])
    resp = authy_api.users.registration_status(user_id)
    if not resp.ok():
       return {'status': 'pending'}
    return resp.content['registration']

```

We pass the `user_id` to the function `get_registration_status()`, which is similar to the JWT identifier discussed earlier, and the application will know that the user id is assigned to this user. 

An unscanned registration status will return:

```python
{'status': 'pending'}
```

Otherwise, it will return:

```python
{'status': 'completed', 'authy_id': <int_value>}
```

We will use this identifier to refer to the user when we want to interact with the Authy service. Any other status in addition to the two above indicates that an error has occurred.

## Send Push Notification to the User

If a registered user has enabled two-factor notification, we need to send a push notification to the user's phone whenever they try to log into their account.

`app/auth/authy.py: Send push notification to a user`

```python
def send_push_authentication(user):
       authy_api = AuthyApiClient(current_app.config['AUTHY_PRODUCTION_API_KEY'])
       resp = authy_api.one_touch.send_request(
           user.authy_id,
           'Login requested for Authy Push Demo',
           details = {
               'Username': user.username,
               'IP Address': request.remote_addr,
           },
           seconds_to_expire=120
       )
       if not resp.ok():
              return None
       return resp.get_uuid()

```

`authy_api.one_touch.send_request()` function initiates the push notification.

* `authy_id` is the identifier that is used to report back to the application when the user has registered fort two-factor authentication
* `details`, a dictionary, helps the user to identify the login attempts that are legitimate. This information will be shown in the Authy app.
* `seconds_to_expire` sets the number of seconds Authy will wait for the user to repsonds to this notifiation.

A universally unique identifier (UUID) is returned which the Authy service assigns to the push request.

Other prameters can be seen in [Twilio Authy documentation](https://www.twilio.com/docs/authy/api/push-authentications#create-an-approval-request).

## Waiting for User o Authorize Login Request

A simple check that the backend can execute can be integrated into the polling cycle:

`app/auth/aythy.py: Authorize login request`

```python
def check_push_notification(uuid):
    authy_api = AuthyApiClient(current_app.config['AUTHY_PRODUCTION_API_KEY'])
    resp = authy_api.one_touch.get_approval_status(uuid)
    if not resp.ok():
       return 'error'
    return resp.content['approval_request']['status]

```

The returned status will remain "pending" untill the user taps either "Approve" or "Deny" buttons, at which point it will change to "Approved" or "Denied" respectively. If the time set out for this request elapses, then the status will change to "expired".