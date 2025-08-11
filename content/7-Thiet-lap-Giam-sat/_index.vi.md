---
title: "7. Thiết lập Giám sát"
chapter: false
weight: 7
---

## Mục tiêu

Thiết lập hệ thống giám sát toàn diện để theo dõi liên tục các hoạt động identity governance, phát hiện anomalies, và đảm bảo compliance.

## Kiến trúc Monitoring

![Kiến trúc Monitoring](/images/7/monitoring-architecture.png)

## Bước 1: Thiết lập CloudWatch Monitoring

### 1.1 Tạo SNS Topic cho Alerts

1. Mở **Amazon SNS** trong console
2. Click **Topics** → **Create topic**
3. Cấu hình:
   - **Type**: Standard
   - **Name**: IdentityGovernanceAlerts
   - **Display name**: Identity Governance Monitoring Alerts

![SNS Topic Creation](/images/7/sns-topic-creation.png)

4. Click **Create subscription**
5. Thêm email subscription cho security team

![SNS Subscription](/images/7/sns-subscription.png)

### 1.2 Tạo CloudWatch Log Group

1. Mở **CloudWatch** console
2. Click **Log groups** → **Create log group**
3. Cấu hình:
   - **Log group name**: /aws/identity-governance/events
   - **Retention setting**: 1 year

![Log Group Creation](/images/7/log-group-creation.png)

### 1.3 Thiết lập CloudWatch Alarms

1. Trong CloudWatch, click **Alarms** → **Create alarm**
2. Tạo các alarms:
   - **Failed Login Attempts**: Nhiều lần đăng nhập thất bại
   - **Privilege Escalation**: Phát hiện leo thang quyền
   - **Off-hours Access**: Truy cập ngoài giờ làm việc

![CloudWatch Alarms](/images/7/cloudwatch-alarms.png)

## Bước 2: Thiết lập Log Analytics

### 2.1 CloudWatch Insights Queries

1. Mở **CloudWatch Logs Insights**
2. Chọn log group: **/aws/cloudtrail**
3. Tạo saved queries:

**Query 1: Failed Login Attempts**
```
fields @timestamp, sourceIPAddress, userIdentity.userName, errorMessage
| filter eventName = "ConsoleLogin" and errorCode != "Success"
| stats count() by userIdentity.userName
| sort count desc
```

![Insights Query 1](/images/7/insights-query-1.png)

**Query 2: Privilege Escalation Events**
```
fields @timestamp, eventName, userIdentity.userName, sourceIPAddress
| filter eventName in ["AttachUserPolicy", "CreateRole", "PutUserPolicy"]
| sort @timestamp desc
```

![Insights Query 2](/images/7/insights-query-2.png)

### 2.2 Tạo Custom Dashboard

1. Trong CloudWatch, click **Dashboards** → **Create dashboard**
2. Tên: **IdentityGovernanceMonitoring**
3. Thêm widgets:
   - **Failed Login Attempts (Line chart)**
   - **Active Users (Number)**
   - **Policy Changes (Log table)**
   - **Geographic Access Map**

![Custom Dashboard](/images/7/custom-dashboard.png)

## Bước 3: Thiết lập Lambda Monitoring Function

### 3.1 Tạo Lambda Function

1. Mở **AWS Lambda** console
2. Click **Create function**
3. Cấu hình:
   - **Function name**: IdentityGovernanceMonitor
   - **Runtime**: Python 3.9
   - **Role**: IdentityGovernanceMonitoringRole

![Lambda Function Creation](/images/7/lambda-function-creation.png)

4. Cấu hình environment variables:
   - **LOG_GROUP_NAME**: /aws/identity-governance/events
   - **SNS_TOPIC_ARN**: ARN của SNS topic

![Lambda Environment Variables](/images/7/lambda-env-variables.png)

5. Thêm Lambda function code:

```python
import boto3
import json
import os
from datetime import datetime, timedelta

class IdentityGovernanceMonitor:
    def __init__(self):
        self.cloudwatch = boto3.client('cloudwatch')
        self.logs_client = boto3.client('logs')
        self.iam_client = boto3.client('iam')
        self.sso_client = boto3.client('sso-admin')
        self.sns_client = boto3.client('sns')
        
        self.log_group = os.environ.get('LOG_GROUP_NAME', '/aws/identity-governance/events')
        self.sns_topic = os.environ.get('SNS_TOPIC_ARN')
    
    def monitor_identity_events(self):
        """Monitor and analyze identity-related events"""
        
        monitoring_results = {
            'timestamp': datetime.now().isoformat(),
            'metrics_collected': [],
            'anomalies_detected': [],
            'alerts_sent': []
        }
        
        # Check for failed login attempts
        failed_logins = self.check_failed_logins()
        if failed_logins > 5:
            self.send_alert(f"High failed login attempts detected: {failed_logins}")
            monitoring_results['anomalies_detected'].append('high_failed_logins')
        
        # Check for privilege escalation
        privilege_changes = self.check_privilege_escalation()
        if privilege_changes:
            self.send_alert(f"Privilege escalation detected: {privilege_changes}")
            monitoring_results['anomalies_detected'].append('privilege_escalation')
        
        return monitoring_results
    
    def check_failed_logins(self):
        """Check for failed login attempts in the last hour"""
        end_time = datetime.now()
        start_time = end_time - timedelta(hours=1)
        
        query = """
        fields @timestamp, sourceIPAddress, userIdentity.userName, errorMessage
        | filter eventName = "ConsoleLogin" and errorCode != "Success"
        | stats count() as failed_attempts
        """
        
        try:
            response = self.logs_client.start_query(
                logGroupName='/aws/cloudtrail',
                startTime=int(start_time.timestamp()),
                endTime=int(end_time.timestamp()),
                queryString=query
            )
            
            # Get query results (simplified for demo)
            return 0  # Would return actual count
        except Exception as e:
            print(f"Error checking failed logins: {e}")
            return 0
    
    def check_privilege_escalation(self):
        """Check for privilege escalation events"""
        escalation_events = [
            'AttachUserPolicy',
            'CreateRole', 
            'PutUserPolicy',
            'AttachRolePolicy'
        ]
        
        # Check recent IAM events
        try:
            # This would query CloudTrail logs for privilege changes
            return []  # Would return actual events
        except Exception as e:
            print(f"Error checking privilege escalation: {e}")
            return []
    
    def send_alert(self, message):
        """Send alert via SNS"""
        try:
            self.sns_client.publish(
                TopicArn=self.sns_topic,
                Message=message,
                Subject='Identity Governance Alert'
            )
        except Exception as e:
            print(f"Error sending alert: {e}")

def lambda_handler(event, context):
    monitor = IdentityGovernanceMonitor()
    results = monitor.monitor_identity_events()
    
    return {
        'statusCode': 200,
        'body': json.dumps(results)
    }
```

### 3.2 Thiết lập EventBridge Schedule

1. Mở **EventBridge** console
2. Click **Rules** → **Create rule**
3. Cấu hình:
   - **Name**: IdentityMonitoringSchedule
   - **Schedule**: Rate(5 minutes)
   - **Target**: Lambda IdentityGovernanceMonitor

![EventBridge Schedule](/images/7/eventbridge-schedule.png)

## Bước 4: Anomaly Detection Setup

### 4.1 CloudWatch Anomaly Detection

1. Trong CloudWatch Metrics, chọn metric cần monitor
2. Click **Actions** → **Create anomaly detector**
3. Cấu hình threshold và notification

![Anomaly Detection](/images/7/anomaly-detection.png)

### 4.2 GuardDuty Integration

1. Mở **GuardDuty** console
2. Cấu hình findings export đến S3
3. Thiết lập EventBridge rule cho GuardDuty findings

![GuardDuty Integration](/images/7/guardduty-integration.png)

## Kết quả Mong đợi

Sau khi hoàn thành:

- ✅ Real-time monitoring của identity events
- ✅ Custom CloudWatch metrics và alarms
- ✅ Automated log analysis
- ✅ Anomaly detection và alerting
- ✅ Comprehensive monitoring dashboard
- ✅ Scheduled monitoring tasks

## Tiếp theo

Chuyển sang [8. Quy trình Vận hành](../8-quy-trinh-van-hanh) để thiết lập quy trình vận hành.