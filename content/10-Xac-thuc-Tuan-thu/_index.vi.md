---
title: "10. Xác thực Tuân thủ"
chapter: false
weight: 10
---

## Mục tiêu

Xác thực và duy trì tuân thủ các framework bảo mật và quy định pháp lý thông qua identity governance, đảm bảo tổ chức đáp ứng các yêu cầu SOX, SOC2, ISO27001, PCI-DSS và các tiêu chuẩn khác.

## Compliance Framework Mapping

![Compliance Framework Mapping](/images/10/compliance-framework-mapping.png)

## Bước 1: Thiết lập AWS Artifact cho Compliance Documentation

### 1.1 Truy cập AWS Artifact

1. Mở **AWS Artifact** trong console
2. Click **Reports** để xem các báo cáo compliance
3. Download các báo cáo cần thiết:
   - **SOC 2 Type II**
   - **ISO 27001**
   - **PCI DSS AOC**

![AWS Artifact](/images/10/aws-artifact.png)

4. Xem lại AWS responsibility matrix
5. Lưu trữ documents trong S3 bucket compliance

![Compliance Documents](/images/10/compliance-documents.png)

## Bước 2: Thiết lập AWS Security Hub

### 2.1 Kích hoạt Security Standards

1. Mở **AWS Security Hub**
2. Click **Security standards**
3. Kích hoạt các standards:
   - **AWS Foundational Security Standard**
   - **CIS AWS Foundations Benchmark**
   - **PCI DSS**

![Security Hub Standards](/images/10/security-hub-standards.png)

4. Cấu hình compliance scoring
5. Thiết lập automated remediation

![Compliance Scoring](/images/10/compliance-scoring.png)

## Bước 3: Tạo Compliance Dashboard

### 3.1 CloudWatch Dashboard

1. Mở **CloudWatch**
2. Click **Dashboards** → **Create dashboard**
3. Tên dashboard: **ComplianceMonitoring**
4. Thêm widgets:
   - **Compliance Score Metrics**
   - **Failed Controls Count**
   - **Remediation Status**

![Compliance Dashboard](/images/10/compliance-dashboard.png)

### 3.2 Thiết lập Compliance Alarms

1. Trong CloudWatch, click **Alarms**
2. Tạo alarm cho:
   - **Low Compliance Score**
   - **Critical Finding Detected**
   - **Failed Remediation**

![Compliance Alarms](/images/10/compliance-alarms.png)

## Bước 4: Automated Compliance Validation

### 4.1 Tạo Lambda Function

1. Mở **AWS Lambda**
2. Click **Create function**
3. Cấu hình:
   - **Function name**: ComplianceValidator
   - **Runtime**: Python 3.9
   - **Role**: ComplianceValidationRole

![Lambda Compliance Function](/images/10/lambda-compliance-function.png)

4. Thêm Lambda function code:

```python
import boto3
import json
from datetime import datetime, timedelta
from enum import Enum

class ComplianceFramework(Enum):
    SOX = "sox"
    SOC2 = "soc2"
    ISO27001 = "iso27001"
    PCI_DSS = "pci_dss"
    NIST = "nist"

class ComplianceValidator:
    def __init__(self):
        self.s3_client = boto3.client('s3')
        self.dynamodb = boto3.resource('dynamodb')
        self.config_client = boto3.client('config')
        self.security_hub = boto3.client('securityhub')
        
        # Compliance requirements mapping
        self.compliance_requirements = {
            ComplianceFramework.SOX: {
                "name": "Sarbanes-Oxley Act",
                "controls": [
                    "access_segregation",
                    "approval_workflows", 
                    "audit_trails",
                    "financial_access_controls"
                ]
            },
            ComplianceFramework.SOC2: {
                "name": "SOC 2 Type II",
                "controls": [
                    "logical_access_controls",
                    "system_monitoring",
                    "change_management",
                    "data_protection"
                ]
            },
            ComplianceFramework.ISO27001: {
                "name": "ISO 27001",
                "controls": [
                    "information_security_policy",
                    "access_control_management",
                    "incident_management",
                    "business_continuity"
                ]
            }
        }
    
    def validate_compliance(self, framework):
        """Validate compliance for specific framework"""
        
        validation_results = {
            'framework': framework.value,
            'timestamp': datetime.now().isoformat(),
            'overall_score': 0,
            'control_results': {},
            'findings': [],
            'recommendations': []
        }
        
        if framework in self.compliance_requirements:
            controls = self.compliance_requirements[framework]['controls']
            
            for control in controls:
                control_score = self.validate_control(control)
                validation_results['control_results'][control] = control_score
                
                if control_score < 80:  # Threshold for compliance
                    validation_results['findings'].append({
                        'control': control,
                        'score': control_score,
                        'severity': 'HIGH' if control_score < 50 else 'MEDIUM'
                    })
            
            # Calculate overall score
            if validation_results['control_results']:
                validation_results['overall_score'] = sum(
                    validation_results['control_results'].values()
                ) / len(validation_results['control_results'])
        
        return validation_results
    
    def validate_control(self, control_name):
        """Validate specific control implementation"""
        
        # Control validation logic based on AWS Config rules
        control_mappings = {
            'access_segregation': self.check_iam_separation(),
            'audit_trails': self.check_cloudtrail_enabled(),
            'logical_access_controls': self.check_mfa_enabled(),
            'system_monitoring': self.check_cloudwatch_monitoring(),
            'data_protection': self.check_encryption_enabled()
        }
        
        return control_mappings.get(control_name, 0)
    
    def check_iam_separation(self):
        """Check IAM role separation"""
        try:
            # Check for proper role separation
            # This would implement actual IAM policy analysis
            return 85  # Demo score
        except Exception:
            return 0
    
    def check_cloudtrail_enabled(self):
        """Check CloudTrail configuration"""
        try:
            cloudtrail = boto3.client('cloudtrail')
            trails = cloudtrail.describe_trails()
            
            if trails['trailList']:
                # Check if trails are properly configured
                return 90
            return 0
        except Exception:
            return 0
    
    def check_mfa_enabled(self):
        """Check MFA enforcement"""
        try:
            # Check MFA policies and enforcement
            return 75  # Demo score
        except Exception:
            return 0
    
    def check_cloudwatch_monitoring(self):
        """Check CloudWatch monitoring setup"""
        try:
            cloudwatch = boto3.client('cloudwatch')
            alarms = cloudwatch.describe_alarms()
            
            if alarms['MetricAlarms']:
                return 80
            return 0
        except Exception:
            return 0
    
    def check_encryption_enabled(self):
        """Check encryption configuration"""
        try:
            # Check S3 bucket encryption, EBS encryption, etc.
            return 85  # Demo score
        except Exception:
            return 0
    
    def generate_compliance_report(self, validation_results):
        """Generate compliance report"""
        
        report = {
            'report_id': f"compliance-{datetime.now().strftime('%Y%m%d-%H%M%S')}",
            'generated_at': datetime.now().isoformat(),
            'framework': validation_results['framework'],
            'overall_compliance': validation_results['overall_score'],
            'status': 'COMPLIANT' if validation_results['overall_score'] >= 80 else 'NON_COMPLIANT',
            'summary': {
                'total_controls': len(validation_results['control_results']),
                'passed_controls': len([s for s in validation_results['control_results'].values() if s >= 80]),
                'failed_controls': len([s for s in validation_results['control_results'].values() if s < 80]),
                'critical_findings': len([f for f in validation_results['findings'] if f['severity'] == 'HIGH'])
            },
            'detailed_results': validation_results
        }
        
        return report

def lambda_handler(event, context):
    validator = ComplianceValidator()
    
    # Validate all frameworks
    all_results = {}
    
    for framework in ComplianceFramework:
        results = validator.validate_compliance(framework)
        report = validator.generate_compliance_report(results)
        all_results[framework.value] = report
    
    return {
        'statusCode': 200,
        'body': json.dumps(all_results, default=str)
    }
```

