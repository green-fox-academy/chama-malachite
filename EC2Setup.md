# Setup an EC2 Computer and deploy a website on AWS

## Scope:
Green Fox Academy, Project phase

## Purpose:
Providing instructions to start an EC2 computer, install applications and deploy a website in Amazon Web Services.

## References:
1.	[Amazon Web Services Login](https://aws.amazon.com/console/)
1.	[AWS EBS Command Line Tool Install for Linux and MacOS](https://github.com/aws/aws-elastic-beanstalk-cli-setup)

## Description:

In order to set up a new EC2 server in Amazon Web Services, follow the instructions below:

1. Initialize the instance
* Log in to Reference 1. with the provided username and password
*	Open the Services menu on the top left, then click on EC2 under Compute
*	The EC2 console allows the review of running instances and starting new ones. Click on the orange Launch Instance button to initiate a new one.
*	The first step in the process is selecting the system for the EC2 instance. The instructions are based on selecting Ubuntu.
*	Choose an instance type that works for your application based on the estimated needs in RAM and CPU processing power. For a test type of instance, it is recommended to choose one that is free tier eligible. Click on Review and Launch.
*	We skipped to step 7. Click on Launch.
*	AWS asks if an existing authentication pair should be used to access the instance, or a new one. Select 'Create a new key pair'.
*	Add a descriptive key name, and download the .pem file to your computer.
*	Click on Launch Instances. AWS will start the EC2. Clicking on View Instances will show all active instances, including the one just started.

2. Connect to the EC2 instance via SSH
*	Click on the Connect button - it opens a popup window with instructions on how to connect to the ESC2 computer via SSH.
*	Run the chmod 400 'mykeyname'.pem command to make the .pem file readable.
*	To reach the EC2 computer, use the command below, replacing the keyname and instance names with yours' details:
```
ssh -i "mykeyname.pem" ubuntu@myamazonec2instance.compute.amazonaws.com
```
*	Before installing programs, you should update the existing apps:
```
sudo apt update
sudo apt upgrade
```
*	Install the needed programs. If Docker is required, getting it from [get.docker.com](get.docker.com) is recommended.
