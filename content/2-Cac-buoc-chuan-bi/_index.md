---
title: "2. Preparation Steps"
chapter: false
weight: 2
---

## Environment Setup

### 1. Create S3 Buckets for Data Storage

1. Navigate to **Amazon S3** service in the AWS Console

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/2/s3b1.png?featherlight=false&width=90pc)

2. Click **Create bucket**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/2/s3b2.png?featherlight=false&width=90pc)

3. Create first bucket for analytics data:
   - **Bucket name**: `identity-governance-analytics`
   - **AWS Region**: Select your preferred region (e.g., us-east-1)
   - **Object Ownership**: ACLs disabled (recommended)

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/2/s3b3.png?featherlight=false&width=90pc)

   - **Block Public Access settings**: Keep all blocked (recommended)
   - **Bucket Versioning**: Enable
   - **Default encryption**: Server-side encryption with Amazon S3 managed keys (SSE-S3)
   - **Bucket Key**: Enable

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/2/s3b4.png?featherlight=false&width=90pc)

4. Click **Create bucket**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/2/s3b5.png?featherlight=false&width=90pc)

5. Create second bucket for compliance reports:
   - **Bucket name**: `identity-governance-reports`
   - **AWS Region**: Same as first bucket
   - **Object Ownership**: ACLs disabled (recommended)
   - **Block Public Access settings**: Keep all blocked (recommended)
   - **Bucket Versioning**: Enable
   - **Default encryption**: Server-side encryption with Amazon S3 managed keys (SSE-S3)
   - **Bucket Key**: Enable
   - **Object Lock**: Enable for compliance retention
6. Click **Create bucket**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/2/s3b6.png?featherlight=false&width=90pc)

7. Verify both buckets are created successfully:

![Navigate to S3](https://trtrantnt.github.io/workshop/images/2/s3b7.png?featherlight=false&width=90pc)

## Infrastructure Preparation

### 1. Enable AWS CloudTrail

1. Navigate to **CloudTrail** service in AWS Console
2. Click **Create trail**

![Open CloudTrail Console](https://trtrantnt.github.io/workshop/images/2/cloudtrailb1.png?featherlight=false&width=90pc)

![Open CloudTrail Console](https://trtrantnt.github.io/workshop/images/2/cloudtrailb2.png?featherlight=false&width=90pc)

#### Step 1: General details

3. Enter basic information:
   - **Trail name**: `IdentityGovernanceTrail`
   - **Enable for all accounts in my organization**: Leave unchecked

![Open CloudTrail Console](https://trtrantnt.github.io/workshop/images/2/cloudtrailb3.png?featherlight=false&width=90pc)

#### Step 2: S3 bucket configuration

4. Configure S3 storage:
   - **Create new S3 bucket**: Select this option (LEAVE BLANK - DO NOT select "Use existing S3 bucket")
   - **S3 bucket name**: CloudTrail will auto-generate name (e.g., aws-cloudtrail-logs-123456789012-abc12345)

5. Configure security settings:
   - **Log file SSE-KMS encryption**: Unchecked (keep default)
   - **Log file validation**: Checked (recommended)

#### Step 3: CloudWatch Logs (Optional)

6. CloudWatch Logs configuration:
   - **CloudWatch Logs**: Unchecked (skip for now)

7. Click **Next**
#### Step 4: Choose log events
8. Select event types to log:
   - **Management events**: Checked
     - **Read**: Checked
     - **Write**: Checked
   - **Data events**: Unchecked (skip)
   - **Insight events**: Unchecked (skip)
9. Click **Next**

#### Step 5: Review and create

10. Review configuration:
    - Confirm trail name
    - Confirm new S3 bucket will be created
    - Confirm management events are enabled
11. Click **Create trail**
**IMPORTANT**: 
- **NEVER** select the `identity-governance-analytics` bucket or any bucket you created before
- **ALWAYS** choose "Create new S3 bucket" to let CloudTrail create its own bucket
- CloudTrail will automatically configure the correct bucket policy, avoiding `InsufficientS3BucketPolicyException` error

![Open CloudTrail Console](https://trtrantnt.github.io/workshop/images/2/cloudtrailb4.png?featherlight=false&width=90pc)

### 2. Enable AWS Security Hub

1. Navigate to **AWS Security Hub** service in AWS Console

![Security Hub Onboard Page](https://trtrantnt.github.io/workshop/images/2/secuHub1.png?featherlight=false&width=90pc)

2. You'll see the **Security Hub Onboard** page

#### Step 1: Configure Security Hub
3. In the **Configure Security Hub** section:
   - Read information about Service Linked Roles (SLRs)
   - Keep default settings
#### Step 2: Delegated Administrator Account
4. In the **Delegated administrator account** section:
   - Choose **Do not select an account** (for single account setup)
#### Step 3: Account Enablement
5. In the **Account enablement** section:
   - ☑️ **Enable Security Hub for my account** (keep checked)
#### Step 4: Delegated Administrator Policy
6. In the **Delegated administrator policy** section:
   - Read policy details
   - Keep default settings
7. Click **Onboard** at the bottom of the page

![Security Hub Onboard Page](https://trtrantnt.github.io/workshop/images/2/secuHub2.png?featherlight=false&width=90pc)

#### Step 5: Verify successful activation
8. After successful onboarding, you'll see the Security Hub dashboard:
   - **Security score** displayed
   - **Findings** start being collected
   - **Standards** automatically enabled

![Security Hub Onboard Page](https://trtrantnt.github.io/workshop/images/2/secuHub3.png?featherlight=false&width=90pc)

### 3. Create DynamoDB Tables

1. Navigate to **DynamoDB** service

![DynamoDB Console](https://trtrantnt.github.io/workshop/images/2/dynab1.png?featherlight=false&width=90pc)

2. Click **Create table**
3. Create first table:
   - **Table name**: `AccessCertifications`
   - **Partition key**: `UserId` (String)
   - **Sort key**: `CertificationDate` (String)
   - **Billing mode**: On-demand

![DynamoDB Console](https://trtrantnt.github.io/workshop/images/2/dynab2.png?featherlight=false&width=90pc)

4. Click **Create table**

5. Create second table:
   - **Table name**: `RiskAssessments`
   - **Partition key**: `AssessmentId` (String)
   - **Billing mode**: On-demand
6. Click **Create table**

![DynamoDB Console](https://trtrantnt.github.io/workshop/images/2/dynab3.png?featherlight=false&width=90pc)

### 4. Create Required IAM Roles

1. Navigate to **IAM** service
2. Click **Roles** in the sidebar

![DynamoDB Console](https://trtrantnt.github.io/workshop/images/2/IAMb1.png?featherlight=false&width=90pc)

3. Click **Create role**
4. Create role for Lambda:
   - **Trusted entity**: AWS service
   - **Service**: Lambda
   - **Role name**: `IdentityGovernanceLambdaRole`
   - **Policies**: Attach `AWSLambdaBasicExecutionRole`

![DynamoDB Console](https://trtrantnt.github.io/workshop/images/2/IAMb2.png?featherlight=false&width=90pc)

## Verification

### 1. Check Enabled Services

1. **CloudTrail**: Go to CloudTrail console, confirm trail is created and active
2. **S3**: Go to S3 console, confirm 3 buckets created (2 your buckets + 1 CloudTrail bucket)
3. **Security Hub**: Go to Security Hub console, confirm service is enabled with security score
4. **DynamoDB**: Go to DynamoDB console, confirm 2 tables created
5. **IAM**: Go to IAM console, confirm Lambda role is created

![Services Verification](https://trtrantnt.github.io/workshop/images/2/services-verification.png?featherlight=false&width=90pc)

### 2. Check Access Permissions

1. Go to **IAM** console
2. Click **Users** and confirm current user has required permissions
3. Click **Roles** and confirm roles are created

![Permissions Check](https://trtrantnt.github.io/workshop/images/2/permissions-check.png?featherlight=false&width=90pc)

## Expected Results

After completing the preparation steps:

- ✅ AWS Account properly configured
- ✅ Required AWS services enabled
- ✅ Base infrastructure deployed
- ✅ Permissions validated
- ✅ Workshop materials ready

## Next Steps

Continue to [3. Access Governance Setup](../3-thiet-lap-access-governance) to start implementing the system.