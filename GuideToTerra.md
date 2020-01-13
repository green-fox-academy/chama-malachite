# Using Terraform to turn code into infrastructure!

## For those special occasions when you want to start 3 beautiful instances with dancing animals. Also, ***GRANNY*** compatible!

## Before you start with this tutorial, you'll need to do a few things in advance:
* If you're a grandmother, you might not know too much about computers - in that case, ask your grandchild for help and tell them to read these instructions carefully
* Make sure you have a basic understanding of Docker and prepare an image you will deploy
* Make sure you have a basic understanding of SSH as well, it is important
* For more information about the topics mentioned above, check out these sites below:

[Docker Installation Guide](https://docs.docker.com/v17.09/engine/installation/)
[Terraform Documentation - Very Useful!](https://learn.hashicorp.com/terraform/getting-started/intro)

## Guide:
* Install Terraform on your computer and configure it - don't forget 'init' !
* Check for errors and if the installation was correct or not
* Create your own -tf file for your project! First, set the correct provider, in our case AWS:
```
provider "aws" {
  profile    = "default"
  region     = "your region goes here"
}
```
* Next we need to create our resource:
```
resource "aws_instance" "name of your instance" {
  ami           = "ami-XXXXXX example" -> For this guide, pick an AMI for Ubuntu. Commands may differ depending on the AMI you use.
  instance_type = "t2.micro" -> Free Tier instance
  count         = "the amount of instances is specified here"
  key_name      = "the name of the .pem file you created" -> if you need to create a key pair, you can access this feature through the EC2 dashboard
  security_groups = ["name of the security group you created for your instances"] -> in case you don't have one, create your own at the EC2 dashboard or in your tf file
  user_data     = "link your script file here"

  tags = {
    Name  = "name your instance" -> optionally, you can create a file that has a list of names, and link it here
  }
}
```
* Let's create our Security Group! We need to open up a few ports to make the instance accessible:
```
resource "aws_security_group" "allow ssh http https" {
name="name of group"
description="description of group"
vpc_id="vpc goes here"

ingress {
from_port=22
to_port=22
protocol="tcp"
cidr_blocks=["0.0.0.0/0"]
}

ingress {
from_port=80
to_port=80
protocol="tcp"
cidr_blocks=["0.0.0.0/0"]
}

ingress {
from_port=443
to_port=443
protocol="tcp"
cidr_blocks=["0.0.0.0/0"]
}

egress {

from_port=0
to_port=0
protocol="-1"
cidr_blocks=["0.0.0.0/0"]

}
}
```
* All that's left is to install Docker on your instances and run your image on them! Don't forget to set the correct ports, and remember - don't install docker with snap command, it won't work properly!
* Below is an example for setting up docker for Ubuntu:
```
#!/bin/bash
sudo apt-get update
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install -y  docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
sudo docker run -d -p 80:80 nameofyourimage
```
* Now all that's left is to type 'terraform apply' and wait for the magic to happen~

## **Congratulations!** You've just created your own instances with Terraform and Docker! Wow!
