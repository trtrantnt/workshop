---
title: "6. Risk Assessment"
chapter: false
weight: 6
---

## Objective

Establish a comprehensive risk assessment system to detect, analyze, and prioritize security risks related to identity and access management.

## Risk Assessment Architecture

```mermaid
graph TB
    A[CloudTrail Events] --> B[Lambda Risk Engine]
    C[IAM Data] --> B
    B --> D[DynamoDB Risk Data]
    D --> E[Security Hub Findings]
    E --> F[CloudWatch Metrics]
    F --> G[SNS Alerts]
    D --> H[QuickSight Dashboard]
```

## Step 1: Lambda Risk Engine Setup

### 1.1 Create Risk Assessment Lambda

1. Open **AWS Lambda** in the console
2. Click **Create function**
3. Configure:
   - **Function name**: IdentityRiskEngine
   - **Runtime**: Python 3.9
   - **Role**: Create new role with DynamoDB and Security Hub permissions

![Create Risk Lambda](/images/6/create-risk-lambda.png?featherlight=false&width=90pc)

### 1.2 Configure Risk Assessment Logic

1. Add environment variables:
   - **RISK_TABLE**: RiskAssessments
   - **SNS_TOPIC**: Risk alert topic ARN

![Risk Lambda Config](/images/6/risk-lambda-config.png?featherlight=false&width=90pc)

2. Upload risk assessment code
3. Configure EventBridge trigger for daily execution

![Lambda EventBridge Trigger](/images/6/lambda-eventbridge-trigger.png?featherlight=false&width=90pc)

## Step 2: Security Hub Integration

### 3.1 Enable AWS Security Hub

1. Open **AWS Security Hub** in the console
2. Click **Go to Security Hub**

![Enable Security Hub](/images/6/enable-security-hub.png?featherlight=false&width=90pc)

3. Enable security standards:
   - **AWS Foundational Security Standard**
   - **CIS AWS Foundations Benchmark**
   - **PCI DSS**

![Security Standards](/images/6/security-standards.png?featherlight=false&width=90pc)

4. Click **Enable Security Hub**

### 3.2 Configure Integrations

1. Go to **Integrations** tab
2. Enable integrations:
   - **AWS Config**
   - **Amazon GuardDuty**
   - **AWS Inspector**

![Security Hub Integrations](/images/6/security-hub-integrations.png?featherlight=false&width=90pc)

3. Configure custom insights for risk assessment

![Custom Insights](/images/6/custom-insights.png?featherlight=false&width=90pc)

## Step 3: CloudWatch Dashboard for Risk Monitoring

### 4.1 Create Risk Assessment Dashboard

1. Open **Amazon CloudWatch** console
2. Click **Dashboards** in sidebar
3. Click **Create dashboard**

![Create Dashboard](/images/6/create-dashboard.png?featherlight=false&width=90pc)

4. Name: **IdentityGovernanceRiskDashboard**
5. Add widgets for:
   - **Security Hub findings**
   - **GuardDuty threats**
   - **Config compliance**

![Dashboard Widgets](/images/6/dashboard-widgets.png?featherlight=false&width=90pc)

### 4.2 Configure Risk Alarms

1. Click **Alarms** in CloudWatch
2. Click **Create alarm**

![Create Alarm](/images/6/create-alarm.png?featherlight=false&width=90pc)

3. Configure alarms for:
   - **High severity findings**
   - **Critical GuardDuty threats**
   - **Config rule violations**

![Risk Alarms](/images/6/risk-alarms.png?featherlight=false&width=90pc)

4. Set SNS notifications

![SNS Notifications](/images/6/sns-notifications.png?featherlight=false&width=90pc)

## Expected Results

After completion:

- ✅ AWS Config monitoring compliance
- ✅ GuardDuty detecting threats
- ✅ Security Hub centralizing findings
- ✅ CloudWatch dashboard for risk metrics
- ✅ Automated alerting for high-risk events
- ✅ Integrated security monitoring

![Risk Assessment Complete](/images/6/risk-assessment-complete.png?featherlight=false&width=90pc)

## Next Steps

Continue to [7. Monitoring Setup](../7-thiet-lap-giam-sat) to set up comprehensive monitoring.