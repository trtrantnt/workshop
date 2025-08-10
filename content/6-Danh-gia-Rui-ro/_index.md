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
    A[Data Sources] --> B[Risk Engine]
    B --> C[Risk Scoring]
    C --> D[Risk Classification]
    D --> E[Risk Prioritization]
    E --> F[Remediation Recommendations]
    F --> G[Risk Dashboard]
    
    H[Threat Intelligence] --> B
    I[Compliance Rules] --> B
    J[Business Context] --> B
```

## Step 1: AWS Config Setup for Risk Assessment

### 1.1 Enable AWS Config

1. Open **AWS Config** in the console
2. Click **Get started**

![Enable AWS Config](/images/6/enable-aws-config.png?featherlight=false&width=90pc)

3. Configure settings:
   - **Resource types**: All supported resource types
   - **Amazon S3 bucket**: Create new bucket
   - **Amazon SNS topic**: Create new topic

![Config Settings](/images/6/config-settings.png?featherlight=false&width=90pc)

4. Choose **AWS Config role**: Create new role
5. Click **Next**

![Config Role](/images/6/config-role.png?featherlight=false&width=90pc)

### 1.2 Add Config Rules

1. Click **Rules** in AWS Config
2. Click **Add rule**

![Add Config Rule](/images/6/add-config-rule.png?featherlight=false&width=90pc)

3. Add security-related rules:
   - **iam-user-mfa-enabled**
   - **root-access-key-check**
   - **iam-password-policy**

![Security Rules](/images/6/security-rules.png?featherlight=false&width=90pc)

## Step 2: GuardDuty Integration

### 2.1 Enable Amazon GuardDuty

1. Open **Amazon GuardDuty** in the console
2. Click **Get started**

![Enable GuardDuty](/images/6/enable-guardduty.png?featherlight=false&width=90pc)

3. Review service permissions
4. Click **Enable GuardDuty**

![GuardDuty Permissions](/images/6/guardduty-permissions.png?featherlight=false&width=90pc)

### 2.2 Configure Threat Intelligence

1. Go to **Settings** in GuardDuty
2. Click **Threat Intelligence**
3. Enable **AWS threat intelligence**

![Threat Intelligence](/images/6/threat-intelligence.png?featherlight=false&width=90pc)

4. Configure **Malware Protection**
5. Enable **S3 Protection**

![Malware Protection](/images/6/malware-protection.png?featherlight=false&width=90pc)

## Step 3: Security Hub Integration

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

## Step 4: CloudWatch Dashboard for Risk Monitoring

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