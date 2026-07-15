# Secure S3 Bucket with Read-Only Access

### PRE-REQUISITES

1. Functional AWS Account
2. I used AWS Default VPC 
3. Access to the AWS Console
4. An Existing EC2 Instance

### GOAL

A user should be able to:
- See a specific S3 bucket with read only access for the Billing Team
- List the files inside it
- Download objects

They **must NOT** be able to:
- Upload files
- Delete files
- Change permissions
- Touch any other bucket

### CREATING THE S3 STORAGE BUCKET VIA GUI 
Step 1: Login and access the AWS console then search for S3 and click on it as highlighted below:

![image.png](<screenshots/image.png>)

Step 2: Click on Create Bucket and fill in the following details as seen below:

Name: **mys3billingstorage**

AWS Region: I maintained the default AWS Region: Europe (Stockholm) eu-north-1

Object Ownership: I chose ACLs Disabled because i want to own the bucket alone.

I enabled Versioning for the following reason (Versioning is not backup—but it’s a powerful safety net.)

- To Protect against accidental deletes
- To Protect against overwrites
- To Support ransomware recovery

I also blocked Public Access, i don’t want it accessible via the Internet.

Then I used the default encryption and added the following tags: “IAM_Project & Cloud_Security_Labs”

Then Click on Create to create the S3 Bucket.

![image.png](<screenshots/image%201.png>)

![image.png](<screenshots/image%202.png>)

![image.png](<screenshots/image%203.png>)

![S3 Bucket (***mys3billingstorage**)* has been created successfully](<screenshots/image%204.png>) S3 Bucket (***mys3billingstorage**) has been created successfully

### GRANTING READ-ONLY ACCESS TO THE S3 BUCKET

For this we would need to access the IAM Dashboard on the AWS Console and choose IAM

![image.png](<screenshots/image%205.png>) From this interface, i want to create a User Group called Billing (because i want them to have read only access to the S3 bucket i created previously) by click on User Groups as seen below

![IAM User Groups](<screenshots/image%206.png>) IAM User Groups

Kindly name the User Group as “Billing_Team” scroll down and save

![Creation of User Group](<screenshots/image%207.png>) Creation of User Group

![User Group Saved Successfully](<screenshots/image%208.png>) User Group Saved Successfully

The Next Step is to create an IAM User and add them to Billing_Team User Group we Created.

We can do this by clicking on the Users, then we would create one and add them to the billing user group

![Create IAM User](<screenshots/image%209.png>) Create IAM User by Typing the username and click on next

I didn’t give this user access to the AWS Management Console because i want them to have only ready access to the S3 resource.

![Jane_Billing IAM User Creation](<screenshots/image%2010.png>) Jane_Billing IAM User Creation

Next Step is to add Jane_Billing into the Billing User Group, I will ignore ignore the Set Permission Boundary for now and choose that later.

![Add IAM User to User Group](<screenshots/image%2011.png>) Add IAM User to User Group

I added the same tags i used previously and clicked on create user

![Create IAM User](<screenshots/image%2012.png>)

![IAM User Successfully Created.](<screenshots/image%2013.png>) IAM User Successfully Created.

### CREATE A POLICY FOR THE IAM USER

To create a Policy, Kindly click on the Policies Tab and choose Create policy.

![image.png](<screenshots/image%2014.png>) The Policy can be created by using the Visual Editor or JSON format and the Service we are choosing is the S3 Bucket which should be ReadOnly.

![Specifying the Services (S3) for our Policy](<screenshots/image%2015.png>) Specifying the Services (S3) for our Policy

Kindly select the S3 Service from the drop down list as seen below:

![image.png](<screenshots/087cee53-dd13-489d-860f-f36c6a1134e7.png>)


Then configure the following for List Object:- i ticked list object to list the contents of the S3 Bucket (***mys3billingstorage***)

![Then configure the following for List Object:- i ticked list object to list the contents of the S3 Bucket (***mys3billingstorage***)](<screenshots/image%2016.png>)


Then configure the same for Read (i ticked GetObject) as seen below

![Then configure the same for Read (i ticked GetObject) as seen below](<screenshots/image%2017.png>)


Next Step is to input a name for the bucket and object under the resource tab

For Bucket i used the following: this allows actions like listbucket to view the contents of the bucket

![For Bucket i used the following: this allows actions like listbucket to view the contents of the bucket](<screenshots/image%2018.png>)


![For Object, i used the following: this allows the GetObject to download any object inside the bucket.](<screenshots/image%2019.png>)

