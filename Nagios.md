# How to monitor a server with Nagios

## Introduction
In this tutorial, we are going to make two AWS EC2 instances: one to run Nagios (codename from now on: `NAGIOS`) and another, that will be monitored by Nagios (codename: `STBM` *(= Server To Be Monitored)*). On `NAGIOS`, we need to install Nagios Core (the free version of theis monitoring service) and a plugin called *check_nrpe*. On `STBM`, we will install the the plugin called *NRPE*. After that, we will connect these too, so the *check_nrpe* plugin on `NAGIOS` can get information about the `STBM` via the *NRPE* plugin sitting on it.

You can make these servers with Terraform or Pulumi, which will not be discussed here, only the contents of the shell scripts to be specified as *user_data*. 
**Important!** When you are configuring the security group for `STBM`, you need to allow access to the *NRPE* port (5666) from `NAGIOS` (basically add port 5666 to your ingress ports in the security group you are making).

The following scripts have comments included explaining every step of the process.

# Install

## The script to set up the server with Nagios (`NAGIOS`) on Ubuntu 18.04

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

## Server with Nagios (`NAGIOS`) with docker

Apart from the regular installation, you can pull a docker image of Nagios as well. In this case, the script can look like this:

```
#! /bin/bash

# installing Docker
sudo apt-get install -yy \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   bionic \
   stable"
sudo apt update
sudo apt install -yy docker-ce docker-ce-cli containerd.io
sudo groupadd docker
sudo usermod -aG docker $USER

# creating the volume for Nagios config files to be added to the container
sudo mkdir /usr/local/nagios/
sudo mkdir /usr/local/nagios/etc/

# pulling and starting the docker container
sudo systemctl start docker
sudo docker pull jasonrivers/nagios:latest
sudo docker run -d --name nagios4  \
  -v /usr/local/nagios/etc/:/opt/nagios/etc/ \
  -p 80:80 jasonrivers/nagios:latest
```

After the server finished initializing, you can navigate to the `/usr/local/nagios/etc/` on the machine, where the configuring process will happen. I have not tested this containerized Nagios though, and I did run into some problems with it, which will not be discussed here.

## The script to set up the server to monitered by Nagios (`STBM`) on Ubuntu 18.04

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

# Manual configuration

## 1. First, make sure these two speak to each other...

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

and and the IP address of `NAGIOS` on line 106 to the IP of the localhost: 

> allowed_hosts=127.0.0.1, **x.x.x.x**

Now, you have to restart the *nrpe* service:

> $ sudo service nrpe restart

