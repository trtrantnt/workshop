---
title: "11. Clean Resources"
chapter: false
weight: 11
---

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

1. Open **AWS Lambda** console
2. Filter functions by workshop names:
   - **IdentityGovernance**
   - **AccessCertification**
   - **ComplianceValidation**

![Lambda Functions List](/images/11/lambda-functions-list.png?featherlight=false&width=90pc)

3. Select workshop functions
4. Click **Actions** → **Delete**

![Delete Lambda Functions](/images/11/delete-lambda-functions.png?featherlight=false&width=90pc)

5. Confirm deletion by typing **delete**

![Confirm Lambda Deletion](/images/11/confirm-lambda-deletion.png?featherlight=false&width=90pc)

### Delete EventBridge Rules

1. Open **Amazon EventBridge** console
2. Go to **Rules**
3. Select workshop rules:
   - **AccessCertificationSchedule**
   - **ComplianceValidationSchedule**

![EventBridge Rules](/images/11/eventbridge-rules.png?featherlight=false&width=90pc)

4. Click **Delete** for each rule

![Delete EventBridge Rules](/images/11/delete-eventbridge-rules.png?featherlight=false&width=90pc)

## Step 2: Step Functions

1. Open **AWS Step Functions** console
2. Select workshop state machines:
   - **AccessCertificationWorkflow**
   - **ComplianceValidationWorkflow**

![Step Functions List](/images/11/step-functions-list.png?featherlight=false&width=90pc)

3. Click **Delete**
4. Confirm deletion

![Delete Step Functions](/images/11/delete-step-functions.png?featherlight=false&width=90pc)

## Step 3: DynamoDB Tables

1. Open **Amazon DynamoDB** console
2. Go to **Tables**
3. Select workshop tables:
   - **OperationalProcedures**
   - **ComplianceEvidence**
   - **AuditFindings**

![DynamoDB Tables](/images/11/dynamodb-tables.png?featherlight=false&width=90pc)

4. Click **Delete** for each table
5. Type **delete** to confirm

![Delete DynamoDB Tables](/images/11/delete-dynamodb-tables.png?featherlight=false&width=90pc)

## Step 4: S3 Buckets

### Empty S3 Buckets

1. Open **Amazon S3** console
2. Identify workshop buckets:
   - **privilege-analytics-***
   - **compliance-reports-***

![S3 Buckets List](/images/11/s3-buckets-list.png?featherlight=false&width=90pc)

3. Select bucket and click **Empty**
4. Type **permanently delete** to confirm

![Empty S3 Bucket](/images/11/empty-s3-bucket.png?featherlight=false&width=90pc)

### Delete S3 Buckets

1. After emptying, select bucket
2. Click **Delete**
3. Type bucket name to confirm

![Delete S3 Bucket](/images/11/delete-s3-bucket.png?featherlight=false&width=90pc)

## Step 5: CloudWatch Resources

### Delete CloudWatch Dashboards

1. Open **Amazon CloudWatch** console
2. Go to **Dashboards**
3. Select workshop dashboards:
   - **IdentityGovernanceRiskDashboard**
   - **DailyOperationsDashboard**

![CloudWatch Dashboards](/images/11/cloudwatch-dashboards.png?featherlight=false&width=90pc)

4. Click **Delete** for each dashboard

![Delete Dashboard](/images/11/delete-dashboard.png?featherlight=false&width=90pc)

### Delete CloudWatch Alarms

1. Go to **Alarms**
2. Select workshop alarms
3. Click **Actions** → **Delete**

![Delete CloudWatch Alarms](/images/11/delete-cloudwatch-alarms.png?featherlight=false&width=90pc)

### Delete Log Groups

1. Go to **Log groups**
2. Select workshop log groups
3. Click **Actions** → **Delete log group**

![Delete Log Groups](/images/11/delete-log-groups.png?featherlight=false&width=90pc)

## Step 6: SNS Topics

1. Open **Amazon SNS** console
2. Go to **Topics**
3. Select workshop topics:
   - **IdentityGovernanceAlerts**
   - **ComplianceAlerts**

![SNS Topics](/images/11/sns-topics.png?featherlight=false&width=90pc)

4. Click **Delete** for each topic
5. Confirm deletion

![Delete SNS Topics](/images/11/delete-sns-topics.png?featherlight=false&width=90pc)

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

## Step 7: Final Verification

### Verify Resource Deletion

1. Check **AWS Cost Explorer** for any remaining charges

![Cost Explorer Verification](/images/11/cost-explorer-verification.png?featherlight=false&width=90pc)

2. Use **AWS Resource Groups** to find tagged resources
3. Search for tag: **Project=IdentityGovernance**

![Resource Groups Check](/images/11/resource-groups-check.png?featherlight=false&width=90pc)

### Final Service Checks

1. **AWS Config**: Disable configuration recorder if not needed
2. **AWS Security Hub**: Disable if not used elsewhere
3. **Amazon GuardDuty**: Disable if not needed
4. **AWS Audit Manager**: Disable data collection

![Final Service Checks](/images/11/final-service-checks.png?featherlight=false&width=90pc)

### Cleanup Verification Report

1. Generate cleanup summary report
2. Document any resources that couldn't be deleted
3. Note any ongoing charges

![Cleanup Verification Report](/images/11/cleanup-verification-report.png?featherlight=false&width=90pc)

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

## Expected Results

After completing the cleanup:

- ✅ All workshop Lambda functions deleted
- ✅ EventBridge rules removed
- ✅ Step Functions state machines deleted
- ✅ DynamoDB tables removed
- ✅ S3 buckets emptied and deleted
- ✅ CloudWatch resources cleaned up
- ✅ SNS topics deleted
- ✅ No ongoing charges from workshop resources
- ✅ Account returned to pre-workshop state

![Cleanup Complete](/images/11/cleanup-complete.png?featherlight=false&width=90pc)

**Workshop cleanup completed successfully! 🎉**

![Workshop Complete](/images/11/workshop-complete.png?featherlight=false&width=90pc)