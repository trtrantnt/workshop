---
title: "2. Preparation Steps"
chapter: false
weight: 2
---

## Prerequisites

### 1. AWS Account
- AWS Account with Administrator privileges
- Permission to create and manage AWS Organizations
- Permission to use IAM Identity Center

### 2. Required Knowledge
- Basic understanding of AWS IAM
- Experience with AWS Organizations
- Knowledge of compliance frameworks (SOX, SOC2, ISO27001)
- Understanding of Python and AWS CLI

### 3. Required Access
- Web browser (Chrome, Firefox, Safari, or Edge)
- Stable internet connection
- AWS Console access
- Administrative permissions in AWS account

## Environment Setup

### 1. Access AWS Console

1. Open your web browser and navigate to [AWS Console](https://console.aws.amazon.com/)
2. Sign in with your AWS account credentials
3. Ensure you have Administrator privileges

![AWS Console Login](images/aws-console-login.png)

### 2. Verify Account Permissions

1. In the AWS Console, click on your account name in the top-right corner
2. Select **My Account** to view account details
3. Verify you have access to:
   - AWS Organizations
   - IAM Identity Center
   - Administrative permissions

![Account Verification](images/account-verification.png)

### 3. Create S3 Buckets for Data Storage

1. Navigate to **S3** service in the AWS Console
2. Click **Create bucket**
3. Create first bucket:
   - **Bucket name**: `privilege-analytics-[YOUR-ACCOUNT-ID]`
   - **Region**: Select your preferred region
   - Keep default settings
   - Click **Create bucket**

![S3 Bucket Creation 1](images/s3-bucket-analytics.png)

4. Create second bucket:
   - **Bucket name**: `compliance-reports-[YOUR-ACCOUNT-ID]`
   - **Region**: Same as first bucket
   - Keep default settings
   - Click **Create bucket**

![S3 Bucket Creation 2](images/s3-bucket-compliance.png)

## Infrastructure Preparation

### 1. Enable AWS CloudTrail

1. Navigate to **CloudTrail** service
2. Click **Create trail**
3. Configure trail settings:
   - **Trail name**: `IdentityGovernanceTrail`
   - **S3 bucket**: Select the `privilege-analytics-[YOUR-ACCOUNT-ID]` bucket
   - **Log file prefix**: `cloudtrail-logs/`
4. Click **Create trail**

![CloudTrail Setup](images/cloudtrail-setup.png)

### 2. Enable AWS Config

1. Navigate to **AWS Config** service
2. Click **Get started**
3. Configure settings:
   - **Resource types**: Select **All resources**
   - **S3 bucket**: Create new bucket or use existing
   - **SNS topic**: Create new topic
4. Click **Next** and **Confirm**

![Config Setup](images/config-setup.png)

### 3. Enable Amazon GuardDuty

1. Navigate to **GuardDuty** service
2. Click **Get Started**
3. Click **Enable GuardDuty**
4. Review the service permissions and click **Enable**

![GuardDuty Setup](images/guardduty-setup.png)

## Expected Results

After completing the preparation steps:

- ✅ AWS Account properly configured
- ✅ Required AWS services enabled
- ✅ Base infrastructure deployed
- ✅ Permissions validated
- ✅ Workshop materials ready

## Next Steps

Continue to [3. Access Governance Setup](../3-thiet-lap-access-governance) to start implementing the system.