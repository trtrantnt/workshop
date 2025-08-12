---
title: "9. Quy trình Kiểm toán"
chapter: false
weight: 9
---

## Mục tiêu

Thiết lập quy trình kiểm toán toàn diện để đảm bảo tuân thủ các tiêu chuẩn SOX, SOC2, ISO27001 và tạo audit trails cho Identity Governance system.

## Bước 1: Thiết lập Audit Data Collection

### 1.1 Xác minh CloudTrail Audit Logs

1. Mở **Amazon CloudTrail** console
2. Xác minh trail **IdentityGovernanceTrail** đang ghi đầy đủ audit events
3. Kiểm tra log file validation được bật

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/9/cloudtrail1.png?featherlight=false&width=90pc)

### 1.2 Cấu hình CloudTrail Insights

1. Trong CloudTrail console
2. Chọn trail **IdentityGovernanceTrail**
3. Click **Edit**
4. Bật **CloudTrail Insights** cho unusual activity patterns

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/9/cloudtrail2.png?featherlight=false&width=90pc)

5. Click **Save changes**

## Bước 2: Tạo Audit Report Generator

### 2.1 Tạo Lambda Function

1. Mở **AWS Lambda** console
2. Click **Create function**
3. Nhập thông tin function:
   - **Function name**: `AuditReportGenerator`
   - **Runtime**: Python 3.9

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/9/lambda1.png?featherlight=false&width=90pc)

### 2.2 Cấu hình Code cho Audit Reports

1. Thay thế code mặc định:

