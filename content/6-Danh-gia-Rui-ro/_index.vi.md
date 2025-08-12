---
title: "6. Đánh giá Rủi ro"
chapter: false
weight: 6
---

## Mục tiêu

Thiết lập hệ thống đánh giá rủi ro toàn diện để phân tích và scoring rủi ro bảo mật dựa trên dữ liệu từ các chương trước.

## Bước 1: Xác minh Dữ liệu từ Các Chương Trước

### 1.1 Kiểm tra DynamoDB Tables

1. Mở **Amazon DynamoDB** trong console
2. Xác minh các tables đã có dữ liệu:
   - **AccessCertifications** (từ chương 4)
   - **RiskAssessments** (từ chương 5)

### 1.2 Kiểm tra Security Hub Findings

1. Vào **AWS Security Hub** console
2. Xác minh có findings được thu thập
3. Kiểm tra compliance standards đang active

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/6/61.png?featherlight=false&width=90pc)

## Bước 2: Tạo Lambda Function cho Risk Assessment

### 2.1 Tạo Lambda Function

1. Mở **AWS Lambda** trong console
2. Click **Create function**
3. Chọn **Author from scratch**
4. Nhập thông tin function:
   - **Function name**: `RiskAssessmentEngine`
   - **Runtime**: Python 3.9
   - **Architecture**: x86_64


5. Click **Create function**

### 2.2 Cấu hình Code cho Lambda Function

1. Trong tab **Code**, thay thế code mặc định bằng code sau:

