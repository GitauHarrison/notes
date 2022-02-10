# Secure Your Domain With SSL

With my application up and running on my server, and with a domain name, I will now secure my domain with SSL. For your reference, these are the topics in our discussion throughout this series:

1. [Deploy your flask app on Linode](/linode/deploy_on_linode.md)
2. [Buy a domain name for your deployed application](/linode/buy_domain.md)
3. [Secure your domain with SSL](/linode/secure_domain_with_ssl.md)

## Table of Contents

This tutorial is divided into the following sections:

- [SSL Overview](#ssl-overview)
- [Working With Let's Encrypt](#working-with-lets-encrypt)

I have noted that next to my domain name on a browser, there is "Not Secure" badge. Clicking on this badge reveals a message that says "Your connection to this site is not secure." 


![Not secure badge](/images/linode/ssl/not_secure_badge.png)

The URL of my application currently is http://www.bolderlearner.com/. What I would like to do is to make my site secure by enabling HTTPS with a free certificate using a service called [Let's Encrypt](https://letsencrypt.org/). To understand the difference between HTTP and HTTPS, let us go over some basics. 

## SSL Overview

In full, SSL stands for Secure Sockets Layer. It is a standard technology used to keep an internet connection secure by safeguarding any sensitive data that is being sent between two systems, such as a server (Linode) and a client (brave browser). It uses encryption algorithms to scramble data in transit. An updated version of the SSL is called the TLS (Transport Layer Security).

HTTPS (Hypertext Transfer Protocol Secure) is a protocol that uses SSL to encrypt the data sent between the client and the server. It is a standard protocol that is used to securely transfer data between a client and a server. The principal motivations for using HTTPS are to protect the privacy of the data and to prevent eavesdropping. It protects against man-in-the-middle-attacks. 

The authentication aspect of HTTPS typically requires a trusted third party to sign server-side digital certificates. We still refer to HTTPS using SSL rather than TLS because the term is more commonly used. The details of the certificate, including the issuing authority and the corporate name of the website owner, can be viewed by clicking on the lock symbol on the browser bar.

## Workign With Let's Encrypt

This is a non-profit organization that is authorized to issue digital certificates. To get started, paste the following URL into your browser to access their page: https://letsencrypt.org/



![Let's Encrypt](/images/linode/ssl/lets_encrypt.png)

Click the "Get Started" button to learn how you can get a free SSL certificate. Typically, you will need to SSH into your server to run the commands needed to get a certificate. Since I have shell access, I will use the [Certbot](https://certbot.eff.org/) ACME command line tool to get a free SSL certificate.

![Cerbot commands](/images/linode/ssl/certbot_commands.gif)

Once on https://certbot.eff.org/, I will identify that my HTTP website is running Nginx on Ubuntu 20. Filling this will provide me several commands that I need to run to get a free SSL certificate. To use certbot, I will need these things:

- Comfort with the command line interface
- A HTTP website what is already online with an open port (80)
- Application hosted on a server that can be accessed via SSH with the ability to run sudo commands

