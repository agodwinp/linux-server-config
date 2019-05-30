# Linux Server Configuration

This repository will guide you through how to take a baseline installation of a Ubuntu Linux Virtual Machine using Amazon Lightsail and prepare it to host web applications. It will include installing updates, securing it from attacks and installing/configuring web and database servers. Finally, it will show you how to deploy and serve my previously built item catalog application.

***

## Design

Include architecture and code design diagrams.

***

## Setup

### Starting Ubuntu Linux VM

Include set up steps from the Udacity project page.

Create ssh keys from local machine, we will use these later.

Add these as the keys to the Amazon Lightsail instance.

### Software Installed

#### Update Package Source List

    $ sudo apt-get update

#### Update Currently Installed Packages

    $ sudo apt-get upgrade

#### Remove Unused Packages

    $ sudo apt-get autoremove

### Configurations

#### SSH Configuration

Edit the sshd_config file to allow ssh requests from port 2200.

    $ sudo nano /etc/ssh/sshd_config

Edit line 5 that says `Port 20` to `Port 2200`. Then exit and save this file.

#### Firewall

First, block all incoming requests.

    $ sudo ufw default deny incoming

Allow all outgoing requests.

    $ sudo ufw default allow outgoing

Allow ssh.

    $ sudo ufw allow 2200/tcp

Allow HTTP (port 80)

    $ sudo ufw allow www

Allow NTP (port 123)

    $ sudo ufw allow 123/tcp

Start the firewall

    $ sudo ufw enable

Log out of the VM go into the `Networking` tab of the Amazon Lightsail instance that you have just created. Add the port 2200 to the accepted lists of ports for the firewall.

Now reboot the VM. 

#### Users

Create a new user called `grader`

    $ sudo adduser grader

Enter a password, but we will disable this later. You can hit enter when prompted to fill in more information, or feel free to fill in the details (Full name, Room number, Work phone, Home phone, Other).

Make `grader` a sudoer. Copy existing sudoers.d file as grader.

    $ sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader

Give read and write permissions to root for this user.

    $ sudo chmod 600 /etc/sudoers.d/grader

Edit the `grader` file in sudoers.d

    $ sudo nano /etc/sudoers.d/grader

Change `ubuntu ALL=(ALL) NOPASSWD:ALL` to `grader ALL=(ALL) NOPASSWD:ALL`

#### Public/Private Key

Log in again as ubuntu

    $ ssh ubuntu@35.178.22.227 -p 2200

Switch user to `grader`.

    $ su - grader

In the root directory, create a directory called `.ssh`.

    $ sudo mkdir /.ssh

Create a file called `authorized_keys`.

    $ sudo touch /.ssh/authorized_keys

Return to the public key that has been generated on your local machine, and view the key using:

    $ cat ~/.ssh/item_catalog.pub

Copy this entire key into the `authorized_key` file within the VM using:

    $ sudo nano /.ssh/authorized_keys

Set up some specific file permissions on the authorized_keys file and the .ssh directory. This is a security measure that ssh enforces to ensure other users cannot gain access to your account. 

    $ sudo chmod 700 /.ssh
    $ sudo chmod 644 /.ssh/authorized_keys

Log out of the grader user

    $ exit

And log out of the VM. Now reboot the VM and from a terminal window, ssh into the VM with your new user and private key using this command:

    $ ssh grader@35.178.22.227 -i ~/.ssh/item_catalog -p 2200

#### Disable Password Authentication

Open the SSH daemon configuration file.

    $ sudo nano /etc/ssh/sshd_config

Scroll down to the line starting with `PasswordAuthentication`. Ensure that after this word, it says `no`. Then save and close the file.

Reload the ssh daemon

    $ sudo systemctl reload sshd

#### Change Timezone

Configure the local timezone to UTC.

    $ sudo dpkg-reconfigure tzdata

Scroll down to `None of the above` and hit enter. Then scroll down again and hit enter when you find `UTC`.

#### Install Apache2 Webserver and mod_wsgi

    $ sudo apt-get install apache2
    $ sudo apt-get install libapache2-mod-wsgi

Check to see if the Apache web server is running by visiting `35.178.22.227:80`. You should be presented with an Apache documentation site. Now we will change this so that it routes to our application.

Configure Apache to handle requests using the WSGI module.

    $ sudo nano /etc/apache2/sites-enabled/000-default.conf

Add the following line above `</Virtualhost>`
    
    $ WSGIScriptAlias / /var/www/html/myapp.wsgi

Restart apache with

    $ sudo apache2ctl restart

Create the `myapp.wsgi` file

    $ sudo nano /var/www/html/myapp.wsgi

