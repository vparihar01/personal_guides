Multiple SSH Keys settings for different github account
=================================================================

Github using SSH client connection. If it is a single-user (first), generate a key pair, the public key saved to github, each time you connect the SSH client sends the local private key (the default ~~ / .ssh / id_rsa) to the server authentication. In the case of single-user public key SSH client send the private key stored on the server connection is naturally paired. But if it is multi-user (first, second), we connected to the second account, the second to save the public key, but SSH client still send the default private key, that is, the first private key, then verify that naturally can not through. However, to achieve multiple accounts under the SSH key to switch on the client to do some configuration can be.

Step 1: Go to SSH key dir
---------------------------------
First, cd to ~ / ssh to use `ssh-keygen -t rsa -C 'your_mail@youremail.com'` to generate new SSH key
You are then prompted for an optional password. After the key is generate you copy & paste it into your GitHub account settings.

Create different public key
---------------------------------

Step 2: Generate a First SSH key
---------------------------------
Create first user with
	$ ssh-keygen -t rsa -C "email@domain.com"
	# Creates a new ssh key, using the provided email as a label

	# Generating public/private rsa key pair.
	# Enter file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]

Which should give you something like this:

	# Your identification has been saved in /Users/you/.ssh/id_rsa.
	# Your public key has been saved in /Users/you/.ssh/id_rsa.pub.
	# The key fingerprint is:
	# 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db email@domain.com

You are then prompted for an optional password. After the key is generate you copy & paste it into your GitHub account settings.

Step 3: Generate a Second SSH key
---------------------------------
Create second user with
	$ ssh-keygen -t rsa -f ~/.ssh/second_rsa -C "email@domain.com"
	# Creates a new ssh key, using the provided email as a label

	# Generating public/private rsa key pair.
	# Enter file in which to save the key (/Users/you/.ssh/second_rsa): [Press enter]

Which should give you something like this:

	# Your identification has been saved in /Users/you/.ssh/second_rsa.
	# Your public key has been saved in /Users/you/.ssh/second_rsa.pub.
	# The key fingerprint is:
	# 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db email@domain.com

Step 4: Add Saved SSK keys
---------------------------------
After Step 3: two SSH keys created at `~/.ssh`:
```    
	~/.ssh/id_rsa
	~/.ssh/second_rsa

```
Then, add these two keys as following
```
	$ ssh-add ~/.ssh/id_rsa
	$ ssh-add ~/.ssh/second_rsa
```
You can delete all cached keys before
```
	$ ssh-add -D
```	
Step 5: Finally, you can check your saved keys
---------------------------------
```
	$ ssh-add -l
```

Step 6: Modify the ssh config
---------------------------------
```
	$ cd ~/.ssh/
	$ touch config
	$ subl -a config
```
Then add below lines into `config` file
```
	#First account
	Host github.com
		HostName github.com
		User git
		IdentityFile ~/.ssh/id_rsa

	#Second account
	Host github.com-second
		HostName github.com
		User git
		IdentityFile ~/.ssh/second_rsa
```		
Step 7: Test everything out
---------------------------------
To make sure everything is working you'll now SSH to GitHub. When you do this, you will be asked to authenticate this action using your password, which for this purpose is the passphrase you created earlier. Don't change the git@github.com part. That's supposed to be there.
```	
	For first SSH key do this:

		ssh -T git@github.com
		# Attempts to ssh to github using first ssh key
 
		You may see this warning:

		# The authenticity of host 'github.com (207.97.227.239)' can't be established.
		# RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
		# Are you sure you want to continue connecting (yes/no)?
		Don't worry, this is supposed to happen. Verify that the fingerprint matches the one here and type "yes".

		# Hi username! You've successfully authenticated, but GitHub does not
		# provide shell access.
		
	For Second SSH key do this:

		ssh -T git@github.com-second
		# Attempts to ssh to github using second ssh key

		You may see this warning:

		# The authenticity of host 'github.com (207.97.227.239)' can't be established.
		# RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
		# Are you sure you want to continue connecting (yes/no)?
		Don't worry, this is supposed to happen. Verify that the fingerprint matches the one here and type "yes".

		# Hi username! You've successfully authenticated, but GitHub does not
		provide shell access.
```	


Step 8: Clone your repo and modify your Git config
---------------------------------------------
With this set up I can clone with my default key as Github suggests:

	Clone your repo
		`git clone git@github.com:username/project.git`

If I want to clone a repository from my second account I can alter the command to use the second SSH key I generated:
	
	Clone your repo
		`git clone git@github.com-second:username/project.git`


	Cd project and modify git config
```
		$ git config user.name "First User"
		$ git config user.email "first_email@email.com" 
 
		$ git config user.name "Second User"
		$ git config user.email "second_email@mail.com" 
```
	Or you can have global git config
		```$ git config --global user.name "First User"
		$ git config --global user.email "email@email.com"```


Then use normal flow to push your code

```	$ git add .
	$ git commit -m "your comments"
	$ git push
```

In fact, if I wanted to I could have a different SSH key for every account I have; GitHub, Bitbucket, or any other service that requires one.