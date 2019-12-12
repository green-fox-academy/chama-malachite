# How to deploy Java web apps to Elastic Beanstalk

## Introduction

This document is designed to help developers deploy their Java web applications to Elastic Beanstalk via the AWS CLI.

## Prerequisites

- You have a Java web app ready in .WAR format.
- You have an AWS account and its key pair.
- You have Python 3 installed.
- You have Pip (Python package manager) installed.

## Install AWS CLI

Use the following terminal command to install the AWS command line. If Pip is installed, it should work on all operating systems.

> $ pip3 install awscli --upgrade --user

To verify if AWS CLI is installed correctly, run the following command:

> $ aws --version

You should see an output similar to this one:

> aws-cli/1.16.298 Python/3.7.5 Linux/5.3.13-300.fc31.x86_64 botocore/1.13.34

## Configuring AWS

As the next step, you need to configure the AWS CLI to use your credentials. Use the following command:

> $ aws configure

You will see four prompts pop us, asking you to enter your key pair, default server location and output format:

> AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE

> AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

> Default region name [None]: us-east-1

> Default output format [None]: table

The output formats can be: json, text or table.

## Using the Elastic Beanstalk service in AWS CLI

### Command structure

AWS CLI commands typically follow the following format:

> aws service-name command

For example, you could view the list of available commands for the Elastic Beanstalk service using the help command:

> aws elasticbeanstalk help

You can query all your currently deployed applications using the following command (it prints the results in the configured output format):

> aws elasticbeanstalk describe-applications

### Deploying the .WAR file to Elastic Beanstalk

Deploying a .WAR file to Elastic Beanstalk via the AWC Console in a browser is a straightforward process. You just need to click "create new web application", choose Java or Tomcat as a platform, upload your .WAR file, and click Upload.

What this does in the background is that it creates an Amazon S3 (Simple Storage Service) bucket, in which it deploys the application, and links it to Elastic Beanstalk. We need this S3 bucket if we want to upload our program via the AWS CLI.

To upload a Java web app via the AWS CLI, follow the steps in the example below. Let's suppose you already have a Java web app running called 'hello-world', which is uploaded to an AWS server in the eu-central-1 region.

#### Step 1: Find the S3 bucket the program runs in

> aws elasticbeanstalk describe-application-versions --application-name hello-world

The output of the command above will contain a line called "S3Bucket" : "elasticbeanstalk-eu-central-1-XXXXXX". This is the address of the S3 bucket we need to upload our .WAR to.

#### Step 2: Upload the .WAR file to the S3 bucket

> aws s3 cp ./hello-world.war s3://elasticbeanstalk-eu-central-1-XXXXXX/s3-hello-world.war

With the above command, we've uploaded our "hello-world.war" file to S3 as "s3-hello-world.war"

#### Step 3: Create a new application version

> aws elasticbeanstalk create-application-version --application-name hello-world --version-label s3-upload --source-bundle S3Bucket=elasticbeanstalk-eu-central-1-XXXXXX,S3Key=s3-hello-world.war

With the above command, we've created a new version for our hello-world app, with the version name "s3-upload". However, we still need to tell Elastic Beanstalk to use this new version. For this, we have to update the environment.

#### Step 4: Get application environment name

> aws elasticbeanstalk describe-environments --application-name hello-world

To identify which environment do we need to update, run the above command. The output will contain a line "EnvironmentName": "hello-world-env", this is our environment name.

#### Step 5: Update the environment

> aws elasticbeanstalk update-environment --environment-name hello-world-env --version-label s3-upload

With this command we tell Elastic Beanstalk to deploy our new version of the app, called "s3-upload". This command will return an output that contains a line "VersionLabel": "s3-upload". If the version label matches the new version we've created in Step 3, the update was successful.

#### Step 6: Verify app health

Re-run the command in Step 4, that describes the environment. The output contains a line "Health": "Green". If the health is green, then the deployment was successful.


