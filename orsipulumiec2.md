# Creating multiple EC2 instances with Pulumi

## Introduction

This documentation is written to help developers starting out with code as infrastructure tools get going with Pulumi and AWS.

## System requirements

- AWS CLI is configured on your machine. Follow these steps: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html
- Node.js 8 or more recent (optional) - if you are planning to use Pulumi with JS.

## Install Pulumi

Install Pulumi.

> $ brew install pulumi / https://www.pulumi.com/docs/get-started/aws/install-pulumi/

Restart your terminal and check if the installation has completed successfully.

> $ pulumi version

## Create a new Pulumi project

Create your Pulumi project in a new folder.

> $ mkdir pulumitut && cd pulumitut

If you are using JS:

> $ pulumi new aws-javascript

A prompt will redirect you to your browser as you need to log into your Pulumi account when you are using Pulumi for the first time.
After you've logged into Pulumi, the CLI will walk you through the project creation. You can specify project name, project description, stack name, and AWS region.
The region we currently use is:

> eu-central-1

If you are using JS, dependencies will be downloaded automatically at this stage.

## Deploying and destroying a Pulumi stack

To deploy our stack we can use the following command:

> $ pulumi up

You will be shown the resources that will be created, modified or deleted. Choosing yes will proceed and deploy the stack.

To remove the resources used by your Pulumi stack, use:

> $ pulumi destroy

To delete the entire stack after destroying the resources, use:

> $ pulumi stack rm

## Creating multiple EC2 instances with Pulumi, using JavaScript

### Step 1

Create a new Pulumi project. (See above)

> $ pulumi new aws-javascript

### Step 2

Open index.js and write the basic architecture.

```
const aws = require("@pulumi/aws");

let size = "t2.micro";     // t2.micro is available in the AWS free tier
let ami = aws.getAmi({
    filters: [{
      name: "name",
      values: ["amzn-ami-hvm-*"],
    }],
    owners: ["137112412989"], // This owner ID is Amazon
    mostRecent: true,
});

let group = new aws.ec2.SecurityGroup("webserver-secgrp", {
    ingress: [
        { protocol: "tcp", fromPort: 22, toPort: 22, cidrBlocks: ["0.0.0.0/0"] },
        { protocol: "tcp", fromPort: 80, toPort: 80, cidrBlocks: ["0.0.0.0/0"] },
        // ^-- ADD THIS LINE
    ],
});

You can add below your bash commands if you'd like:

let userData = // <-- ADD THIS DEFINITION
`#!/bin/bash
echo "Hello, World!" > index.html
nohup python -m SimpleHTTPServer 80 &`;

let server = new aws.ec2.Instance("web-server-www", {
    instanceType: size,
    securityGroups: [ group.name ], // reference the group object above
    ami: ami.id,
    userData: userData,             // <-- ADD THIS LINE
});

exports.publicIp = server.publicIp;
exports.publicHostName = server.publicDns;
```

The last two rows contain auxiliary code only, which prints information about the created server instance for the user.
```


```

### Step 3

Modify server creation so that Pulumi creates multiple servers during deployment.

```
let serverNames = ["dev", "test", "prod"];
let serverArray = [];

for (let i = 0; i < serverNames.length; ++i) {
    serverArray[i] = new aws.ec2.Instance(serverNames[i], {
        instanceType: size,
        securityGroups: [ group.name ], // reference the security group resource above
        ami: ami.id,
        userData: userData,
    });
}
```

### Step 4

Deploy your stack.

> $ pulumi up

Don't forget to clean up after yourself when you don't need the servers any more. :)

> $ pulumi destroy

To delete the stack itself, use:

> $ pulumi stack rm