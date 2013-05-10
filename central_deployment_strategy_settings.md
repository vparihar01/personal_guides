Initial Server Setup

The first thing you will need on our server is a user account on the server which will be responsible for providing a container for your application’s install.

A user account -:
Every time we have to create a deploy user on remote server . Which is responsible for all application's install and running.

To set up this new user, run these commands on the server:

useradd -d /home/deploy -m -s /bin/bash deploy
passwd deploy

Set a new password for the user and remember it, as you will require it in just a moment.


Key-based authentication
The next thing to set up is secure key-based authentication on the server. This will involve setting up a private key on your central deployment server i.e deploy.prometheussports.com, copying over the related public key to the server, asserting that you can now login without providing a password, and then disabling password authentication on the server to increase security.

On the remote server, set up an .ssh directory to contain the new public key for a user by running these commands:
mkdir /home/deploy/.ssh
chown deploy:deploy /home/deploy/.ssh
chmod 700 /home/deploy/.ssh

This directory is used to authenticate key-based authentication when using SSH.

On your server_name.com server, use www-data user id_dsa  private key.

For this onserver_name.com server do like this -:

root@git:~#
root@git:~#su www-data


Now you will need to copy the public version of this key over to the new server. To do this, run this command:
scp .ssh/id_dsa.pub deplot@[your server's address]:/home/deploy/.ssh/authorized_keys

Once you’ve set this up, you will then be able to setup project in webistrano with user deploy in it for suceesfull and hassel free deployment:
ssh deploy@[your server's address] -i [your home directory]/.ssh/deploy