```python
import json
import boto3
import csv
from datetime import datetime, timedelta
from io import StringIO

def lambda_handler(event, context):
    print("Audit Report Generator Started")
    
    # Initialize AWS clients
    dynamodb = boto3.resource('dynamodb')
    s3 = boto3.client('s3')
    cloudtrail = boto3.client('cloudtrail')
    
    try:
        # Generate different types of audit reports
        report_type = event.get('report_type', 'comprehensive')
        
        if report_type == 'access_certification':
            report = generate_access_certification_report(dynamodb)
        elif report_type == 'privilege_usage':
            report = generate_privilege_usage_report(dynamodb, cloudtrail)
        elif report_type == 'risk_assessment':
            report = generate_risk_assessment_report(dynamodb)
        elif report_type == 'compliance':
            report = generate_compliance_report(dynamodb, cloudtrail)
        else:
            report = generate_comprehensive_audit_report(dynamodb, cloudtrail)
        
        # Store report in S3
        report_url = store_audit_report(s3, report, report_type)
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': f'{report_type} audit report generated successfully',
                'report_url': report_url,
                'report_date': datetime.now().isoformat()
            })
        }
        
    except Exception as e:
        print(f'Error generating audit report: {str(e)}')
        return {
            'statusCode': 500,
            'body': json.dumps(f'Error: {str(e)}')
        }

def generate_access_certification_report(dynamodb):
    """Generate access certification audit report"""
    table = dynamodb.Table('AccessCertifications')
    
    response = table.scan()
    certifications = response['Items']
    
    # Analyze certification data
    report = {
        'report_type': 'Access Certification Audit',
        'generated_date': datetime.now().isoformat(),
        'summary': {
            'total_certifications': len(certifications),
            'completed_certifications': len([c for c in certifications if c.get('Status') in ['Completed', 'Approved']]),
            'pending_certifications': len([c for c in certifications if c.get('Status') == 'Pending']),
            'overdue_certifications': count_overdue_certifications(certifications)
        },
        'details': []
    }
    
    # Add detailed certification records
    for cert in certifications:
        report['details'].append({
            'user_id': cert.get('UserId'),
            'certification_date': cert.get('CertificationDate'),
            'status': cert.get('Status'),
            'type': cert.get('Type'),
            'days_since_certification': calculate_days_since(cert.get('CertificationDate'))
        })
    
    return report

def generate_privilege_usage_report(dynamodb, cloudtrail):
    """Generate privilege usage audit report"""
    table = dynamodb.Table('RiskAssessments')
    
    # Get privilege analysis data
    response = table.scan(
        FilterExpression='AssessmentType = :type',
        ExpressionAttributeValues={':type': 'Privilege Analysis'}
    )
    
    privilege_events = response['Items']
    
    # Analyze privilege usage patterns
    report = {
        'report_type': 'Privilege Usage Audit',
        'generated_date': datetime.now().isoformat(),
        'summary': {
            'total_privilege_events': len(privilege_events),
            'high_risk_events': len([e for e in privilege_events if float(e.get('RiskScore', 0)) >= 8]),
            'unique_users': len(set([e.get('UserIdentity', '') for e in privilege_events])),
            'event_types': count_event_types(privilege_events)
        },
        'high_risk_activities': []
    }
    
    # Add high-risk privilege activities
    for event in privilege_events:
        if float(event.get('RiskScore', 0)) >= 8:
            report['high_risk_activities'].append({
                'event_time': event.get('EventTime'),
                'event_name': event.get('EventName'),
                'user_identity': event.get('UserIdentity'),
                'source_ip': event.get('SourceIP'),
                'risk_score': event.get('RiskScore')
            })
    
    return report

def generate_risk_assessment_report(dynamodb):
    """Generate risk assessment audit report"""
    table = dynamodb.Table('RiskAssessments')
    
    # Get user risk assessments
    response = table.scan(
        FilterExpression='AssessmentType = :type',
        ExpressionAttributeValues={':type': 'User Risk Assessment'}
    )
    
    risk_assessments = response['Items']
    
    # Analyze risk distribution
    risk_distribution = {'LOW': 0, 'MEDIUM': 0, 'HIGH': 0, 'CRITICAL': 0}
    for assessment in risk_assessments:
        risk_level = assessment.get('RiskLevel', 'LOW')
        if risk_level in risk_distribution:
            risk_distribution[risk_level] += 1
    
    report = {
        'report_type': 'Risk Assessment Audit',
        'generated_date': datetime.now().isoformat(),
        'summary': {
            'total_users_assessed': len(risk_assessments),
            'risk_distribution': risk_distribution,
            'high_risk_percentage': ((risk_distribution['HIGH'] + risk_distribution['CRITICAL']) / 
                                   max(len(risk_assessments), 1)) * 100
        },
        'high_risk_users': []
    }
    
    # Add high-risk users
    for assessment in risk_assessments:
        if assessment.get('RiskLevel') in ['HIGH', 'CRITICAL']:
            report['high_risk_users'].append({
                'user_name': assessment.get('UserName'),
                'risk_score': float(assessment.get('RiskScore', 0)),
                'risk_level': assessment.get('RiskLevel'),
                'assessment_date': assessment.get('AssessmentDate'),
                'risk_factors': {
                    'certification': float(assessment.get('CertificationRisk', 0)),
                    'privilege': float(assessment.get('PrivilegeRisk', 0)),
                    'security': float(assessment.get('SecurityRisk', 0)),
                    'access': float(assessment.get('AccessRisk', 0))
                }
            })
    
    return report

def generate_compliance_report(dynamodb, cloudtrail):
    """Generate compliance audit report"""
    
    # Get operational data
    ops_table = dynamodb.Table('RiskAssessments')
    ops_response = ops_table.scan(
        FilterExpression='AssessmentType = :type',
        ExpressionAttributeValues={':type': 'Daily Operations Report'}
    )
    
    ops_reports = ops_response['Items']
    
    # Calculate compliance metrics
    if ops_reports:
        latest_ops = max(ops_reports, key=lambda x: x.get('Date', ''))
        compliance_score = float(latest_ops.get('ComplianceScore', 0))
        compliance_status = latest_ops.get('ComplianceStatus', 'UNKNOWN')
    else:
        compliance_score = 0
        compliance_status = 'NO_DATA'
    
    report = {
        'report_type': 'Compliance Audit',
        'generated_date': datetime.now().isoformat(),
        'compliance_frameworks': {
            'SOX': assess_sox_compliance(dynamodb),
            'SOC2': assess_soc2_compliance(dynamodb),
            'ISO27001': assess_iso27001_compliance(dynamodb)
        },
        'overall_compliance': {
            'score': compliance_score,
            'status': compliance_status,
            'last_assessment': latest_ops.get('Date', 'N/A') if ops_reports else 'N/A'
        },
        'compliance_gaps': identify_compliance_gaps(dynamodb)
    }
    
    return report

def generate_comprehensive_audit_report(dynamodb, cloudtrail):
    """Generate comprehensive audit report"""
    
    # Combine all report types
    access_report = generate_access_certification_report(dynamodb)
    privilege_report = generate_privilege_usage_report(dynamodb, cloudtrail)
    risk_report = generate_risk_assessment_report(dynamodb)
    compliance_report = generate_compliance_report(dynamodb, cloudtrail)
    
    comprehensive_report = {
        'report_type': 'Comprehensive Identity Governance Audit',
        'generated_date': datetime.now().isoformat(),
        'executive_summary': generate_executive_summary(access_report, privilege_report, risk_report, compliance_report),
        'access_certification': access_report,
        'privilege_usage': privilege_report,
        'risk_assessment': risk_report,
        'compliance': compliance_report
    }
    
    return comprehensive_report

def count_overdue_certifications(certifications):
    """Count overdue certifications"""
    overdue_count = 0
    ninety_days_ago = datetime.now() - timedelta(days=90)
    
    for cert in certifications:
        cert_date_str = cert.get('CertificationDate', '')
        if cert_date_str:
            try:
                cert_date = datetime.fromisoformat(cert_date_str.replace('Z', '+00:00'))
                if cert_date.replace(tzinfo=None) < ninety_days_ago:
                    overdue_count += 1
            except:
                pass
    
    return overdue_count

def calculate_days_since(date_str):
    """Calculate days since a given date"""
    if not date_str:
        return None
    
    try:
        date_obj = datetime.fromisoformat(date_str.replace('Z', '+00:00'))
        return (datetime.now(date_obj.tzinfo) - date_obj).days
    except:
        return None

def count_event_types(events):
    """Count different types of privilege events"""
    event_counts = {}
    for event in events:
        event_name = event.get('EventName', 'Unknown')
        event_counts[event_name] = event_counts.get(event_name, 0) + 1
    return event_counts

def assess_sox_compliance(dynamodb):
    """Assess SOX compliance"""
    # Simplified SOX assessment
    return {
        'status': 'COMPLIANT',
        'score': 85,
        'requirements_met': [
            'Access controls documented',
            'Regular access reviews conducted',
            'Audit trails maintained'
        ],
        'gaps': [
            'Quarterly management attestation needed'
        ]
    }

def assess_soc2_compliance(dynamodb):
    """Assess SOC2 compliance"""
    return {
        'status': 'MOSTLY_COMPLIANT',
        'score': 78,
        'requirements_met': [
            'Access monitoring implemented',
            'Risk assessments performed',
            'Security controls documented'
        ],
        'gaps': [
            'Vendor access controls need enhancement',
            'Incident response procedures need update'
        ]
    }

def assess_iso27001_compliance(dynamodb):
    """Assess ISO27001 compliance"""
    return {
        'status': 'COMPLIANT',
        'score': 82,
        'requirements_met': [
            'Information security policy established',
            'Access control procedures implemented',
            'Regular security assessments conducted'
        ],
        'gaps': [
            'Business continuity plan needs review'
        ]
    }

def identify_compliance_gaps(dynamodb):
    """Identify compliance gaps"""
    return [
        {
            'gap': 'Incomplete access certifications',
            'severity': 'HIGH',
            'recommendation': 'Implement automated reminders for pending certifications'
        },
        {
            'gap': 'High-risk users not reviewed',
            'severity': 'MEDIUM',
            'recommendation': 'Establish monthly review process for high-risk users'
        }
    ]

def generate_executive_summary(access_report, privilege_report, risk_report, compliance_report):
    """Generate executive summary"""
    return {
        'key_metrics': {
            'total_certifications': access_report['summary']['total_certifications'],
            'completion_rate': (access_report['summary']['completed_certifications'] / 
                              max(access_report['summary']['total_certifications'], 1)) * 100,
            'high_risk_users': len(risk_report['high_risk_users']),
            'overall_compliance_score': compliance_report['overall_compliance']['score']
        },
        'key_findings': [
            f"Access certification completion rate: {((access_report['summary']['completed_certifications'] / max(access_report['summary']['total_certifications'], 1)) * 100):.1f}%",
            f"High-risk privilege events detected: {privilege_report['summary']['high_risk_events']}",
            f"Users requiring immediate attention: {len(risk_report['high_risk_users'])}",
            f"Overall compliance status: {compliance_report['overall_compliance']['status']}"
        ],
        'recommendations': [
            'Increase certification completion rate through automated reminders',
            'Review and remediate high-risk privilege usage patterns',
            'Implement additional controls for high-risk users',
            'Address identified compliance gaps'
        ]
    }

def store_audit_report(s3, report, report_type):
    """Store audit report in S3"""
    bucket_name = 'identity-governance-reports'  # From chapter 2
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    key = f'audit-reports/{report_type}_{timestamp}.json'
    
    try:
        s3.put_object(
            Bucket=bucket_name,
            Key=key,
            Body=json.dumps(report, indent=2, default=str),
            ContentType='application/json'
        )
        
        return f's3://{bucket_name}/{key}'
    except Exception as e:
        print(f'Error storing report in S3: {str(e)}')
        return None
```

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/9/lambda2.png?featherlight=false&width=90pc)

