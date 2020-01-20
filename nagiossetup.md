# Setting Up Nagios with Terraform on AWS
---
Setting up Nagios on an EC2 instance with Terraform

## Introduction
---
This documentation is written to help developers setting up Nagios from scartch on an AWS EC2 instance with a Terraform file.

## System requirements
---
- AWS CLI is configured on your machine. Follow these steps: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html
- Terraform is configured on your machine. Detailed installation can be found here: https://learn.hashicorp.com/terraform/getting-started/install.html
- A text editor of your choice (e.g.: Visual Studio Code, Brackets etc.)

## Create a new Terraform project
---
Create a new folder for your Terraform project on your computer.

You will need to create the following files in your project folder:
* my_test_key.pub
* .gitignore
* application.sh
* main.tf
* variables.tf

Let's start putting things in each of the above mentioned files:
---
First, we will generate a public key so we can ssh into the EC2 instance:
Open up another terminal window, and execute the following:
```
cd ~/.ssh/
ssh-keygen -t rsa -b 2048 -v
```
For the filename, let's just enter 'my_test_key', so that the full location of the private key is '~/.ssh/my_test_key' and the public key is located at '~/.ssh/my_test_key.pub' . Let's leave the passwords blank.
Back in the project directory, execute the following command:
```
cat ~/.ssh/my_test_key.pub > my_test_key.pub
```

Once you ran 'terraform apply', and pull the IP address out of the final output, you should be able to successfuly SSH into the machine using the following code:
```
ssh IPaddress -l ubuntu -i ~/.ssh/my_test_key
```
---

- .gitignore:
```
// Compiled files
*.tfstate
*.tfstate.backup
//Module directory
.terraform/
// Sensitive Files
/variables.tf
node_modules
```
---

- application.sh : this will run on the EC2 instance. It will create a user for the EC2 server that you can use to ssh to the instance. It will download and install Nagios on the EC2 instance, then create a username and password for Nagios so you will be able to login to: IPaddress/nagios

```
#!/bin/bash
cd /etc/ssh
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' sshd_config
sudo service ssh restart
sudo useradd -m -p $(openssl passwd -1 PASSWORD) USERNAME
sudo apt-get update
sudo apt-get install -y autoconf gcc libc6 make wget unzip apache2 php libapache2-mod-php7.2 libgd-dev
cd /tmp
wget -O nagioscore.tar.gz https://github.com/NagiosEnterprises/nagioscore/archive/nagios-4.4.5.tar.gz
tar xzf nagioscore.tar.gz
cd /tmp/nagioscore-nagios-4.4.5/
sudo ./configure --with-httpd-conf=/etc/apache2/sites-enabled
sudo make all
sudo make install-groups-users
sudo usermod -a -G nagios www-data
sudo make install
sudo make install-daemoninit
sudo make install-commandmode
sudo make install-config
sudo make install-webconf
sudo a2enmod rewrite
sudo a2enmod cgi
sudo ufw allow Apache
sudo ufw reload
sudo htpasswd -b -c /usr/local/nagios/etc/htpasswd.users NAGIOSUSERNAME NAGIOSPASSWORD
sudo systemctl restart apache2.service
sudo systemctl start nagios.service
sudo apt-get install -y autoconf gcc libc6 libmcrypt-dev make libssl-dev wget bc gawk dc build-essential snmp libnet-snmp-perl gettext
cd /tmp
wget --no-check-certificate -O nagios-plugins.tar.gz https://github.com/nagios-plugins/nagios-plugins/archive/release-2.2.1.tar.gz
tar zxf nagios-plugins.tar.gz
cd /tmp/nagios-plugins-release-2.2.1/
sudo ./tools/setup
sudo ./configure
sudo make
sudo make install
```
---

- main.tf 
```
provider "aws" {
  profile    = "default"
  region     = "eu-central-1"
}
resource "aws_key_pair" "PUBKEY" {
  key_name = "PUBKEY"
  public_key = file("my_test_key.pub")
}
resource "aws_security_group" "NAGIOSSECGROUP" {
  name        = "NAGIOSSECGROUP"
  description = "Open SSH, 80 and 3000"
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 3000
    to_port     = 3000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port       = 0
    to_port         = 0
    protocol        = "-1"
    cidr_blocks     = ["0.0.0.0/0"]
  }
}
resource "aws_instance" "example" {
  ami           = "ami-0cc0a36f626a4fdf5"
  instance_type = "t2.micro"
  key_name = aws_key_pair.PUBKEY.key_name
  security_groups = ["${aws_security_group.NAGIOSSECGROUP.name}"]
  user_data       = file("application.sh")
  tags = {
    Name = "Nagios"
  }
  provisioner "local-exec" {
    command = "echo ${aws_instance.example.public_ip} > ip_address.txt"
  }
}
```

- variables.tf: (swap everything with capital letters below,this file will be in the .gitignore file so you won't upload it to git. Never upload this data anywhere publically on the internet!!!)
---
```
variable "aws_access_key" {
  default = "WRITE_YOUR_AWS_ACCESSKEY_HERE"
}

variable "aws_secret_key" {
  default = "WRITE_YOUR_AWS_SECRETKEY_HERE"
}

variable "aws_region" {
  default = "WRITE_WHICH_REGION_YOU_WANT_TO_CREATE_YOUR_INSTANCE"
}
```

## Run Terraform
---
```
terraform init
```
```
terraform apply
```
Type yes when prompted and wait for the magic to happen.

Login to your AWS account,find your newly created instance. When AWS finished initializing, use the IPv4 Public IP/nagios endpoint to access Nagios with the NAGIOSUSERNAME NAGIOSPASSWORD you created in the application.sh file.

If you don't need your instance anymore, you can simply destroy it by typing the following in your project's terminal:
```
terraform destroy
```
Type yes after prompted.
---

## Enjoy!


