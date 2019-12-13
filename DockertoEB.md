# Deploying Docker Image to Elastic Beanstalk through AWSCLI

## For those special occasions when you want to feed your whale some beans. Now ***GRANNY*** compatible!

## Before you start with this tutorial, you'll need to do a few things in advance:
* If you're a grandmother, you might not know too much about computers - in that case, ask your grandchild for help and tell them to read these instructions carefully
* Make sure you have a basic understanding of Docker and prepare an image you will deploy
* Install and configure AWSCLI and EBCLI if it's not already installed - for these you'll need python and pip
* In case you need help, check out these sites below:
(https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html "AWSCLI Installing Guide")
(https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install-advanced.html "EBCLI Installing Guide")
(https://docs.docker.com/v17.09/engine/installation/ "Docker Installing Guide")

## Guide:
* Create a folder for your project, then change your directory in the console to that folder
* If you don't have a dockerfile prepared, create one - don't forget FROM and EXPOSE!
* Check if awscli and ebcli is installed on your computer by running the following command:
```
aws --version
eb --version
```
* If they're both installed correctly, we're good to go. Configure your AWSCLI with this command (if it's not already configured)
```
aws configure
```
* Now that everything is ready, let's initialize our Elastic Beanstalk:
```
eb init
```
* Then create the environment:
```
eb create
```
## **Congratulations!** You've just deployed your docker container to Elastic Beanstalk!