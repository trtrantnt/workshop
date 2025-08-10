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

![Create EventBridge Rule](/images/4/create-eventbridge-rule.png?featherlight=false&width=90pc)

4. Enter rule details:
   - **Name**: AccessCertificationSchedule
   - **Description**: Quarterly access certification review
   - **Event bus**: default

![Rule Details](/images/4/rule-details.png?featherlight=false&width=90pc)

5. In **Define pattern**, select **Schedule**
6. Choose **Fixed rate every** and enter **90 days**

![Schedule Pattern](/images/4/schedule-pattern.png?featherlight=false&width=90pc)

7. Click **Next**

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

## Step 3: Step Functions Workflow

### 3.1 Create State Machine

1. Open **AWS Step Functions** in the console
2. Click **Create state machine**

![Create State Machine](/images/4/create-state-machine.png?featherlight=false&width=90pc)

3. Choose **Write your workflow in code**
4. Select **Standard** type

![State Machine Type](/images/4/state-machine-type.png?featherlight=false&width=90pc)

5. Enter the workflow definition in JSON format
6. Name the state machine: **AccessCertificationWorkflow**

![State Machine Definition](/images/4/state-machine-definition.png?featherlight=false&width=90pc)

### 3.2 Configure IAM Role

1. Create or select an IAM role for Step Functions
2. Ensure it has permissions to invoke Lambda functions

![Step Functions IAM Role](/images/4/stepfunctions-iam-role.png?featherlight=false&width=90pc)

3. Click **Create state machine**

## Step 4: Connect EventBridge to Lambda

### 4.1 Add Lambda Target to EventBridge Rule

1. Go back to **EventBridge** console
2. Select the rule **AccessCertificationSchedule**
3. Click **Targets** tab
4. Click **Add target**

![Add Lambda Target](/images/4/add-lambda-target.png?featherlight=false&width=90pc)

5. Configure target:
   - **Target type**: AWS service
   - **Service**: Lambda function
   - **Function**: AccessCertificationTrigger

![Configure Lambda Target](/images/4/configure-lambda-target.png?featherlight=false&width=90pc)

6. Click **Add** and then **Update rule**

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