Lets head to our `NAGIOS`. You can check whether it is working by visiting its nagios endpoint in the browser (*http://**x.x.x.x**/nagios*). In the menu on the left, under Hosts and Services, you should see mostly green OK statuses indicating that it is successfully monitoring itself. Wouldn't it be nice, if it monitored our `STBM` server instead? First of all, lets check if it can reach `STBM` by running the following command, with the IP-address of `STBM` at the end:

> $ /usr/local/nagios/libexec/check_nrpe -H **x.x.x.x**

Which should return with the version number of the NRPE plugin in `STBM`

> NRPE v4.0.0

Very nice!

## 2. Configure the servers to watch

From now on, the directory `/usr/local/nagios/etc/` on `NAGIOS` will be the one we configure by making and changing the *.cfg* files. If we list the already existing files and directories here, we will see a `nagios.cfg` file and an `objects` folder among others. First, open the `nagios.cfg` file

> $ sudo nano /usr/local/nagios/etc/nagios.cfg

and uncomment line 51 

> $ cfg_dir=/usr/local/nagios/etc/servers

Now, we can make the servers directory and put our server config files there.

> $ sudo mkdir /usr/local/nagios/etc/servers

Start this by making a copy of the localhost.cfg file in the `objects` folder inside of the `servers` directory.

> $ sudo cp /usr/local/nagios/etc/objects/localhost.cfg /usr/local/nagios/etc/servers

This `localhost.cfg` is the one telling Nagios what to monitor on our (*Linux*) local machine, therefore it is relatively easy to modify it to monitor our `STBM`. You can even rename it to `STBM.cfg` for example, if you like. Then open `STBM.cfg`, and change the host definition in the first part of the cfg file like this, using the IP-address of our `STBM`:

```
define host {

    use                     linux-server
    host_name               STBM
    alias                   Linux Server Ubuntu 18.04.
    address                 x.x.x.x
}
```

The next part in the file is host **GROUP DEFINITIONS**, you can comment it out for now. The third section is called **SERVICE DEFINITIONS**, where you should change the *host_names* from localhosst to STBM, like this:

```
define service {

    use                     local-service           ; Name of service template to use
    host_name               STBM
    service_description     PING
    check_command           check_ping!100.0,20%!500.0,60%
}
```

Now we turn to the commands it should execute, but don't forget about this file, as we will return to this to modify the *check_command* part of these services as well soon.

## 3. Configure the commands

### The check_nrpe command

Now open the `commands.cfg` file on `NAGIOS` in the `objects` folder. 

> sudo nano /usr/local/nagios/etc/objects/commands.cfg

There are a lot of predefined commands here, except for one called `check_nrpe` - let's go ahead and create it (place somewhere in the list of commands, for example under the **SAMPLE SERVICE CHECK COMMANDS** section):

```
define command {

    command_name    check_nrpe
    command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
```

This our main command to reach out to our `STBM` instance and invoke its commands defined on `STBM` and then read its result. 

### The commands on `STBM`

Lets check these commands on `STBM` by running:

> $ sudo nano /usr/local/nagios/etc/nrpe.cfg

Lines 300-304 where you should look for the commands for the NRPE plugin. They can remain mostly untouched, except for the `check_hda1` command. Its default behavior is that it checks the load on the /dev/hda1 partition, but the actual drive of our machine might be named differently. To figure it out, exit from the config file and run:

> $ lsblk

The output of this command is going to be similar to this in the case of a micro aws EC2 instance with Ubuntu Server 18.04:

```
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0     7:0    0   89M  1 loop /snap/core/7713
loop1     7:1    0   18M  1 loop /snap/amazon-ssm-agent/1480
loop2     7:2    0 89.1M  1 loop /snap/core/8268
xvda    202:0    0    8G  0 disk 
└─xvda1 202:1    0    8G  0 part /
```

In this case, our root (/) partition that we are looking for is called *xvda1*. Go back to line 302 in our `nrpe.cfg` file and change the command like this:

> command[check_xvda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/xvda1

After restarting the NRPE plugin (just in case) with 

> $ sudo service nrpe restart

we can check whether this commands work:

> $ /usr/local/nagios/libexec/check_nrpe -H localhost -c check_xvda1

If the output is similar to this:

> DISK OK - free space: /var/tmp 6008 MiB (76.44% inode=90%);| /var/tmp=1851MiB;6300;7088;0;7876

you are good to go!

### Calling the commands on `STBM` from `NAGIOS`

The way you are going to run these commands from the `NAGIOS` server is the following: go back to edit the `STBM.cfg` file on `NAGIOS`

> $ sudo nano /usr/local/nagios/etc/servers/STBM.cfg

and change the command (*check_command*) for checking the disk space like this:

```
define service {

    use                     local-service           ; Name of service template to use
    host_name               STBM
    service_description     Root Partition
    check_command           check_nrpe!check_xvda1
}
```

Similarly, you can change the command for checking the currently logged in users to `check_nrpe!check_users` and so on. To break this down a little bit: it uses the previously defined *check_nrpe* command with the argument `check_users` (after the `!`) that is defined in the `nrpe.cfg` file on the `STBM` machine - as far as my knowledge goes.

You shall verify your Nagios configuration files by running 

> $ /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

on your `NAGIOS` machine. If there are errors, fix them. If everything is fine, restart Nagios using

> $ sudo service nagios restart

## 4. Check Nagios server in your browser

Visit the nagios endpoint of your `NAGIOS` instance (*http://**x.x.x.x**/nagios*). Now, under the *Hosts* and *Sevices* tab on the left, our good ol' STBM can also be found. Can this journey be over already? Not even close.

## 5. Setting up an email service if the root partition check turns red

### Installing mail

First, we need to install `mailutils` so we can use /bin/mail on our system.

> $ sudo apt install -y mailutils

A Postfix Configuration prompt will be shown during the installation, feel free to choose the default values ("*Internet Site*" as type of mail configuration, and the default system mail name).

Now you can send emails from the terminal using the following sytax:

> $ echo "testing message" | mail -s "message subject" username@example.com

Check the email client you sent your test message to, especially look into your Spam folder, as the test message most probably landed there.

### Setting up an event handler in Nagios

Next, we will set up an event handler in `NAGIOS` which will execute a command similar to the previous one whenever the disk usage is over 60% on our `STBM` instance. To do this, we need to further modify our `commands.cfg` and `STBM.cfg` files, and we will also write a shell script that will send us an email.

Let's start with the command. Open the previously edited `commands.cfg` file by running

> $ sudo nano /usr/local/nagios/etc/objects/commands.cfg

and add the following lines somewhere in the file, so the `send_email` command would call our not yet existing `send_email.sh` script in our not yet existing `scripts` directory, and pass 4 arguments to it.

```
define command{

    command_name send_email
    command_line /usr/local/nagios/etc/scripts/send_email.sh $SERVICESTATE$ $HOSTNAME$ $HOSTADDRESS$
}
```

Make this script exist by creating a `scripts` directory in `/usr/local/nagios/etc`, and creating the `send_email.sh` file in it:

> $ sudo mkdir /usr/local/nagios/etc/scripts && cd /usr/local/nagios/etc/scripts

> $ sudo touch send_email.sh && sudo nano send_email.sh

Put a script similar like this inside:

```
#!/bin/sh
#
# Event handler script for sending an email when a state reaches a value
#
# What state is the HTTP service in?
case "$1" in
OK)
	# Everything is OK, so don't do anything...
	;;
WARNING)
    # Aha!  The disk service appears to have a problem - perhaps we should send the warning email...
	mail -s "$2 (IP $3) disk warning" youremail@gmail.com
	;;
UNKNOWN)
	# We don't know what might be causing an unknown error, so don't do anything...
	;;
CRITICAL)
	# Aha!  The disk service appears to have a problem - perhaps we should send the warning email...
    mail -s "$2 (IP $3) disk critical" youremail@gmail.com
	;;
esac
exit 0
```
This will use the first argument we pass (`$SERVICESTATE$`) as `$1` to determine the current state of the disk, and if its state is *warning* or *critical*, it sends an email to the above specified email address with the name (`$2`) and the IP-address (`$3`) of the specific host (in this case, the `STBM`). You should make this script runnable with the following command:

> $ sudo chmod +x /usr/local/nagios/etc/servers/scripts/send_email.sh

What is still miss, is to add this event listener to the Root Partition service in our `STBM.cfg` file. So open it,

> $ sudo nano /usr/local/nagios/etc/servers/STBM.cfg

and add the `event_handler` line to the `Root Partiton` service:

```
define service {

    use                     local-service           ; Name of service template to use
    host_name               STBM
    service_description     Root Partition
    check_command           check_nrpe!check_xvda1
    event_handler           send_email
}
```

Lets verify if the check_xvda1 command does what we want on our `STBM` machine. Open the `nrpe.cfg` file with

> $ sudo nano /usr/local/nagios/etc/nrpe.cfg

and look at its line 302 (we already modified it once):

`command[check_xvda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/xvda1`

The percentages after the `-w` and `-c` flags mean that its state becomes *warning* as soon as only 20% free space is left, and it becomes critical when only 10% is left. Our task is to send email when it is 60% full, so we should write 40% instead of the 20%. If we would like to try out whether it works, we can set a higher number, for example 95% first.

Restart the nrpe on `STBM` with

> $ sudo service nrpe restart

and the nagios service on `NAGIOS` with

> $ sudo service nagios restart

And thats it, I believe!