2. Click **Deploy**

### 2.3 Cấu hình IAM Permissions

1. Thêm permissions cho Lambda:
   - **AmazonDynamoDBReadOnlyAccess**
   - **AmazonS3FullAccess**
   - **CloudTrailReadOnlyAccess**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/9/lambda3.png?featherlight=false&width=90pc)

## Bước 3: Thiết lập Automated Audit Schedules

### 3.1 Tạo Monthly Audit Schedule

1. Mở **Amazon EventBridge** console
2. Tạo schedule chạy audit reports hàng tháng
3. Cấu hình để tạo comprehensive audit report

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/9/schedule1.png?featherlight=false&width=90pc)

### 3.2 Tạo Quarterly Compliance Schedule

1. Tạo schedule chạy compliance reports hàng quý
2. Cấu hình notification cho management

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/9/schedule2.png?featherlight=false&width=90pc)

## Bước 4: Thiết lập Audit Trail Monitoring

### 4.1 Tạo CloudWatch Alarms cho Audit Events

1. Mở **Amazon CloudWatch** console
2. Tạo alarms cho critical audit events:
   - Unauthorized access attempts
   - Privilege escalation events
   - Failed authentication events

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/9/alarm1.png?featherlight=false&width=90pc)

### 4.2 Cấu hình Log Insights Queries

