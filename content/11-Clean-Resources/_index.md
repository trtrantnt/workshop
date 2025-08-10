---
title: "11. Clean Resources"
chapter: false
weight: 11
---

# 11. Clean Resources

## Overview

This section provides comprehensive instructions for cleaning up all AWS resources created during the Identity Governance workshop to avoid unnecessary charges.

## Important Notes

⚠️ **Warning**: Following these cleanup steps will permanently delete all resources and data created during the workshop. Make sure you have backed up any important configurations or data before proceeding.

## Cleanup Order

Resources should be cleaned up in the following order to avoid dependency conflicts:

1. Lambda Functions and EventBridge Rules
2. Step Functions State Machines
3. DynamoDB Tables
4. S3 Buckets and Objects
5. CloudWatch Resources
6. IAM Roles and Policies
7. CloudFormation Stacks
8. AWS Organizations (if created)
9. IAM Identity Center (if no longer needed)

## Step 1: Lambda Functions and EventBridge

### Delete Lambda Functions

```bash
# List all Lambda functions created during workshop
aws lambda list-functions --query 'Functions[?contains(FunctionName, `IdentityGovernance`) || contains(FunctionName, `Compliance`) || contains(FunctionName, `AccessReview`)].FunctionName' --output table

# Delete Lambda functions
aws lambda delete-function --function-name IdentityGovernanceMonitor
aws lambda delete-function --function-name AccessReviewGenerator
aws lambda delete-function --function-name ComplianceValidationEngine
aws lambda delete-function --function-name RiskAssessmentEngine
aws lambda delete-function --function-name CertificationNotifier
```

### Delete EventBridge Rules

```bash
# List EventBridge rules
aws events list-rules --query 'Rules[?contains(Name, `IdentityGovernance`) || contains(Name, `Compliance`) || contains(Name, `Certification`)].Name' --output table

# Delete EventBridge rules
aws events delete-rule --name AccessCertificationSchedule
aws events delete-rule --name ComplianceValidationSchedule
aws events delete-rule --name RiskAssessmentSchedule
```

## Step 2: Step Functions

```bash
# List Step Functions
aws stepfunctions list-state-machines --query 'stateMachines[?contains(name, `Certification`) || contains(name, `Governance`)].name' --output table

# Delete Step Functions
aws stepfunctions delete-state-machine --state-machine-arn arn:aws:states:REGION:ACCOUNT:stateMachine:CertificationWorkflow
```

## Step 3: DynamoDB Tables

```bash
# List DynamoDB tables
aws dynamodb list-tables --query 'TableNames[?contains(@, `Certification`) || contains(@, `Operations`) || contains(@, `Compliance`) || contains(@, `Risk`)]' --output table

# Delete DynamoDB tables
aws dynamodb delete-table --table-name CertificationTasks
aws dynamodb delete-table --table-name OperationsLog
aws dynamodb delete-table --table-name ComplianceEvidence
aws dynamodb delete-table --table-name RiskMonitoring
aws dynamodb delete-table --table-name AuditFindings
```

## Step 4: S3 Buckets

### Empty and Delete S3 Buckets

```bash
# Get account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# List workshop S3 buckets
aws s3 ls | grep -E "(privilege-analytics|compliance-reports|identity-governance)"

# Empty buckets first
aws s3 rm s3://privilege-analytics-${ACCOUNT_ID} --recursive
aws s3 rm s3://compliance-reports-${ACCOUNT_ID} --recursive

# Delete buckets
aws s3 rb s3://privilege-analytics-${ACCOUNT_ID}
aws s3 rb s3://compliance-reports-${ACCOUNT_ID}
```

## Step 5: CloudWatch Resources

### Delete CloudWatch Dashboards

```bash
# List dashboards
aws cloudwatch list-dashboards --query 'DashboardEntries[?contains(DashboardName, `IdentityGovernance`) || contains(DashboardName, `Compliance`)].DashboardName' --output table

# Delete dashboards
aws cloudwatch delete-dashboards --dashboard-names IdentityGovernanceRiskDashboard IdentityGovernanceMonitoring ComplianceDashboard
```

