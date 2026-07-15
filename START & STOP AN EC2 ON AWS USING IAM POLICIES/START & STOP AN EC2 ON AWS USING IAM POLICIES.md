# START & STOP AN EC2 ON AWS USING IAM POLICIES.

### PRE-REQUISITES

1. Functional AWS Account
2. Access to the AWS Console
3. An Existing EC2 Instance.

### GOAL

We’re creating an IAM permission set that allows **only**:

✅ Start EC2 instances

✅ Stop EC2 instances

❌ No create

❌ No terminate

❌ No modify

❌ No delete

An EC2 “Start / Stop only” user needs **exactly two permissions**:

- ec2:StartInstances
- ec2:StopInstances

### STEP 1: Create an IAM User to operate the EC2 Instance.

- Go to **IAM → Users**
- Click **Create user**
- Username: ec2-operators
- **Do NOT enable console access** (CLI/API only is cleaner for labs)
- Click **Next**
- I will attach the policies manually

![image.png](image.png)

**STEP 2: Create a Custom Policy using the Visual Editor**

After clicking Next, You will see the  **Set permissions page**

1. Select **Attach policies directly**
2. Click **Create policy**
3. Choose **Visual editor**

### **STEP 3 Configure the EC2 permissions (this is the core)**

**1.** Choose the service then select **EC2**

![Select Service and choose EC2](image%201.png)

Select Service and choose EC2

Choose actions: Expand Actions and Click on Write

Then Tick ONLY

- StartInstances
- StopInstances

Do **NOT** select:

- TerminateInstances ❌
- RunInstances ❌
- RebootInstances ❌
- ModifyInstanceAttribute ❌

![Expand Write Actions](image%202.png)

Expand Write Actions

![Scroll down and Tick Only StartInstances & StopInstances.](image%203.png)

Scroll down and Tick Only StartInstances & StopInstances.

Then Click on Resources and choose a specific EC2 that’s already existing, to configure this, 

[I will have to retrieve the instance ID of the EC2 Instance from searching EC2 → Instances → choosing the Specific Instance and checking for the instance ID then copy it.]

![image.png](image%204.png)

Then head back to  Resources on the Create Policy Page then do the following using the Visual Editor:

- Select **Specific**
- Under **Instance**, click **Add ARN**
- Fill it in like this:
- **Region**:: The region your instance is in (e.g. mine is eu-north-1)

![image.png](image%205.png)

- **Resource Instance** : Paste the Instance ID for the EC2 you copied earlier then click on Add ARNs

You will have the below screenshot after filling all the details.

![Specific ARN for EC2 with Instance ID: i-09201cc89e07ecdb6](image%206.png)

Specific ARN for EC2 with Instance ID: i-09201cc89e07ecdb6

After Adding the ARN, then click on Next.

![After Adding the ARN, then click on Next.](image%207.png)

After Adding the ARN, then click on Next.

You will see the Review and Create Page, Kindly fill in the Name of the Policy and Description

Name: ec2-start-stop

Description: This Policy allows an IAM user called ec2-operators to start and stop an instance called Wazuh

Then click on create policy.

![You will see the Review and Create Page, Kindly fill in the Name of the Policy and Description](image%208.png)

You will see the Review and Create Page, Kindly fill in the Name of the Policy and Description

![Policy created successfully.](image%209.png)

Policy created successfully.

When you click on View Policy, you can see the JSON Version of what we created by clicking on Policy Version and clicking on the Version 1 (+) plus sign to open it

![image.png](image%2010.png)

{
"Version": "2012-10-17",
"Statement": [
{
"Sid": "VisualEditor0",
"Effect": "Allow",
"Action": [
"ec2:StartInstances",
"ec2:StopInstances"
],
"Resource": "arn:aws:ec2:eu-north-1:178735646603:instance/i-09201cc89e07ecdb6"
}
]
}

### STEP 4:  Attach Policy to User (ec2-operators)

We are going to attach the Policy we created to the IAM User we also created earlier called “ec2-operators”.

Kindly head back to the set permissions page where we were creating the IAM user and we would see the policy we just created, if you cannot see it click on the refresh policies icon.

![If you cannot see the policy you created then click on the refresh policies icon.](image%2011.png)

If you cannot see the policy you created then click on the refresh policies icon.

From the screenshot above, we can see the policy we just created, we would tick it and click on next.

![Tick the Created policy and click on Next](image%2012.png)

Tick the Created policy and click on Next

The Review & Create Page will be loaded then you can review then click on the Create User Button

![Click on Create User after Reviewing the details of the User and the attached policy.](image%2013.png)

Click on Create User after Reviewing the details of the User and the attached policy.

