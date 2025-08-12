---
title: "5. Privilege Analytics"
chapter: false
weight: 5
---

## Objective

Analyze and monitor privilege usage to detect security risks, excessive permissions, and abnormal patterns through CloudTrail logs.

## Step 1: Verify CloudTrail Data

### 1.1 Check CloudTrail Logs

1. Open **Amazon CloudTrail** in the console
2. Verify that trail **IdentityGovernanceTrail** was created in chapter 2
3. Check S3 bucket containing CloudTrail logs
### 1.2 Verify S3 Bucket has CloudTrail Data

1. Go to **Amazon S3** console
2. Find CloudTrail bucket (name like `aws-cloudtrail-logs-xxx`)
3. Verify that log files are being created

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/5.1.png?featherlight=false&width=90pc)

## Step 2: Create Lambda Function for Privilege Analytics

### 2.1 Create Lambda Function

1. Open **AWS Lambda** in the console
2. Click **Create function**
3. Choose **Author from scratch**
4. Enter function details:
   - **Function name**: `PrivilegeAnalyticsEngine`
   - **Runtime**: Python 3.9
   - **Architecture**: x86_64

![Navigate to S3](https://trtrantnt.github.io/workshop/images/5/ld1.png?featherlight=false&width=90pc)

5. Click **Create function**

### 2.2 Configure Lambda Function Code

1. In the **Code** tab, replace the default code with the following:

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

![Navigate to S3](https://trtrantnt.github.io/workshop/images/5/ld2.png?featherlight=false&width=90pc)

2. Click **Deploy** to save changes

### 2.3 Configure IAM Role for Lambda

1. Go to **Configuration** tab
2. Click **Permissions**
3. Click on the role name to open IAM console
4. Click **Add permissions** → **Attach policies**
5. Search and attach the following policies:
   - **AmazonS3ReadOnlyAccess**
   - **AmazonDynamoDBFullAccess**
6. Click **Add permissions**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/5/ld3.png?featherlight=false&width=90pc)

## Step 3: Setup S3 Event Trigger

### 3.1 Configure S3 Trigger for Lambda

1. In Lambda function **PrivilegeAnalyticsEngine**
2. Click **Add trigger**
3. Select **S3** from dropdown

![Navigate to S3](https://trtrantnt.github.io/workshop/images/5/tg1.png?featherlight=false&width=90pc)

4. Configure trigger:
   - **Bucket**: Select CloudTrail S3 bucket
   - **Event type**: All object create events
   - **Prefix**: `AWSLogs/` (optional)
   - **Suffix**: `.json.gz`

![Navigate to S3](https://trtrantnt.github.io/workshop/images/5/tg2.png?featherlight=false&width=90pc)

5. Click **Add**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/5/tg3.png?featherlight=false&width=90pc)

## Step 4: Create CloudWatch Dashboard

### 4.1 Create Dashboard for Privilege Analytics

1. Open **Amazon CloudWatch** console
2. Click **Dashboards** in the sidebar
3. Click **Create dashboard**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/5/cw1.png?featherlight=false&width=90pc)

4. Enter dashboard name: `PrivilegeAnalyticsDashboard`
5. Click **Create dashboard**

### 4.2 Add Widget 1: Lambda Invocations

1. Click **Add widget**
2. Select **Line** chart
3. Click **Next**
4. Configure metric:
   - **Namespace**: AWS/Lambda
   - **Metric name**: Invocations
   - **Dimensions**: FunctionName = PrivilegeAnalyticsEngine
5. Click **Select metric**
6. Set widget name: "Lambda Invocations"
7. Click **Create widget**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/5/cw2.png?featherlight=false&width=90pc)

### 4.3 Add Widget 2: Lambda Errors

1. Click **Add widget** (in dashboard)
2. Select **Line** chart → **Next**
3. Configure metric:
   - **Namespace**: AWS/Lambda
   - **Metric name**: Errors
   - **Dimensions**: FunctionName = PrivilegeAnalyticsEngine
4. Click **Select metric**
5. Set widget name: "Lambda Errors"
6. Click **Create widget**

### 4.4 Add Widget 3: Lambda Duration

1. Click **Add widget**
2. Select **Line** chart → **Next**
3. Configure metric:
   - **Namespace**: AWS/Lambda
   - **Metric name**: Duration
   - **Dimensions**: FunctionName = PrivilegeAnalyticsEngine
4. Click **Select metric**
5. Set widget name: "Lambda Duration (ms)"
6. Click **Create widget**

### 4.5 Add Widget 4: DynamoDB Write Activity

1. Click **Add widget**
2. Select **Line** chart → **Next**
3. Configure metric:
   - **Namespace**: AWS/DynamoDB
   - **Metric name**: ConsumedWriteCapacityUnits
   - **Dimensions**: TableName = RiskAssessments
4. Click **Select metric**
5. Set widget name: "DynamoDB Write Activity"
6. Click **Create widget**

**Note**: This metric shows write activity to DynamoDB, indicating when new risk assessments are stored.

### 4.6 Save Dashboard

1. Click **Save dashboard** in the top right corner
2. Dashboard will display 4 widgets monitoring:
   - Number of Lambda invocations
   - Lambda errors
   - Lambda execution time
   - Number of items in DynamoDB

![Navigate to S3](https://trtrantnt.github.io/workshop/images/5/cw3.png?featherlight=false&width=90pc)

## Step 5: Test Privilege Analytics

### 5.1 Create Test Activity

1. Go to **IAM** console
2. Perform some actions to generate CloudTrail logs:
   - Create test user
   - Attach/detach policies
   - Create test role

![Navigate to S3](https://trtrantnt.github.io/workshop/images/5/test1.png?featherlight=false&width=90pc)

### 5.2 Check Lambda Execution

1. Go to **AWS Lambda** console
2. Select function **PrivilegeAnalyticsEngine**
3. Click **Monitor** tab
4. View CloudWatch logs to verify function execution

![Navigate to S3](https://trtrantnt.github.io/workshop/images/5/test2.png?featherlight=false&width=90pc)

### 5.3 Verify DynamoDB Records

1. Go to **Amazon DynamoDB** console
2. Select table **RiskAssessments**
3. Click **Explore table items**
4. Verify new records with AssessmentType = 'Privilege Analysis'

![Navigate to S3](https://trtrantnt.github.io/workshop/images/5/test3.png?featherlight=false&width=90pc)

## Expected Results

After completion:

- ✅ CloudTrail logs analyzed automatically
- ✅ Lambda function processing privilege events
- ✅ Risk scoring for privilege actions
- ✅ DynamoDB storing analysis results
- ✅ CloudWatch dashboard monitoring
- ✅ Real-time privilege monitoring

![Privilege Analytics Complete](https://trtrantnt.github.io/workshop/images/5/complete.png?featherlight=false&width=90pc)

## Next Steps

Continue to [6. Risk Assessment](../6-danh-gia-rui-ro) to set up comprehensive risk assessment.