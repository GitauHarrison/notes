# Send Your Users Free Airtime

![Hero Image](/images/africas_talking/airtime/hero.jpeg)

How best can you appreciate your users or online clients? Say they have successfully purchased a product or service, and you want to show gratitude as a business. Or, they may be loyal clients who have stood by you for some time as a business, or as an organization.

One way can be to top up their voice call time. You can give them airtime. After successfully purchasing a product or service (in the case of an e-commerce site), you can send some airtime to their phone.

### Table of Contents

- [Welcome to Africa's Talking (AT)](#welcome-to-africas-talking-at)
- [Getting Started With the Airtime API from AT](#getting-started-with-the-airtime-api-from-at)
- [Putting It Together](#putting-it-together)
- [Notes](#notes)

## Welcome to Africa's Talking (AT)

Africa's Talking aims to power communication solutions in Africa, hence the name. They have quite a number of valuable services from SMS to USSD, Voice, Payments, and Airtime. In this article, I will show you how you can utilize their Airtime API to add the appreciation feature to a web application, Flask web app to be exact.

### Utilizing The Airtime API

```python
# airtime.py

import africastalking

username = "sandbox"
api_key = "your-api-key"

africastalking.initialize(username, api_key)

airtime = africastalking.Airtime

phone_number = "+2547xxxxxxxx"
currency_code = "KES" # Change this to your country's code
amount = 50

try:
    response = airtime.send(
      phone_number=phone_number,
      amount=amount,
      currency_code=currency_code)
    print(response)
except Exception as e:
    print(f"Encountered an error while sending airtime. '
          'More error details below\n {e}")
```

This is a very simple and direct Python script that makes use of the Airtime API from Africa's Talking, AT in short. The API needs the following to work:

- SDK ( `username` and `api_key`)
- Customer `phone_number`
- Your country's `currency`
- The `amount` to send

## Getting Started With the Airtime API from AT

These are the things you will need to do to work with the Airtime API:

- [Create a FREE account](#create-account)
- [Initialize the SDK](#initialize-the-sdk)
- [Initialize the Airtime Service](#initialize-the-airtime-service)
- [The Client](#the-client)
- [Sending](#sending)
- [Summary](#summary)

### Create Account

Africa's Talking has a comprehensive tutorial on the Airtime API. To get started, you will need to create an account with them. It is FREE.

![Create account](/images/africas_talking/airtime/create_account.png)

### Initialize the SDK

The SDK needs two things: `username` and `api_key`. These are found when you create an app. Africa's Talking provides a default "sandbox" app to start with.

![Initialize sdk](/images/africas_talking/airtime/initialize_sdk.png)

The `username` can be found at the top-right of the app. With the sandbox app, you are required to use "sandbox" for the `username`. The `api_key` can be found by clicking on "Settings > API Key" as seen below.

![Create api key](/images/africas_talking/airtime/create_api.png)

Enter your password to generate an API Key. Copy it somewhere because it will not be shown to you again. Remember, the value of the `api_key` is SECRET, so be sure to maintain it as such. Within your application, you may save it in a .env file.

```python
#.env

AT_API_KEY=''
AT_USERNAME=''
```

Access them as environment variables.

```python
# config.py

import os

class Config(object):
    
    # Africa's Talking configurations
    AT_API_KEY = os.environ.get("AT_API_KEY")
    AT_USERNAME = os.environ.get("AT_USERNAME")
```

### Initialize the Airtime Service

With the SDK in place, it is time to initialize the airtime service.

```python
# airtime.py

from app import app

africastalking.initialize(app.config["AT_USERNAME"], app.config["AT_API_KEY"])
airtime = africastalking.Airtime
```

The service is stored in a variable `airtime`. It will be used below to send a customer some airtime.


### The Client

Before crediting a customer's airtime balance, we at least need to identify them and specify what we would like to send them.

```python
# airtime.py

phone_number = current_user.phone
amount = app.config["AIRTIME_AMOUNT"]
currency_code = app.config["CURRENCY"]
```

Once again, the values are sourced from environment variables. Only the `phone_number` is gotten from the app's context.


### Sending

The `airtime.send()` function is used to send a customer airtime.

```python
# airtime.py

try:
  response = airtime.send(
    phone_number=phone_number,
    amount=amount,
    currency_code=currency_code)
  
  # See the output
  print(response)
except Exception as e:
  # See any possible errors
  print(f"An error occured. More info:\n\n {e}")
```

To ensure that the application does not crash in the event an error occurs, we can use the `try-except` block. We try sending some airtime. If not successful, then we print the error on the terminal so we can know what went wrong.


### Summary

The final script that sends a customer airtime is as below:

```python
# airtime.py

# Import AT package (pip3 install africastalking)
import africastalking
from app import app
from flask_login import current_user

def airtime():
  # Initialize SDK
  africastalking.initialize(
    app.config["AT_USERNAME"],
    app.config["AT_API_KEY"])

  # Connect to the airtime service
   airtime = africastalking.Airtime
  
  # To client
  phone_number = current_user.phone
  amount = app.config["AIRTIME_AMOUNT"]
  currency_code = app.config["CURRENCY"]
  
  # Send
  try:
    response = airtime.send(
      phone_number=phone_number,
      amount=amount,
      currency_code=currency_code)
    
    # See the output
    print(response)
  except Exception as e:
    # See any possible errors
    print(f"An error occured. More info:\n\n {e}")
```

Remember, to use the africastalking package, you will need to install it in your active virtual environment:

```python
# Activate your virtualenvironment
$ python3 -m venv venv      # create
$ source venv/bin/activate  # activate

# Alternatively, you may use virtualenvwrapper
# It creates and activates a virtual environment
$ mkvirtualenv venv

# Install
(venv)$ pip3 install africastalking
```

It is always recommended to do the installation in an active virtual environment primarily to isolate the needs of one application from another.


### Putting It Together

Now that the airtime module is in place, we can now show some appreciation to the customer.

```python
# routes.py

from app import app
from app.airtime import airtime

@app.route('/thank-you')
def thank_you():
  airtime()
  return render_template("index.html", title="Thanks")
```

### Notes

The sandbox app is ideal for testing. You should be able to see the necessary output in your terminal due to the use of the `print()` statements. However, the recipient identified by the current user's phone number will not receive the airtime top-up. You will need to switch to a "live" app, one credited with cash, and enabled by Africa's Talking team. To get started with the "live" app,

- Return to the [home](https://medium.com/r/?url=https%3A%2F%2Faccount.africastalking.com%2F) page

    ![Home](/images/africas_talking/airtime/home.png)

- Click on any of the existing teams or create a new team.

    ![Team](/images/africas_talking/airtime/team.png)

- Click on the "Create App" button to create a new app.

    ![Create app](/images/africas_talking/airtime/create_app.png)

- Your app will be listed in the new team.

    ![App list](/images/africas_talking/airtime/app_list.png)

- Click on the app name (in my case "test_test").

    ![App](/images/africas_talking/airtime/app.png)

- The app currently has KES 0.00 balance in the wallet. As much as the script may work, the recipient will not get any airtime until there is some money in her wallet. So, be sure to top up the wallet. Head over to "Billing > Payment Methods" on the left sidebar menu. Select your preferred method, and follow the steps.

- The last step would be to enable the Airtime service. This is done by Africa's Talking itself. Send an email to airtime@africastalking.com, notifying them of your intention to use the service. You will be asked to provide a few information for verification. If you are a Kenyan, you may be asked to provide a copy of your National ID card too.

- Hoping all goes well, and you should be good to go. You can test out the script.

### A Working Example

If you would like to see an example of a project utilizing the service, check out [sending-free-airtime-after-purchase](https://medium.com/r/?url=https%3A%2F%2Fgithub.com%2FGitauHarrison%2Fsending-free-airtime-after-purchase) on GitHub.
