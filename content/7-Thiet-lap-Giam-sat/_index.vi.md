---
title: "7. Thiết lập Giám sát"
chapter: false
weight: 7
---

## Mục tiêu

Thiết lập hệ thống giám sát toàn diện với CloudWatch Alarms, Dashboard và SNS notifications để theo dõi Identity Governance metrics.

## Bước 1: Xác minh CloudWatch Metrics

### 1.1 Kiểm tra Lambda Metrics

1. Mở **Amazon CloudWatch** console
2. Click **Metrics** ở sidebar
3. Xác minh metrics từ các Lambda functions:
   - AccessCertificationTrigger
   - PrivilegeAnalyticsEngine
   - RiskAssessmentEngine

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/metrics1.png?featherlight=false&width=90pc)

### 1.2 Kiểm tra DynamoDB Metrics

1. Trong CloudWatch Metrics
2. Click **AWS/DynamoDB**
3. Xác minh metrics cho tables:
   - AccessCertifications
   - RiskAssessments

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/metrics2.png?featherlight=false&width=90pc)

## Bước 2: Tạo CloudWatch Alarms

### 2.1 Tạo Alarm cho Lambda Errors

1. Trong **CloudWatch** console
2. Click **Alarms** ở sidebar
3. Click **Create alarm**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/alarm1.png?featherlight=false&width=90pc)

4. Chọn metric:
   - **Namespace**: AWS/Lambda
   - **Metric**: Errors
   - **Function**: AccessCertificationTrigger

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/alarm2.png?featherlight=false&width=90pc)

5. Cấu hình conditions:
   - **Threshold type**: Static
   - **Condition**: Greater than
   - **Threshold value**: 0
   - **Period**: 5 minutes

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/alarm3.png?featherlight=false&width=90pc)

6. Cấu hình actions:
   - **SNS topic**: RiskAssessmentAlerts (từ chương 6)
   - **Alarm name**: `Lambda-AccessCertification-Errors`

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/alarm4.png?featherlight=false&width=90pc)

7. Click **Create alarm**

### 2.2 Tạo Alarm cho High Risk Users

1. Click **Create alarm**
2. Chọn **Custom metric**
3. Tạo custom metric cho high risk users:
   - **Namespace**: IdentityGovernance
   - **Metric**: HighRiskUserCount

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/alarm5.png?featherlight=false&width=90pc)

4. Cấu hình threshold:
   - **Condition**: Greater than 5
   - **Period**: 1 hour

5. Click **Create alarm**

## Bước 3: Tạo Comprehensive Dashboard

### 3.1 Tạo Main Dashboard

1. Trong **CloudWatch** console
2. Click **Dashboards**
3. Click **Create dashboard**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/dashboard1.png?featherlight=false&width=90pc)

4. Nhập dashboard name: `IdentityGovernanceDashboard`
5. Click **Create dashboard**

### 3.2 Thêm Lambda Metrics Widget

1. Click **Add widget**
2. Chọn **Line** chart
3. Cấu hình metrics:
   - Lambda Invocations
   - Lambda Errors
   - Lambda Duration

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/dashboard2.png?featherlight=false&width=90pc)

4. Click **Create widget**

### 3.3 Thêm DynamoDB Metrics Widget

1. Click **Add widget**
2. Chọn **Number** widget
3. Cấu hình metrics:
   - DynamoDB Item Count
   - DynamoDB Read/Write Capacity

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/dashboard3.png?featherlight=false&width=90pc)

4. Click **Create widget**

### 3.4 Thêm Risk Assessment Widget

1. Click **Add widget**
2. Chọn **Pie** chart
3. Tạo custom metrics cho risk levels:
   - High Risk Users
   - Medium Risk Users
   - Low Risk Users

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/dashboard4.png?featherlight=false&width=90pc)

4. Click **Create widget**

## Bước 4: Thiết lập Custom Metrics Lambda

### 4.1 Tạo Lambda Function cho Custom Metrics

1. Mở **AWS Lambda** console
2. Click **Create function**
3. Nhập thông tin function:
   - **Function name**: `CustomMetricsPublisher`
   - **Runtime**: Python 3.9

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/lambda1.png?featherlight=false&width=90pc)

### 4.2 Cấu hình Code cho Custom Metrics

1. Thay thế code mặc định:

```python
import json
import boto3
from datetime import datetime
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

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/lambda2.png?featherlight=false&width=90pc)

2. Click **Deploy**

### 4.3 Cấu hình IAM Permissions

1. Thêm permissions cho Lambda:
   - **AmazonDynamoDBReadOnlyAccess**
   - **CloudWatchFullAccess**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/lambda3.png?featherlight=false&width=90pc)

### 4.4 Tạo EventBridge Schedule

1. Tạo schedule để chạy CustomMetricsPublisher mỗi 15 phút
2. Cấu hình tương tự như các chương trước

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/lambda4.png?featherlight=false&width=90pc)

## Bước 5: Kiểm tra Monitoring System

### 5.1 Test Custom Metrics

1. Chạy Lambda function **CustomMetricsPublisher**
2. Kiểm tra CloudWatch Metrics có custom metrics mới

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/test1.png?featherlight=false&width=90pc)

### 5.2 Test Alarms

1. Tạo test error trong Lambda function
2. Xác minh alarm được trigger
3. Kiểm tra SNS notification

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/test2.png?featherlight=false&width=90pc)

### 5.3 Xác minh Dashboard

1. Vào **IdentityGovernanceDashboard**
2. Xác minh tất cả widgets hiển thị data
3. Kiểm tra real-time updates

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/7/test3.png?featherlight=false&width=90pc)

## Kết quả Mong đợi

Sau khi hoàn thành:

- ✅ Comprehensive CloudWatch Dashboard
- ✅ Automated alarms for critical metrics
- ✅ Custom metrics for Identity Governance KPIs
- ✅ SNS notifications for alerts
- ✅ Real-time monitoring of all components
- ✅ Historical trend analysis

![Hoàn thành Monitoring Setup](https://trtrantnt.github.io/workshop/images/7/complete.png?featherlight=false&width=90pc)

## Tiếp theo

Chuyển sang [8. Quy trình Vận hành](../8-quy-trinh-van-hanh) để thiết lập các quy trình vận hành hàng ngày.