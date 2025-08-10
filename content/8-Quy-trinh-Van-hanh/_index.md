---
title: "8. Operational Procedures"
chapter: false
weight: 8
---

# 8. Operational Procedures

## Objective

Establish standardized operational procedures to manage identity governance effectively, ensuring consistency and compliance.

## Process Overview

```mermaid
graph TB
    A[Daily Operations] --> B[Weekly Reviews]
    B --> C[Monthly Assessments]
    C --> D[Quarterly Certifications]
    
    E[Incident Response] --> F[Investigation]
    F --> G[Remediation]
    G --> H[Documentation]
    
    I[Change Management] --> J[Approval Process]
    J --> K[Implementation]
    K --> L[Validation]
```

## Step 1: Daily Operations Dashboard

### 1.1 Create Operations Dashboard

1. Open **Amazon CloudWatch** console
2. Click **Dashboards**
3. Click **Create dashboard**

![Create Operations Dashboard](/images/8/create-operations-dashboard.png?featherlight=false&width=90pc)

4. Name: **DailyOperationsDashboard**
5. Add widgets for daily monitoring:
   - **Failed login attempts (last 24h)**
   - **High-risk alerts**
   - **System health status**
   - **Pending certifications**

![Operations Dashboard Widgets](/images/8/operations-dashboard-widgets.png?featherlight=false&width=90pc)

### 1.2 Set Up Daily Checklist Automation

1. Open **AWS Systems Manager**
2. Click **Documents** in sidebar
3. Click **Create document**

![Create SSM Document](/images/8/create-ssm-document.png?featherlight=false&width=90pc)

4. Create **Automation document** for daily checks:
   - **Name**: DailyOperationsChecklist
   - **Document type**: Automation

![SSM Document Details](/images/8/ssm-document-details.png?featherlight=false&width=90pc)

5. Configure automation steps for:
   - **Security alerts review**
   - **System health validation**
   - **Backup status check**

![Automation Steps](/images/8/automation-steps.png?featherlight=false&width=90pc)

## Step 2: Standard Operating Procedures (SOPs)

### 2.1 Create SOP Documentation in Wiki

1. Open **Amazon WorkDocs** or create **S3 bucket** for documentation
2. Create folder structure for SOPs

![SOP Documentation](/images/8/sop-documentation.png?featherlight=false&width=90pc)

3. Create **SOP-001: Daily Monitoring Procedures**
4. Include sections:
   - **Purpose and Scope**
   - **Daily procedures timeline**
   - **Escalation criteria**
   - **Contact information**

![Daily Monitoring SOP](/images/8/daily-monitoring-sop.png?featherlight=false&width=90pc)

### 2.2 Implement SOP Tracking

1. Create **DynamoDB table** for SOP tracking:
   - **Table name**: OperationalProcedures
   - **Primary key**: procedure_id

![DynamoDB SOP Table](/images/8/dynamodb-sop-table.png?featherlight=false&width=90pc)

2. Create **Lambda function** to track SOP execution
3. Configure **EventBridge** rules for SOP scheduling

![SOP Tracking Lambda](/images/8/sop-tracking-lambda.png?featherlight=false&width=90pc)

## Step 3: Weekly Review Automation

### 3.1 Set Up Weekly Review Reports

1. Open **Amazon QuickSight**
2. Create **Weekly Review Dashboard**

![Weekly Review Dashboard](/images/8/weekly-review-dashboard.png?featherlight=false&width=90pc)

3. Configure data sources:
   - **CloudWatch metrics**
   - **Security Hub findings**
   - **Config compliance data**

![Weekly Review Data Sources](/images/8/weekly-review-data-sources.png?featherlight=false&width=90pc)

### 3.2 Automate Weekly Report Generation

1. Create **Lambda function** for weekly reports
2. Configure **EventBridge** rule:
   - **Schedule**: Every Monday at 9:00 AM
   - **Target**: Weekly report Lambda

![Weekly Report Automation](/images/8/weekly-report-automation.png?featherlight=false&width=90pc)

3. Configure report delivery via **SES**

![Email Report Delivery](/images/8/email-report-delivery.png?featherlight=false&width=90pc)

## Step 4: Incident Response Runbooks

### 4.1 Create Digital Runbooks

1. Open **AWS Systems Manager**
2. Click **Documents**
3. Create **Command document** for incident response

![Create Runbook Document](/images/8/create-runbook-document.png?featherlight=false&width=90pc)

4. Configure runbook sections:
   - **Incident classification**
   - **Initial response steps**
   - **Investigation procedures**
   - **Escalation matrix**

![Runbook Configuration](/images/8/runbook-configuration.png?featherlight=false&width=90pc)

### 4.2 Implement Incident Tracking

1. Create **ServiceNow integration** or use **AWS Support Cases**
2. Configure **ChatBot** for Slack/Teams notifications

![Incident Tracking Integration](/images/8/incident-tracking-integration.png?featherlight=false&width=90pc)

3. Set up **PagerDuty** integration for on-call management

![PagerDuty Integration](/images/8/pagerduty-integration.png?featherlight=false&width=90pc)

### 4.3 Test Runbook Procedures

1. Schedule **tabletop exercises**
2. Document lessons learned
3. Update runbooks based on feedback

![Runbook Testing](/images/8/runbook-testing.png?featherlight=false&width=90pc)

## Expected Results

After completion:

- ✅ CloudWatch dashboard for daily operations
- ✅ Automated SOP execution and tracking
- ✅ Weekly review reports via QuickSight
- ✅ Digital incident response runbooks
- ✅ Integrated incident tracking system
- ✅ Automated operational workflows

![Operational Procedures Complete](/images/8/operational-procedures-complete.png?featherlight=false&width=90pc)

## Next Steps

Continue to [9. Audit Procedures](../9-quy-trinh-kiem-toan) to set up audit processes.