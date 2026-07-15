# AWS Continuous Compliance with Configuration and Simple Notification Service (SNS)

Project Overview

To implement continuous monitoring in AWS that detects public S3 bucket exposure and automatically sends email alerts.

### Prerequisites

- Active AWS account
- Valid Email address for SNS Alerts

### Step 1: Enable AWS Config

1. Open **AWS Management Console**
2. Navigate to **AWS Config**
3. Click **Get started**
4. Under **Resource Method > Recording Strategy choose**
    - Select **All Resource Type with Customizable Overrides**
    - For Default Settings choose Continuous Recording
    - Don’t add any Overrides.
5. Under **Data governance**:
    - Select **Use an existing AWS Config service-linked role**
6. Under **Delivery channel**:
    - Choose Create a bucket → I created an S3 Bucket called: **paul-bucket-drop**
    - SNS topic → **Leave unset for now**
7. Click Next
AWS Config is now **actively recording supported resources in that region (eu-north-1)

![AWS Config: Recording Method:](<Screenshot/image.png>) AWS Config: Recording Method: Select **All Resource Type with Customizable Overrides**

![ Delivery Channel: I created an S3 Bucket called: paul-bucket-drop](<Screenshot/image 1.png>) Delivery Channel: I created an S3 Bucket called: paul-bucket-drop

### **Step 2.1: Add a Rule & Configure the Rule**

After Clicking Next, you will see Step 2: Rule by the Left hand side of the dashboard,

Under AWS Managed Rules, Search for ***“s3-bucket-public-read-prohibited”***  Tick the rule and click on next,

[This rule checks whether **any S3 bucket allows public READ access]**

Then Click on Confirm to save the settings, status becomes:

- **COMPLIANT (Green)** if private
- **NON_COMPLIANT (Red)** if public

![AWS Managed Rule: ***s3-bucket-public-read-prohibited”***](<Screenshot/image 2.png>) AWS Managed Rule: ***s3-bucket-public-read-prohibited”***

![Review the AWS Config and Click on Confirm](<Screenshot/image 3.png>) Review the AWS Config and Click on Confirm

![AWS Config Dashboard has been Setup Successfully](<Screenshot/image 4.png>) AWS Config Dashboard has been Setup Successfully

![AWS Managed Rule: ***s3-bucket-public-read-prohibited”***](<Screenshot/image 5.png>) AWS Managed Rule: ***s3-bucket-public-read-prohibited”***

### **Step 3: Create SNS (Email Alert System)**

Open Amazon SNS using the Search bar , Search for SNS, then Click Amazon SNS (Simple Notification Service)

Then Click on Start with an Overview, Click on Topics → Create Topic → Select Type: Standard → Choose a Topic Name: (**Paul-Security-Alerts)** and click on Create Topic

Leave the other settings untouched, you fill in the display name (I created this to to trigger email alerts for compliance reporting)

![SNS Topic Created Successfully](<Screenshot/image 6.png>)

SNS Topic Created Successfully

### **Step 3.1 : Create Email Subscription**

While on the on the topic details page, click **"Create subscription"**

Configure the following

- **Protocol**: Select **"Email"**
- **Endpoint**: Enter your personal email address

Click **Create subscription** Status will show **Pending confirmation**

- Check your email inbox (including spam/junk folders)
- Look for an email from **"AWS Notifications"** with subject like: **AWS Notification - Subscription Confirmation**
- You should see a confirmation page in your browser
- Return to SNS console - the subscription status should now show **"Confirmed"**

![Creation of Email Subscription for **Paul-Security-Alerts**](<Screenshot/image 7.png>) Creation of Email Subscription for **Paul-Security-Alerts**



![Email Confirmation](<Screenshot/image 8.png>) Email Confirmation



![Email Confirmation Status showing Confirmed on Topics Dashboard](<Screenshot/image 9.png>) Email Confirmation Status showing Confirmed on Topics Dashboard

### **Step 3.2: Connect AWS Config to SNS.**

Go to AWS Config Setting → Open **AWS Config →**In the left menu, click **Settings**

Under Data and Delivery Menu Click on Edit

Navigate to SNS, Tick “Stream configuration changes and notifications to an Amazon SNS topic”

Select Choose Topic from Account and select the drop down and choose “Paul-Security-Alerts”

![SNS Topic Name Attached Successfully ](<Screenshot/image 10.png>) SNS Topic Name Attached Successfully

### **Step 4: TESTING THE POLICY BY CREATING A PUBLIC S3 BUCKET**

- In the AWS Console, search for **S3**
- Click Create bucket using a unique name: **<BUCKET_NAME>**
- Region still remains eu-north-1, then uncheck “block all public access” also tick the disclaimer about “I acknowledge that the current settings might result in this bucket and the objects within becoming public.” and click on

We also need to add a public bucket policy to our created bucket

- Still under **Permissions →**Scroll to **Bucket policy →** Click **Edit →** Paste this policy below:

```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<BUCKET_NAME>/*"
    }
  ]
}
```

This policy makes all objects in the S3 bucket  publicly readable

"Principal": "*",- This means ALL Users including those on the internet have read access to the objects in this bucket.

### **Step 4.1 : Check AWS Config for Non- Compliance Alerts**

1. Open **AWS Config**
2. Click **Rules**
3. Click s3-bucket-public-read-prohibited

You should now see:

- Status: **NON_COMPLIANT (Red)**
- Resource: **<BUCKET_NAME>**

![AWS Config Alert Showing S3 Bucket is Non-Compliant](<Screenshot/image 11.png>) AWS Config Alert Showing S3 Bucket is Non-Compliant

### **Step 4.2: Email Alerts for  Non- Compliance Alerts**

![Email Alert showing Non- Compliance Status for S3 Bucket with Public Access](<Screenshot/image 12.png>) Email Alert showing Non-Compliant Status of S3 Bucket with Public Access

### **Step 4.3:  Fix: Make the Bucket Private so meet Compliance.**

The Fix would be to revert the policy because we enabled public read access hence we need to block public read access.

- Go to **Amazon S3 →**Open **<BUCKET_NAME> →**Go to the **Permissions** tab
- Scroll to **Bucket policy →**Click **Edit →Delete the entire policy →**Click **Save changes**
- Check **Block all public access →** Click **Save changes**

![Remove Public Access from S3 Bucket-  *<BUCKET_NAME>*](<Screenshot/image 13.png>) Remove Public Access from S3 Bucket-*<BUCKET_NAME>*

![Block all Public Access Enabled Successfully](<Screenshot/image 14.png>) Block all Public Access Enabled Successfully

The Rule has been updated from Non Compliant to Compliant, Same with the Email Alerts

![AWS Config Alert showing S3 Bucket is now Compliant](<Screenshot/image 15.png>) AWS Config Alert showing S3 Bucket is now Compliant

![Email Alert showing S3 Bucket is now Compliant](<Screenshot/image 16.png>) Email Alert showing S3 Bucket is now Compliant

The Email Alert below shows when deleted the Bucket Policy

![The Email Alert  shows the deletion of the Bucket Policy](<Screenshot/image 17.png>) The Email Alert  shows the deletion of the Bucket Policy


![This Captures the enabling of the block all public access box rule. ](<Screenshot/image 18.png>) This Email Alert Captures the enabling of the "block all public access" Bucket Settings rule


![Bucket is now Compliant](<Screenshot/image 19.png>) The Bucket is now Compliant

Conclusion.

This project demonstrated automated detection and remediation of insecure S3 configurations using AWS Config and SNS, ensuring continuous visibility and control over Cloud Compliance