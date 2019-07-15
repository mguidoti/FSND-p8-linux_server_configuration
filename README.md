## Linux Server Configuration

> If you consider the LinkedIn and GitHub profile optional exercises as projects, this is the eighth project of the [Full Stack Web Development Nanodegree](https://in.udacity.com/course/full-stack-web-developer-nanodegree--nd004/) program, from Udacity.



### Overview

This project's goal is to set up a Linux server to host one of the web applications developed throughout the [Udacity's Full Stack Web Development Nanodegree](https://in.udacity.com/course/full-stack-web-developer-nanodegree--nd004/). 

The configuration involved creating a new user, giving it the right credentials, creating a SSH key, protecting the server, etc.

[Amazon Lightsail](https://lightsail.aws.amazon.com) was the service used in this project, with the basic plan and a 512 MB RAM, 1 vCPU, 20 GB SSD machine. 



### Server Basic Information

**Public IP:** 34.201.53.27

**SSH port:** 2200

**App URL:** http://34.201.53.27.xip.io



### Configuration Steps

#### Start a new Ubuntu Linux server, connect into it using SSH

The [Udacity's Get Started on Lighsail](https://classroom.udacity.com/nanodegrees/nd004-br/parts/8893c4ba-3622-413f-8622-090527207969/modules/aa87e6b9-d442-498d-99a4-eeebf02183d7/lessons/38095062-abb6-4c55-a3a8-885e8aea4c83/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e) guide was used to create this server's instance and to connect into it using SSH. The instance was created with Ubuntu 16.04. 



#### Update all currently installed packages

1. Update package source list: `sudo apt-get update`

2. Update all installed packages: `sudo apt-get upgrade`

   When asked, choose to install the "package maintainer's version".

3. Remove unnecessary packages: `sudo apt-get autoremove`



#### Change the SSH port from **22** to **2200. Configure the Lightsail firewall to allow it**

1. Use `sudo nano /etc/ssh/sshd_config` to open the file and edit line 4 from **Port 22** to **Port 2200**. Save and quit nano.
2. Reload SSH service: `sudo service ssh restart`
3. On your instance dashboard, **Networking** tab, add a Custom TCP application with Port range = 2200 to your instance's Firewall.



_From this point onward, if I close the browser connection with my server, I won't be able to come back. Thus, I downloaded [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html), my default private key from Lightsail, and managed to continue with the next steps from this direct connetion through PuTTY._



#### Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

1. Check the current UFW status: `sudo ufw status`

   It's expected that the UFW is **disabled**;

2. Block all incoming requests initially: `sudo ufw default deny incoming` 

3. Allow all outgoing connections: `sudo ufw default allow outgoing`

4. Allow SSH setting port to 2200: `sudo ufw allow 2200/tcp`

5. Allow HTTP on its default port 80: `sudo ufw allow www`

6. Allow NTP on its default port 123: `sudo ufw allow ntp`

7. Enable firewall: `sudo ufw enable`

8. Check UFW states once again: `sudo ufw status`




#### Give `grader` access

1. Create a new user account named **grader**: `sudo adduser grader`

2. Give **grader** the permission to **sudo**: 
   1. Open **/etc/sudoers** to check if it includes `#includedir /etc/sudoers.d` (usually on its last line), or include it if absent: `sudo nano /etc/sudoers`
   2. Create a file named **grader** in **/etc/sudoers.d**: `sudo touch /etc/sudoers.d/grader`
   3. Check if the file was correctly created: `sudo ls -al /etc/sudoers.d`
   4. Open the recently created file (**/etc/sudoers.d/grader**) with `sudo nano /etc/sudoers.d/grader` and add a single line containing `grader ALL=(ALL:ALL) ALL` , save the file
   5. Check if the file was correctly saved: `sudo cat /etc/sudoers.d/grader`
   
3. Create an SSH key pair for **grader** using **ssh-keygen**.
   
   a. On local machine:
   
      1. Create an SSH key pair for grader on my local machine: `ssh-keygen`
         1. As I'm on Windows10, I set the file name to **grader** with the following directory: `c:\Users\Marcus\.ssh\grader` 
         2. Leave empty the passphrase
      2. Read and copy the content of the recently created grader.pub: `cat ~/ssh/grader.pub`
   
   b. On server:
   
   1. Create a **/home/grader/.ssh** folder: `sudo mkdir /home/grader/.ssh`
   2. Create an **authorized_keys** file inside the **/.ssh** folder: `sudo touch /home/grader/.ssh/authorized_keys` 
   3. Edit the recently created file to add the copied SSH key, not forgetting to save after it: `sudo nano /home/grader/.ssh/authorized_keys`
   4. Check if the file now contains the key: `sudo cat /home/grader/.ssh/authorized_keys`
   5. Change ownership status for **/.ssh** folder: `sudo chown grader:grader /home/grader/.ssh` 
   6. Check if the user **grader** is the owner of the folder **/.ssh**: `sudo ls -al /home/grader`
   7. Change the credentials of **/.ssh** folder: `sudo chmod 700 /home/grader/.ssh`
   8. Change the **/.ssh/authorized_keys** credentials: `sudo chmod 644 /home/grader/.ssh/authorized_keys`
   9. Reload the SSH service once again: `sudo service ssh restart`



_From now on, we can login from your OS using Bash with the grader user:_ `ssh grader@PUBLIC-IP -p 2200 -i PATH-TO-MY-KEY-FILE`

_In my case, **PUBLIC-IP** is `34.201.53.27` and **PATH-TO-MY-KEY-FILE** is `~/.ssh/grader`._



**_The SSH Key was sent in the "Notes to Reviewer field"_**



#### Remove root user access and password authentication

1. Open the right file by using `sudo nano /etc/ssh/sshd_config`
2. Change `PermitRootLogin` to **no**
3. Set, if not already, `PasswordAuthentication`to **no**
4. Add, on the last line, **DenyUsers root**
5. Save the file before exiting



#### Configure the local timezone to UTC.

1. First, check the current date: `date`
2. Then, if not in UTC yet, change it with `sudo timedatectl set-timezone UTC`
3. Now check it again: `date` 



_My server was already set to UTC, thus, steps 2-3 were unnecessary._



#### Install `Apache` and `mod_wsgi` for python 3

1. For `Apache`: `sudo apt-get install apache2`

 	2. For `mod_wsgi`: `sudo apt-get install libapache2-mod-wsgi`



#### Install and configure `PostgreSQL`:

1. Instal **PostgreSQL** with `sudo apt-get install postgresql`

2. To create a new database user named `catalog` with limited permissions to my catalog application database: 

   1. Login as **postgres**: `sudo su - postgres`

   2. Start PostgreSQL shell: `psql`

   3. Create a new database (in my case, **themehospitals**) and a user named **catalog**: 

      ```
      postgres=# CREATE DATABASE themehospitals;
      postgres=# CREATE USER catalog;
      ```

   4. Set password for the user **catalog**: `postgres=# ALTER ROLE catalog WITH PASSWORD 'SECRET-MAGIC-WORD';`

   5.  Give full access to the database **themehospitals** to the user **catalog**: `postgres=# GRANT ALL PRIVILEGES ON DATABASE themehospitals TO catalog;`

   6. Exit PostgreSQL shell: `postgres=# \q`

   7. Logout from **postgres** user: `exit`



**_The user password was also sent in the "Notes to Reviewer field"_** 



#### Install `git` and my Item Catalog project repository

1. Install Git with `sudo apt-get install git`
2. Move to the directory watched by Apache with `cd /var/www`
3. Create the application directory with `sudo mkdir ItemCatalog`
4. Move to the recently created folder: `cd ItemCatalog`
5. Initiate an empty git repository in the current folder: `sudo git init`
6. Add my [Item Catalog App Repo](https://github.com/mguidoti/FSND-p4-item_catalog) as a remote repository: `sudo git remote add origin https://github.com/mguidoti/FSND-p4-item_catalog.git`
7. Check if the remote repository was added successfully: `sudo git remote -v`
8. Pull the remote repository: `sudo git pull origin master`
9. Check if the files were downloaded successfully: `sudo ls`



#### Make necessary modifications on Item Catalog project files

1. Rename **hospitals.py** to `__init__.py` with `sudo mv hospitals.py __init__.py`
2. Edit **initiate_database.py**, **fill_database.y**, and the now renamed `__init.py__`, and change all occurrences of **'sqlite:///themehospitals.db'** to `'postgresql://catalog:password@localhost/themehospitals'`, editing the files with: `sudo nano 'FILE-NAME'`



_Remember: the password was given in the "Notes to Reviewer field"._



#### Install all libraries on Item Catalog project requirements.txt

1. First, install pip: `sudo apt-get install python-pip`
3. Then, use it to install all dependencies listed on **requirements.txt**: `sudo pip install -r requirements.txt`



#### Initiate Item Catalog database

1. To create the database: `sudo python initiate_database.py`
2. To fill the database: `sudo python fill_database.py`



_Remember, to understand why it's necessary to run two *.py to get this project started, refer to the [original README](https://github.com/mguidoti/FSND-p4-item_catalog/blob/master/README.md)._



#### Config `Apache`

1. Create ItemCatalog.conf: `sudo nano /etc/apache2/sites-available/ItemCatalog.conf`

2. Add the following to the recently created and opened file:

   ```
   <VirtualHost *:80>
           ServerName 34.201.53.27
           ServerAdmin marcus.guidoti@gmail.com
           ServerAlias 34.201.53.27.xip.io
   	WSGIScriptAlias / /var/www/ItemCatalog/itemcatalog.wsgi
   	<Directory /var/www/ItemCatalog>
   		Order allow,deny
   		Allow from all
   	</Directory>
   	Alias /static /var/www/ItemCatalog/static
   	<Directory /var/www/ItemCatalog/static/>
   		Order allow,deny
   		Allow from all
   	</Directory>
   	ErrorLog ${APACHE_LOG_DIR}/error.log
   	LogLevel warn
   	CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```

3. Enable the virtual host: `sudo a2ensite ItemCatalog`

4. Activate the new configuration: `sudo service apache2 reload`



#### Create and config the *.wsgi file

1. To create AND edit the desired file: `sudo nano /var/www/ItemCatalog/itemcatalog.wsgi`

2. Add the following to the recently created and opened file:

   ```
   #!/usr/bin/python
   import sys
   import logging
   
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/ItemCatalog")
   
   from __init__ import app as application
   application.secret_key = 'super_secret_key'
   ```



_The application.secret_key is correctly provided above. I know, I know... I should have changed it, but I didn't, and now it looks like a fake/dummy secret key but it's the real one!_



#### Restart and reload `Apache`

1. Restart: `sudo service apache2 restart`
2. Reload: `sudo service apache2 reload`



#### Logging

If something breaks or go wrong, you can access a log for this application with the following command:

```
sudo tail -f /var/log/apache2/error.log 
```



### Third-party Resources Consulted

The following sources were consulted in order to finish this project:



#### Courses

[Udacity's Linux Commandas Line Basics](https://www.udacity.com/course/viewer#!/c-ud595-nd)

[Udacity's Configuring Linux Web Servers](https://www.udacity.com/course/viewer#!/c-ud299-nd)



#### Documentation

[Ubuntu](http://manpages.ubuntu.com/)

[Apache - HTTP Server Project](https://httpd.apache.org/)

[PostgreSQL](https://www.postgresql.org/docs)

[WSGI](https://wsgi.readthedocs.io/en/latest/)

[Lightsail and PuTTY](https://lightsail.aws.amazon.com/ls/docs/en_us/articles/lightsail-how-to-set-up-putty-to-connect-using-ssh)



#### Webcasts

[SSH: How to access a remote server and edit files](https://www.youtube.com/watch?v=HcwK8IWc-a8)



#### GitHub Repositories

[@andrevst](https://github.com/andrevst/fsnd-p6-linux-server-configuration)

[@jungleBadger](https://github.com/jungleBadger/-nanodegree-linux-server)

[@bcko](https://github.com/bcko/Ud-FS-LinuxServerConfig-LightSail)

[@leandrocl2005](https://github.com/leandrocl2005/aws_lightsail_config_for_flask_python3)



#### Q&A

[Apache error “Could not reliably determine the server's fully qualified domain name”](https://askubuntu.com/questions/256013/apache-error-could-not-reliably-determine-the-servers-fully-qualified-domain-n)

[How to connect to Amazon LightSail instance from Windows 10](https://manjaro.site/how-to-connect-to-amazon-lightsail-from-windows/)

[How to Set Up Time Synchronization on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-time-synchronization-on-ubuntu-16-04)