---
title: "7. Monitoring Setup"
chapter: false
weight: 7
---

## Objective

Set up comprehensive monitoring system with CloudWatch Alarms, Dashboard and SNS notifications to track Identity Governance metrics.

## Step 1: Verify CloudWatch Metrics

### 1.1 Check Lambda Metrics

1. Open **Amazon CloudWatch** console
2. Click **Metrics** in sidebar
3. Verify metrics from Lambda functions:
   - AccessCertificationTrigger
   - PrivilegeAnalyticsEngine
   - RiskAssessmentEngine

### 1.2 Check DynamoDB Metrics

1. In CloudWatch Metrics
2. Click **AWS/DynamoDB**
3. Verify metrics for tables:
   - AccessCertifications
   - RiskAssessments

## Step 2: Create CloudWatch Alarms

### 2.1 Create Alarm for Lambda Errors

1. In **CloudWatch** console
2. Click **Alarms** in sidebar
3. Click **Create alarm**
4. Select metric:
   - **Namespace**: AWS/Lambda
   - **Metric**: Errors
   - **Function**: AccessCertificationTrigger
5. Configure conditions:
   - **Threshold type**: Static
   - **Condition**: Greater than
   - **Threshold value**: 0
   - **Period**: 5 minutes
6. Configure actions:
   - **SNS topic**: RiskAssessmentAlerts (from chapter 6)
   - **Alarm name**: `Lambda-AccessCertification-Errors`
7. Click **Create alarm**

### 2.2 Create Alarm for High Risk Users

1. Click **Create alarm**
2. Select **Custom metric**
3. Create custom metric for high risk users:
   - **Namespace**: IdentityGovernance
   - **Metric**: HighRiskUserCount
4. Configure threshold:
   - **Condition**: Greater than 5
   - **Period**: 1 hour
5. Click **Create alarm**

## Step 3: Create Comprehensive Dashboard

### 3.1 Create Main Dashboard

1. In **CloudWatch** console
2. Click **Dashboards**
3. Click **Create dashboard**
4. Enter dashboard name: `IdentityGovernanceDashboard`
5. Click **Create dashboard**

### 3.2 Add Lambda Metrics Widget

1. Click **Add widget**
2. Select **Line** chart
3. Configure metrics:
   - Lambda Invocations
   - Lambda Errors
   - Lambda Duration
4. Click **Create widget**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/7/cw3.png?featherlight=false&width=90pc)

### 3.3 Add DynamoDB Metrics Widget

1. Click **Add widget**
2. Select **Number** widget
3. Configure metrics:
   - DynamoDB Item Count
   - DynamoDB Read/Write Capacity
4. Click **Create widget**

### 3.4 Add Risk Assessment Widget

1. Click **Add widget**
2. Select **Pie** chart
3. Create custom metrics for risk levels:
   - High Risk Users
   - Medium Risk Users
   - Low Risk Users
4. Click **Create widget**

## Step 4: Setup Custom Metrics Lambda

### 4.1 Create Lambda Function for Custom Metrics

1. Open **AWS Lambda** console
2. Click **Create function**
3. Enter function details:
   - **Function name**: `CustomMetricsPublisher`
   - **Runtime**: Python 3.9

### 4.2 Configure Code for Custom Metrics

1. Replace default code:

