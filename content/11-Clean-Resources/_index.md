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
   - **AccessCertificationTrigger**
   - **PrivilegeAnalyticsEngine**
   - **RiskAssessmentEngine**
   - **CustomMetricsPublisher**
   - **DailyOperationsEngine**
   - **AuditReportGenerator**
   - **E2EValidationTest**
3. Select each function
4. Click **Actions** → **Delete**
5. Confirm deletion by typing **confirm**

![Lambda Functions](https://trtrantnt.github.io/workshop/images/11/del1.png?featherlight=false&width=90pc)

### Delete EventBridge Rules

1. Open **Amazon EventBridge** console
2. Go to **Schedules**
3. Select workshop rules:
   - **AccessCertificationSchedule**
   - **ComplianceValidationSchedule**
   - **RiskAssessmentSchedule**
   - **DailyOperationsSchedule**
   - **WeeklyOperationsReview**
4. Click **Delete** for each rule

![EventBridge Rules](https://trtrantnt.github.io/workshop/images/11/del2.png?featherlight=false&width=90pc)

## Step 2: DynamoDB Tables

1. Open **Amazon DynamoDB** console
2. Go to **Tables**
3. Select workshop tables:
   - **AccessCertifications**
   - **RiskAssessments**
   - **CertificationTasks**
   - **OperationsLog**
   - **ComplianceEvidence**
   - **RiskMonitoring**
   - **AuditFindings**
4. Click **Delete** for each table
5. Type **delete** to confirm

![DynamoDB Tables](https://trtrantnt.github.io/workshop/images/11/del3.png?featherlight=false&width=90pc)

## Step 3: S3 Buckets

### Empty S3 Buckets

1. Open **Amazon S3** console
2. Identify workshop buckets:
   - **identity-governance-analytics**
   - **identity-governance-reports**
   - **aws-cloudtrail-logs-*** (CloudTrail bucket)
3. Select each bucket and click **Empty**
4. Type `permanently delete` to confirm

### Delete S3 Buckets

1. After emptying, select each bucket
2. Click **Delete**
3. Type bucket name to confirm

![Empty S3 Bucket](https://trtrantnt.github.io/workshop/images/11/del5.png?featherlight=false&width=90pc)

## Step 4: CloudWatch Resources

### Delete CloudWatch Dashboards

1. Open **Amazon CloudWatch** console
2. Go to **Dashboards**
3. Select workshop dashboards:
   - **IdentityGovernanceRiskDashboard**
   - **IdentityGovernanceOperations**
   - **DailyOperationsDashboard**
4. Click **Delete** for each dashboard

### Delete CloudWatch Alarms

1. Go to **Alarms**
2. Select workshop alarms:
   - **Lambda-AccessCertification-Errors**
   - **HighRiskUserCount-Alarm**
   - **DynamoDB-ReadErrors**
   - **S3-AccessErrors**
3. Click **Actions** → **Delete**

### Delete Log Groups

1. Go to **Log groups**
2. Select workshop log groups:
   - `/aws/lambda/AccessCertificationTrigger`
   - `/aws/lambda/PrivilegeAnalyticsEngine`
   - `/aws/lambda/RiskAssessmentEngine`
   - `/aws/lambda/CustomMetricsPublisher`
   - `/aws/lambda/DailyOperationsEngine`
   - `/aws/lambda/AuditReportGenerator`
   - `/aws/lambda/E2EValidationTest`
3. Click **Actions** → **Delete log group**

## Step 5: SNS Topics

1. Open **Amazon SNS** console
2. Go to **Topics**
3. Select workshop topics:
   - **IdentityGovernanceAlerts**
   - **ComplianceAlerts**
   - **RiskAssessmentAlerts**
   - **DailyOperationsAlerts**

4. Click **Delete** for each topic
5. Confirm deletion

## Step 6: IAM Resources

### Delete IAM Roles

1. Navigate to **IAM** service in AWS Console
2. Click **Roles** in the sidebar
3. Search for workshop roles:
   - **IdentityGovernanceLambdaRole**
   - **ComplianceValidationRole**
   - **CertificationWorkflowRole**
4. Select each role and click **Delete**
5. Type role name to confirm deletion

![Empty S3 Bucket](https://trtrantnt.github.io/workshop/images/11/del6.png?featherlight=false&width=90pc)

### Delete Custom IAM Policies

1. Click **Policies** in the sidebar
2. Filter by **Customer managed**
3. Search for workshop policies:
   - **SecurityAuditPolicy**
   - **IdentityGovernancePolicy**
   - **ComplianceValidationPolicy**

4. Select each policy and click **Actions** → **Delete**
5. Confirm deletion

### Delete IAM Users and Groups

1. Click **Users** in the sidebar
2. Select workshop users and click **Delete**
3. Click **User groups** in the sidebar
4. Select workshop groups and click **Delete**

## Step 7: Clean up IAM Identity Center

### Delete Permission Set Assignments

1. Navigate to **IAM Identity Center**
2. Click **AWS accounts** in the sidebar
3. Select your account and click **Remove access**

### Delete Permission Sets

1. Click **Permission sets** in the sidebar
2. Select workshop permission sets:
   - **SecurityAuditor**
   - **ComplianceReviewer**
3. Click **Delete**

![Empty S3 Bucket](https://trtrantnt.github.io/workshop/images/11/del7.png?featherlight=false&width=90pc)

### Delete Users and Groups

1. Click **Users** in the sidebar
2. Select workshop users and click **Delete**

3. Click **Groups** in the sidebar
4. Select workshop groups and click **Delete**

## Step 8: Clean up CloudTrail

1. Navigate to **CloudTrail** service
2. Click **Trails** in the sidebar
3. Select **IdentityGovernanceTrail**
4. Click **Delete**

![CloudTrail](https://trtrantnt.github.io/workshop/images/11/del8.png?featherlight=false&width=90pc)

## Console-Based Cleanup Checklist

For systematic cleanup through AWS Console, follow this checklist:

### ✅ Cleanup Checklist

**Lambda Functions:**

- [ ] AccessCertificationTrigger
- [ ] PrivilegeAnalyticsEngine
- [ ] RiskAssessmentEngine
- [ ] CustomMetricsPublisher
- [ ] DailyOperationsEngine
- [ ] AuditReportGenerator
- [ ] E2EValidationTest

**EventBridge Rules:**

- [ ] AccessCertificationSchedule
- [ ] ComplianceValidationSchedule
- [ ] RiskAssessmentSchedule
- [ ] DailyOperationsSchedule
- [ ] WeeklyOperationsReview

**DynamoDB Tables:**

- [ ] AccessCertifications
- [ ] RiskAssessments
- [ ] CertificationTasks
- [ ] OperationsLog
- [ ] ComplianceEvidence
- [ ] RiskMonitoring
- [ ] AuditFindings

**S3 Buckets:**

- [ ] identity-governance-analytics
- [ ] identity-governance-reports
- [ ] aws-cloudtrail-logs-[ACCOUNT-ID]-[HASH]

**CloudWatch Resources:**

- [ ] IdentityGovernanceRiskDashboard
- [ ] IdentityGovernanceOperations
- [ ] DailyOperationsDashboard
- [ ] All workshop alarms
- [ ] All workshop log groups

**SNS Topics:**

- [ ] IdentityGovernanceAlerts
- [ ] ComplianceAlerts
- [ ] RiskAssessmentAlerts
- [ ] DailyOperationsAlerts

**IAM Resources:**

- [ ] Workshop IAM roles
- [ ] Workshop custom policies (if any)

**Optional Resources:**

- [ ] AWS Security Hub (if not needed)
- [ ] AWS CloudTrail (if not needed)


## Step 9: Cleanup Verification

### Check Remaining Resources

1. Check **AWS Cost Explorer** to confirm no charges are ongoing
2. Use **AWS Resource Groups** to find tagged resources
3. Search for tag: **Project=IdentityGovernance**

### Check Final Services

1. **AWS Config**: Disable configuration recorder if not needed
2. **AWS Security Hub**: Disable if not used elsewhere
3. **Amazon GuardDuty**: Disable if not needed
4. **AWS Audit Manager**: Disable data collection

## Workshop Cleanup Completed

Workshop cleanup completed successfully! 🎉
