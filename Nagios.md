# How to monitor a server with Nagios

## Introduction
In this tutorial, we are going to make two AWS EC2 instances: one to run Nagios (codename from now on: `NAGIOS`) and another, that will be monitored by Nagios (codename: `STBM` *(= Server To Be Monitored)*). On `NAGIOS`, we need to install Nagios Core (the free version of theis monitoring service) and a plugin called *check_nrpe*. On `STBM`, we will install the the plugin called *NRPE*. After that, we will connect these too, so the *check_nrpe* plugin on `NAGIOS` can get information about the `STBM` via the *NRPE* plugin sitting on it.

You can make these servers with Terraform or Pulumi, which will not be discussed here, only the contents of the shell scripts to be specified as *user_data*. 
**Important!** When you are configuring the security group for `STBM`, you need to allow access to the *NRPE* port (5666) from `NAGIOS` (basically add port 5666 to your ingress ports in the security group you are making).

## The script to set up the server with Nagios on Ubuntu 18.04

```
#!/bin/bash

# Creating user "dev"
cd /etc/ssh
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' sshd_config
sudo service ssh restart
sudo useradd -m -p $(openssl passwd -1 password) dev

# Installing Nagios (https://support.nagios.com/kb/article/nagios-core-installing-nagios-core-from-source-96.html#Ubuntu)

# Prerequisites
# Perform these steps to install the pre-requisite packages.
sudo apt-get update
sudo apt-get install -y autoconf gcc libc6 make wget unzip apache2 php libapache2-mod-php7.2 libgd-dev

# Downloading the Source
cd /tmp
wget -O nagioscore.tar.gz https://github.com/NagiosEnterprises/nagioscore/archive/nagios-4.4.5.tar.gz
tar xzf nagioscore.tar.gz
 
# Compile
cd /tmp/nagioscore-nagios-4.4.5/
sudo ./configure --with-httpd-conf=/etc/apache2/sites-enabled
sudo make all

# Create User And Group
# This creates the nagios user and group. The www-data user is also added to the nagios group.
sudo make install-groups-users
sudo usermod -a -G nagios www-data

# Install Binaries
# This step installs the binary files, CGIs, and HTML files.
sudo make install
 
# Install Service / Daemon
# This installs the service or daemon files and also configures them to start on boot.
# Information on starting and stopping services will be explained further on.
sudo make install-daemoninit

# Install Command Mode
# This installs and configures the external command file.
sudo make install-commandmode
 
# Install Configuration Files
# This installs the *SAMPLE* configuration files. These are required as Nagios needs some configuration files to allow it to start.
sudo make install-config

# Install Apache Config Files
# This installs the Apache web server configuration files and configures Apache settings.
sudo make install-webconf
sudo a2enmod rewrite
sudo a2enmod cgi

# Configure Firewall
# You need to allow port 80 inbound traffic on the local firewall so you can reach the Nagios Core web interface.
sudo ufw allow Apache
sudo ufw reload
 
# Create nagiosadmin User Account
# You'll need to create an Apache user account to be able to log into Nagios.
# The following command will create a user account called nagiosadmin and you will be prompted to provide a password for the account.
sudo htpasswd -b -c /usr/local/nagios/etc/htpasswd.users nagiosadmin strongpassword

# Start Apache Web Server
# Need to restart it because it is already running.
sudo systemctl restart apache2.service

# Start Service / Daemon
# This command starts Nagios Core.
sudo systemctl start nagios.service


# Test Nagios
# Nagios is now running, to confirm this you need to log into the Nagios Web Interface.
# Point your web browser to the ip address or FQDN of your Nagios Core server, for example:
# http://10.25.5.143/nagios
# http://core-013.domain.local/nagios
# You will be prompted for a username and password. The username is nagiosadmin (you created it in a previous step) and the password is what you provided earlier.
# Once you have logged in you are presented with the Nagios interface. Congratulations you have installed Nagios Core.

# BUT WAIT ...
# Currently you have only installed the Nagios Core engine. You'll notice some errors under the hosts and services along the lines of:
# (No output on stdout) stderr: execvp(/usr/local/nagios/libexec/check_load, ...) failed. errno is 2: No such file or directory 
# These errors will be resolved once you install the Nagios Plugins, which is covered in the next step.


# Installing The Nagios Plugins
# Nagios Core needs plugins to operate properly. The following steps will walk you through installing Nagios Plugins.
# These steps install nagios-plugins 2.2.1. Newer versions will become available in the future and you can use those in the following installation steps. Please see the releases page on GitHub for all available versions.
# Please note that the following steps install most of the plugins that come in the Nagios Plugins package. However there are some plugins that require other libraries which are not included in those instructions. Please refer to the following KB article for detailed installation instructions:
# Documentation - Installing Nagios Plugins From Source
 

# Prerequisites
# Make sure that you have the following packages installed.
sudo apt-get install -y autoconf gcc libc6 libmcrypt-dev make libssl-dev wget bc gawk dc build-essential snmp libnet-snmp-perl gettext
 
# Downloading The Source
cd /tmp
wget --no-check-certificate -O nagios-plugins.tar.gz https://github.com/nagios-plugins/nagios-plugins/archive/release-2.3.1.tar.gz
tar zxf nagios-plugins.tar.gz
 
# Compile + Install
cd /tmp/nagios-plugins-release-2.3.1/
sudo ./tools/setup
sudo ./configure
sudo make
sudo make install
 

# Test Plugins
# Point your web browser to the ip address or FQDN of your Nagios Core server, for example:
# http://10.25.5.143/nagios
# http://core-013.domain.local/nagios
# Go to a host or service object and "Re-schedule the next check" under the Commands menu. The error you previously saw should now disappear and the correct output will be shown on the screen.


# Service / Daemon Commands
# sudo systemctl start nagios.service
# sudo systemctl stop nagios.service
sudo systemctl restart nagios.service
# sudo systemctl status nagios.service


# Monitoring Host Setup
# On the monitoring host (the machine that runs Nagios), you'll need to do just a few things:
# • Install the check_nrpe plugin
# • Create a Nagios command definition for using the check_nrpe plugin
# • Create Nagios host and service definitions for monitoring the remote host
# These instructions assume that you have already installed Nagios on this machine according to the
# quickstart installation guide. The configuration examples that are given reference templates that are
# defined in the sample localhost.cfg and commands.cfg files that get installed if you follow the quickstart.

# i. Install the check_nrpe Plugin
# Download the source code tarball of the NRPE addon (visit https://www.nagios.org/downloads/nagios-core-
# addons/ for links to the latest versions). At the time of writing, the latest version of NRPE was 3.2.1.
mkdir /home/ubuntu/downloads
cd /home/ubuntu/downloads
wget https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-4.0.0/nrpe-4.0.0.tar.gz
# Extract the NRPE source code tarball:
tar xzf nrpe-4.0.0.tar.gz
cd nrpe-4.0.0
# Compile the NRPE addon:
sudo ./configure
sudo make check_nrpe
# Install the NRPE plugin.
sudo make install-plugin
```

