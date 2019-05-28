# 3. Linux Server Configuration

This repository will guide you through how to set up a Ubuntu Linux VM using Amazon Lightsail, and access a deployed item-catalog application from the server.

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

#### Install Apache2 Webserver

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

***

## Usage

How to connect to application.

***

## Authors

Arun Godwin Patel