```python
import json
import boto3
from datetime import datetime, timedelta
from decimal import Decimal

def lambda_handler(event, context):
    print("Risk Assessment Engine Started")
    
    # Initialize AWS clients
    dynamodb = boto3.resource('dynamodb')
    securityhub = boto3.client('securityhub')
    iam = boto3.client('iam')
    
    try:
        # Collect data from various sources
        certification_data = get_certification_data(dynamodb)
        privilege_data = get_privilege_analysis_data(dynamodb)
        security_findings = get_security_hub_findings(securityhub)
        iam_data = get_iam_data(iam)
        
        # Perform comprehensive risk assessment
        risk_assessment = perform_risk_assessment(
            certification_data, privilege_data, security_findings, iam_data
        )
        
        # Store assessment results
        store_risk_assessment(risk_assessment, dynamodb)
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': 'Risk assessment completed',
                'totalUsers': len(risk_assessment['user_risks']),
                'highRiskUsers': len([u for u in risk_assessment['user_risks'] if u['riskLevel'] == 'HIGH']),
                'overallRiskScore': risk_assessment['overall_risk_score']
            })
        }
        
    except Exception as e:
        print(f'Error in risk assessment: {str(e)}')
        return {
            'statusCode': 500,
            'body': json.dumps(f'Error: {str(e)}')
        }

def get_certification_data(dynamodb):
    """Get certification data from DynamoDB"""
    table = dynamodb.Table('AccessCertifications')
    response = table.scan()
    return response['Items']

def get_privilege_analysis_data(dynamodb):
    """Get privilege analysis data from DynamoDB"""
    table = dynamodb.Table('RiskAssessments')
    response = table.scan(
        FilterExpression='AssessmentType = :type',
        ExpressionAttributeValues={':type': 'Privilege Analysis'}
    )
    return response['Items']

def get_security_hub_findings(securityhub):
    """Get Security Hub findings"""
    try:
        response = securityhub.get_findings(
            Filters={
                'RecordState': [{'Value': 'ACTIVE', 'Comparison': 'EQUALS'}],
                'WorkflowStatus': [{'Value': 'NEW', 'Comparison': 'EQUALS'}]
            },
            MaxResults=100
        )
        return response['Findings']
    except Exception as e:
        print(f'Error getting Security Hub findings: {str(e)}')
        return []

def get_iam_data(iam):
    """Get IAM users and roles data"""
    users = []
    roles = []
    
    try:
        # Get users
        paginator = iam.get_paginator('list_users')
        for page in paginator.paginate():
            users.extend(page['Users'])
        
        # Get roles
        paginator = iam.get_paginator('list_roles')
        for page in paginator.paginate():
            roles.extend(page['Roles'])
            
    except Exception as e:
        print(f'Error getting IAM data: {str(e)}')
    
    return {'users': users, 'roles': roles}

def perform_risk_assessment(cert_data, priv_data, sec_findings, iam_data):
    """Perform comprehensive risk assessment"""
    
    user_risks = []
    
    # Assess each user
    for user in iam_data['users']:
        user_name = user['UserName']
        
        # Calculate risk factors
        cert_risk = calculate_certification_risk(user_name, cert_data)
        priv_risk = calculate_privilege_risk(user_name, priv_data)
        security_risk = calculate_security_findings_risk(user_name, sec_findings)
        access_risk = calculate_access_pattern_risk(user)
        
        # Calculate overall user risk score
        overall_score = (cert_risk * 0.25 + priv_risk * 0.35 + 
                        security_risk * 0.25 + access_risk * 0.15)
        
        risk_level = get_risk_level(overall_score)
        
        user_risk = {
            'userName': user_name,
            'riskScore': round(overall_score, 2),
            'riskLevel': risk_level,
            'factors': {
                'certification': cert_risk,
                'privilege': priv_risk,
                'security': security_risk,
                'access': access_risk
            },
            'assessmentDate': datetime.now().isoformat()
        }
        
        user_risks.append(user_risk)
    
    # Calculate overall organizational risk
    if user_risks:
        overall_risk_score = sum(u['riskScore'] for u in user_risks) / len(user_risks)
    else:
        overall_risk_score = 0
    
    return {
        'user_risks': user_risks,
        'overall_risk_score': round(overall_risk_score, 2),
        'assessment_date': datetime.now().isoformat(),
        'total_findings': len(sec_findings)
    }

def calculate_certification_risk(user_name, cert_data):
    """Calculate certification-related risk"""
    base_risk = 5.0
    
    # Check if user has recent certifications
    user_certs = [c for c in cert_data if c.get('UserId') == user_name]
    
    if not user_certs:
        return 8.0  # High risk if no certifications
    
    # Check certification recency
    recent_cert = max(user_certs, key=lambda x: x.get('CertificationDate', ''))
    cert_date = datetime.fromisoformat(recent_cert['CertificationDate'].replace('Z', '+00:00'))
    days_since_cert = (datetime.now(cert_date.tzinfo) - cert_date).days
    
    if days_since_cert > 90:
        base_risk += 2
    elif days_since_cert > 30:
        base_risk += 1
    
    return min(base_risk, 10.0)

def calculate_privilege_risk(user_name, priv_data):
    """Calculate privilege-related risk"""
    user_priv_events = [p for p in priv_data if user_name in str(p.get('UserIdentity', ''))]
    
    if not user_priv_events:
        return 3.0  # Low risk if no privilege events
    
    # Calculate average risk score from privilege events
    avg_risk = sum(float(p.get('RiskScore', 5)) for p in user_priv_events) / len(user_priv_events)
    return min(avg_risk, 10.0)

def calculate_security_findings_risk(user_name, findings):
    """Calculate risk based on Security Hub findings"""
    base_risk = 2.0
    
    # Count findings related to IAM/access
    iam_findings = [f for f in findings if 'iam' in f.get('Type', '').lower() or 
                   'access' in f.get('Title', '').lower()]
    
    if len(iam_findings) > 5:
        base_risk += 4
    elif len(iam_findings) > 2:
        base_risk += 2
    elif len(iam_findings) > 0:
        base_risk += 1
    
    return min(base_risk, 10.0)

def calculate_access_pattern_risk(user):
    """Calculate risk based on access patterns"""
    base_risk = 3.0
    
    # Check password age
    password_last_used = user.get('PasswordLastUsed')
    if password_last_used:
        days_since_login = (datetime.now(password_last_used.tzinfo) - password_last_used).days
        if days_since_login > 90:
            base_risk += 2
        elif days_since_login > 30:
            base_risk += 1
    else:
        base_risk += 3  # Never logged in
    
    return min(base_risk, 10.0)

def get_risk_level(score):
    """Convert risk score to risk level"""
    if score >= 8:
        return 'CRITICAL'
    elif score >= 6:
        return 'HIGH'
    elif score >= 4:
        return 'MEDIUM'
    else:
        return 'LOW'

def store_risk_assessment(assessment, dynamodb):
    """Store risk assessment results in DynamoDB"""
    table = dynamodb.Table('RiskAssessments')
    
    # Store overall assessment
    table.put_item(
        Item={
            'AssessmentId': f"risk-assessment-{datetime.now().isoformat()}",
            'AssessmentType': 'Comprehensive Risk Assessment',
            'OverallRiskScore': Decimal(str(assessment['overall_risk_score'])),
            'TotalUsers': len(assessment['user_risks']),
            'HighRiskUsers': len([u for u in assessment['user_risks'] if u['riskLevel'] in ['HIGH', 'CRITICAL']]),
            'AssessmentDate': assessment['assessment_date']
        }
    )
    
    # Store individual user risks
    for user_risk in assessment['user_risks']:
        table.put_item(
            Item={
                'AssessmentId': f"user-risk-{user_risk['userName']}-{datetime.now().isoformat()}",
                'AssessmentType': 'User Risk Assessment',
                'UserName': user_risk['userName'],
                'RiskScore': Decimal(str(user_risk['riskScore'])),
                'RiskLevel': user_risk['riskLevel'],
                'CertificationRisk': Decimal(str(user_risk['factors']['certification'])),
                'PrivilegeRisk': Decimal(str(user_risk['factors']['privilege'])),
                'SecurityRisk': Decimal(str(user_risk['factors']['security'])),
                'AccessRisk': Decimal(str(user_risk['factors']['access'])),
                'AssessmentDate': user_risk['assessmentDate']
            }
        )
```


