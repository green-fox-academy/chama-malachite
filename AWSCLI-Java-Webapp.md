#How to deploy Java web apps to Elastic Beanstalk

##Introduction

This document is designed to help developers deploy their Java web applications to Elastic Beanstalk via the AWS CLI.

##Prerequisites

- You have a Java web app ready in .WAR format.
- You have an AWS account and its key pair.
- You have Python 3 installed.
- You have Pip (Python package manager) installed.

##Install AWS CLI

Use the following terminal command to install the AWS command line. If Pip is installed, it should work on all operating systems.

> $ pip3 install awscli --upgrade --user

To verify if AWS CLI is installed correctly, run the following command:

> $ aws --version

You should see an output similar to this one:

> aws-cli/1.16.298 Python/3.7.5 Linux/5.3.13-300.fc31.x86_64 botocore/1.13.34

##Configuring AWS

As the next step, you need to configure the AWS CLI to use your credentials. Use the following command:

> $ aws configure

You will see four prompts pop us, asking you to enter your key pair, default server location and output format:

> AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
> AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
> Default region name [None]: us-east-1
> Default output format [None]: table

The output formats can be: json, text or table.

##Using the Elastic Beanstalk service in AWS CLI

###Command structure

AWS CLI commands typically follow the following format:

> aws service-name command

For example, you could view the list of available commands for the Elastic Beanstalk service using the help command:

> aws elasticbeanstalk help

You can query all your currently deployed applications using the following command (it prints the results in the configured output format):

> aws elasticbeanstalk describe-applications

###Deploying the .WAR file to Elastic Beanstalk

Deploying a .WAR file to Elastic Beanstalk via the AWC Console in a browser is a straightforward process. You just need to create a new application, choose Java, upload your .WAR file, and click Upload & Deploy.

However, what this does in the background is that it creates an Amazon S3 (Simple Storage Service) bucket, in which it deploys the application, and links it to Elastic Beanstalk.


