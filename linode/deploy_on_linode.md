# Deploy to Linode Servers

During this tutorial, I will walk you through the steps you can follow to deploy your flask application on Linode servers. For your reference, these are the topics I am going to discuss:

1. [Deploy your flask app on Linode](/linode/deploy_on_linode.md)
2. [Buy a domain name for your deployed application](/linode/buy_domain.md)
3. [Secure your domain with SSL](/linode/secure_domain_with_ssl.md)

## Table of Contents

If you would like to skip to a particular section within this tutorial, you can click on any of these links.

- [Overview](#overview)
- [Create a linode account](#create-a-linode-account)
- [Create a linode](#create-a-linode)
- [Log into your virtual machine](#log-into-your-virtual-machine)
- [Set up your linux server](#set-up-your-linux-server)
- [Add application on the linux server](#add-application-on-the-linux-server)

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

Copy and paste your SSH Access code in one of the terminal windows to access your linode server. I will paste mine as follows:

```python
$ ssh root@139.162.221.92

# Output

The authenticity of host '139.162.221.92 (139.162.221.92)' can't be established.                      │
ECDSA key fingerprint is SHA256:bBMaK2zHPang8gz2GC/cI/F/Jlo/h6iumhBWYILtcKk.                          │
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type "yes" to continue your connection. When prompted, key in your linode's root password which you had created in your dashboard. If successful, you will be connected to your linode's server. Your terminal will change from `harry@harry:~$` to `root@localhost:~#`. _You are now working with a barebones Ubuntu server_. 


## Set Up Your Linux Server

As is normally the case with your local machine's server, we need to set up a few things in our new linode server:

- Upgrade the software the first time you log into any linux system
    <br>
    ```python
    root@localhost:~# apt update && apt upgrade

    # Patiently wait for the update and upgrade to complete since it may take several minutes to finish
    ```
<br>

- Create a host name in this machine:
    <br>
    ```python
    root@localhost:~# hostnamectl set-hostname bolderlearner
    ```

    - To test that the host name has actually been set, run:
        ```python
        root@localhost:~# hostname

        # Output
        bolderlearner
        ```
<br>

- Set your hostname in a host file:
    <br>

    ```python
    root@localhost:~# nano /etc/host
    ```
    - This will open the file `host` in the `nano` editor. If you are not familiar with this editor, consider searching it online to learn more. However, for this tutorial, there are only a few commands that we shall be using while working with the `nano` editor. 
    
    - You can use the arrow keys to move around. Begin by adding these lines in the `/etc/host` file which is currently opened in your `nano` editor:
        <br>
        ```python
        127.0.0.1         localhost
        139.162.221.92    bolder learner
        ```
    - The first is the IP address of the localhost whereas the second is the IP address of my linode server as seen in the dashboard.

    - To save this file, hit `ctrl + x`. It will ask if you want to save this file. Type in "y" and press `Enter`. You will have saved this file upon exiting.
    <br>
    ![Host file](/images/linode/etc_host_file.png)
<br>

- Create a limited user. 
At the moment, we are logged in as the root user, who has unlimited priviledges and can execute any command. However, it is best practice to add a user who has limited priviledges. This user will still be able to run admin commands using `sudo`, which is much safer than running everything as root. Run this command to add a new user:
    <br>
    ```python
    root@localhost:~# adduser gitauharrison
    ```
    - You will be prompted to create a new password for this user. Please do so. Ensure that you can remember that password. You will be asked a few more questions such as "Full Name" etc. You can fill that up or leave them blank. This step is optional.
    <br>
    ![Add user](/images/linode/adduser.png)
<br>

- Add the user to a sudo group so that they can run admin commands
    <br>
    ```python
    root@localhost:~# adduser gitauharrison sudo

    # Output
    Adding user `gitauharrison' to group `sudo' ...
    Adding user gitauharrison to group sudo
    Done.
    ```
<br>

- Logout of root user and log in as `gitauharrison` user. It is just best practice to work as a limited user. Run:
    <br>
    ```python
    root@localhost:~# exit

    # Output
    logout
    Connection to 139.162.221.92 closed.
    ```

    - This will log you out as a `root` user. Notice that I am taken back to my local machine.
    <br>
    ![Logout root user](/images/linode/logout_root_user.png)
    <br>

    - Log back in as user `gitauharrison`. To do this, simply press the up arrow key on your keyboard to access your linode's SSH Access command. Mine was `ssh root@139.162.221.92`. Instead of hitting `Enter` just yet, I will replace `root` in the SSH Access command with `gitauharrison`. Now, the new command that I need to run to start my _bolderlearner_ linode server will be:
        <br>
        ```python
        $ ssh gitauharrison@139.162.221.92
        ```
    - Hit `Enter`. You will be required to key in your `gitauharrison` password that we had set earlier. Do so and you will notice that you will be logged in as your new user. For me, this is `gitauharrison@bolderlearner:~$`
<br>

- Set up SSH Authentication. This will allow me to log into my server without having to supply my password every time. This is quite convinient.
    - Make a `.ssh` directory from the home directory.
        <br>
        ```python
        gitauharrison@bolderlearner:~$ mkdir .ssh
        ```
    - If you are not sure if your are in your home directory or not, simply run the command `pwd` in the terminal. `pwd` stands for `print working directory`. Another way to find out if you are in your home directory is to look at your terminal. If you can see a tilde character (`~`), then you are in your home directory.
    <br>

    - __Moving to your other terminal window running your local machine__, run the following command:
        <br>
        ```python
        $ ssh-keygen -b 4096 
        ``` 
        - You will be asked to create a password. You can add one if you want more security, but again, you can still leave it blank. This is okay. You will also be asked to overwrite an existing `id_rsa` file. Type "y" for yes to continue.
        <br>
        - Notice that you will have two files created, _id_rsa_ and _id_rsa.pub_ which is our public key. We will need to move our public key to our linode server. To do this, run the `scp` (secure copy) command in your terminal:
            <br>
            ```python
            $ scp ~/.ssh/id_rsa.pub gitauharrison@139.162.221.92:~/.ssh/authorized_keys
            ```
            - Here, we are securely copying the _id_rsa.pub_ file to our new linode user. I am using the colon punctuation mark to specify what location I want the file to be copied to. I will save the contents of _id_rsa.pub_ in a file called _authorized_keys_ found in the hidden `.ssh` folder.
        <br>
    - __Moving back to the new linode user terminal window__, let us test to see if the new _id_rsa.pub_ key has been copied. Run the `ls` command to list all the files found in `.ssh` folder:
        <br>
        ```python
        gitauharrison@bolderlearner:~$ ls .ssh/

        # Output
        authorized_keys
        ```
<br>

- Update SSH permissions where the owner of the directory has  read, write and execute permissions on the directory and read and write permissions on the files. 
    <br>
    ```python
    gitauharrison@bolderlearner:~$ sudo chmod 700 ~/.ssh/
    ```

    - You will be required to provide your user password. Here, we are updating the permissions on the `ssh` folder.
    <br>
    ```python
    gitauharrison@bolderlearner:~$ sudo chmod 600 ~/.ssh/*
    ```

    - Here, we are updating the permissions on all files found inside `.ssh` folder.

    - In Linux, the `chmod` command is used to change permissions of files and directories. The first digit in the command `chmod 700`, that is number 7, define the permissions for the owner/user of that file or folder which is basically means the user has _read_, _write_ and _execute_ rights. The second digit, 0, means no permissions, and it applies to the group of that directory. The last digit, 0, applies to everyone else. The second command `chmod 600` applies the 6 hundred permissions to the owner, the group and everyone else on the files found in the `.ssh` folder.

    - Now, if I exit the server by running `exit` in the terminal, and hit `Enter` again after pasting `ssh gitauharrison@139.162.221.92`, I should be able to SSH into the virtual machine without a password.
<br>

- Disallow root logins over SSH. 

    - As the new user, update the SSH Config file. 
        <br>
        ```python
        gitauharrison@bolderlearner:~$ sudo nano /etc/ssh/sshd_config
        ```

        - This will open the SSH configuration file in the `nano` editor. There are two values that we need to change in this configuration file. Using the down arrow key on your keyboard, scroll down until you can see:

            1. `PermitRootLogin`. Currently, it is set to "yes". Change it to "no" by deleting the existing value and writing "no". We have a limited user who now has `sudo` access and, therefore, there is no need to log into my machine as `root`. 

            2. `PasswordAuthentication`. It is most likely commented out so make sure you uncomment it. If it is not commented out, then simply change its value from "yes" to "no". With SSH setup, there is really no need to log in using a password. 

        - Press `ctrl + x` to exit. On your way out, type "y" to save the file using its current name.
    <br>
    - Now, what we need to do is to restart the SSH service. Run the command below:
        <br>
        ```python
        gitauharrison@bolderlearner:~$ sudo systemctl restart sshd
        ```

    - If you would like to learn more about SSH Key-based authentication, [click here](/linode/ssh_key_based_authentication.md) to check out the SSH Authentication.
<br>

- Set up a firewall to monitor and control incoming and outgoing network traffic based on predetermined security rules. For your information, a firewall creates a barrier between a trusted network and an untrusted network. An example of an untrusted network can be the internet.
    <br>
    - Install `uncomplicated firewall`:
        <br>

        ```python
            gitauharrison@bolderlearner:~$ sudo apt install ufw
        ```
    
    - Add rules while working with `ufw`:
        <br>
        
        ```python
        gitauharrison@bolderlearner:~$ sudo ufw default allow outgoing

        gitauharrison@bolderlearner:~$ sudo ufw defualt deny incoming
        ```

    - We want to configure these allow rules to allow for SSH, HTTP and any other port that we may want to access from the outside of our server. Begin by running the commands below. I have provided a brief explanation of what each command does.
        <br>
        ```python
        gitauharrison@bolderlearner:~$ sudo ufw allow ssh 
        ```

        - This will allow me to SSH into our server, without which our firewall will prevent its use.
        <br>
        ```python
        gitauharrison@bolderlearner:~$ sudo ufw allow 5000
        ``` 
        
        - This will allow me to access port 5000 which will be used by our flask development server. Using this port will enable me to test my application before going live on a production server.
        <br>

        ```python
        gitauharrison@bolderlearner:~$ sudo ufw enable
        ```

        - The command above basically enables all that we have set for our SSH and port 5000. When prompted, type "y" to activate and enable firewall on system startup.
        <br>

        - To see the status of the things I have allowed, I can run the command below:
        <br>
        ```python
        gitauharrison@bolderlearner:~$ sudo ufw status

        # Output
        Status: active

        To                         Action      From
        --                         ------      ----
        22/tcp                     ALLOW       Anywhere                  
        5000                       ALLOW       Anywhere                  
        22/tcp (v6)                ALLOW       Anywhere (v6)             
        5000 (v6)                  ALLOW       Anywhere (v6)

        # Port 22 is SSH; there is port 5000 as well
        ```

## Add Application on the Linux Server