# How to create multiple EC2 instances with Pulumi

## Introduction

This documentation is written to help developers starting out with code as infrastructure tools get going with Pulumi and AWS.

## Prerequisites

- AWS CLI is configured on your machine. You can find a detailed how-to in my AWSCLI Java Webapp Deployment documentation.
- Node.js 8 or more recent (optional) - if you are planning to use Pulumi with JS.
- Python 3.6 or more recent (optional) - if you are planning to use Pulumi with Python.

## Install Pulumi

Use the following terminal command to install Pulumi.

> $ curl -fsSL https://get.pulumi.com | sh

Restart your shell and check if the installation has completed successfully.

> $ pulumi version

## Create a new Pulumi project

Create your Pulumi project in a new folder.

> $ mkdir pulumitut && cd pulumitut

If you are using JS:

> $ pulumi new aws-javascript

If you are using Python:

> $ pulumi new aws-python

A prompt will redirect you to your browser as you need to log into your Pulumi account when you are using Pulumi for the first time.
After you've logged into Pulumi, the CLI will walk you through the project creation. You can specify project name, project description, stack name, and AWS region.
The region we currently use is:

> eu-central-1

If you are using JS, dependencies will be downloaded automatically at this phase.

If you are using Python, a virtual environment needs to be set up and the dependencies need to be installed manually using the following commands:

> $ virtualenv -p python3 venv

> $ source venv/bin/activate

> $ pip3 install -r requirements.txt

Our infrastructure code will be written in index.js in case of JS, and \_\_main\_\_.py in case of Python.

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
    ],
});

let server = new aws.ec2.Instance("webserver-www", {
    instanceType: size,
    securityGroups: [ group.name ], // reference the security group resource above
    ami: ami.id,
});

exports.publicIp = server.publicIp;
exports.publicHostName = server.publicDns;
```

The last two rows contain auxiliary code only, which prints information about the created server instance for the user.

### Step 3

Configure security group to your liking.
In this example we open port 80 of the servers that we are creating.

```
let group = new aws.ec2.SecurityGroup("webserver-secgrp", {
    ingress: [
        { protocol: "tcp", fromPort: 22, toPort: 22, cidrBlocks: ["0.0.0.0/0"] },
        { protocol: "tcp", fromPort: 80, toPort: 80, cidrBlocks: ["0.0.0.0/0"] },
    ],
});
```

### Step 4

Add user defined shell commands that should be executed on the newly created servers.

```
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
```

### Step 5

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

### Step 6

Deploy your stack.

> $ pulumi up

Don't forget to clean up after yourself when you don't need the servers any more. :)

> $ pulumi destroy

To delete the stack itself, use:

> $ pulumi stack rm

## Creating multiple EC2 instances with Pulumi, using Python

*"There are some who call me... Tim?!"*

Now we'll do the same with the might of Python.

### Step 1

Create a new Pulumi project.

> $ pulumi new aws-python

### Step 2

Create new Python virtual environment

```
$ virtualenv -p python3 venv
$ source venv/bin/activate
$ pip3 install -r requirements.txt
```

### Step 3

Open \_\_main\_\_.py and write the basic architecture.

```
import pulumi
import pulumi_aws as aws

size = 't2.micro'
ami = aws.get_ami(most_recent="true",
                  owners=["137112412989"],
                  filters=[{"name":"name","values":["amzn-ami-hvm-*"]}])

group = aws.ec2.SecurityGroup('webserver-secgrp',
    description='Enable HTTP access',
    ingress=[
        { 'protocol': 'tcp', 'from_port': 22, 'to_port': 22, 'cidr_blocks': ['0.0.0.0/0'] }
    ])

server = aws.ec2.Instance('webserver-www',
    instance_type=size,
    security_groups=[group.name], # reference security group from above
    ami=ami.id)

pulumi.export('publicIp', server.public_ip)
pulumi.export('publicHostName', server.public_dns)
```

The last two rows contain auxiliary code only, which prints information about the created server instance for the user.

### Step 4

Configure security group to your liking.
In this example we open port 80 of the servers that we are creating.

```
group = aws.ec2.SecurityGroup('webserver-secgrp',
    description='Enable HTTP access',
    ingress=[
        { 'protocol': 'tcp', 'from_port': 22, 'to_port': 22, 'cidr_blocks': ['0.0.0.0/0'] },
        { 'protocol': 'tcp', 'from_port': 80, 'to_port': 80, 'cidr_blocks': ['0.0.0.0/0'] }
    ])
```

### Step 5

Add user defined shell commands that should be executed on the newly created servers.

```
user_data = """
#!/bin/bash
echo "Hello, World!" > index.html
nohup python -m SimpleHTTPServer 80 &
"""

server = ec2.Instance('webserver-www',
    instance_type=size,
    security_groups=[group.name], # reference security group from above
    user_data=user_data, # <-- ADD THIS LINE
    ami=ami.id)
```

### Step 6

Modify server creation so that Pulumi creates multiple servers during deployment.

```
server_names = ["webserver-www-dev", "webserver-www-test", "webserver-www-prod"]
server_array = []
export_array = ["dev", "test", "prod"]


for i in range(3):
    server_array.append(aws.ec2.Instance(server_names[i],
        instance_type=size,
        security_groups=[group.name], # reference security group from above
        user_data=user_data, # <-- ADD THIS LINE
        ami=ami.id))
    
for i in range(3):
    pulumi.export('publicIp' + export_array[i], server_array[i].public_ip)
    pulumi.export('publicHostName' + export_array[i], server_array[i].public_dns)
```

### Step 7

Deploy your stack.

> $ pulumi up

Don't forget to clean up after yourself when you don't need the servers any more. :)

> $ pulumi destroy

To delete the stack itself, use:

> $ pulumi stack rm


