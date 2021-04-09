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

