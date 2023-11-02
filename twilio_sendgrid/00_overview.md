# Twilio SendGrid

Before we can understand what Twilio SendGrid is, let us first know what [Twilio](https://www.twilio.com/try-twilio?promo=WNPWrR) is. If you are coming across Twilio for the first time, it is a popular customer engagement functionality provider that allows developers to build unique communication features. There have quite a number of communication services they offer through APIs such as SMS, Voice, Video and Authentication. Should developers want to integrate email communication functionalities, Twilio offers the SendGrid API. 

Twilio SendGrid enables developers to build platforms or services that increase customer engagement by ensuring that the delivering of emails is reliable even at scale, be it just 10 emails or 100,000 emails.


## Consider Using Twilio

Given your skill level, Twilio and it extensive set of API-driven services can quickly enable personalized communications and automated messaging in your system(s). It is this personalized touch that enhances customer engagement thereby driving marketing, sales and cusotmer service functions.

## Twilio Use Cases

You can utilize the Twilio Voice API to make calls. For example, when developing a ride-sharing platform such as Uber, you can integrate the "Call Driver/Rider" feature where your application makes an API call to Twilio to initiate and connect a call. 

TWilio can also be used to sent verificaiton messages to users of an application. Say you want users to sign up for your newsletters. Before such a user is saved to your email list, you may consider verifying them first where you require them to share a token number sent to them via email or SMS. Once verified, the application adds them to your database.

It is also possible to build a video conferencing platform, similar to Zoom, using their programmable Video APIs. You can develop a fully functional video app where real-time conversation is online.

## Twilio SendGrid

Twilio alleges that they process over 100 billion emails every month, from account creation to customer nurtures to purchase receipts. SendGrid not only offers the sending or emails, but you can use it to receive emails too. They utilize their proprietary mail transfer agent and infrastructure to enable users achieve optimal inbox placement while making the process of sending email scalable and flexible.

Trhoughout this series, you will learn how to utilize Twilio SendGrid to add email integration to your Flask application, especially on an authenticated domain. Below are the items we shall look at:

- [SendGrid Overview](/twilio_sendgrid/00_overview.md) (this article)
- [Add Email Support In Your Flask App](/email_support_in_flask.md)
- [Create A Twilio SendGrid Account](/twilio_sendgrid/01_create_acccount.md)
- [Send Emails Using Twilio SendGrid](/twilio_sendgrid/02_send_emails_using_sendgrid.md)
- [Receive Emails Using Twilio SendGrid](/twilio_sendgrid/03_receive_emails_using_sendgrid.md)
- [Verify An Email Address Using Twilio Sendgrid](/twilio_sendgrid/04_email_verification.md)