## The script to set up the server to monitered by Nagios on Ubuntu 18.04

```
#!/bin/bash
cd /etc/ssh
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' sshd_config
sudo service ssh restart
sudo useradd -m -p $(openssl passwd -1 password) dev

sudo apt-get update -y

# Install the Nagios Plugins
# Create a directory for storing the downloads, if you don't already have one.
mkdir /home/ubuntu/downloads
cd /home/ubuntu/downloads
# Make sure that you have the following packages installed.
sudo apt-get install -y autoconf gcc libc6 libmcrypt-dev make libssl-dev wget bc gawk dc build-essential snmp libnet-snmp-perl gettext
# Download the source code tarball of the Nagios plugins (visit http://www.nagios.org/downloads/ for links to
# the latest versions). At the time of writing, the latest stable version of the Nagios plugins was 2.3.1.
wget http://nagios-plugins.org/download/nagios-plugins-2.3.1.tar.gz
# Extract the Nagios plugins source code tarball.
tar xzf nagios-plugins-2.3.1.tar.gz
cd nagios-plugins-2.3.1
# Note: on some systems, you will have to run the extract this way:
# gunzip -c nagios-plugins-2.2.1.tar.gz | tar xf -
# Compile and install the plugins.
./configure
make
make install
# Depending on the version of the plugins, the permissions on the plugin directory and the plugins may need
# to be fixed at this point. If so run the following commands:
useradd nagios
groupadd nagios
usermod -a -G nagios nagios
chown nagios.nagios /usr/local/nagios
chown -R nagios.nagios /usr/local/nagios/libexec

# Install xinetd
# If you will be running NRPE per-connections, some distributions, such as Fedora Core 6, don't ship with
# xinetd installed by default, so install it with the following command:
sudo apt-get install -y xinetd

#  Install the NRPE daemon
# Download the source code tarball of the NRPE addon (visit https://www.nagios.org/downloads/nagios-coreaddons/ for links to the latest versions). At the time of writing, the latest version of NRPE was 3.2.1.
cd /home/ubuntu/downloads
wget https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-4.0.0/nrpe-4.0.0.tar.gz
# Extract the NRPE source code tarball:
tar xzf nrpe-4.0.0.tar.gz
cd nrpe-4.0.0
# Compile the NRPE addon:
./configure
make all
# If you didn't create the groups and users in (i) above, do it now:
make install-groups-users
# Install the NRPE plugin (for testing), daemon, and sample daemon configuration file.
make install
make install-config
# If you want NRPE to run per-connection under inetd, xinetd, launchd, systemd, smf, etc. run the following command:
make install-inetd
# Make sure nrpe 5666/tcp is in your /etc/services file, if applicable.
# If you want to run NRPE all the time under init, launchd, systemd, smf, etc. run the followning command:
make install-init
# You may need to reload or restart the controlling daemon using one of the following (or similar) commands:
service xinetd restart
systemctl reload xinetd # systemd
systemctl enable nrpe && systemctl start nrpe # systemd
svcadm enable nrpe # smf
initctl reload-configuration # upstart

# If everything worked, add the hostname or IP address of the nagios server to the /etc/xinetd.d/nrpe
# file, or /etc/hosts-allow and hosts-deny.

# vi. Open firewall rules
# If the server has a firewall running, you need to allow access to the NRPE port (5666) from the Nagios server.
# In Ubuntu, the firewall is disabled by default (check: $ sudo ufw status)
```