2. Click **Deploy** để lưu thay đổi

### 2.3 Cấu hình IAM Role cho Lambda

1. Chuyển đến tab **Configuration**
2. Click **Permissions**
3. Click vào role name để mở IAM console
4. Click **Add permissions** → **Attach policies**
5. Tìm và attach các policies sau:
   - **AmazonDynamoDBFullAccess**
   - **SecurityHubReadOnlyAccess**
   - **IAMReadOnlyAccess**
6. Click **Add permissions**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/6/ld3.png?featherlight=false&width=90pc)

## Bước 3: Thiết lập EventBridge Schedule cho Risk Assessment

### 3.1 Tạo Scheduled Rule

1. Mở **Amazon EventBridge** trong AWS Console
2. Click **Schedules** ở sidebar
3. Click **Create schedule**
4. Cấu hình schedule:
   - **Name**: `RiskAssessmentSchedule`
   - **Description**: `Daily risk assessment execution`
   - **Schedule pattern**: Rate-based schedule
   - **Rate**: 1 Day
   - **Target**: Lambda function `RiskAssessmentEngine`

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/6/eb1.png?featherlight=false&width=90pc)

5. Click **Create schedule**

## Bước 4: Tạo SNS Topic cho Risk Alerts

### 4.1 Tạo SNS Topic

1. Mở **Amazon SNS** console
2. Click **Topics** ở sidebar
3. Click **Create topic**
4. Cấu hình topic:
   - **Type**: Standard
   - **Name**: `RiskAssessmentAlerts`
   - **Display name**: `Risk Assessment Alerts`
5. Click **Create topic**

### 4.2 Tạo Subscription

1. Trong topic **RiskAssessmentAlerts**
2. Click **Create subscription**
3. Cấu hình subscription:
   - **Protocol**: Email
   - **Endpoint**: your-email@example.com


4. Click **Create subscription**
5. Xác nhận subscription qua email

## Bước 5: Kiểm tra Risk Assessment

### 5.1 Test Lambda Function

1. Vào **AWS Lambda** console
2. Chọn function **RiskAssessmentEngine**
3. Click **Test** để tạo test event
4. Sử dụng default test event và click **Test**


### 5.2 Xác minh DynamoDB Results

1. Vào **Amazon DynamoDB** console
2. Chọn table **RiskAssessments**
3. Click **Explore table items**
4. Xác minh có records mới với AssessmentType = 'Comprehensive Risk Assessment'


### 5.3 Kiểm tra CloudWatch Logs

1. Vào **Amazon CloudWatch** console
2. Click **Log groups**
3. Tìm log group cho Lambda function
4. Xem logs để xác minh risk assessment đã chạy thành công


## Kết quả Mong đợi

Sau khi hoàn thành:

- ✅ Comprehensive risk assessment engine
- ✅ Multi-factor risk scoring (certification, privilege, security, access)
- ✅ Automated daily risk assessments
- ✅ Risk level categorization (LOW, MEDIUM, HIGH, CRITICAL)
- ✅ SNS alerts for high-risk findings
- ✅ Historical risk tracking in DynamoDB


## Tiếp theo

Chuyển sang [7. Thiết lập Giám sát](../7-thiet-lap-giam-sat) để thiết lập hệ thống giám sát và alerting.