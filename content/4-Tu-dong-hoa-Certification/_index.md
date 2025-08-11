---
title: "4. Certification Automation"
chapter: false
weight: 4
---

## Objective

Automate access certification processes to ensure access rights are reviewed periodically and comply with security requirements.

## Automation Architecture

```mermaid
graph TB
    A[EventBridge Schedule] --> B[Lambda Trigger]
    B --> C[Access Review Generator]
    C --> D[Certification Workflow]
    D --> E[Manager Approval]
    E --> F[Remediation Actions]
    F --> G[Compliance Report]
```

## Step 1: EventBridge Scheduler Setup

### 1.1 Create Scheduled Rule

1. Open **Amazon EventBridge** in AWS Console
2. Click **Rules** in the sidebar
3. Click **Create rule**

#### Step 1: Define rule detail
4. Enter rule information:
   - **Name**: `AccessCertificationSchedule`
   - **Description**: `Quarterly access certification review`
   - **Event bus**: default
   - **Enable the rule on the selected event bus**: ✅ Checked

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/eb1.png?featherlight=false&width=90pc)

5. In **Rule type**, select **Schedule**
   - Choose "A rule that runs on a schedule"
6. Click **Next**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/eb2.png?featherlight=false&width=90pc)

#### Step 2: Define schedule
7. In **Occurrence**, select **Recurring schedule**
   - Choose "Recurring schedule" because we want to run quarterly reviews
8. In **Schedule pattern**, select **Rate-based schedule**
9. Enter **90** and select **Days**
10. In **Flexible time window**, enter **15** minutes
    - Allows the schedule to run within 15 minutes after the start time
    - Helps reduce system load and increases flexibility
11. Click **Next**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/eb3.png?featherlight=false&width=90pc)

## Step 2: Lambda Function Setup

### 2.1 Create Lambda Function

1. Open **AWS Lambda** in the console
2. Click **Create function**

![Create Lambda Function](/images/4/create-lambda-function.png?featherlight=false&width=90pc)

3. Choose **Author from scratch**
4. Enter function details:
   - **Function name**: AccessCertificationTrigger
   - **Runtime**: Python 3.9
   - **Architecture**: x86_64

![Lambda Function Details](/images/4/lambda-function-details.png?featherlight=false&width=90pc)

5. Click **Create function**

### 2.2 Configure Lambda Function Code

1. In the **Code** tab, replace the default code
2. Upload the certification logic code

![Lambda Code Editor](/images/4/lambda-code-editor.png?featherlight=false&width=90pc)

3. Click **Deploy** to save changes

### 2.3 Set Environment Variables

1. Go to **Configuration** tab
2. Click **Environment variables**
3. Click **Edit**

![Environment Variables](/images/4/environment-variables.png?featherlight=false&width=90pc)

4. Add required variables:
   - **S3_BUCKET**: certification-data-bucket
   - **SNS_TOPIC**: certification-notifications

![Add Environment Variables](/images/4/add-env-variables.png?featherlight=false&width=90pc)

## Step 3: DynamoDB Setup for Certification Data

### 3.1 Create DynamoDB Table

1. Open **Amazon DynamoDB** in the console
2. Click **Create table**

![Create DynamoDB Table](/images/4/create-dynamodb-table.png?featherlight=false&width=90pc)

3. Enter table details:
   - **Table name**: AccessCertifications
   - **Partition key**: UserId (String)
   - **Sort key**: CertificationDate (String)
   - **Billing mode**: On-demand
4. Click **Create table**

![DynamoDB Table Details](/images/4/dynamodb-table-details.png?featherlight=false&width=90pc)

### 3.2 Configure Lambda for DynamoDB Integration

1. Return to Lambda function **AccessCertificationTrigger**
2. Add DynamoDB permissions to IAM role
3. Update code to write certification records

![Lambda DynamoDB Integration](/images/4/lambda-dynamodb-integration.png?featherlight=false&width=90pc)

## Step 4: Connect EventBridge to Lambda

### 4.1 Add Lambda Target to EventBridge Rule

#### Step 3: Select target
1. In **Target API**, select **AWS Lambda**
2. Choose API **Invoke**
3. In **Lambda function**, select **AccessCertificationTrigger**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/eb4.png?featherlight=false&width=90pc)

4. Click **Next**

#### Step 4: Configure tags (Optional)
5. Skip the tags section, click **Next**

#### Step 5: Review and create
6. Review configuration:
   - Rule name: AccessCertificationSchedule
   - Schedule: Rate(90 days)
   - Target: Lambda function
7. Click **Create rule**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/4/eb5.png?featherlight=false&width=90pc)

## Step 5: Test the Automation

### 5.1 Manual Test Execution

1. In EventBridge, select your rule
2. Click **Actions** → **Test rule**

![Test EventBridge Rule](/images/4/test-eventbridge-rule.png?featherlight=false&width=90pc)

3. Monitor Lambda function execution in CloudWatch Logs

![CloudWatch Logs](/images/4/cloudwatch-logs.png?featherlight=false&width=90pc)

## Expected Results

After completion:

- ✅ Automated quarterly access reviews
- ✅ EventBridge scheduled triggers
- ✅ Lambda function processing
- ✅ Step Functions workflow orchestration
- ✅ Audit trail and monitoring

![Certification Automation Complete](/images/4/automation-complete.png?featherlight=false&width=90pc)

## Next Steps

Continue to [5. Privilege Analytics](../5-phan-tich-dac-quyen) to set up privilege analysis.