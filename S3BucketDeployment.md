# Deploying an application to an S3 Bucket on AWS

## Scope:
Green Fox Academy, Project phase

## Purpose:
Providing instructions to upload applications to an S3 bucket in Amazon Web Services.

## References:
1. [AWS EBS Command Line Tool Install for Linux and MacOS](https://github.com/aws/aws-elastic-beanstalk-cli-setup)

## Description:

In order to upload an application to a S3 bucket in Amazon Web Services, follow the instructions below:

1. Utilizing the browser's graphical interface
* Log into AWS via your browser, using the supplied username (malachite3) and password (Malacka100).
* Open the Services tab on the header, and find the S3 option under Storage, click on it.
* Click on Create Bucket - the Create Bucket popup window opens.
*	Add a bucket name and select the region (EU - Frankfurt is suggested), then click Next.
*	Click through the Configure Optios tab.
*	On the Set Permissions tab, uncheck the Block all public access field, then hit Next.
*	Review the details and click on Create Bucket.

*	Find your bucket in the list and click on its name to open it.
*	Click on the Upload button on the left side to upload files and folders into the bucket.
*	Drag and drop the files into the area, then click Next.
*	On the Set Permissions tab, set the Manage Public Permissions to Grant public read access to this object, then click Next.
*	Keep the Storage class on standard, click Next. Click on Upload on the next screen.

2. Use the AWS CLI
*	Log into the AWS CLI
*	List all the S3 buckets linked to your account:
```
aws2 s3 ls
```
*	Move your files and folders over to the S3 bucket with a command similar to below:
```
aws2 s3 cp '~/Documents/index.html' s3://my-bucket/
```


* If you want to host a static website, go to the Properties tab.
*	Click on Static Website Hosting.
*	Select "Use this bucket to host a website". Enter the filepath to the index.html to both Index document and Error document. The website's endpoint is already shown above the selection buttons. Click Save.
*	Re-open the Static Website Hosting tab to find the Endpoint again.

