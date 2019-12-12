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
* Initialize a git repository with:
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
* Add a 'start' method to your package.json, following the parameters below:
	1. use node, not nodemon
	1. do not use server.js or app.js as your main file where the server starts
```
  "scripts": {
    "start": "node main.js",
  },

```
* Create your application, using port 8081 (AWS uses an engine called 'nginx' by default, which listens on this specific port).
* Once your application is ready, make sure there are no changes to be committed, then deploy the application with the following:
```
eb deploy
```
* You can find a link to your website through the AWS user's Elastic Beanstalk applications. Alternatively, typing in the code below the shell opens it in your favorite browser:
```
eb open
```
* If a revision is made, the changes can be manually pushed by running the deploy command again.
