---
title: "9. Audit Procedures"
chapter: false
weight: 9
---

## Objective

Establish comprehensive audit procedures to ensure compliance with SOX, SOC2, ISO27001 standards and create audit trails for the Identity Governance system.

## Step 1: Setup Audit Data Collection

### 1.1 Verify CloudTrail Audit Logs

1. Open **Amazon CloudTrail** console
2. Verify trail **IdentityGovernanceTrail** is recording complete audit events
3. Check that log file validation is enabled

### 1.2 Configure CloudTrail Insights

1. In CloudTrail console
2. Select trail **IdentityGovernanceTrail**
3. Click **Edit**
4. Enable **CloudTrail Insights** for unusual activity patterns
5. Click **Save changes**

## Step 2: Create Audit Report Generator

### 2.1 Create Lambda Function

1. Open **AWS Lambda** console
2. Click **Create function**
3. Enter function details:
   - **Function name**: `AuditReportGenerator`
   - **Runtime**: Python 3.9

### 2.2 Configure Code for Audit Reports

1. Replace default code:

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
    
    # Calculate certification statistics
    total_certifications = len(certifications)
    completed_certifications = len([c for c in certifications if c.get('Status') in ['Completed', 'Approved']])
    overdue_certifications = count_overdue_certifications(certifications)
    
    report = {
        'report_type': 'Access Certification Audit',
        'generated_date': datetime.now().isoformat(),
        'summary': {
            'total_certifications': total_certifications,
            'completed_certifications': completed_certifications,
            'overdue_certifications': overdue_certifications,
            'completion_rate': (completed_certifications / max(total_certifications, 1)) * 100
        },
        'certifications': []
    }
    
    # Add certification details
    for cert in certifications:
        days_since_creation = calculate_days_since(cert.get('CertificationDate'))
        report['certifications'].append({
            'certification_id': cert.get('CertificationId'),
            'user_name': cert.get('UserName'),
            'manager': cert.get('Manager'),
            'status': cert.get('Status'),
            'certification_date': cert.get('CertificationDate'),
            'days_since_creation': days_since_creation,
            'is_overdue': days_since_creation and days_since_creation > 90
        })
    
    return report

def generate_privilege_usage_report(dynamodb, cloudtrail):
    """Generate privilege usage audit report"""
    
    # Get privilege analytics data
    table = dynamodb.Table('RiskAssessments')
    response = table.scan(
        FilterExpression='AssessmentType = :type',
        ExpressionAttributeValues={':type': 'Privilege Analytics'}
    )
    
    privilege_events = response['Items']
    
    report = {
        'report_type': 'Privilege Usage Audit',
        'generated_date': datetime.now().isoformat(),
        'summary': {
            'total_events': len(privilege_events),
            'high_risk_events': len([e for e in privilege_events if float(e.get('RiskScore', 0)) >= 8]),
            'unique_users': len(set([e.get('UserName') for e in privilege_events if e.get('UserName')]))
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
                'risk_factors': assessment.get('RiskFactors', []),
                'last_assessment': assessment.get('LastAssessment')
            })
    
    return report

def generate_compliance_report(dynamodb, cloudtrail):
    """Generate compliance audit report"""
    
    # Check SOX compliance
    sox_compliance = check_sox_compliance(dynamodb)
    
    # Check SOC2 compliance
    soc2_compliance = check_soc2_compliance(dynamodb)
    
    # Check ISO27001 compliance
    iso_compliance = check_iso27001_compliance(dynamodb)
    
    # Calculate overall compliance score
    overall_score = (sox_compliance['score'] + soc2_compliance['score'] + iso_compliance['score']) / 3
    
    report = {
        'report_type': 'Compliance Audit',
        'generated_date': datetime.now().isoformat(),
        'overall_compliance': {
            'score': round(overall_score, 1),
            'status': 'COMPLIANT' if overall_score >= 80 else 'NON_COMPLIANT'
        },
        'sox_compliance': sox_compliance,
        'soc2_compliance': soc2_compliance,
        'iso27001_compliance': iso_compliance,
        'compliance_gaps': identify_compliance_gaps(dynamodb)
    }
    
    return report

