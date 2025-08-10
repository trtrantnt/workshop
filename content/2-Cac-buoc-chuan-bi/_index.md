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

### 3. Required Tools
- Configured AWS CLI
- Python 3.9 or higher
- Git to clone code examples
- Text editor or IDE

## Environment Setup

### 1. Configure AWS CLI

```bash
# Install AWS CLI
pip install awscli

# Configure credentials
aws configure
```

### 2. Check Permissions

```bash
# Check current account
aws sts get-caller-identity

# Check Organizations permissions
aws organizations describe-organization
```

### 3. Create S3 Buckets for Data

```bash
# Create bucket for analytics data
aws s3 mb s3://privilege-analytics-$(aws sts get-caller-identity --query Account --output text)

# Create bucket for compliance reports
aws s3 mb s3://compliance-reports-$(aws sts get-caller-identity --query Account --output text)
```

## Infrastructure Preparation

### 1. Enable AWS Services

```bash
# Enable CloudTrail
aws cloudtrail create-trail \
  --name IdentityGovernanceTrail \
  --s3-bucket-name privilege-analytics-$(aws sts get-caller-identity --query Account --output text)

# Enable Config
aws configservice put-configuration-recorder \
  --configuration-recorder name=IdentityGovernanceRecorder,roleARN=arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):role/aws-service-role/config.amazonaws.com/AWSServiceRoleForConfig

# Enable GuardDuty
aws guardduty create-detector --enable
```

## Expected Results

After completing the preparation steps:

- ✅ AWS Account properly configured
- ✅ Required AWS services enabled
- ✅ Base infrastructure deployed
- ✅ Permissions validated
- ✅ Workshop materials ready

## Next Steps

Continue to [3. Access Governance Setup](../3-thiet-lap-access-governance) to start implementing the system.