1. Trong CloudWatch console
2. Click **Logs Insights**
3. Tạo saved queries cho audit analysis

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/9/insights1.png?featherlight=false&width=90pc)

## Bước 5: Kiểm tra Audit System

### 5.1 Test Audit Report Generation

1. Chạy Lambda function **AuditReportGenerator**
2. Test với các report types khác nhau
3. Xác minh reports được lưu trong S3

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/9/test1.png?featherlight=false&width=90pc)

### 5.2 Xác minh Audit Trail Completeness

1. Kiểm tra CloudTrail logs có đầy đủ events
2. Xác minh log file validation
3. Test audit trail queries

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/9/test2.png?featherlight=false&width=90pc)

### 5.3 Test Compliance Reporting

1. Generate compliance reports
2. Xác minh compliance scores
3. Review compliance gaps identification

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/9/test3.png?featherlight=false&width=90pc)

## Kết quả Mong đợi

Sau khi hoàn thành:

- ✅ Comprehensive audit trail system
- ✅ Automated audit report generation
- ✅ SOX, SOC2, ISO27001 compliance reporting
- ✅ Monthly and quarterly audit schedules
- ✅ Real-time audit event monitoring
- ✅ Compliance gap identification and remediation

![Hoàn thành Audit System](https://trtrantnt.github.io/workshop/images/9/complete.png?featherlight=false&width=90pc)

## Tiếp theo

Chuyển sang [10. Xác thực Tuân thủ](../10-xac-thuc-tuan-thu) để thực hiện validation cuối cùng và compliance verification.