#### Install Git

Install Git

    $ sudo apt-get install git

Configure your username

    $ git config --global user.name <username>

Configure your email

    $ git config --global user.email <email>

#### Clone the Item Catalog app from Github

cd into `/var/www`

    $ cd /var/www

Make a new directory for the git repo

    $ sudo mkdir catalog

Change ownership of this directory to grader

    $ sudo chown grader catalog

Move inside the newly created folder and clone the repo from Github into a folder called `catalog`

    $ cd catalog
    $ git clone https://github.com/agodwinp/udacity-item-catalog.git catalog

cd into the git repo and checkout the deployment branch

    $ cd catalog
    $ git checkout deployment

Make a catalog.wsgi file to serve the application over mod_wsgi, and then edit this file with nano

    $ touch catalog.wsgi
    $ nano catalog.wsgi

Include the following code within this file

    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application

Save and exit from this file.

#### Install virtual environment, Flask and project dependencies

Install pip, the tool for installing Python packages and update

    $ sudo apt-get install python-pip
    $ sudo pip install --upgrade pip

If virtualenv is not installed, use pip to install it using the following command

    $ sudo pip install virtualenv

Move to the parent catalog folder and then create a new virtual environment

    $ cd /var/www/catalog
    $ sudo virtualenv venv

Activate the virtual environment

    $ source venv/bin/activate

Change permissions to the virtual environment folder

    $ sudo chmod -R 777 venv

Install all the projects dependencies

    $ sudo pip install -r catalog/requirements.txt

Install psycopg2

    $ sudo pip install psycopg2-binary
    $ deactivate

#### Configure and enable a new virtual host

Create a virtual host config file

    $ sudo nano /etc/apache2/sites-available/catalog.conf

Include the following lines of code in this file

    <VirtualHost *:80>
        ServerName 35.178.22.227
        ServerAlias ec2-35-178-22-227.eu-west-2.compute.amazonaws.com
        ServerAdmin admin@35.178.22.227
        WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
        WSGIProcessGroup catalog
        WSGIScriptAlias / /var/www/catalog/catalog/catalog.wsgi
        <Directory /var/www/catalog/catalog/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/catalog/catalog/static
        <Directory /var/www/catalog/catalog/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>

Enable the new virtual host

    $ sudo a2ensite catalog

To activate the new configuration reload the apache server

    $ sudo service apache2 reload

#### Install and configure PostgreSQL

Install some necessary Python packages for working with PostgreSQL

    $ sudo apt-get install libpq-dev python-dev

Install PostgreSQL

    $ sudo apt-get install postgresql postgresql-contrib

Postgres is automatically creating a new user during its installation, whose name is 'postgres'. That is a trusted user who can access the database software. So let's change the user then connect to the database system

    $ sudo su - postgres
    $ psql

Create a new user called 'catalog' with a password

    # CREATE USER catalog WITH PASSWORD 'password';

Give catalog user the CREATEDB capability

    # ALTER USER catalog CREATEDB;

Create the 'catalog' database owned by catalog user

    # CREATE DATABASE catalog WITH OWNER catalog;

Connect to the database:

    # \c catalog

Revoke all rights
        
    # REVOKE ALL ON SCHEMA public FROM public;

Lock down the permissions to only let catalog role create tables

    # GRANT ALL ON SCHEMA public TO catalog;

Log out from PostgreSQL and then return to the grader user on the server

    # \q
    $ exit

Setup the database with

    $ python /var/www/catalog/catalog/populatedb.py

To prevent potential attacks from the outer world we double check that no remote connections to the database are allowed. Open the file `ph_hba_conf` and edit it to look like below

    $ sudo nano /etc/postgresql/9.3/main/pg_hba.conf

    local   all             postgres                                peer
    local   all             all                                     peer
    host    all             all             127.0.0.1/32            md5
    host    all             all             ::1/128                 md5

#### Update OAuth Authorised Javascript Origins and Domains

To let users correctly log in with Google Sign-In you must add http://ec2-35-178-22-227.eu-west-2.compute.amazonaws.com to the list of authorized URI's for this Client ID.

#### Restart Apache

Restart Apache to launch the app


    $ sudo service apache2 restart

If at any point the web server does not serve the item catalog application any longer, you can view the apache2 error log by running the following command on thr server

    $ sudo cat /var/log/apache2/error.log

***

## Usage

- IP Address: 35.178.22.227
- Accessible SSH Port: 2200
- Application URL: http://ec2-35-178-22-227.eu-west-2.compute.amazonaws.com

***

## Authors

Arun Godwin Patel