# Deploy to Linode Servers

During this tutorial, I will walk you through the steps you can follow to deploy your flask application on Linode servers. For your reference, these are the topics I am going to discuss:

1. [Deploy your flask app on Linode](/linode/deploy_on_linode.md)
2. [Buy a domain name for your deployed application](/linode/buy_domain.md)
3. [Secure your domain with SSL](/linode/secure_domain_with_ssl.md)


## Overview


The assumption is that you have already built an application. However, you probably already know that a locally running app is only accessible to you and no one else. There is a tool such as `ngrok` that you can use to create a tunnel and provision a publicly accessible URL. However, we would like to make our application accessible to anyone who has internet through a browser such as Brave or Google Chrome.

For the purposes of this tutorial, I will be hosting an elearning application built using Python and the Flask micro-framework. You can access the project on [this GitHub repository](https://github.com/GitauHarrison/somasoma-eLearning-app) and use it to follow along. Alternatively, if you have built one for yourself, ensure that you have it on GitHub, or an alterntive version control system.


## Create Linode Account


When you deploy to a Linux server, you are are going to be using a virtual machine hosted by your provider, in this case it is Linode. To begin, ensure that you have created an account on Linode. If you haven't already, [click here](https://login.linode.com/signup) to sign up. Occassionally, Linode will offer you free hosting credits as soon as you sign up. So, be sure to check that out and take advantage of it.


## Create a Linode

Deploying to a Linux server takes a lot of effort to set up, but this method offers you a lot of flexibility on the kind of control you will have over your application and web server. Free tier hosting options normally offer only basic functionalities, but should you desire to have more control, say you want to use a custom domain name like my-app.com rather than something such as my-app.herokuapp.com, you will be required to upgrade to a paid version. Even so, you may not have all the control you might want.

I already have an account with Linode, but I will still show you how you can get started from scratch. Below, you will see that I already have one server called _official_personal_website_.

![Linode Dashboard](/images/linode/sample_linode.png)

To create you own server, do the following:

- Click on the Create button on the top-left or the top-right of your dashboard.
- Select "Linode", which is the name of their Linux server
- Choose the "Ubuntu" image (your Operating System)
- Select your region (it is best to choose one that is closest to you)
- Decide on what plan you'd prefer. For the purposes of this tutorial, I will choose Nanode 1 GB, a low performance machine and the cheapest since I do not anticipate so much traffic and activity to go on in the application.
- Give a descriptive label for your linode, say _bolderlearner_ since this is going to be an elearning application.
- Provide a stronger root password for your own server. You need to remember this as you will need it to log into the system.
- Optionally, you can choose to back up your server if it is for production
- Finally, click the "Create Linode" button on the far right to create your server

![New linode](/images/linode/new_linode.gif)

Once the new linode has finished provisioning, click on it. You will see how you can access your machine using the SSH command. In the case of _bolderlearner_ linode, the SSH Access command is `ssh root@139.162.221.92`.


## Log Into Your Virtual Machine

You will see me use different terminal windows moving forward. If you are a bit curious as to how I am able to do this, then I recommend that you look at [byobu](https://www.byobu.org/), a text-based window manager and terminal multiplexer. Alternatively, if you are using [VS Code](https://code.visualstudio.com/), you can simply do the following:

- Press **ctrl + shift + `** to open a new terminal window
- Press **ctrl + shift + 5** to split the terminal window

The reason for having the split terminal windows is to allow me to run commands on my local machine and commands on linode server separately. It is just so much easier working like this rather than having to go back and forth between my local machine and my linode server.

![Terminal windows](/images/linode/terminal_windows.png)

The window on the left will be my linode server while the one on the right will be my local machine's. I have copied and pasted my linode's root SSH command in the left terminal window to hightlight this.

Copy and paste your SSH Access code in one of the terminal windows to access our linode server. I will paste mine as follows:

```python
$ ssh root@139.162.221.92

# Output

The authenticity of host '139.162.221.92 (139.162.221.92)' can't be established.                      │
ECDSA key fingerprint is SHA256:bBMaK2zHPang8gz2GC/cI/F/Jlo/h6iumhBWYILtcKk.                          │
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type "yes" to continue your connection. When prompted, key in your linode's root password which you had created in your dashboard. If successful, you will be connected to your linode's server. Your terminal will change from `harry@harry:~$` to `root@localhost:~#`. _You are now working with a barebones Ubuntu server_. 