### 4.2 Thiết lập EventBridge Schedule

1. Mở **EventBridge**
2. Tạo rule cho daily compliance check
3. Target: Lambda ComplianceValidator function

![EventBridge Compliance Rule](/images/10/eventbridge-compliance-rule.png)

## Bước 5: Compliance Evidence Collection

### 5.1 Sử dụng AWS Config

1. Sử dụng **AWS Config** để validate:
   - **IAM password policies**
   - **S3 bucket encryption**
   - **CloudTrail logging**
   - **VPC security groups**

![Config Compliance Rules](/images/10/config-compliance-rules.png)

### 5.2 Automated Evidence Storage

1. Cấu hình Config để lưu compliance evidence vào S3
2. Thiết lập retention policies
3. Tạo compliance reports tự động

![Evidence Collection](/images/10/evidence-collection.png)

## Kết quả Mong đợi

Sau khi hoàn thành workshop này, bạn sẽ có:

### ✅ Comprehensive Identity Governance System
- Centralized access management với AWS IAM Identity Center
- Automated access certification workflows
- Real-time privilege analytics và risk assessment
- Continuous monitoring và alerting

### ✅ Compliance Framework Implementation
- SOX, SOC2, ISO27001, PCI-DSS compliance validation
- Automated evidence collection
- Regulatory reporting capabilities
- Audit trail maintenance

### ✅ Operational Excellence
- Standardized operational procedures
- Incident response capabilities
- Change management processes
- Performance monitoring

### ✅ Audit và Governance
- Comprehensive audit framework
- Automated control testing
- Finding tracking và remediation
- Management reporting

## Best Practices Summary

1. **Implement Least Privilege**: Chỉ cấp quyền tối thiểu cần thiết
2. **Automate Where Possible**: Tự động hóa các quy trình lặp lại
3. **Monitor Continuously**: Giám sát liên tục các hoạt động
4. **Document Everything**: Ghi chép đầy đủ cho audit trail
5. **Regular Reviews**: Thực hiện review định kỳ
6. **Stay Updated**: Cập nhật theo các thay đổi compliance

## Tài liệu Tham khảo

- [AWS IAM Identity Center Documentation](https://docs.aws.amazon.com/singlesignon/)
- [AWS Organizations Best Practices](https://docs.aws.amazon.com/organizations/)
- [SOX Compliance Guidelines](https://www.sec.gov/about/laws/soa2002.pdf)
- [SOC 2 Framework](https://www.aicpa.org/interestareas/frc/assuranceadvisoryservices/aicpasoc2report.html)
- [ISO 27001 Standard](https://www.iso.org/isoiec-27001-information-security.html)

## Hỗ trợ

Nếu bạn gặp vấn đề trong quá trình triển khai, vui lòng:
1. Kiểm tra CloudWatch Logs để debug
2. Xem lại IAM permissions
3. Tham khảo AWS documentation
4. Liên hệ team support nếu cần thiết

**Workshop hoàn thành thành công! 🎉**