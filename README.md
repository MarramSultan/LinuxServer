

# Linux Server Configuration


## Description
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

## IP address and Hostname
**IP address**: 3.8.2.228
**Hostname**: [http://ec2-3-8-2-228.eu-west-2.compute.amazonaws.com](http://ec2-3-8-2-228.eu-west-2.compute.amazonaws.com)
**SSH Port**: 2200
## Configure Linux Instance 
Steps to start working on Amazon Lightsail is provided by udacity [here](https://classroom.udacity.com/nanodegrees/nd004-connect/parts/226fb92a-d5dc-4d10-add0-c1dabff6ee69/modules/56cf3482-b006-455c-8acd-26b37b6458d2/lessons/046c35ef-5bd2-4b56-83ba-a8143876165e/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e).

## Linux Configuration

 1- Updates all system packages to most recent versions, and remove by running the commands:

Run this command to get information about new and updated packages available

    $ sudo apt-get update
Install the newest versions of all packages currently installed on the system that requires no additional installation or modification

    $ sudo apt-get upgrade
Install any extra packages' updates that would require additional installation or removal

    $ sudo apt-get dist-upgrade
To remove any unessery installed packages 

    $ sudo apt-get autoremove

2- Install the software **finger** to be able to create users, we install **finger** using the command  `$ sudo apt-get install finger` , after that we'd be able to run the command `$ finger` to see users authorized to login the server.

3- Create  grader user by running `$ sudo adduser grader`, it'll ask for a password and retyping the password again, other information are optional to fill. To login into the server use `ssh grader@3.8.2.228 -p 22` and by providing the password that was earlier created, we'll log in the server as **grader** user.

4- To able grader to run sudo commands, modify **/etc/sudoers** to include the grader. Edit this file by running `$ sudo nano  /etc/sudoers` then, right below the line `root  ALL=(ALL:ALL) ALL`,  add the following line `grader  ALL=(ALL:ALL) ALL` , exit and save changes.

5-  Create an SSH key pair for grader using the **ssh-keygen** tool. As local, run the command `$ ssh-keygen` , then keep the same path and change only the last part of the default name to **linuxServer** ,after that it'll ask you to write passphrase that you should memorize since you'll be asked to insert in the next login using this created key. As local also, copy the content of the public key **linuxServer.pub** using `cat /YOURPATH/.ssh/linuxServer.pub`.

6- Login to the server as grader, then:
- create .ssh directory and an authorized_keys file where the public key you copied earlier will be stored, to do that run the commands:

    `$ mkdir .ssh`
   ` $ touch .ssh/authorized_keys`
    `$ nano .ssh/authorized_keys`
- Paste the public key in the editor, then save the changes.
- Run these commands to change permissions: `chmod 700 .ssh`
and `chmod 644 .ssh/authorized_keys`

- Change settings so that login using password is not allowed by editing the file `$ sudo nano /etc/ssh/sshd_config`, change `passwordAthuntication` option to no. Restart services to apply changes using `$ sudo service sshd restart`. Now, we can login as the grader using `ssh grader@**3.8.2.228** -p 22 -i ~/.ssh/linuxserver`.

7- Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

    $ sudo ufw  default deny incoming
    $ sudo ufw default allow outgoing
    $ sudo ufw allow 2200/tcp
    $ sudo ufw allow www
    $ sudo ufw allow ntp
    $ sudo ufw deny 22/tcp
    $ sudo ufw deny 22
    $ sudo ufw enable 
    $ sudo ufw status 

-  `$ sudo nano /etc/ssh/sshd_config `- edit Port 20 line to Port 2200 and save changes.
-   In Lightsail dashboard, go to Networking and make sure changes applied successfully.
-   Exit and reconnect with ssh command using -p 2200. `ssh grader@52.56.250.121 -p 2200 -i ~/.ssh/linuxserver`.

8- Disable ssh login for root user: 
-   Edit the file `$ sudo nano /etc/ssh/sshd_config`.
-   Change `PermitRootLogin prohibit-password` to `PermitRootLogin no`, save changes and exit nano.
-   Restart ssh with `$ sudo service ssh restart`.

9- Configure the local timezone to UTC by running `$ sudo dpkg-reconfigure tzdata `, then choose Time Zone: UTC. It’s already set up to UTC in my server.

10- sudo apt-get install apache2, mod-wsgi, and git:

    $ sudo apt-get install apache2
    $ sudo apt-get install libapache2-mod-wsgi python-dev
    $ sudo apt-get install git
   11- enable mod-wsgi by running `$ sudo a2enmod wsgi` and restart Apache using `$ sudo service apache2 restart`. Confirm success installation of Apache by visiting **3.8.2.228**.[xip.io](http://xip.io) and see the welcome page of Apache2.


## Application Deployment 
1- Go back to console.developer.google.com and update the json file (client_sercrets.json) of the Flask Application we want to deploy so it contain the public ip of the server, and  redownload it again.

2- Create a directory "catalog"  and make the user "grader" the owner:

    $ cd /var/www
    $ sudo mkdir catalog
    $ sudo chown -R grader:grader catalog
    $ cd catalog
3- Since the grader is the owner now, login as grader to be able to clone the Flask Application, make sure we are in the catalog directory and then run this command:
`$ git clone https://github.com/MarramSultan/item_catalog.git catalog`

4- Create catalog.wsgi file `$ sudo nano catalog.wsgi` and paste the following

    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")
    
    from catalog import app as application
    application.secret_key = 'super_secret_key'

5- Apply changes to the Flask application file to operate with no errors:
- Rename  item_catalog.py, (the main application file) in the catalog application folder to __init\__.py by `$ mv item_catalog.py  __init__.py`.
-  Update files by changing the path to client_secrets.json or fb_client_secrets.json to the complete path  `/var/www/catalog/catalog/client_secrets.json`  ( it happens to be only in the __init\__.py file for me).

6- Prepare the virtual environment in ` /var/www/catalog` directory by running the following we'll install and activate vent and change venv permission:

    $ sudo pip install virtualenv
    $ sudo virtualenv venv
    $ source venv/bin/activate
    $ sudo chmod -R 777 venv
7- Install all packages needed for the application to work:

    $ sudo apt-get install python-pip
    $ sudo pip install flask
    $ sudo pip install httplib2 oauth2client  
    $ sudo pip install sqlalchemy
    $ sudo pip install psycopg2
    $ sudo pip install requests

8- Configure and enable a new virtual host:
- Run `$ sudo nano /etc/apache2/sites-available/catalog.conf`.
- Paste the following into the editor:
```
    <VirtualHost *:80>
    ServerName 3.8.2.228
    ServerAlias ec2-3-8-2-228.eu-west-2.compute.amazonaws.com
    ServerAdmin admin@35.167.27.204
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
    Order allow,deny
    Allow from all
    <\/Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
    Order allow,deny    
    Allow from all   
    <\/Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    <\/VirtualHost>
  ```
  9- Enable to virtual host and disable the default host:

     $ sudo a2ensite catalog.conf 
     $ a2dissite 000-default.conf 
10- Setting the database by installing PostgreSQL:

    $ sudo apt-get install libpq-dev python-dev
    $ sudo apt-get install postgresql postgresql-contrib
    $ sudo su - postgres
    $ psql
11-  Create a database user 'catalog' and assign password:

    postgres=# CREATE USER catalog WITH PASSWORD ‘myPassword’;
    postgres=# ALTER USER catalog CREATEDB;
    postgres=# CREATE DATABASE catalog with OWNER catalog;
    postgres=# \c catalog
    catalog=# REVOKE ALL ON SCHEMA public FROM public;
    catalog=# GRANT ALL ON SCHEMA public TO catalog;
    catalog=# \q
    $ exit
12- Using nano again to edit  __init\__.py, database_setup.py,and helper.py files to change the database engine from `sqlite://countries.db` to `postgresql://catalog:myPassword@localhost/catalog`.

13- Restart  apache server `$ sudo service apache2 restart`, then open the browser to test the and IP address and hostname both load my application.


## Third-party resources:
-   [https://github.com/mulligan121/Udacity-Linux-Configuration](https://github.com/mulligan121/Udacity-Linux-Configuration)
-   Find the host name :[https://whatismyipaddress.com/ip-hostname](https://whatismyipaddress.com/ip-hostname)
-   [https://stackoverflow.com/questions/14547631/python-locale-error-unsupported-locale-setting](https://stackoverflow.com/questions/14547631/python-locale-error-unsupported-locale-setting)
-   [https://ubuntuforums.org/showthread.php?t=1222909](https://ubuntuforums.org/showthread.php?t=1222909)
-   [https://askubuntu.com/questions/81585/what-is-dist-upgrade-and-why-does-it-upgrade-more-than-upgrade](https://askubuntu.com/questions/81585/what-is-dist-upgrade-and-why-does-it-upgrade-more-than-upgrade)



