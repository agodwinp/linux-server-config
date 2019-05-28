# 3. Linux Server Configuration

This repository will guide you through how to set up a Ubuntu Linux VM using Amazon Lightsail, and access my deployed item-catalog application from the server.

***

## Design

Include architecture and code design diagrams.

***

## Setup

### Starting Ubuntu Linux VM

Include set up steps from the Udacity project page.

### Software Installed

#### Update Package Source List

    $ sudo apt-get update

#### Update Currently Installed Packages

    $ sudo apt-get upgrade

#### Remove Unused Packages

    $ sudo apt-get autoremove

### Configurations

#### Firewall

First, block all incoming requests.

    $ sudo ufw default deny incoming

Allow all outgoing requests.

    $ sudo ufw default allow outgoing

Allow ssh.

    $ sudo ufw allow ssh
    $ sudo ufw allow 2200/tcp

Allow HTTP (port 80)

    $ sudo ufw allow www

Allow NTP (port 123)

    $ sudo ufw allow 123/tcp


Firewall, users, database etc...

***

## Usage

How to connect to application.

***

## Authors

Arun Godwin Patel