For Object, i used the following: this allows the GetObject to download any object inside the bucket.

![Adding a ARN for out ](<screenshots/image%2020.png>)

After this, we will click on next to give the policy a name and a description and click on Create Policy.

Name: Billingsreadonlys3bucket

Description: I want the billing team to have read only access to this s3 bucket.

![Adding the Name and Policy Description](<screenshots/image%2021.png>) Adding the Name and Policy Description

![Policy Created successfully](<screenshots/image%2022.png>) Policy Created successfully

## ATTACH POLICY TO THE BILLING USER GROUP

To achieve this from the IAM Dashboard, Navigate to User Groups then Click on the Billing User Group and attach the policy.

![To achieve this from the IAM Dashboard, Navigate to User Groups then Click on the Billing User Group and attach the policy.](<screenshots/image%2023.png>)


Kindly select the policy we created then click on attach policy

![Kindly select the policy we created then click on attach policy](<screenshots/image%2024.png>) Policy has been attached successfully

![Policy has been attached Successfully.](<screenshots/image%2025.png>) Policy has been attached Successfully.

The Screenshot below shows the JSON Equivalent of our the Read only (List and GetObject) policy we created.

![The JSON Equivalent of our the Read only (List and GetObject) policy we created.](<screenshots/image%2026.png>) The JSON Equivalent of our the Read only (List and GetObject) policy we created.

### TESTING THE POLICY

Since i didn’t create an AWS console for my Billing user, i will be testing this using the AWS CLI.

I will also create an access Key for the account i used to access the AWS console (This user should have admin right to do this)

[Create new access keys for an IAM user - Amazon Keyspaces (for Apache Cassandra)](https://docs.aws.amazon.com/keyspaces/latest/devguide/create.keypair.html)

### INSTALL AWS CLI ON KALI MACHINE

Then I will initialize AWS console in order to access the resources within AWS like my S3 Storage via the CLI

I am creating access keys for Jane_Billing under the Billing_Team User Group, she doesn’t have AWS Console access, so she will be using the AWS CLI Version 2 from her Kali Linux Machine.

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

Command to Install on Kali or any Linux Distro:

```bash
curl "[https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip](https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip)" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

![IAM User does NOT have AWS Console access, but has Access Keys.](<screenshots/image%2027.png>) IAM User does NOT have AWS Console access, but has Access Keys.

To login to Jane_Billing on the AWS CLI V2 run this command:

```bash
aws configure
```

Then input the access keys for Jane_Billing

Then Run this command to verify the IAM user is logged in

```bash
aws sts get-caller-identity
```

So the AWS CLI v2 has been installed on the Kali Machine as seen below and it shows the IAM User linked to the Access key.

![Logging in with Access Keys for IAM User](<screenshots/image%2028.png>)

Logging in with Access Keys for IAM User

The Response above shows that Jane_Billing is confirmed logged in this Kali Machine.

I uploaded a text file to my S3 Storage from the S3 Dashboard using my IAM User with Admin Privileges so we can test using Jane_Billing IAM account.

![image.png](<screenshots/image%2029.png>)

### TEST: LIST OBJECTS IN THE S3 BUCKET

So Let’s check if Jane_Billing can list the content of the S3 Bucket

```bash
aws s3 ls s3://mys3billingstorage #This returns the content of the S3 Bucket

```

![image.png](<screenshots/image%2030.png>)

From the screenshot above, Jane was able to read the content of the S3 Bucket.

### TEST: DOWNLOAD  THE TEXT FILE FROM THE S3 BUCKET

Jane is able to download the Jay.txt file into her machine using the command below:

```bash
aws s3 cp s3://mys3billingstorage/Jay.txt ./ # download
```

![image.png](<screenshots/image%2031.png>)

### TEST: UPLOAD FILE TO S3 BUCKET

```bash
echo "testing write access" > test-write.txt
aws s3 cp test-write.txt s3://mys3billingstorage/
```

```bash
**upload failed: ./test-write.txt to s3://mys3billingstorage/test-write.txt An error occurred (AccessDenied) when calling the PutObject operation: User: arn:aws:iam::178735646603:user/Jane_Billing is not authorized to perform: s3:PutObject on resource: "arn:aws:s3:::mys3billingstorage/test-write.txt" because no identity-based policy allows the s3:PutObject action**
```
![image.png](<screenshots/image%2032.png>)

From the Output above, Jane doesn’t have access to upload files to the S3 Bucket