# 3. Linux Server Configuration

This repository will guide you through how to set up a Ubuntu Linux VM using Amazon Lightsail, and access my deployed item-catalog application from the server.

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

#### Users

Create a new user called `grader`

    $ sudo adduser grader

Enter a password, but we will disable this later. You can hit enter when prompted to fill in more information, or feel free to fill in the details (Full name, Room number, Work phone, Home phone, Other)

Make `grader` a sudoer. Copy existing sudoers.d file as grader.

    $ sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader

Give read and write permissions to root for this user.

    $ sudo chmod 600 /etc/sudoers.d/grader

Edit the `grader` file in sudoers.d

    $ sudo nano /etc/sudoers.d/grader

Change `ubuntu ALL=(ALL) NOPASSWD:ALL` to `grader ALL=(ALL) NOPASSWD:ALL`

#### Public/Private Key

Use ssh-keygen to generate a private/public key pair on your local machine.

Log in again as ubuntu

switch user using `su - grader`

Now log into the VM using the newly created user `grader`. You will be prompted to enter the password that you originally set, but we will remove the ability to log in with a password shortly.

First go into the `Networking` tab of the Amazon Lightsail instance that you have just created. Add the port 2200 to the accepted lists of ports for the firewall.

Reboot the instance.

    $ ssh grader@35.178.22.227 -p 2200

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

And reboot the VM.

From a terminal window, ssh into the VM using this command:

    $ ssh grader@35.178.22.227 -i ~/.ssh/item_catalog -p 2200

***

## Usage

How to connect to application.

***

## Authors

Arun Godwin Patel