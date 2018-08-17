# git-server-docker

This docker image was taken from another github repo, I just made some modifications to adapt some steps to run in Windows 10.

A lightweight Git Server Docker image built with Alpine Linux. Copy from [GitHub](https://github.com/jkarlosb/git-server-docker) and [Docker Hub](https://hub.docker.com/r/jkarlos/git-server-docker/)

!["image git server docker" "git server docker"](https://raw.githubusercontent.com/jkarlosb/git-server-docker/master/git-server-docker.jpg)

### How to build the images

	FOR WINDOWS USERS:
	-------------------------------
	Before to run the docker image created, convert the "start.sh" file to unix format, you can do this downloading the "dos2unix" tool from [Sourceforge](https://sourceforge.net/projects/dos2unix/)

	C:\<PATH_OF_YOUR_CODE>>dos2unix.exe start.sh

	Once the file was changed, you can build the images as follow

	$ docker build -t <USERNAME>/<TAG>:<VERSION> .

	USERNAME: Commonly your nickname or account name
	TAG: Commonly the component name, like gitserver or something like that
	VERSION: If you want to tag with a version put it, or else leave in blank, docker will put latest by default
	".": Don't forget the PATH, in this case I put "." (Dot) beacuse I'm in the repo directory

	$ docker build -t spcruzaley/gitserver .


### Basic Usage

How to run the container in port 2222 with two volumes: keys volume for public keys and repos volume for git repositories:

	FOR LINUX USERS:
	----------------------------
	$ docker run -d -p 2222:22 -v ~/git-server/keys:/git-server/keys -v ~/git-server/repos:/git-server/repos spcruzaley/gitserver

	FOR WINDOWS USERS:
	----------------------------
	C:\Users\salvador.perez> docker run -d -p 2222:22 -v c:/Users/salvador.perez/git-server/keys:/git-server/keys -v c:/Users/salvador.perez/git-server/repos:/git-server/repos --name gitserver spcruzaley/gitserver:latest

How to use a public key:

Get in as interactive way to docker image as follow:

	$ docker exec -it gitserver sh

Go to /home/git/.ssh directory and generate the keys as follow:

	/home/git/.ssh # ssh-keygen -t rsa

In the following steps just type "Enter" to set the default values, you will to see something like this:

	Generating public/private rsa key pair.
	Enter file in which to save the key (/root/.ssh/id_rsa):
	Created directory '/root/.ssh'.
	Enter passphrase (empty for no passphrase):
	Enter same passphrase again:
	Your identification has been saved in /root/.ssh/id_rsa.
	Your public key has been saved in /root/.ssh/id_rsa.pub.
	The key fingerprint is:
	SHA256:YxtaO0UYH3BF/A+tgnZrf6LiZ98A0LtuHs/9pbG0G6M root@4fae6831e15c
	The key's randomart image is:
	+---[RSA 2048]----+
	|        o.o+o    |
	|         = o.    |
	|        . + .. . |
	|         . . .o .|
	|        S ..o  + |
	|       + *o oo. .|
	|      . +. .oo* .|
	|         ...**.@o|
	|         ..BE+X+=|
	+----[SHA256]-----+

    Copy them to keys folder: 
	- From host: $ cp ~/.ssh/id_rsa.pub /git-server/keys
	- From remote: $ scp ~/.ssh/id_rsa.pub user@host:/git-server/keys
	You need restart the container when keys are updated:
	$ docker restart <container-id>
	
How to check that container works (you must to have a key):

	$ ssh git@<ip-docker-server> -p 2222
	...
	Welcome to git-server-docker!
	You've successfully authenticated, but I do not
	provide interactive shell access.
	...

NOTE: If you have error messages like this:

	Permission denied (publickey,keyboard-interactive).

Try to make the following change (But this change will cause each time that yout execute a git command
the prompt will require you the git password):

	In server host go to
	$ cd /etc/ssh

Once there, open the file "sshd_config" and change

	PasswordAuthentication no

to

	By this --> PasswordAuthentication yes

Restart the server and try again

How to create a new repo (From Server as root):

	# cd /git-server/repos/
	# mkdir test-repo.git
	# cd test-repo.git
	# git init --bare --shared=true

How to use the repo (From client):

	$ mkdir test-repo
	$ cd test-repo

Let's to initialize a repo

	$ git init

Add files

	$ touch file.txt
	$ git add .
	$ git commit -m "File added"

Now we'll to add the remote server to our local repository

	$ git remote add origin ssh://git@ip.to.the.server:2222/git-server/repos/test-repo.git
	$ git push origin master
	git@ip.to.the.server's password:
	Counting objects: 1, done.
	Writing objects: 100% (1/1), 218 bytes | 218.00 KiB/s, done.
	Total 1 (delta 0), reused 0 (delta 0)
	To ssh://ip.to.the.server:2222/git-server/repos/test-repo.git
	* [new branch]      master -> master

How clone a repository:

	$ git clone ssh://git@<ip-docker-server>:2222/git-server/repos/myrepo.git

### Arguments

* **Expose ports**: 22
* **Volumes**:
 * */git-server/keys*: Volume to store the users public keys
 * */git-server/repos*: Volume to store the repositories

### SSH Keys

How generate a pair keys in client machine:

	$ ssh-keygen -t rsa

How upload quickly a public key to host volume:

	$ scp ~/.ssh/id_rsa.pub user@host:~/git-server/keys

### Build Image

How to make the image:

	$ docker build -t git-server-docker .
	
### Docker-Compose

You can edit docker-compose.yml and run this container with docker-compose:

	$ docker-compose up -d
