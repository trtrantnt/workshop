---
title: "8. Operational Procedures"
chapter: false
weight: 8
---

## Objective

Establish daily operational procedures to maintain and optimize the Identity Governance system, including automation workflows and operational procedures.

## Step 1: Create Operational Dashboard

### 1.1 Create Operations Dashboard

1. Open **Amazon CloudWatch** console
2. Click **Dashboards**
3. Click **Create dashboard**
4. Enter dashboard name: `IdentityGovernanceOperations`
5. Add widgets for:
   - Daily certification status
   - Risk assessment trends
   - System health metrics
   - Operational KPIs

## Step 2: Create Daily Operations Lambda

### 2.1 Create Lambda Function

1. Open **AWS Lambda** console
2. Click **Create function**
3. Enter function details:
   - **Function name**: `DailyOperationsEngine`
   - **Runtime**: Python 3.9

![Navigate to S3](https://trtrantnt.github.io/workshop/images/8/ld1.png?featherlight=false&width=90pc)

### 2.2 Configure Code for Daily Operations

1. Replace default code:

```python
import json
import boto3
from datetime import datetime, timedelta
from decimal import Decimal

def lambda_handler(event, context):
    print("Daily Operations Engine Started")
    
    # Initialize AWS clients
    dynamodb = boto3.resource('dynamodb')
    sns = boto3.client('sns')
    ses = boto3.client('ses')
    
    try:
        # Perform daily operations
        operations_report = perform_daily_operations(dynamodb)
        
        # Generate daily report
        daily_report = generate_daily_report(operations_report)
        
        # Send notifications
        send_daily_notifications(sns, ses, daily_report)
        
        # Store operations log
        store_operations_log(dynamodb, operations_report)
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': 'Daily operations completed',
                'report': daily_report
            })
        }
        
    except Exception as e:
        print(f'Error in daily operations: {str(e)}')
        return {
            'statusCode': 500,
            'body': json.dumps(f'Error: {str(e)}')
        }

def perform_daily_operations(dynamodb):
    """Perform daily operational tasks"""
    
    operations_report = {
        'date': datetime.now().strftime('%Y-%m-%d'),
        'certification_status': check_certification_status(dynamodb),
        'risk_summary': get_risk_summary(dynamodb),
        'system_health': check_system_health(dynamodb),
        'compliance_status': check_compliance_status(dynamodb)
    }
    
    return operations_report

def check_certification_status(dynamodb):
    """Check certification status for the day"""
    table = dynamodb.Table('AccessCertifications')
    
    # Get certifications from last 24 hours
    yesterday = (datetime.now() - timedelta(days=1)).isoformat()
    
    response = table.scan(
        FilterExpression='CertificationDate > :date',
        ExpressionAttributeValues={':date': yesterday}
    )
    
    total_certifications = len(response['Items'])
    
    # Count by status
    status_counts = {}
    for item in response['Items']:
        status = item.get('Status', 'Unknown')
        status_counts[status] = status_counts.get(status, 0) + 1
    
    return {
        'total_certifications': total_certifications,
        'status_breakdown': status_counts,
        'completion_rate': calculate_completion_rate(status_counts)
    }

def get_risk_summary(dynamodb):
    """Get current risk summary"""
    table = dynamodb.Table('RiskAssessments')
    
    # Get latest risk assessments
    response = table.scan(
        FilterExpression='AssessmentType = :type',
        ExpressionAttributeValues={':type': 'User Risk Assessment'}
    )
    
    risk_summary = {'LOW': 0, 'MEDIUM': 0, 'HIGH': 0, 'CRITICAL': 0}
    total_users = 0
    
    for item in response['Items']:
        risk_level = item.get('RiskLevel', 'LOW')
        if risk_level in risk_summary:
            risk_summary[risk_level] += 1
            total_users += 1
    
    return {
        'total_users': total_users,
        'risk_distribution': risk_summary,
        'high_risk_percentage': ((risk_summary['HIGH'] + risk_summary['CRITICAL']) / max(total_users, 1)) * 100
    }

def check_system_health(dynamodb):
    """Check overall system health"""
    
    # Check if all components are functioning
    health_status = {
        'certification_engine': check_lambda_health('AccessCertificationTrigger'),
        'privilege_analytics': check_lambda_health('PrivilegeAnalyticsEngine'),
        'risk_assessment': check_lambda_health('RiskAssessmentEngine'),
        'custom_metrics': check_lambda_health('CustomMetricsPublisher'),
        'database_health': check_dynamodb_health(dynamodb)
    }
    
    overall_health = all(health_status.values())
    
    return {
        'overall_status': 'HEALTHY' if overall_health else 'DEGRADED',
        'component_status': health_status
    }

def check_compliance_status(dynamodb):
    """Check compliance status"""
    
    # Calculate compliance metrics
    cert_table = dynamodb.Table('AccessCertifications')
    risk_table = dynamodb.Table('RiskAssessments')
    
    # Get recent certifications
    thirty_days_ago = (datetime.now() - timedelta(days=30)).isoformat()
    
    cert_response = cert_table.scan(
        FilterExpression='CertificationDate > :date',
        ExpressionAttributeValues={':date': thirty_days_ago}
    )
    
    recent_certifications = len(cert_response['Items'])
    
    # Get high-risk users
    risk_response = risk_table.scan(
        FilterExpression='AssessmentType = :type AND RiskLevel IN (:high, :critical)',
        ExpressionAttributeValues={
            ':type': 'User Risk Assessment',
            ':high': 'HIGH',
            ':critical': 'CRITICAL'
        }
    )
    
    high_risk_users = len(risk_response['Items'])
    
    # Calculate compliance score
    compliance_score = calculate_compliance_score(recent_certifications, high_risk_users)
    
    return {
        'compliance_score': compliance_score,
        'recent_certifications': recent_certifications,
        'high_risk_users': high_risk_users,
        'status': 'COMPLIANT' if compliance_score >= 80 else 'NON_COMPLIANT'
    }

def calculate_completion_rate(status_counts):
    """Calculate certification completion rate"""
    total = sum(status_counts.values())
    completed = status_counts.get('Completed', 0) + status_counts.get('Approved', 0)
    return (completed / max(total, 1)) * 100

def check_lambda_health(function_name):
    """Check Lambda function health (simplified)"""
    # In real implementation, check CloudWatch metrics
    return True

def check_dynamodb_health(dynamodb):
    """Check DynamoDB health"""
    try:
        # Simple health check by listing tables
        tables = ['AccessCertifications', 'RiskAssessments']
        for table_name in tables:
            table = dynamodb.Table(table_name)
            table.table_status  # This will raise exception if table doesn't exist
        return True
    except Exception:
        return False

def calculate_compliance_score(recent_certs, high_risk_users):
    """Calculate overall compliance score"""
    base_score = 100
    
    # Deduct points for lack of recent certifications
    if recent_certs < 10:
        base_score -= 20
    elif recent_certs < 20:
        base_score -= 10
    
    # Deduct points for high-risk users
    if high_risk_users > 10:
        base_score -= 30
    elif high_risk_users > 5:
        base_score -= 15
    elif high_risk_users > 0:
        base_score -= 5
    
    return max(base_score, 0)

def generate_daily_report(operations_report):
    """Generate daily operations report"""
    
    report = {
        'date': operations_report['date'],
        'summary': {
            'total_certifications': operations_report['certification_status']['total_certifications'],
            'completion_rate': round(operations_report['certification_status']['completion_rate'], 2),
            'high_risk_users': operations_report['risk_summary']['risk_distribution']['HIGH'] + 
                             operations_report['risk_summary']['risk_distribution']['CRITICAL'],
            'system_status': operations_report['system_health']['overall_status'],
            'compliance_score': operations_report['compliance_status']['compliance_score']
        },
        'recommendations': generate_recommendations(operations_report)
    }
    
    return report

def generate_recommendations(operations_report):
    """Generate operational recommendations"""
    recommendations = []
    
    # Check completion rate
    completion_rate = operations_report['certification_status']['completion_rate']
    if completion_rate < 80:
        recommendations.append("Low certification completion rate. Consider sending reminder notifications.")
    
    # Check high-risk users
    high_risk_count = (operations_report['risk_summary']['risk_distribution']['HIGH'] + 
                      operations_report['risk_summary']['risk_distribution']['CRITICAL'])
    if high_risk_count > 5:
        recommendations.append(f"High number of high-risk users ({high_risk_count}). Review and remediate immediately.")
    
    # Check system health
    if operations_report['system_health']['overall_status'] != 'HEALTHY':
        recommendations.append("System health degraded. Check component status and resolve issues.")
    
    # Check compliance
    if operations_report['compliance_status']['status'] != 'COMPLIANT':
        recommendations.append("Compliance status is non-compliant. Review certification and risk management processes.")
    
    return recommendations

def send_daily_notifications(sns, ses, daily_report):
    """Send daily notifications"""
    
    # Prepare notification message
    message = format_daily_report_message(daily_report)
    
    # Send SNS notification
    try:
        sns.publish(
            TopicArn='arn:aws:sns:region:account:RiskAssessmentAlerts',  # Update with actual ARN
            Subject='Daily Identity Governance Report',
            Message=message
        )
    except Exception as e:
        print(f'Error sending SNS notification: {str(e)}')

def format_daily_report_message(report):
    """Format daily report message"""
    
    message = f"""
Daily Identity Governance Report - {report['date']}

SUMMARY:
- Total Certifications: {report['summary']['total_certifications']}
- Completion Rate: {report['summary']['completion_rate']}%
- High Risk Users: {report['summary']['high_risk_users']}
- System Status: {report['summary']['system_status']}
- Compliance Score: {report['summary']['compliance_score']}

RECOMMENDATIONS:
"""
    
    for rec in report['recommendations']:
        message += f"- {rec}\n"
    
    return message

def store_operations_log(dynamodb, operations_report):
    """Store daily operations log"""
    table = dynamodb.Table('RiskAssessments')
    
    table.put_item(
        Item={
            'AssessmentId': f"daily-ops-{operations_report['date']}",
            'AssessmentType': 'Daily Operations Report',
            'Date': operations_report['date'],
            'CertificationCount': operations_report['certification_status']['total_certifications'],
            'CompletionRate': Decimal(str(operations_report['certification_status']['completion_rate'])),
            'HighRiskUsers': operations_report['risk_summary']['risk_distribution']['HIGH'] + 
                           operations_report['risk_summary']['risk_distribution']['CRITICAL'],
            'SystemStatus': operations_report['system_health']['overall_status'],
            'ComplianceScore': operations_report['compliance_status']['compliance_score'],
            'ComplianceStatus': operations_report['compliance_status']['status']
        }
    )
```

1. Click **Deploy**

### 2.3 Configure IAM Permissions

1. Add permissions for Lambda:
   - **AmazonDynamoDBFullAccess**
   - **AmazonSNSFullAccess**
   - **AmazonSESFullAccess**

## Step 3: Setup Automated Workflows

### 3.1 Create EventBridge Schedule for Daily Operations

1. Open **Amazon EventBridge** console
2. Click **Create rule**
3. Configure rule:
   - **Name**: `DailyOperationsSchedule`
   - **Rule type**: Schedule
   - **Schedule expression**: `cron(0 8 * * ? *)`  # Daily at 8:00 AM
   - **Target**: Lambda function `DailyOperationsEngine`

### 3.2 Create Weekly Summary Schedule

1. Create additional EventBridge rule:
   - **Name**: `WeeklyOperationsReview`
   - **Schedule expression**: `cron(0 9 ? * MON *)`  # Every Monday at 9:00 AM
2. Configure to trigger weekly summary Lambda

### 3.3 Create Operations Notification Topics

1. Open **Amazon SNS** console
2. Create topic: `DailyOperationsAlerts`
3. Add subscriptions for operations team
4. Configure email notifications for daily reports

## Step 4: Create Operational Runbooks

### 4.1 Create S3 Bucket for Documentation

1. Open **Amazon S3** console
2. Create bucket: `identity-governance-runbooks`
3. Create folder structure:
   - `/sop/` - Standard Operating Procedures
   - `/runbooks/` - Incident Response Runbooks
   - `/templates/` - Document Templates
4. Upload operational procedures and documentation

### 4.2 Create Systems Manager Documents

1. Open **AWS Systems Manager** console
2. Click **Documents** → **Create document**
3. Create automation documents for common tasks:
   - Daily health checks
   - Incident response procedures
   - Emergency access procedures

## Step 5: Test Operational Workflows

### 5.1 Test Daily Operations

1. Run Lambda function **DailyOperationsEngine** manually
2. Verify daily report is generated correctly
3. Check SNS notifications are delivered
4. Validate DynamoDB logging is working

### 5.2 Verify Operations Dashboard

1. Check **IdentityGovernanceOperations** dashboard in CloudWatch
2. Verify all widgets display correct metrics
3. Test real-time metric updates
4. Configure dashboard sharing for operations team

### 5.3 Test Automation Workflows

1. Verify EventBridge scheduled executions
2. Check error handling and retry logic
3. Test notification delivery channels
4. Validate logging and monitoring

## Expected Results

After completion:

- ✅ Automated daily operations workflows
- ✅ Comprehensive operational dashboard
- ✅ Daily and weekly reporting automation
- ✅ Operational runbooks and procedures
- ✅ Health monitoring and alerting
- ✅ Compliance tracking and reporting

## Next Steps

Proceed to [9. Audit Procedures](../9-quy-trinh-kiem-toan) to set up audit processes and compliance procedures.
 
 