# Deploying NodeJS to EBS via AWSCLI

## Scope:
Green Fox Academy, Project phase, Node.js based projects

## Purpose:
Providing instructions to deploy Node.js (and Express) based applications to Amazon Web Services via Elastic Beanstalk.

## References:
1. [AWS EBS Command Line Tool Install for Linux and MacOS](https://github.com/aws/aws-elastic-beanstalk-cli-setup)
2. [Configuring the EB CLI Tool](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-configuration.html)

## Description:

In order to deploy a Node.js based application to Amazon Web Services using Elastic Beanstalk, follow the instructions below:

* Install the EBS Command Line tool (see References 1. for instructions)
* Make a folder for the project and cd into it
* Initialize npm and add all needed dependencies
```
npm init
npm i --save express
```
* Initialize a git repository with
```
git init
```
* Create an Elatric Beanstalk environment:
```
eb init --platform node.js --region eu-west-3
```
* Create a .gitignore and verify that it has the following added:
```
node_modules

# Elastic Beanstalk Files
.elasticbeanstalk/*
!.elasticbeanstalk/*.cfg.yml
!.elasticbeanstalk/*.global.yml

```
*
*
*
*
*
*
