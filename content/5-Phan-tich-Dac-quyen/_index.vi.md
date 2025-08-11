---
title: "5. Phân tích Đặc quyền"
chapter: false
weight: 5
---

## Mục tiêu

Phân tích và giám sát việc sử dụng đặc quyền để phát hiện rủi ro bảo mật, quyền thừa, và các pattern bất thường thông qua CloudTrail logs.

## Bước 1: Xác minh CloudTrail Data

### 1.1 Kiểm tra CloudTrail Logs

1. Mở **Amazon CloudTrail** trong console
2. Xác minh trail **IdentityGovernanceTrail** đã được tạo trong chương 2
3. Kiểm tra S3 bucket chứa CloudTrail logs

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/ct1.png?featherlight=false&width=90pc)

### 1.2 Xác minh S3 Bucket có CloudTrail Data

1. Vào **Amazon S3** console
2. Tìm bucket CloudTrail (tên dạng `aws-cloudtrail-logs-xxx`)
3. Xác minh có log files được tạo

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/ct2.png?featherlight=false&width=90pc)

## Bước 2: Tạo Lambda Function cho Privilege Analytics

### 2.1 Tạo Lambda Function

1. Mở **AWS Lambda** trong console
2. Click **Create function**
3. Chọn **Author from scratch**
4. Nhập thông tin function:
   - **Function name**: `PrivilegeAnalyticsEngine`
   - **Runtime**: Python 3.9
   - **Architecture**: x86_64

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/lambda1.png?featherlight=false&width=90pc)

5. Click **Create function**

### 2.2 Cấu hình Code cho Lambda Function

1. Trong tab **Code**, thay thế code mặc định bằng code sau:

```python
import json
import boto3
import gzip
from datetime import datetime, timedelta
from urllib.parse import unquote_plus

def lambda_handler(event, context):
    print("Privilege Analytics Engine Started")
    
    # Initialize AWS clients
    s3 = boto3.client('s3')
    dynamodb = boto3.resource('dynamodb')
    
    # Get the object from the event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = unquote_plus(event['Records'][0]['s3']['object']['key'])
    
    try:
        # Download and decompress CloudTrail log
        response = s3.get_object(Bucket=bucket, Key=key)
        
        if key.endswith('.gz'):
            content = gzip.decompress(response['Body'].read())
        else:
            content = response['Body'].read()
        
        # Parse CloudTrail log
        log_data = json.loads(content.decode('utf-8'))
        
        # Analyze privilege usage
        privilege_events = analyze_privilege_events(log_data['Records'])
        
        # Store analysis results
        store_analysis_results(privilege_events, dynamodb)
        
        return {
            'statusCode': 200,
            'body': json.dumps(f'Processed {len(privilege_events)} privilege events')
        }
        
    except Exception as e:
        print(f'Error processing {key}: {str(e)}')
        return {
            'statusCode': 500,
            'body': json.dumps(f'Error: {str(e)}')
        }

def analyze_privilege_events(records):
    """Analyze CloudTrail records for privilege usage patterns"""
    privilege_events = []
    
    high_privilege_actions = [
        'CreateUser', 'DeleteUser', 'AttachUserPolicy', 'DetachUserPolicy',
        'CreateRole', 'DeleteRole', 'AttachRolePolicy', 'DetachRolePolicy',
        'PutUserPolicy', 'DeleteUserPolicy', 'PutRolePolicy', 'DeleteRolePolicy'
    ]
    
    for record in records:
        event_name = record.get('eventName', '')
        
        if event_name in high_privilege_actions:
            privilege_event = {
                'eventTime': record.get('eventTime'),
                'eventName': event_name,
                'userIdentity': record.get('userIdentity', {}),
                'sourceIPAddress': record.get('sourceIPAddress'),
                'userAgent': record.get('userAgent'),
                'awsRegion': record.get('awsRegion'),
                'riskScore': calculate_risk_score(record)
            }
            privilege_events.append(privilege_event)
    
    return privilege_events

def calculate_risk_score(record):
    """Calculate risk score for privilege event (1-10 scale)"""
    base_score = 5
    
    # High-risk actions
    high_risk_actions = ['DeleteUser', 'DeleteRole', 'DetachUserPolicy']
    if record.get('eventName') in high_risk_actions:
        base_score += 3
    
    # External IP access
    source_ip = record.get('sourceIPAddress', '')
    if not source_ip.startswith('10.') and not source_ip.startswith('172.') and not source_ip.startswith('192.168.'):
        base_score += 2
    
    # Console vs API access
    user_agent = record.get('userAgent', '')
    if 'console' not in user_agent.lower():
        base_score += 1
    
    return min(base_score, 10)

def store_analysis_results(privilege_events, dynamodb):
    """Store analysis results in DynamoDB"""
    table = dynamodb.Table('RiskAssessments')
    
    for event in privilege_events:
        table.put_item(
            Item={
                'AssessmentId': f"privilege-{datetime.now().isoformat()}",
                'EventTime': event['eventTime'],
                'EventName': event['eventName'],
                'UserIdentity': json.dumps(event['userIdentity']),
                'SourceIP': event['sourceIPAddress'],
                'RiskScore': event['riskScore'],
                'AssessmentType': 'Privilege Analysis'
            }
        )
```

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/lambda2.png?featherlight=false&width=90pc)

