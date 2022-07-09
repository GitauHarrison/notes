# Download Encrypted PDF Copy of Database Data using Python and Flask


Now that your flask application is capable of storing user data in a database, a user may want to download a copy of that data. Partly inspired by the MPesa technology, you will learn how to download an encrypted copy of the database data.


## Background Story

> [M-Pesa](https://www.safaricom.co.ke/personal/m-pesa) is a mobile phone-based money transfer service, payments and micro-financing service, launched in 2007 by Vodafone and Safaricom, the largest mobile network operator in Kenya.

Safaricom allows its MPesa users to send and receive money from Safaricom and other mobile network users in Kenya. One feature that all MPesa, just like banks, users enjoy, is the provision of a transactions statement. An Mpesa user can request a copy of their transactions statement from the Safaricom MPesa app. On the web, a user can download a copy whereas on mobile, the user will receive a copy of the statement via email. See the image below.

![Mpesa statement](images/download_encrypted_pdf/mpesa_statement.png)

Before a user can open the document, they will be required to identify themselves, usually by providing their national identity number. As of this writing, Safaricom not only requires a user to provide their national ID number, but also a one-time token sent to their phones.

This tutorial does not intend to reproduce a clone of the MPesa statement, but rather to show how to download an encrypted copy of the database data. You will learn how to collect users' data, store it in the database, and then download an encrypted copy of that data.