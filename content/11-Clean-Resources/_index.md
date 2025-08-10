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

1. Navigate to **IAM** service in AWS Console
2. Click **Roles** in the sidebar
3. Search for workshop roles:
   - **IdentityGovernanceLambdaRole**
   - **ComplianceValidationRole**
   - **CertificationWorkflowRole**

![IAM Roles List](/images/11/iam-roles-list.png)

4. Select each role and click **Delete**
5. Type role name to confirm deletion

![Delete IAM Role](/images/11/delete-iam-role.png)

### Delete Custom IAM Policies

1. Click **Policies** in the sidebar
2. Filter by **Customer managed**
3. Search for workshop policies:
   - **SecurityAuditPolicy**
   - **IdentityGovernancePolicy**
   - **ComplianceValidationPolicy**

![IAM Policies List](/images/11/iam-policies-list.png)

4. Select each policy and click **Actions** → **Delete**
5. Confirm deletion

![Delete IAM Policy](/images/11/delete-iam-policy.png)

### Delete IAM Users and Groups

1. Click **Users** in the sidebar
2. Select workshop users and click **Delete**

![Delete IAM Users](/images/11/delete-iam-users.png)

3. Click **User groups** in the sidebar
4. Select workshop groups and click **Delete**

![Delete IAM Groups](/images/11/delete-iam-groups.png)

## Step 8: IAM Identity Center Cleanup

### Remove Permission Set Assignments

1. Navigate to **IAM Identity Center**
2. Click **AWS accounts** in the sidebar
3. Select your account and click **Remove access**

![Remove SSO Access](/images/11/remove-sso-access.png)

### Delete Permission Sets

1. Click **Permission sets** in the sidebar
2. Select workshop permission sets:
   - **SecurityAuditor**
   - **ComplianceReviewer**
3. Click **Delete**

![Delete Permission Sets](/images/11/delete-permission-sets.png)

### Delete Users and Groups

1. Click **Users** in the sidebar
2. Select workshop users and click **Delete**

![Delete SSO Users](/images/11/delete-sso-users.png)

3. Click **Groups** in the sidebar
4. Select workshop groups and click **Delete**

![Delete SSO Groups](/images/11/delete-sso-groups.png)

## Step 9: AWS Config Cleanup

1. Navigate to **AWS Config** service
2. Click **Settings** in the sidebar
3. Click **Edit** and then **Delete configuration recorder**

![Delete Config Recorder](/images/11/delete-config-recorder.png)

4. Confirm deletion by typing **delete**

## Step 10: CloudTrail Cleanup

1. Navigate to **CloudTrail** service
2. Click **Trails** in the sidebar
3. Select **IdentityGovernanceTrail**
4. Click **Delete**

![Delete CloudTrail](/images/11/delete-cloudtrail.png)

5. Type trail name to confirm deletion

## Console-Based Cleanup Checklist

For systematic cleanup through AWS Console, follow this checklist:

### ✅ Cleanup Checklist

**Lambda Functions:**
- [ ] IdentityGovernanceMonitor
- [ ] AccessReviewGenerator  
- [ ] ComplianceValidationEngine
- [ ] RiskAssessmentEngine
- [ ] CertificationNotifier

**EventBridge Rules:**
- [ ] AccessCertificationSchedule
- [ ] ComplianceValidationSchedule
- [ ] RiskAssessmentSchedule

**Step Functions:**
- [ ] AccessCertificationWorkflow
- [ ] ComplianceValidationWorkflow

**DynamoDB Tables:**
- [ ] CertificationTasks
- [ ] OperationsLog
- [ ] ComplianceEvidence
- [ ] RiskMonitoring
- [ ] AuditFindings

**S3 Buckets:**
- [ ] privilege-analytics-[ACCOUNT-ID]
- [ ] compliance-reports-[ACCOUNT-ID]

**CloudWatch Resources:**
- [ ] IdentityGovernanceRiskDashboard
- [ ] DailyOperationsDashboard
- [ ] All workshop alarms
- [ ] All workshop log groups

**SNS Topics:**
- [ ] IdentityGovernanceAlerts
- [ ] ComplianceAlerts

**IAM Resources:**
- [ ] Workshop IAM roles
- [ ] Workshop IAM policies
- [ ] Workshop IAM users
- [ ] Workshop IAM groups

**IAM Identity Center:**
- [ ] Permission set assignments
- [ ] Permission sets
- [ ] SSO users and groups

**Other Services:**
- [ ] AWS Config recorder
- [ ] CloudTrail trail
- [ ] GuardDuty detector (if not needed)

![Cleanup Checklist](/images/11/cleanup-checklist.png)

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