2. Click **Deploy** để lưu thay đổi

### 2.3 Cấu hình IAM Role cho Lambda

1. Chuyển đến tab **Configuration**
2. Click **Permissions**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/lambda3.png?featherlight=false&width=90pc)

3. Click vào role name để mở IAM console
4. Click **Add permissions** → **Attach policies**
5. Tìm và attach các policies sau:
   - **AmazonS3ReadOnlyAccess**
   - **AmazonDynamoDBFullAccess**
6. Click **Add permissions**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/lambda4.png?featherlight=false&width=90pc)

## Bước 3: Thiết lập S3 Event Trigger

### 3.1 Cấu hình S3 Trigger cho Lambda

1. Trong Lambda function **PrivilegeAnalyticsEngine**
2. Click **Add trigger**
3. Chọn **S3** từ dropdown

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/trigger1.png?featherlight=false&width=90pc)

4. Cấu hình trigger:
   - **Bucket**: Chọn CloudTrail S3 bucket
   - **Event type**: All object create events
   - **Prefix**: `AWSLogs/` (optional)
   - **Suffix**: `.json.gz`

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/trigger2.png?featherlight=false&width=90pc)

5. Click **Add**

## Bước 4: Tạo CloudWatch Dashboard

### 4.1 Tạo Dashboard cho Privilege Analytics

1. Mở **Amazon CloudWatch** console
2. Click **Dashboards** ở sidebar
3. Click **Create dashboard**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/cw1.png?featherlight=false&width=90pc)

4. Nhập dashboard name: `PrivilegeAnalyticsDashboard`
5. Click **Create dashboard**

### 4.2 Thêm Widgets cho Dashboard

1. Click **Add widget**
2. Chọn **Line** chart
3. Cấu hình metric:
   - **Namespace**: AWS/Lambda
   - **Metric**: Invocations
   - **Function**: PrivilegeAnalyticsEngine

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/cw2.png?featherlight=false&width=90pc)

4. Click **Create widget**
5. Thêm thêm widgets cho:
   - Lambda Errors
   - Lambda Duration
   - DynamoDB Item Count

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/cw3.png?featherlight=false&width=90pc)

## Bước 5: Kiểm tra Privilege Analytics

### 5.1 Tạo Test Activity

1. Vào **IAM** console
2. Thực hiện một số actions để tạo CloudTrail logs:
   - Tạo test user
   - Attach/detach policies
   - Tạo test role

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/test1.png?featherlight=false&width=90pc)

### 5.2 Kiểm tra Lambda Execution

1. Vào **AWS Lambda** console
2. Chọn function **PrivilegeAnalyticsEngine**
3. Click tab **Monitor**
4. Xem CloudWatch logs để xác minh function đã chạy

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/test2.png?featherlight=false&width=90pc)

### 5.3 Xác minh DynamoDB Records

1. Vào **Amazon DynamoDB** console
2. Chọn table **RiskAssessments**
3. Click **Explore table items**
4. Xác minh có records mới với AssessmentType = 'Privilege Analysis'

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/5/test3.png?featherlight=false&width=90pc)

## Kết quả Mong đợi

Sau khi hoàn thành:

- ✅ CloudTrail logs được phân tích tự động
- ✅ Lambda function xử lý privilege events
- ✅ Risk scoring cho các privilege actions
- ✅ DynamoDB lưu trữ kết quả phân tích
- ✅ CloudWatch dashboard giám sát
- ✅ Real-time privilege monitoring

![Hoàn thành Privilege Analytics](https://trtrantnt.github.io/workshop/images/5/complete.png?featherlight=false&width=90pc)

## Tiếp theo

Chuyển sang [6. Đánh giá Rủi ro](../6-danh-gia-rui-ro) để thiết lập đánh giá rủi ro toàn diện.