# Connecting to GitHub with SSH

#### What is SSH?

You can connect and authenticate to remote serves such as GitHub without supplying your _username_ and _password_ during each visit. It is recommended that you review your `SSH` key regularly and revoke any that may be invalid or compromised, just as you would your email account.
<br>

#### Check for existing `SSH` Keys

```python
$ ls -al ~/.ssh
```

This lists all your available `SSH` keys. Check to see if you have an public `SSH` key. The file naming, by default, could be as follows:

```
id_rsa.pub
```
<br>
#### Generate a  new `SSH` key

If you have none listed from the step above, you may want to generate a public and a private key pair:

```python
$ ssh-keygen -t rsa -b 4096 -C "<your GitHub email address goes here>"
```
This will create a new `ssh` key using the email you have provided. When prompted to enter a file to save your `ssh` key, just press `Enter` to save it in the default file location.

```python
S Enter a file in which to save the key (/home/you/.ssh/id_rsa): [Press enter]
```
Type a secure _passphrase_ when prompted.
<br>

#### Adding your `SSH` key to the ssh-agent

In the background, you can start your ssh-agent as below:

```python
$ eval "$(ssh-agent -s)"
```

```python
$ ssh-add ~/.ssh/id_rsa
```

`id_rsa` is the file that holds your key. Check how to list your files above. This file is hidden hence the use of `~/.`
<br>
#### Adding your `SSH` key to your GitHub account

Copy your `ssh` key to your clipboard [ctrl + shift + C from your terminal]. **Make sure not to add newlines or whitespaces while you copy to the clipboard** as  shown below:

```python
$ sudo apt install xclip
```
This downloads and install xclip.

```python
$ xclip -sel clip < ~/.ssh/id_rsa.pub
```
This copies the content of the _id_rsa.pub_ file to your clipboard. You can substitute the file name to match the exact name in your system.

* Log into your GitHub account. Click on settings as shown in the image below:<br>

![GitHub Settings](/images/github_settings.png)
<br>

* Click on _SSH and GPG keys_ from the left menu with _Settings_<br>

![SSH and GPG keys](/images/ssh_gpa_keys.png)
<br>
* Click the green _New SSH key_ or _Add SSH key_ button<br>

![New SSH key](/images/new_ssh.png)
<br>

### Changing remote's URL

If you look at one of your GitHub repository, you will see a green _Code_ button . Click on the drop down menu to locate `HTTPS`, `SSH` and `GitHub CLI`
![Code](/images/https_ssh.png)

* If you are updating to `https`, your URL will look as below:
```python
https://github.com/GitauHarrison/work_gossip_chat_app.git
```
* If you are updating to `ssh`, your URL will look as below:
```python
git@github.com:GitauHarrison/work_gossip_chat_app.git
```
* If you are updating to `GitHub CLI`, your URL will look as below:

```python
gh repo clone GitauHarrison/work_gossip_chat_app 
```
<br>

#### Testing your `SSH` connection

```python
$ ssh -T git@github.com
```
This command will attempt to `ssh` to GitHub. You may see a warning. Verify the fingerprint  and then type _yes_.

Verify that the resulting message contains your _username_.
<br>

#### Working with `ssh` passphrases

To add extra layer of security, you will need a passphrase to your `ssh`. The reason behind this is because whenever someone gains access to your computer, they also gain access to every system that uses that key. `ssh-agents` are used to securely save your passphrases so you don't have to enter it all over again.

To add or change your passphrases for your existing private key without generating a new keypair, run the command below:

```python
$ ssh-keygen -p
```
Press `Enter` to save your file in the default location, type in your new passphrase and press `Enter` again.
<br>

### Additional Notes:

###### Switching remote URL from `SSH` to `HTTPS`

As ealier noted, whenever you use `https`, you will be prompted to provide your GitHub _username_ and _password_

1. List all your remotes to know what remote name you want to change:
```python
$ git remote -V
```

2. Change to `HTTPS`:
```python
$ git remote set-url origin https://github.com/GitauHarrison/work_gossip_chat_app.git
```
Replace `https://github.com/GitauHarrison/work_gossip_chat_app.git` with your own repo link.

3. Verify remote URL has changed:
```python
$ git remote -V
```

###### Switching remote URL from `HTTPS` to `SSH`

The commands needed for this are similar as those above. Run each at a time:

```python
$ git remote -v
$ git remote set-url origin git@github.com:GitauHarrison/work_gossip_chat_app.git
$ git remote -v
```
