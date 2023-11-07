# Create Twilio SendGrid Account


Is it your first time implementing email support in your Flask app? If so, start [here](/email_support_in_flask.md). If not, check out the other parts of the Twilio SendGrid series below:

- [SendGrid Overview](/twilio_sendgrid/00_overview.md)
- [Create A Twilio SendGrid Account](/twilio_sendgrid/01_create_acccount.md) (this article)
- [Send Emails Using Twilio SendGrid](/twilio_sendgrid/02_send_emails_using_sendgrid.md)
- [Receive Emails Using Twilio SendGrid](/twilio_sendgrid/03_receive_emails_using_sendgrid.md)
- [Verify An Email Address Using Twilio Sendgrid](/twilio_sendgrid/04_email_verification.md)



## Create Account

To configure an email verification solution, you have to connect your Twilio and SendGrid accounts. [Click here to create a free account](https://signup.sendgrid.com/), which allows you to send up to 100 emails per day.

![Create Account](/images/sendgrid/create_account/create_account.png)

Notice that you can use your email address as your username. Alternatively, you can uncheck the box "User email address as username" to provide your own username.

In the event you fail to use the account for some time and it becomes inactive, you will not be able to utilize the service. You will need to create another account.

- If you created your first account using your email address as your username (name@email.com), then a new account can be created by providing a new username (name).
- Uncheck the box "User email address as username"
- You can use the same email address (name@email.com) when creating this new account.
![Create a new account again](/images/sendgrid/create_account/new_account_again.png)
- You will be asked to provide more details about yourself
![About yourself](/images/sendgrid/create_account/your_info.png)
- Once the details are sent, you will be asked to provide further information by contacting the [Support team](support@twilio.zendesk.com).
![Contact Support](/images/sendgrid/create_account/contact_support.png)
- They would want to know more about you and your intention for using the service.


## Log In

Once approved, you can proceed to [log into your new account](https://app.sendgrid.com/login). You may have to set up your account a little, so follow the flow of post-registration instructions such as setting up two-factor authentication.

![Login](/images/sendgrid/create_account/login.png)

You will be asked to futher set up a few things to get started.


## Create A Single Sender

The first thing to do is to create a Single Sender Identity. This is done by clicking the large blue button "Create a Single Sender" as seen below.

![Create Single Sender](/images/sendgrid/create_account/create_sender_id.png)

You will be required to include your contact information.

![Create Sender](/images/sendgrid/create_account/create_sender.png)

Click **Create** to create your sender. An verification email will be sent to your inbox. Once verified, you are all set.