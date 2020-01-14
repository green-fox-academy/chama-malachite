# How to create multiple EC2 instances with Terraform

## Introduction

This documentation is written to help developers starting out with code as infrastructure tools get going with Terraform and AWS.

## Prerequisites

- AWS CLI is configured on your machine. You can find a detailed how-to in the AWSCLI Java Webapp Deployment documentation.

## Install Terraform

You can search for and install Terraform with your package manager, or find the [appropriate package](https://www.terraform.io/downloads.html) for your system and download it. Terraform is packaged as a zip archive.

After downloading Terraform, unzip the package. Terraform runs as a single binary named terraform. Any other files in the package can be safely removed and Terraform will still function.

The final step is to make sure that the terraform binary is available on the PATH. See [this page](https://stackoverflow.com/questions/14637979/how-to-permanently-set-path-on-linux-unix) for instructions on setting the PATH on Linux and Mac. [This page](https://stackoverflow.com/questions/1618280/where-can-i-set-path-to-make-exe-on-windows) contains instructions for setting the PATH on Windows. The easiest way is to unzip the contents to a folder that is already in the PATH, e.g. to /usr/local/bin on Linux.

Restart your shell and check if the installation has completed successfully.

> $ terraform version

## Create a new Terraform project

Create your Terraform project in a new folder.

> $ mkdir terraform && cd terraform

Create at least one *.tf file in which you define at least one provider. To build infrastucture on AWS, you can use this sample code snippet:

```
provider "aws" {
  profile    = "default"
  region     = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"
}
```

## Deploying and destroying a Terraform infrastructure

First you need to initialize your terraform project by navigating to the folder with the *.tf files and run the following command:

> $ terraform init

This will initialize various local settings and data that will be used by subsequent commands.
You can test your project with the following command:

> $ terraform plan

This will show whether there are any problem in your *.tf files and shows the data on the respurces terraform will add, change and destroy, before deployment.
To deploy our stack we can use the following command:

> $ terraform apply

After a summary, you will be asked whether you want to perform the actions described above.
Only 'yes' will be accepted to approve, which needs to be typed in for the process to continue.

To remove the resources used by your Terraform infrastructure, use:

> $ terraform destroy

## Creating multiple EC2 instances with Terraform

In the following, we will take a look of the most important blocks needed to make a terraform project which creates multiple,  EC2 instances that are added to a Security Group and are accessible later via ssh protocol. The syntax that will be used is for the latest version of Terraform (v12).

### Step 1 - AWS provider

The first thing you need to specify is a provider, which is responsible for creating and managing resources. Multiple provider blocks can exist if a Terraform configuration is composed of multiple providers, which is a common situation. Now we only need AWS now, which needs a region and the credentials (access key, secret key) to be specified. This can be done multiple way, such as:

```
provider "aws" {
  region     = "eu-central-1"
  access_key = "my-access-key"
  secret_key = "my-secret-key"
}
```

However, if these credentials were specified previously in AWS CLI on your machine, Terraform looks up the necessary information in its default directory, if the keys are left out.

```
provider "aws" {
  region     = "eu-central-1"
}
```

You can optionally specify a different location for the credentials in the configuration by providing the shared_credentials_file attribute. 
The AWS profile can be a custom one, but here we can use the default value:

```
provider "aws" {
  region                  = "eu-central-1"
  shared_credentials_file = "/Users/tf_user/.aws/creds"
  profile                 = "default"
}
```

### Step 2 - AWS key-pair resource

In the next part we start to add resources to our terraform file. The resource block defines a resource that exists within the infrastructure, such as an EC2 instance. There are [informative documentations](https://www.terraform.io/docs/providers/aws/index.html) on all the resources in connection with an AWS EC2 instance. 

If we would like to access our server instances via ssh, first we will need to generate an ssh key on our machine, and then add a resource to our .tf file called aws_key_pair. 

#### Generating an ssh key

In your terminal, navigate to a directory where you would like to store your ssh keys (e.g. ~/Creds), and run the following command:

> $ ssh-keygen -o

It will open a promt in which you need to specify the nam of your key, in this case "terraform-test" for example. Next a passphrase will be asked from you but it is not necessary for now, you can skip it by pressing ENTER. Thus, your key pair has been generated which is obviously two files: a private key with no extension and a public key with .pub extension.

#### Adding aws_key_pair resource to the terraform file

The resource block has two strings before opening the block: the resource type and the resource name. In our example, the resource type is "aws_key_pair" and the name is "terraform-test". Within the block, the name of the key and the path of the public key (the one with the *.pub extension) has to be specified.

```
resource "aws_key_pair" "terraform-test" {
  key_name   = "terraform-test"
  public_key = file("~/Creds/terraform-test.pub")
}
```

### Step 3 - AWS Security Group resource

It is advised to add our instances to be created to a Security Group. Security Groups can be imported using the security group id, e.g.

> $ terraform import aws_security_group.elb_sg sg-903004f8

Or one can be created within the terraform project by adding an "aws_security_group" resource to our .tf file. Within the resource block, a name, description, ingress port(s) and an egress port needs to be specified. You can open as many port as you need. Port 22 is necessary for accessing our instances via ssh, port 80 is for http, port 443 is for https protocol.

```
resource "aws_security_group" "helloworld-group" {
  name        = "helloworld-group"
  description = "My security group for my helloworld application"
  
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
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Step 4 - AWS instance resource

Now we finally arrived at starting our EC2 instances! 

#### Adding one EC2 instance

Observe the following code snippet:

```
resource "aws_instance" "prod" {
  ami           = "ami-0cc0a36f626a4fdf5"
  instance_type = "t2.micro"
  key_name      = aws_key_pair.terraform-test.key_name
  user_data     = file("helloworld.sh")

  security_groups = [
    aws_security_group.helloworld-group.name
  ]

  tags = {
    Name = "server-prod"
  }
}
```

- Inside this block, the first thing is an Amazon Machine Image (AMI). The most easy way to find the code of a desired AMI is to navigate to the Instances page on the AWS website, check the top ribbon if we are located in the desired region, and launch a new instance, and copy the id of an AMI we want to use from the next page. In this case, the AMI of the latest Ubuntu 18.04 server x64 version is used.

- The second element is the instance type, in this case it is a t2.micro instance since it is free tier eligible.

- The third is the key name of which we initialize the instance to access it later via ssh. Here we use the one created in **Step 2** by referring to its name in the following format: *resource_type*.*resource_name*.*block_element*

- user_data is a path to a shell script, which will run at the start of our instance. The script can be used e.g. to install docker on the system, pull a docker image and start a container running our app on port 80.

- Next one is the security groups we want to add it to, referencing the one previously created in **Step 3** by referring to its name in the following format: *resource_type*.*resource_name*.*block_element*

- And the last in this case is the tags part, where only the Name is specified this time. This is what will appear on the AWS Instances page under the column "Name".

There are two other useful elements we can add inside our instance block, to copy files or directories, and running commands when the instance is started (instead of using the user_data element). The next code snippet demonstrates a use case when a shell script is copied to the instance and then we use commands to run it:

```
resource "aws_instance" "prod" {
  ami           = "ami-0cc0a36f626a4fdf5"
  instance_type = "t2.micro"
  key_name      = aws_key_pair.terraform-test.key_name

  security_groups = [
    aws_security_group.helloworld-group.name
  ]

  tags = {
    Name = "server-prod"
  }

  provisioner "file" {
    source      = "helloworld.sh"
    destination = "/tmp/helloworld.sh"
    connection {
      type = "ssh"
      host = self.public_ip
      user = "ubuntu"
      private_key = file("~/Creds/terraform-test")
    }
  }

  provisioner "remote-exec" {
    inline = [
      "chmod +x /tmp/helloworld.sh",
      "sh /tmp/helloworld.sh",
    ]
    connection {
      type = "ssh"
      host = self.public_ip
      user = "ubuntu"
      private_key = file("~/Creds/terraform-test")
    }
  }
}
```

- The "file" provisioner can be used to move files or folders from a source directory to a destination directory in our instance.

- The "remote-exec" provisioner can be used to run commands on the instance. The commands can be specified inline, or you can add the path to an existing shell script on your machine, like this:

```
  provisioner "remote-exec" {
    script = "helloworld.sh"
    connection {
      type = "ssh"
      host = self.public_ip
      user = "ubuntu"
      private_key = file("~/Creds/terraform-test")
    }
  }
```

Both of these provisioners need a connection in which you need to specify the path to your secret key, generated in **Step 2**.

#### Adding multiple EC2 instances

The easiest way to add multiple EC2 instances to your terraform project is to copy-paste the previous block as many times as it is needed. There is another way however, to loop through an array of instance names for example. This is demonstrated through a snippet from the main.tf file, and the content of the vars.tf file:

main.tf:

```
resource "aws_instance" "dev" {
  count         = var.instance_count
  ami           = lookup(var.ami,var.aws_region)
  instance_type = var.instance_type
  key_name      = aws_key_pair.terraform-test.key_name
  user_data     = file(element(var.instance_scripts, count.index))

  security_groups = [
    aws_security_group.akijakya-greeter.name
  ]

  tags = {
    Name  = element(var.instance_tags, count.index)  # element syntax: element(list, index)
  }
}
```

vars.tf:

```
variable "instance_count" {
  default = "3"
}

variable "ami" {
  type = map(string)

  default = {
    "eu-central-1" = "ami-0cc0a36f626a4fdf5"
  }
}

variable "instance_type" {
  default = "t2.micro"
}

variable "instance_scripts" {
  type = list(string)
  default = ["scripts/prod.sh", "scripts/test.sh", "scripts/dev.sh"]
}

variable "instance_tags" {
  type = list(string)
  default = ["server-prod", "server-test", "server-dev"]
}

variable "aws_region" {
  default = "eu-central-1"
}
```

The different part of the block are:

- count: the number of instances we would like to start, stored in the variable "instance_count" in the vars.tf file
- ami: the id of the AMI we would like to use, stored in the variable "ami" in the vars.tf file
- instance_type: the type of the instance we would like to start, stored in the variable "instance_type" in the vars.tf file
- key_name: the name of the key we initialize the instance with to access it later via ssh. Here we use the one created in **Step 2** by referring to its name in the following format: *resource_type*.*resource_name*.*block_element*
- user_data: the paths to the different shell scripts, stored in an array in the variable "instance_scripts" in the vars.tf file. The syntax of the element method is the following: element(list, index).
- security_groups: the security groups we want to add it to, referencing the one previously created in **Step 3** by referring to its name in the following format: *resource_type*.*resource_name*.*block_element*
- tags: the tags part, where only the Name is specified this time. This is what will appear on the AWS Instances page under the column "Name".