### Delete CloudWatch Alarms

```bash
# List alarms
aws cloudwatch describe-alarms --query 'MetricAlarms[?contains(AlarmName, `IdentityGovernance`) || contains(AlarmName, `Compliance`) || contains(AlarmName, `Risk`)].AlarmName' --output table

# Delete alarms
aws cloudwatch delete-alarms --alarm-names HighFailedLoginAttempts PrivilegeEscalationDetected HighRiskEventsAlarm ComplianceViolationAlarm
```

### Delete Log Groups

```bash
# List log groups
aws logs describe-log-groups --query 'logGroups[?contains(logGroupName, `identity-governance`) || contains(logGroupName, `compliance`) || contains(logGroupName, `aws/lambda/Identity`) || contains(logGroupName, `aws/lambda/Compliance`)].logGroupName' --output table

# Delete log groups
aws logs delete-log-group --log-group-name /aws/identity-governance/events
aws logs delete-log-group --log-group-name /aws/lambda/IdentityGovernanceMonitor
aws logs delete-log-group --log-group-name /aws/lambda/ComplianceValidationEngine
```

## Step 6: SNS Topics

```bash
# List SNS topics
aws sns list-topics --query 'Topics[?contains(TopicArn, `IdentityGovernance`) || contains(TopicArn, `Compliance`) || contains(TopicArn, `Risk`)].TopicArn' --output table

# Delete SNS topics
aws sns delete-topic --topic-arn arn:aws:sns:REGION:ACCOUNT:IdentityGovernanceAlerts
aws sns delete-topic --topic-arn arn:aws:sns:REGION:ACCOUNT:ComplianceAlerts
aws sns delete-topic --topic-arn arn:aws:sns:REGION:ACCOUNT:RiskAlerts
```

## Step 7: IAM Resources

### Delete IAM Roles

```bash
# List IAM roles created for workshop
aws iam list-roles --query 'Roles[?contains(RoleName, `IdentityGovernance`) || contains(RoleName, `Compliance`) || contains(RoleName, `Certification`)].RoleName' --output table

# Detach policies and delete roles
aws iam detach-role-policy --role-name IdentityGovernanceLambdaRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam delete-role --role-name IdentityGovernanceLambdaRole

aws iam delete-role --role-name ComplianceValidationRole
aws iam delete-role --role-name CertificationWorkflowRole
```

### Delete Custom IAM Policies

```bash
# List custom policies
aws iam list-policies --scope Local --query 'Policies[?contains(PolicyName, `IdentityGovernance`) || contains(PolicyName, `Compliance`)].PolicyName' --output table

# Delete custom policies
aws iam delete-policy --policy-arn arn:aws:iam::ACCOUNT:policy/IdentityGovernancePolicy
aws iam delete-policy --policy-arn arn:aws:iam::ACCOUNT:policy/ComplianceValidationPolicy
```

## Step 8: CloudFormation Stacks

```bash
# List CloudFormation stacks
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE --query 'StackSummaries[?contains(StackName, `identity-governance`) || contains(StackName, `compliance`)].StackName' --output table

# Delete CloudFormation stacks
aws cloudformation delete-stack --stack-name identity-governance-base
aws cloudformation delete-stack --stack-name identity-governance-monitoring
aws cloudformation delete-stack --stack-name identity-governance-compliance
```

## Step 9: AWS Config (if enabled for workshop)

```bash
# Stop configuration recorder
aws configservice stop-configuration-recorder --configuration-recorder-name IdentityGovernanceRecorder

# Delete configuration recorder
aws configservice delete-configuration-recorder --configuration-recorder-name IdentityGovernanceRecorder

# Delete delivery channel
aws configservice delete-delivery-channel --delivery-channel-name IdentityGovernanceDeliveryChannel
```

## Step 10: CloudTrail (if created for workshop)

```bash
# List CloudTrail trails
aws cloudtrail describe-trails --query 'trailList[?contains(Name, `IdentityGovernance`)].Name' --output table

# Delete CloudTrail
aws cloudtrail delete-trail --name IdentityGovernanceTrail
```