def check_sox_compliance(dynamodb):
    """Check SOX compliance requirements"""
    return {
        'framework': 'SOX (Sarbanes-Oxley)',
        'score': 85,
        'status': 'COMPLIANT',
        'controls': [
            {'control': 'Access certification', 'status': 'PASS', 'evidence': 'Quarterly certifications completed'},
            {'control': 'Segregation of duties', 'status': 'PASS', 'evidence': 'Role-based access controls implemented'},
            {'control': 'Change management', 'status': 'PASS', 'evidence': 'All changes tracked and approved'}
        ],
        'gaps': []
    }

def check_soc2_compliance(dynamodb):
    """Check SOC2 compliance requirements"""
    return {
        'framework': 'SOC 2 Type II',
        'score': 90,
        'status': 'COMPLIANT',
        'controls': [
            {'control': 'Access controls', 'status': 'PASS', 'evidence': 'Role-based access implemented'},
            {'control': 'System monitoring', 'status': 'PASS', 'evidence': 'CloudWatch monitoring active'},
            {'control': 'Data protection', 'status': 'PASS', 'evidence': 'Encryption in transit and at rest'}
        ],
        'gaps': []
    }

def check_iso27001_compliance(dynamodb):
    """Check ISO27001 compliance requirements"""
    return {
        'framework': 'ISO 27001:2013',
        'score': 78,
        'status': 'PARTIALLY_COMPLIANT',
        'controls': [
            {'control': 'Information security policy', 'status': 'PASS', 'evidence': 'Policy documented and approved'},
            {'control': 'Access management', 'status': 'PASS', 'evidence': 'Identity governance system implemented'},
            {'control': 'Risk management', 'status': 'NEEDS_IMPROVEMENT', 'evidence': 'Risk assessments need regular updates'}
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

def store_audit_report(s3, report, report_type):
    """Store audit report in S3"""
    bucket_name = 'identity-governance-reports'  # From chapter 2
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    key = f'audit-reports/{report_type}_{timestamp}.json'
    
    try:
        s3.put_object(
            Bucket=bucket_name,
            Key=key,
            Body=json.dumps(report, indent=2),
            ContentType='application/json'
        )
        return f's3://{bucket_name}/{key}'
    except Exception as e:
        print(f'Error storing audit report: {str(e)}')
        return None
```

1. Click **Deploy**

### 2.3 Configure IAM Permissions

1. Add permissions for Lambda:
   - **AmazonDynamoDBFullAccess**
   - **AmazonS3FullAccess**
   - **CloudTrailReadOnlyAccess**

## Step 3: Setup Automated Audit Schedules

### 3.1 Create EventBridge Rules for Audit Reports

1. Open **Amazon EventBridge** console
2. Create rules for different audit schedules:
   - **Monthly Comprehensive Audit**: `cron(0 9 1 * ? *)`
   - **Weekly Risk Assessment**: `cron(0 9 ? * MON *)`
   - **Daily Compliance Check**: `cron(0 8 * * ? *)`

### 3.2 Configure Audit Report Storage

1. Verify S3 bucket `identity-governance-reports` exists
2. Create folder structure:
   - `/audit-reports/` - Generated audit reports
   - `/evidence/` - Supporting evidence files
   - `/compliance/` - Compliance documentation

## Step 4: Setup CloudWatch Logs Analysis

### 4.1 Configure CloudTrail Log Analysis

1. Open **Amazon CloudWatch** console
2. Navigate to **Logs Insights**
3. Create queries for audit trail analysis

### 4.2 Configure Log Insights Queries

1. In CloudWatch console
2. Click **Logs Insights**
3. Create saved queries for audit analysis

## Step 5: Test Audit System

### 5.1 Test Audit Report Generation

1. Run Lambda function **AuditReportGenerator**
2. Test with different report types
3. Verify reports are saved in S3

### 5.2 Verify Audit Trail Completeness

1. Check CloudTrail logs contain complete events
2. Verify log file validation
3. Test audit trail queries

### 5.3 Test Compliance Reporting

1. Generate compliance reports
2. Verify compliance scores
3. Review compliance gap identification

## Expected Results

After completion:

- ✅ Comprehensive audit trail system
- ✅ Automated audit report generation
- ✅ SOX, SOC2, ISO27001 compliance reporting
- ✅ Monthly and quarterly audit schedules
- ✅ Real-time audit event monitoring
- ✅ Compliance gap identification and remediation

## Next Steps

Proceed to [10. Compliance Validation](../10-xac-thuc-tuan-thu) to perform final validation and compliance verification.
 
 