```python
import json
import boto3
from datetime import datetime, timedelta
from boto3.dynamodb.conditions import Key

def lambda_handler(event, context):
    print("Custom Metrics Publisher Started")
    
    # Initialize AWS clients
    dynamodb = boto3.resource('dynamodb')
    cloudwatch = boto3.client('cloudwatch')
    
    try:
        # Get risk assessment data
        risk_metrics = get_risk_metrics(dynamodb)
        
        # Get certification metrics
        cert_metrics = get_certification_metrics(dynamodb)
        
        # Publish custom metrics
        publish_metrics(cloudwatch, risk_metrics, cert_metrics)
        
        return {
            'statusCode': 200,
            'body': json.dumps('Custom metrics published successfully')
        }
        
    except Exception as e:
        print(f'Error publishing metrics: {str(e)}')
        return {
            'statusCode': 500,
            'body': json.dumps(f'Error: {str(e)}')
        }

def get_risk_metrics(dynamodb):
    """Get risk assessment metrics from DynamoDB"""
    table = dynamodb.Table('RiskAssessments')
    
    # Get latest user risk assessments
    response = table.scan(
        FilterExpression='AssessmentType = :type',
        ExpressionAttributeValues={':type': 'User Risk Assessment'}
    )
    
    risk_levels = {'LOW': 0, 'MEDIUM': 0, 'HIGH': 0, 'CRITICAL': 0}
    
    for item in response['Items']:
        risk_level = item.get('RiskLevel', 'LOW')
        if risk_level in risk_levels:
            risk_levels[risk_level] += 1
    
    return risk_levels

def get_certification_metrics(dynamodb):
    """Get certification metrics from DynamoDB"""
    table = dynamodb.Table('AccessCertifications')
    
    response = table.scan()
    
    total_certifications = len(response['Items'])
    recent_certifications = 0
    
    # Count recent certifications (last 30 days)
    thirty_days_ago = (datetime.now() - timedelta(days=30)).isoformat()
    
    for item in response['Items']:
        cert_date = item.get('CertificationDate', '')
        if cert_date > thirty_days_ago:
            recent_certifications += 1
    
    return {
        'total': total_certifications,
        'recent': recent_certifications
    }

def publish_metrics(cloudwatch, risk_metrics, cert_metrics):
    """Publish custom metrics to CloudWatch"""
    
    # Publish risk metrics
    for risk_level, count in risk_metrics.items():
        cloudwatch.put_metric_data(
            Namespace='IdentityGovernance',
            MetricData=[
                {
                    'MetricName': f'{risk_level}RiskUserCount',
                    'Value': count,
                    'Unit': 'Count',
                    'Timestamp': datetime.now()
                }
            ]
        )
    
    # Publish certification metrics
    cloudwatch.put_metric_data(
        Namespace='IdentityGovernance',
        MetricData=[
            {
                'MetricName': 'TotalCertifications',
                'Value': cert_metrics['total'],
                'Unit': 'Count',
                'Timestamp': datetime.now()
            },
            {
                'MetricName': 'RecentCertifications',
                'Value': cert_metrics['recent'],
                'Unit': 'Count',
                'Timestamp': datetime.now()
            }
        ]
    )
    
    print(f"Published metrics: Risk={risk_metrics}, Cert={cert_metrics}")
```

2. Click **Deploy**

### 4.3 Configure IAM Permissions

1. Add permissions for Lambda:
   - **AmazonDynamoDBReadOnlyAccess**
   - **CloudWatchFullAccess**

### 4.4 Create EventBridge Schedule

1. Create schedule to run CustomMetricsPublisher every 15 minutes
2. Configure similar to previous chapters

## Step 5: Test Monitoring System

### 5.1 Test Custom Metrics

1. Run Lambda function **CustomMetricsPublisher**
2. Check CloudWatch Metrics for new custom metrics

### 5.2 Test Alarms

1. Create test error in Lambda function
2. Verify alarm is triggered
3. Check SNS notification

### 5.3 Verify Dashboard

1. Go to **IdentityGovernanceDashboard**
2. Verify all widgets display data
3. Check real-time updates

## Expected Results

After completion:

- ✅ Comprehensive CloudWatch Dashboard
- ✅ Automated alarms for critical metrics
- ✅ Custom metrics for Identity Governance KPIs
- ✅ SNS notifications for alerts
- ✅ Real-time monitoring of all components
- ✅ Historical trend analysis

## Next Steps

Proceed to [8. Operational Procedures](../8-quy-trinh-van-hanh) to set up daily operational procedures.