## Automated Cleanup Script

For convenience, here's a comprehensive cleanup script:

```bash
#!/bin/bash

echo "Starting Identity Governance Workshop Cleanup..."

# Get account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REGION=$(aws configure get region)

echo "Account ID: $ACCOUNT_ID"
echo "Region: $REGION"

# Function to check if resource exists before deletion
check_and_delete() {
    local resource_type=$1
    local resource_name=$2
    local delete_command=$3
    
    echo "Checking $resource_type: $resource_name"
    if eval "$delete_command" 2>/dev/null; then
        echo "✅ Deleted $resource_type: $resource_name"
    else
        echo "⚠️  $resource_type not found or already deleted: $resource_name"
    fi
}

# Delete Lambda functions
echo "🧹 Cleaning up Lambda functions..."
LAMBDA_FUNCTIONS=("IdentityGovernanceMonitor" "AccessReviewGenerator" "ComplianceValidationEngine" "RiskAssessmentEngine")
for func in "${LAMBDA_FUNCTIONS[@]}"; do
    check_and_delete "Lambda function" "$func" "aws lambda delete-function --function-name $func"
done

# Delete DynamoDB tables
echo "🧹 Cleaning up DynamoDB tables..."
DYNAMODB_TABLES=("CertificationTasks" "OperationsLog" "ComplianceEvidence" "RiskMonitoring" "AuditFindings")
for table in "${DYNAMODB_TABLES[@]}"; do
    check_and_delete "DynamoDB table" "$table" "aws dynamodb delete-table --table-name $table"
done

# Delete S3 buckets
echo "🧹 Cleaning up S3 buckets..."
S3_BUCKETS=("privilege-analytics-${ACCOUNT_ID}" "compliance-reports-${ACCOUNT_ID}")
for bucket in "${S3_BUCKETS[@]}"; do
    echo "Emptying S3 bucket: $bucket"
    aws s3 rm s3://$bucket --recursive 2>/dev/null || echo "Bucket $bucket not found"
    check_and_delete "S3 bucket" "$bucket" "aws s3 rb s3://$bucket"
done

# Delete CloudFormation stacks
echo "🧹 Cleaning up CloudFormation stacks..."
CF_STACKS=("identity-governance-base" "identity-governance-monitoring" "identity-governance-compliance")
for stack in "${CF_STACKS[@]}"; do
    check_and_delete "CloudFormation stack" "$stack" "aws cloudformation delete-stack --stack-name $stack"
done

echo "🎉 Cleanup completed!"
echo "Note: Some resources may take a few minutes to be fully deleted."
echo "Please check the AWS Console to verify all resources have been removed."
```

## Verification

After running the cleanup, verify that all resources have been deleted:

### Check Remaining Resources

```bash
# Check Lambda functions
aws lambda list-functions --query 'Functions[?contains(FunctionName, `IdentityGovernance`)].FunctionName'

# Check DynamoDB tables
aws dynamodb list-tables --query 'TableNames[?contains(@, `Certification`) || contains(@, `Compliance`)]'

# Check S3 buckets
aws s3 ls | grep -E "(privilege-analytics|compliance-reports)"

# Check CloudFormation stacks
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE --query 'StackSummaries[?contains(StackName, `identity-governance`)].StackName'
```

## Cost Verification

After cleanup, monitor your AWS billing dashboard to ensure no unexpected charges from remaining resources.

## Troubleshooting

### Common Issues

1. **Dependency Errors**: Some resources may have dependencies. Delete dependent resources first.
2. **Permission Errors**: Ensure you have sufficient permissions to delete all resources.
3. **Region Issues**: Make sure you're deleting resources in the correct region.

### Manual Cleanup

If automated cleanup fails, manually delete resources through the AWS Console:

1. Go to each service console
2. Search for resources with names containing "IdentityGovernance", "Compliance", or "Certification"
3. Delete resources individually

## Final Notes

- This cleanup removes ALL workshop resources
- Double-check before running cleanup commands
- Some resources may have a brief delay before deletion
- Monitor your AWS bill to ensure no charges continue

**Workshop cleanup completed! 🎉**