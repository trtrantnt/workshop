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

![Navigate to S3](https://trtrantnt.github.io/workshop/images/5/5.1.png?featherlight=false&width=90pc)

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

### 5.1 Create Test User to Generate CloudTrail Events

1. Go to **IAM** console
2. Click **Users** in the left sidebar
3. Click **Create user**
4. Enter User name: `test-privilege-user`
5. Click **Next**
6. Select **Attach policies directly**
7. Search and select **ReadOnlyAccess**
8. Click **Next** → **Create user**

### 5.2 Perform High-Privilege Actions

1. In IAM console, select the **test-privilege-user** you just created
2. Click **Permissions** tab
3. Click **Add permissions** → **Attach policies directly**
4. Search and attach **PowerUserAccess** policy
5. Click **Add permissions**
6. Then **Remove** the **PowerUserAccess** policy to create more events
7. Create test role:
   - Click **Roles** in sidebar
   - Click **Create role**
   - Select **AWS service** → **Lambda**
   - Click **Next** → **Next**
   - Role name: `test-privilege-role`
   - Click **Create role**

### 5.3 Wait for CloudTrail Processing

1. CloudTrail needs time to write logs to S3
2. Check CloudTrail S3 bucket:
   - Go to **S3** console
   - Find CloudTrail bucket (name like `aws-cloudtrail-logs-xxx`)
   - Verify new log files are being created

### 5.4 Verify Lambda Function Execution

1. Go to **AWS Lambda** console
2. Select function **PrivilegeAnalyticsEngine**
3. Click **Monitor** tab
4. Check **Invocations** graph - should show activity
5. Click **View CloudWatch logs**
6. Select the latest log stream
7. Verify logs like:
   ```
   Privilege Analytics Engine Started
   Processed X privilege events
   ```

### 5.5 Verify DynamoDB Records

1. Go to **Amazon DynamoDB** console
2. Click **Tables** in sidebar
3. Select table **RiskAssessments**
4. Click **Explore table items**
5. Verify new records with:
   - **AssessmentType**: 'Privilege Analysis'
   - **EventName**: 'AttachUserPolicy', 'DetachUserPolicy', 'CreateRole'
   - **RiskScore**: Value from 1-10
   - **EventTime**: Recent timestamp

### 5.6 Check CloudWatch Dashboard

1. Go to **CloudWatch** console
2. Click **Dashboards**
3. Select **PrivilegeAnalyticsDashboard**
4. Verify widgets display data:
   - **Lambda Invocations**: Should show spike when function runs
   - **Lambda Errors**: Should be 0
   - **Lambda Duration**: Execution time
   - **DynamoDB Write Activity**: Should show activity when writing data

### 5.7 Test Real-time Monitoring

1. Perform additional privilege action:
   - Create new user: `test-user-2`
   - Attach policy **IAMReadOnlyAccess**
2. Wait 5-10 minutes
3. Refresh DynamoDB table to see new record
4. Check if dashboard updates metrics

### 5.8 Troubleshooting (if no data)

**If Lambda doesn't run:**
1. Check S3 trigger is configured correctly
2. Verify CloudTrail is creating log files in S3
3. Check IAM permissions of Lambda role

**If no data in DynamoDB:**
1. Check CloudWatch logs of Lambda function
2. Verify table name in code: 'RiskAssessments'
3. Confirm Lambda has write permissions to DynamoDB

**If Dashboard doesn't show data:**
1. Wait 5-15 minutes for metrics to appear
2. Check metric names and dimensions are correct
3. Refresh dashboard page

## Expected Results

After completion:

- ✅ CloudTrail logs are automatically analyzed
- ✅ Lambda function processes privilege events
- ✅ Risk scoring for privilege actions
- ✅ DynamoDB stores analysis results
- ✅ CloudWatch dashboard monitoring
- ✅ Real-time privilege monitoring

## Next Steps

Proceed to [6. Risk Assessment](../6-danh-gia-rui-ro) to set up comprehensive risk assessment.