![User has been created successfully.](image%2014.png)

User has been created successfully.

### STEP 5: Create Access Key for IAM User and Test Policy using Ubuntu.

To Create the Access Keys for the IAM User, Kindly click on View from the User Created Successfully Banner.

Then Click on Create Access Key, we are doing this because we didn’t assign the IAM User access to the AWS Management Console.

![Create Access Key for IAM User (ec2-operators)](image%2015.png)

Create Access Key for IAM User (ec2-operators)

You can follow the steps in this AWS documentation: https://docs.aws.amazon.com/keyspaces/latest/devguide/create.keypair.html

When you click on create access key, you will be redirected to this page, Kindly select the CLI option which is best for the IAM User

For this, you will need to install the AWS CLI V2 on the Machine you intend to use

![Choose CLI and Click on Next](image%2016.png)

Choose CLI and Click on Next

![Fill in the Description for the purpose of creating the access key](image%2017.png)

Fill in the Description for the purpose of creating the access key

Click on Create Access Key and download the CSV file and store properly, Click on Done to exit the Interface

Testing the Policy with the IAM User on Ubuntu Machine

Recall while creating the Access Keys for the IAM User (ec2-operators), we would need to install AWS CLI V2 to access the EC2 to test our policy, let’s install that on our Ubuntu Machine.

To Install AWS CLI V2, follow this guide: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

Run this command on your Ubuntu Terminal to install it

```yaml
curl "[https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip](https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip)" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

aws --version #This verifies the version of the AWS CLI.

![Install AWS CLI V2 on Ubuntu Machine](image%2018.png)

Install AWS CLI V2 on Ubuntu Machine

Then let’s sign in with the IAM User and test our policies.

We need to login to our IAM User with the commands below

```yaml
aws configure #it will prompt for the access keys for the IAM User.
```

```yaml
aws sts get-caller-identity #this verifies if i am logged in

```

![logging in to our IAM User on Ubuntu](image%2019.png)

logging in to our IAM User on Ubuntu

As you can see above we are logged in to our IAM User (ec2-operators) then we  can run our test.

### TEST 1: START INSTANCE

So we want to start our instance with instance ID: i-09201cc89e07ecdb6 by running this command below:

aws ec2 start-instances --instance-ids i-09201cc89e07ecdb6

![Instance started successfully](image%2020.png)

Instance started successfully

From the response above, it shows that the EC2 Instance was powered off and its current state means it is in the process of powering up.

Checking from the AWS Console using my IAM User with Admin Access, the instance is running.

![Instance was started and currently running when confirmed by IAM User with Admin Access](image%2021.png)

Instance was started and currently running when confirmed by IAM User with Admin Access

### TEST 2: CHECK INSTANCE STATUS & TERMINATE INSTANCE

So i want to stop the  instance with instance ID: i-09201cc89e07ecdb6 by running this commands

Terminate Instance

aws ec2 terminate-instances --instance-ids i-09201cc89e07ecdb6

![Unauthorized Operation when terminating  instance.](image%2022.png)

Unauthorized Operation when terminating  instance.

Response:

 (UnauthorizedOperation) when calling the TerminateInstances operation: You are not authorized to perform this operation. User: arn:aws:iam::178735646603:user/ec2-operators is not authorized to perform: ec2:TerminateInstances on resource: arn:aws:ec2:eu-north-1:178735646603:instance/i-09201cc89e07ecdb6 because no identity-based policy allows the ec2:TerminateInstances action. 

From the response we can see the IAM User isn’t authorized to perform the operation to terminate this specific instance.

Check Instance Status.

To check the Instance status, we would use this command

aws ec2 describe-instances --instance-ids i-09201cc89e07ecdb6

![Unauthorized Operation when checking instance status](image%2023.png)

Unauthorized Operation when checking instance status

Response: An error occurred (UnauthorizedOperation) when calling the DescribeInstances operation: You are not authorized to perform this operation. User: arn:aws:iam::178735646603:user/ec2-operators is not authorized to perform: ec2:DescribeInstances because no identity-based policy allows the ec2:DescribeInstances action

From the response we can see the IAM User isn’t authorized to perform the operation to terminate this specific instance.

### TEST 3: STOP INSTANCE

To stop this instance, we would run this command:

aws ec2 stop-instances --instance-ids i-09201cc89e07ecdb6

![Instance was stopped](image%2024.png)

Instance was stopped

From the Response above, it shows the previous state was running but it’s being stopped

Validating using an IAM User with Admin Access shows it has actually stopped.

![Instance was stopped when confirmed with IAM User with Admin access.](image%2025.png)

Instance was stopped when confirmed with IAM User with Admin access.