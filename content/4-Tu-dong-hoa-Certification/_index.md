---
title: "4. Certification Automation"
chapter: false
weight: 4
---

## Objective

Automate access certification processes to ensure access rights are reviewed periodically and comply with security requirements.

## Step 1: Verify DynamoDB Table

### 1.1 Check Existing Table

1. Open **Amazon DynamoDB** in the console
2. Verify that the `AccessCertifications` table was created in chapter 2
3. This table will be used to store certification data

## Step 2: Create Lambda Function

### 2.1 Create Lambda Function

1. Open **AWS Lambda** in the console
2. Click **Create function**
3. Choose **Author from scratch**
4. Enter function details:
   - **Function name**: `AccessCertificationTrigger`
   - **Runtime**: Python 3.9
   - **Architecture**: x86_64

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/ld1.png?featherlight=false&width=90pc)

5. Click **Create function**

### 2.2 Configure Lambda Function Code

1. In the **Code** tab, replace the default code with the following:

```python
import json
import boto3
from datetime import datetime

def lambda_handler(event, context):
    print("Access Certification Trigger Started")
    
    # Initialize AWS clients
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('AccessCertifications')
    
    # Create certification record
    response = table.put_item(
        Item={
            'UserId': 'system',
            'CertificationDate': datetime.now().isoformat(),
            'Status': 'Triggered',
            'Type': 'Quarterly Review'
        }
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps('Certification process triggered successfully')
    }
```

2. Click **Deploy** to save changes

### 2.3 Configure IAM Role for Lambda

1. Go to **Configuration** tab

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/ld2.png?featherlight=false&width=90pc)

2. Click **Permissions**
3. Click on the role name to open IAM console
4. Click **Add permissions** → **Attach policies**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/ld3.png?featherlight=false&width=90pc)

5. Search and attach policy **AmazonDynamoDBFullAccess**
6. Select **Add permissions**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/ld4.png?featherlight=false&width=90pc)

## Step 3: Setup EventBridge Scheduler

### 3.1 Create Scheduled Rule

1. Open **Amazon EventBridge** in AWS Console
2. Click **Rules** in the sidebar
3. Click **Create rule**

#### Step 1: Define rule detail
4. Enter rule information:
   - **Name**: `AccessCertificationSchedule`
   - **Description**: `Quarterly access certification review`
   - **Event bus**: default
   - **Enable the rule on the selected event bus**

5. In **Rule type**, select **Schedule**
6. Click **Next**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/eb1.png?featherlight=false&width=90pc)

#### Step 2: Define schedule
7. In **Occurrence**, select **Recurring schedule**
8. In **Schedule pattern**, select **Rate-based schedule**
9. Enter **90** and select **Days**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/eb2.png?featherlight=false&width=90pc)

10. In **Flexible time window**, enter **15** minutes
11. Click **Next**

#### Step 3: Select target
12. In **Target API**, select **AWS Lambda Invoke**
13. In **Lambda function**, select **AccessCertificationTrigger**
14. Click **Next**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/eb3.png?featherlight=false&width=90pc)

#### Step 4: Configure tags (Optional)
15. Skip the tags section, click **Next**

#### Step 5: Review and create
16. Review configuration and click **Create rule**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/eb4.png?featherlight=false&width=90pc)

## Step 4: Test the Automation

### 4.1 Verify EventBridge Schedule

1. In **Amazon EventBridge** console
2. Click **Schedules** in the sidebar (not Rules)
3. Verify that schedule **AccessCertificationSchedule** is created and **Enabled**

### 4.2 Test Lambda Function Manually

1. Go to **AWS Lambda** console
2. Select function **AccessCertificationTrigger**
3. Click **Test** to create a test event
4. Use default test event and click **Test**
5. Check execution results

### 4.3 Verify DynamoDB Record

1. Go to **Amazon DynamoDB** console
2. Select table **AccessCertifications**
3. Click **Explore table items**
4. Verify that a new record was created by the Lambda function

## Expected Results

After completion:

- ✅ DynamoDB table for certification data storage
- ✅ Lambda function processing certification logic
- ✅ EventBridge scheduled triggers quarterly
- ✅ Automated quarterly access reviews
- ✅ Audit trail and monitoring

## Next Steps

Continue to [5. Privilege Analytics](../5-phan-tich-dac-quyen) to set up privilege analysis.