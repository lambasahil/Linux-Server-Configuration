# Linux Server Configuration

### IP Adderss
> 54.197.7.212

> To `ssh` into th grader user with private key:
```
$ ssh grader@54.197.7.212 -i grader -p 2200
```

### External Resources Used:
1. **StackOverflow**
2. **DigitalOcean Tutorials**
3. **SQLAlchemy Documentation**

## Steps to configure your own server on Amazon AWS

#### 1. Create an Amazon AWS account.
Visit Amazon AWS [Home Page](https://aws.amazon.com) and click on `Sign In To Console` button to create your accout and sign in.
#### 2. Create an EC2 instance.
After successfully logging in, create a new EC2 instance that will be used as a server for our project. Click [here](https://docs.aws.amazon.com/efs/latest/ug/gs-step-one-create-ec2-resources.html) for instructions on setting up an EC2 instance.
#### 3. Create a new user.
After the instance is up and running, a new user is to be created. Every hacker knows that there is a default `ubuntu` user that can be used to log in to your server and get sudo access. To eleminate the possibility, a new user called `grader` is created so that no one knows that which user is acting as our server.

```
$ ssh ubuntu@54.197.7.212
$ sudo adduser grader
Adding user 'grader' ...
Adding new group 'grader' (1001) ...
Adding new user 'grader' (1001) with group `grader' ...
Creating home directory `/home/grader' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for temp1
Enter the new value, or press ENTER for the default
	Full Name []: grader
	Room Number []:
	Work Phone []:
	Home Phone []:
	Other []:
Is the information correct? [Y/n] y
```
#### 4. Give sudo access to the user
To configure the firewall, the user created should have the `sudo` access. The information about all the sudo users is in the `/etc/sudoers` file.

Whenever a system update occurs, there is a chance that this file gets overridden and your configuration is deleted. To avoid the case, we'll add our sudo user information in the `/etc/sudoers.d` directory.

Create a new file by `$ sudo nano /etc/sudoers.d/grader` command. Add the following line to set the new user as sudo.
```
grader ALL=(ALL) NOPASSWD:ALL
```
Save the file and try running a sudo command such as `$ sudo apt-get update`.

#### 5. Configure the firewall of the instance.
After the user has been created, the firewall has to be configured. Firewall is responsible for the security of the system. It controls which ports are open for the system.

Uncomplex Firewall `ufw` is such tool that is used to configure the firewall.

The `ssh` protocol by default works on port 22 and everyone knows it. To eleminate the case of remote login using `ssh`, the default port can be changed.

The configuration file for `ssh` lies in `/etc/ssh/sshd_config` file. Edit this file using `nano` or `vi` editor and find the line:
```
# Port 22
```
Change the port to 2200 and save the file.
Now, the `ssh` service needs to be restarted to apply the changes.
```
$ sudo service ssh restart
```
Initially the status of `ufw` is inactive. All the
configuration will be done before activating the `ufw`
```
$ sudo ufw status
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow ssh
$ sudo ufw allow 2200
$ sudo ufw allow http
$ sudo ufw allow nst
$ sudo ufw enable
```

#### 6. Key-Pair authentication
By default, all the users can be logged in using the password but it can be a problem if someone knows the password to the user. To avoid this, Key-pair authentication is used. In this, a Public and Private key is generated. The public key is kept on the server and the private key is with the user. When someone tries to log in using the private key, the contents are decrypted and matched with the public key. If the keys match, only then the user is logged in.

To enable such authenticaton, a key pair is generated on the local system.
```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/nikhil/.ssh/id_rsa): grader
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in grader.
Your public key has been saved in grader.pub.
The key fingerprint is:
SHA256:GUSf/NLCuUXcHd2nYzhpcBMHrowWWMXulGBX21WB4+E nikhil@nikhil-Inspiron-5559
The key's randomart image is:
+---[RSA 2048]----+
|       .+o..+oooB|
|       +oo++oO o=|
|      ..o+++B+=.o|
|         B+*=E+  |
|        SoB.+o . |
|       .  .=     |
|          .      |
|                 |
|                 |
+----[SHA256]-----+
```
Now, we have generated a private key `grader` and a public key `grader.pub`. To use a key to log in to the system, the contents of the public key are added to the `.ssh/authorized_keys` file so that the system knows which keys are authorized.

Log back into the server and add create a file `authorized_keys` in `.ssh` directory.
```
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```
Now the contents of the public key generated are to be pasted in the `authorized_keys` file. It can be done in 2 ways.

* **Manually copying the contents.**

  Open the public key file and copy the contents of the file and paste in the `authorized_keys` file.
* **ssh-id-copy command**

  `ssh-id-copy` command can be used to copy authomatically add the public key to your server.
```
$ ssh-copy-id -i grader.pub -p 2222 grader@54.197.7.212
```

After the public key has been placed on the server, the password login needs to be disabled so that key-pair based authentication is the only way of log in.

```
$ sudo nano /etc/ssh/sshd_config
```
Search for line:
```
PasswordAuthentication yes
```
and change it to `no`. After saving the file, the `ssh` service has to be restarted to apply the changes.
```
$ sudo service ssh restart
```
**Now you should be able to ssh into your aws machine using your private key**

```
$ ssh -i grader -p 2222 grader@54.197.7.212
```

### 7. Clone the project on server.
Now the server is secure of all attacks. Clone the project onto the server.
```
$ git clone https://github.com/nikhilbansal97/ItemCatalog.git
$ git branch aws
$ git checkout aws
```

### 8. Install Aapche2
Apache2 is a open source http server project that will be used to handle all the `http` requests to our server.

```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi
```

After `apache2` has been installed, it has to be configured to run with our flask application.
Create a directory structure as defined below in the `/var/www` directory.

* ItemCatalog
  * ItemCatalog
    * templates
    * static
    * index.html
    * project_server.py
  * project_server.wsgi

Add the following contents to the `wsgi` file.
```
import sys
sys.path.insert(0, '/var/www/ItemCatalog/ItemCatalog')

from project_server import app as application
application.secret_key = 'my_secret_key'
```
Now, this wsgi file needs to be enabled. Add the following lines to the `/etc/apache2/sites-enabled/000-default.conf` file.
```
WSGIDaemonProcess project_server
WSGIScriptAlias / /var/www/ItemCatalog/project_server.wsgi

<Directory ItemCatalog>
    WSGIProcessGroup project_server
    WSGIApplicationGroup %{GLOBAL}
    Order deny,allow
    Allow from all
</Directory>
```
**Restart the web server**
```
$ sudo apache2ctl restart
```
### 9. Install PostgreSQL

For the project, postgresql database will be used.
```
$ sudo apt-get install postgresql postgresql-contrib
```
Switch to the `postgresql` user and create a user and a new database.

```
$ su -i -u postgres
postgres@: createuser nikhil
postgres@: psql
psql=# CREATE DATABASE marvels WITH OWNER nikhil;
psql=# REVOKE ALL ON SCHEMA public FROM public;
psql=# GRANT ALL ON SCHEMA public TO nikhil;
psql=# ALTER USER nikhil WITH PASSWORD 'nikhil'
```
