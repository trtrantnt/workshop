---
title: "5. Privilege Analytics"
chapter: false
weight: 5
---

## Objective

Analyze and monitor privilege usage to detect security risks, excessive permissions, and anomalous patterns.

## Analytics Architecture

```mermaid
graph TB
    A[CloudTrail Logs] --> B[S3 Data Lake]
    C[Config Data] --> B
    D[IAM Data] --> B
    B --> E[Athena Queries]
    E --> F[Analytics Engine]
    F --> G[Risk Scoring]
    G --> H[QuickSight Dashboard]
    F --> I[Anomaly Detection]
    I --> J[Alerts]
```

## Step 1: CloudTrail Setup for Data Collection

### 1.1 Create CloudTrail

1. Open **AWS CloudTrail** in the console
2. Click **Create trail**

![Create CloudTrail](/images/5/create-cloudtrail.png?featherlight=false&width=90pc)

3. Configure trail settings:
   - **Trail name**: PrivilegeAnalyticsTrail
   - **Apply trail to all regions**: Yes
   - **Management events**: Read and Write

![CloudTrail Configuration](/images/5/cloudtrail-config.png?featherlight=false&width=90pc)

4. Configure S3 bucket:
   - **Create new S3 bucket**: Yes
   - **S3 bucket name**: privilege-analytics-logs-[account-id]
   - **Log file prefix**: cloudtrail-logs/

![S3 Bucket Configuration](/images/5/s3-bucket-config.png?featherlight=false&width=90pc)

5. Enable **Log file validation**
6. Click **Create trail**

## Step 2: Amazon Athena Setup

### 2.1 Create Athena Database

1. Open **Amazon Athena** in the console
2. Click **Query editor**
3. Set up query result location if prompted

![Athena Query Editor](/images/5/athena-query-editor.png?featherlight=false&width=90pc)

4. Create database for privilege analytics:

```sql
CREATE DATABASE privilege_analytics_db;
```

![Create Database](/images/5/create-database.png?featherlight=false&width=90pc)

### 2.2 Create CloudTrail Table

1. Use the database:

```sql
USE privilege_analytics_db;
```

2. Create table for CloudTrail logs:

![Create CloudTrail Table](/images/5/create-cloudtrail-table.png?featherlight=false&width=90pc)

### 2.3 Run Analytics Queries

1. Query for high-privilege activities:

```sql
SELECT 
  useridentity.username,
  eventname,
  sourceipaddress,
  eventtime,
  COUNT(*) as event_count
FROM cloudtrail_logs
WHERE eventname IN ('AssumeRole', 'AttachUserPolicy', 'CreateRole')
GROUP BY useridentity.username, eventname, sourceipaddress, eventtime
ORDER BY event_count DESC;
```

![Analytics Query Results](/images/5/analytics-query-results.png?featherlight=false&width=90pc)

## Step 3: QuickSight Dashboard Setup

### 3.1 Create QuickSight Account

1. Open **Amazon QuickSight** in the console
2. Sign up for QuickSight if not already done
3. Choose **Standard** edition

![QuickSight Signup](/images/5/quicksight-signup.png?featherlight=false&width=90pc)

### 3.2 Create Data Source

1. Click **Datasets** in QuickSight
2. Click **New dataset**
3. Choose **Athena** as data source

![Create Dataset](/images/5/create-dataset.png?featherlight=false&width=90pc)

4. Configure Athena connection:
   - **Data source name**: PrivilegeAnalytics
   - **Database**: privilege_analytics_db
   - **Table**: cloudtrail_logs

![Athena Data Source](/images/5/athena-data-source.png?featherlight=false&width=90pc)

### 3.3 Create Analysis Dashboard

1. Click **Create analysis**
2. Add visualizations for:
   - Top users by activity
   - High-risk events timeline
   - Geographic access patterns

![QuickSight Dashboard](/images/5/quicksight-dashboard.png?featherlight=false&width=90pc)

3. Configure filters and parameters
4. Publish the dashboard

![Publish Dashboard](/images/5/publish-dashboard.png?featherlight=false&width=90pc)

## Step 4: Set Up Automated Analysis

### 4.1 Create Lambda for Risk Scoring

1. Open **AWS Lambda** console
2. Create function **PrivilegeRiskScoring**
3. Configure to run daily via EventBridge

![Risk Scoring Lambda](/images/5/risk-scoring-lambda.png?featherlight=false&width=90pc)

### 4.2 Configure CloudWatch Alarms

1. Open **CloudWatch** console
2. Create alarms for high-risk activities
3. Set SNS notifications

![CloudWatch Alarms](/images/5/cloudwatch-alarms.png?featherlight=false&width=90pc)

## Expected Results

After completion:

- ✅ CloudTrail collecting privilege data
- ✅ Athena tables for analytics queries
- ✅ QuickSight dashboard for visualization
- ✅ Automated risk scoring
- ✅ CloudWatch monitoring and alerts

![Privilege Analytics Complete](/images/5/privilege-analytics-complete.png?featherlight=false&width=90pc)

## Next Steps

Continue to [6. Risk Assessment](../6-danh-gia-rui-ro) to set up comprehensive risk assessment.