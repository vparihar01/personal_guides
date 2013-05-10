# Adding user as admin on server
Adding user to server and giving the admin previleges to that user

## Add a User
To add a user, issue the following command and replace with **new_user** the user name of your choice:
```
   adduser new_user
```

> Note: After this initial step,you'll need Super User (sudo) privileges to complete these administrative/simple tasks.

## Assign User sudo priviledges
To assign the **new_user** user sudo privileges, issue the following command, which invokes the nano editor by default on Ubuntu:
```
visudo
```
At the end of the file add your administravie user name (in place of our example **new_user** user below) and the following text string:
```
new_user   ALL=(ALL) ALL
``` 
When you are finished adding this line, exit, confirm, and save the file, using these commands:
* Press the key combination Ctrl-X to exit.

* Press y to confirm the changes.

* Press the Enter key to save the file as `/etc/sudoers.tmp`.


# PasswordLess Login for All Users who access the server.
Now next step is to not let every user to access the server using username and password

Set Up Public and Private Keys (SSH keygen) -: From Cloud Documentation
One effective way of securing SSH access to your Cloud Server is to use a public/private key. This means that a public key is placed on the server and the private key is on your local workstation. This makes it impossible for someone to log in using just a password - they must have the private key. This consists of 3 basic steps: create the key on your local workstation, copy the public key to the Cloud Server, and set the correct permissions for the key. 

The following instructions assume you use Linux or OS X. For Windows instructions, see Key generation using Putty for Windows.

## Step 1. Create the Public and Private Keys

Create a folder to hold your keys on your LOCAL workstation:
```
mkdir ~/.ssh
```
To create the ssh keys, on your local workstation enter:
```
ssh-keygen -t rsa
```
> If you do not want a passphrase then just press enter when prompted.

The **id_rsa** and **id_rsa.pub** are created in the **.ssh** directory. The **id_rsa.pub** file holds the public key. You'll place this file on you server.

The **id_rsa** file is your private key. Never show, give away, or keep this file on a public computer.

## Step 2. Copy the Public Key

You can use the scp command to place the public key your server. 

While still on your local computer enter this command, substituting your admin user for "demo" below:
```
scp ~/.ssh/id_rsa.pub demo@123.45.67.890:/home/demo/
When prompted, enter the admin user password.
```
Change the IP address to your server and the location to your admin user's home directory (remember the admin user in this example is called demo).

### Step 3. Modify SSH Permissions

Create a directory on the admin user's home folder on your server called .ssh and move the pub key into it, as shown in the following examples:
```
mkdir /home/demo/.ssh
mv /home/demo/id_rsa.pub /home/demo/.ssh/authorized_keys
```
Set the correct permissions on the key using the following commands, changing the "demo" user and group to your admin user and group:
```
chown -R demo:demo /home/demo/.ssh
chmod 700 /home/demo/.ssh
chmod 600 /home/demo/.ssh/authorized_keys
```
Congratulations! You have now successfully created the key on your local computer, copied the public key to your Cloud Server, and set the correct permissions for the key. 

Modify the SSH Configuration
Keeping the SSH service on the default port of 22 makes it an easy target. We recommend changing the default SSH configuration to make it more secure. 

Issue the following command:
```
nano /etc/ssh/sshd_config
Modify, check, and add the following values: 
Port 22                           <--- change to a port of your choosing
 Protocol 2
 PermitRootLogin no
 PasswordAuthentication no
 UseDNS no
 AllowUsers demo
```
Change the default port of 22 to one of your choosing, turn off root logins, and define which users can log in.
> Note: The port number can be any integer between 1025 and 65536 (inclusive). Be sure to note the new port number and remember to avoid port conflicts if you later configure additional listening processes.

PasswordAuthentication has been turned off as we setup the public/private key earlier. If you intend to access your server from different computers, you may want to leave PasswordAuthentication set to yes. Only use the private key if the local computer is secure (i.e. don't put the private key on a work computer).

> Note that these settings are not enabled yet. First you should create a simple firewall using iptables before restarting ssh using the new port.

Restart ssh now.
```
service ssh restart
```