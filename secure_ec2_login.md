# Log into an EC2 server using a username and password combination

## Prerequisites

- AWS account
- AWS CLI configured
- Terraform
- The ability to use Terraform (peruse the Terraform documentation by akijakya if lacking)

## Step 1

Create an AWS EC2 instance via Terraform, using an Ubuntu AMI preferably (though CentOS is fine as well).
Allow SSH communication via port 22.
Specify an init.sh shell script as user_data.

## Step 2

Let's write the shell script now.

```
#!/bin/bash
cd /etc/ssh
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' sshd_config
sudo service ssh restart
sudo useradd -m -p $(openssl passwd -1 $password) $username
```

In case you are running a CentOS based distro, substitute
```
sudo service ssh restart
```
for
```
sudo service sshd restart
```

Let's break it down.
To allow username & password authentication, you need to modify the sshd_config in the /etc/ssh directory.
You need to change `PasswordAuthentication no` to `PasswordAuthentication yes`.
The standard Linux program sed is used to replace parts of the file through the terminal.
Then, the ssh daemon needs to be restarted for the configuration change to take place.
Finally, create a new user by specifying username and password.

## Step 3

Terraform apply time!
You can now log in using ssh.

```
ssh $public_ip -l $username
```