## Manual configuration to make these two to speak to each other...

We need to ssh into both of these servers, and check first, whether everything is working as it should be.
On `STBM` run the following command:

> $ netstat -at | egrep "nrpe|5666"

which should return something like

> tcp       0      0 0.0.0.0:nrpe            0.0.0.0:*               LISTEN     
tcp6       0      0 [::]:nrpe               [::]:*                  LISTEN 

Also, the following command 

> $ /usr/local/nagios/libexec/check_nrpe -H localhost

should return with the version number of the NRPE plugin

> NRPE v4.0.0

If everything seem to be working, lets add the IP-address of `NAGIOS` to the *NRPE* config file. Open the following file using your favorite terminal-based text editor:

> $ sudo nano /usr/local/nagios/etc/nrpe.cfg

and change the IP address on line 98 to the IP of `NAGIOS`: 

> allowed_hosts=x.x.x.x

Now, you have to restart the *xinetd* and *nrpe* services:

> $ sudo service xinetd restart

> $ sudo service nrpe restart

Lets head to our `NAGIOS`. You can check whether it is working by visiting its nagios endpoint in the browser (*http://x.x.x.x/nagios*). In the menu on the left, under Hosts and Services, you should see mostly green OK statuses indicating that it is successfully monitoring itself. Wouldn't it be nice, if it monitored our `STBM` server instead? First of all, lets check if it can reach `STBM` by running the following command, with the IP-address of `STBM` at the end:

> $ /usr/local/nagios/libexec/check_nrpe -H x.x.x.x

Which should return with the version number of the NRPE plugin in `STBM`

> NRPE v4.0.0

Very nice!