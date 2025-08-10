---
title: "2. Các bước chuẩn bị"
chapter: false
weight: 2
---

## Yêu cầu Tiên quyết

### 1. AWS Account
- AWS Account với quyền Administrator
- Quyền tạo và quản lý AWS Organizations
- Quyền sử dụng IAM Identity Center

### 2. Kiến thức cần thiết
- Hiểu biết cơ bản về AWS IAM
- Kinh nghiệm với AWS Organizations
- Kiến thức về compliance frameworks (SOX, SOC2, ISO27001)
- Hiểu biết về Python và AWS CLI

### 3. Công cụ cần thiết
- AWS CLI đã được cấu hình
- Python 3.9 hoặc cao hơn
- Git để clone code examples
- Text editor hoặc IDE

## Thiết lập môi trường

### 1. Cấu hình AWS CLI

```bash
# Cài đặt AWS CLI
pip install awscli

# Cấu hình credentials
aws configure
```

### 2. Kiểm tra quyền

```bash
# Kiểm tra account hiện tại
aws sts get-caller-identity

# Kiểm tra quyền Organizations
aws organizations describe-organization
```

### 3. Tạo S3 Bucket cho dữ liệu

```bash
# Tạo bucket cho analytics data
aws s3 mb s3://privilege-analytics-$(aws sts get-caller-identity --query Account --output text)

# Tạo bucket cho compliance reports
aws s3 mb s3://compliance-reports-$(aws sts get-caller-identity --query Account --output text)
```

## Chuẩn bị Infrastructure

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

### 2. Tạo IAM Roles cần thiết

```bash
# Tạo role cho Lambda functions
aws iam create-role \
  --role-name IdentityGovernanceLambdaRole \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "Service": "lambda.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
      }
    ]
  }'

# Attach policies
aws iam attach-role-policy \
  --role-name IdentityGovernanceLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

## Chuẩn bị Code và Templates

### 1. Download Workshop Materials

```bash
# Clone workshop repository
git clone https://github.com/aws-samples/identity-governance-workshop.git
cd identity-governance-workshop

# Install Python dependencies
pip install -r requirements.txt
```

### 2. Chuẩn bị CloudFormation Templates

```yaml
# infrastructure-base.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Base infrastructure for Identity Governance Workshop'

Resources:
  # S3 Buckets
  AnalyticsDataBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub 'privilege-analytics-${AWS::AccountId}'
      VersioningConfiguration:
        Status: Enabled

  ComplianceReportsBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub 'compliance-reports-${AWS::AccountId}'
      VersioningConfiguration:
        Status: Enabled

  # DynamoDB Tables
  OperationsLogTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: OperationsLog
      AttributeDefinitions:
        - AttributeName: operation_id
          AttributeType: S
      KeySchema:
        - AttributeName: operation_id
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST

  CertificationTasksTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: CertificationTasks
      AttributeDefinitions:
        - AttributeName: task_id
          AttributeType: S
      KeySchema:
        - AttributeName: task_id
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST
```

### 3. Deploy Base Infrastructure

```bash
# Deploy base infrastructure
aws cloudformation deploy \
  --template-file infrastructure-base.yaml \
  --stack-name identity-governance-base \
  --capabilities CAPABILITY_IAM
```

## Validation

### 1. Kiểm tra Services

```bash
# Kiểm tra CloudTrail
aws cloudtrail describe-trails

# Kiểm tra S3 buckets
aws s3 ls | grep -E "(privilege-analytics|compliance-reports)"

# Kiểm tra DynamoDB tables
aws dynamodb list-tables
```

### 2. Test Permissions

```python
import boto3

def test_permissions():
    """Test required AWS permissions"""
    
    try:
        # Test IAM permissions
        iam = boto3.client('iam')
        iam.list_users(MaxItems=1)
        print("✅ IAM permissions OK")
        
        # Test Organizations permissions
        org = boto3.client('organizations')
        org.describe_organization()
        print("✅ Organizations permissions OK")
        
        # Test SSO permissions
        sso = boto3.client('sso-admin')
        sso.list_instances()
        print("✅ SSO permissions OK")
        
        print("🎉 All permissions validated successfully!")
        
    except Exception as e:
        print(f"❌ Permission error: {str(e)}")

if __name__ == "__main__":
    test_permissions()
```

## Kết quả Mong đợi

Sau khi hoàn thành các bước chuẩn bị:

- ✅ AWS Account được cấu hình đúng
- ✅ Các AWS services cần thiết đã được enable
- ✅ Base infrastructure đã được deploy
- ✅ Permissions đã được validate
- ✅ Workshop materials đã sẵn sàng

## Tiếp theo

Chuyển sang [3. Thiết lập Access Governance](../3-thiet-lap-access-governance) để bắt đầu triển khai hệ thống.