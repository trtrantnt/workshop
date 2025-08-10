---
title: "11. Dọn dẹp Tài nguyên"
chapter: false
weight: 11
---

# Dọn dẹp Tài nguyên

## Tổng quan

Phần này cung cấp hướng dẫn toàn diện để dọn dẹp tất cả tài nguyên AWS được tạo trong workshop Identity Governance nhằm tránh phát sinh chi phí không cần thiết.

## Lưu ý quan trọng

⚠️ **Cảnh báo**: Thực hiện các bước dọn dẹp này sẽ xóa vĩnh viễn tất cả tài nguyên và dữ liệu được tạo trong workshop. Hãy đảm bảo bạn đã sao lưu các cấu hình hoặc dữ liệu quan trọng trước khi tiến hành.

## Thứ tự Dọn dẹp

Tài nguyên nên được dọn dẹp theo thứ tự sau để tránh xung đột phụ thuộc:

1. Lambda Functions và EventBridge Rules
2. Step Functions State Machines
3. DynamoDB Tables
4. S3 Buckets và Objects
5. CloudWatch Resources
6. IAM Roles và Policies
7. CloudFormation Stacks
8. AWS Organizations (nếu đã tạo)
9. IAM Identity Center (nếu không còn cần thiết)

## Bước 1: Lambda Functions và EventBridge

### Xóa Lambda Functions

```bash
# Liệt kê tất cả Lambda functions được tạo trong workshop
aws lambda list-functions --query 'Functions[?contains(FunctionName, `IdentityGovernance`) || contains(FunctionName, `Compliance`) || contains(FunctionName, `AccessReview`)].FunctionName' --output table

# Xóa Lambda functions
aws lambda delete-function --function-name IdentityGovernanceMonitor
aws lambda delete-function --function-name AccessReviewGenerator
aws lambda delete-function --function-name ComplianceValidationEngine
aws lambda delete-function --function-name RiskAssessmentEngine
aws lambda delete-function --function-name CertificationNotifier
```

### Xóa EventBridge Rules

```bash
# Liệt kê EventBridge rules
aws events list-rules --query 'Rules[?contains(Name, `IdentityGovernance`) || contains(Name, `Compliance`) || contains(Name, `Certification`)].Name' --output table

# Xóa EventBridge rules
aws events delete-rule --name AccessCertificationSchedule
aws events delete-rule --name ComplianceValidationSchedule
aws events delete-rule --name RiskAssessmentSchedule
```

## Bước 2: Step Functions

```bash
# Liệt kê Step Functions
aws stepfunctions list-state-machines --query 'stateMachines[?contains(name, `Certification`) || contains(name, `Governance`)].name' --output table

# Xóa Step Functions
aws stepfunctions delete-state-machine --state-machine-arn arn:aws:states:REGION:ACCOUNT:stateMachine:CertificationWorkflow
```

## Bước 3: DynamoDB Tables

```bash
# Liệt kê DynamoDB tables
aws dynamodb list-tables --query 'TableNames[?contains(@, `Certification`) || contains(@, `Operations`) || contains(@, `Compliance`) || contains(@, `Risk`)]' --output table

# Xóa DynamoDB tables
aws dynamodb delete-table --table-name CertificationTasks
aws dynamodb delete-table --table-name OperationsLog
aws dynamodb delete-table --table-name ComplianceEvidence
aws dynamodb delete-table --table-name RiskMonitoring
aws dynamodb delete-table --table-name AuditFindings
```

## Bước 4: S3 Buckets

### Làm trống và Xóa S3 Buckets

```bash
# Lấy account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Liệt kê workshop S3 buckets
aws s3 ls | grep -E "(privilege-analytics|compliance-reports|identity-governance)"

# Làm trống buckets trước
aws s3 rm s3://privilege-analytics-${ACCOUNT_ID} --recursive
aws s3 rm s3://compliance-reports-${ACCOUNT_ID} --recursive

# Xóa buckets
aws s3 rb s3://privilege-analytics-${ACCOUNT_ID}
aws s3 rb s3://compliance-reports-${ACCOUNT_ID}
```

## Bước 5: CloudWatch Resources

### Xóa CloudWatch Dashboards

```bash
# Liệt kê dashboards
aws cloudwatch list-dashboards --query 'DashboardEntries[?contains(DashboardName, `IdentityGovernance`) || contains(DashboardName, `Compliance`)].DashboardName' --output table

# Xóa dashboards
aws cloudwatch delete-dashboards --dashboard-names IdentityGovernanceRiskDashboard IdentityGovernanceMonitoring ComplianceDashboard
```

### Xóa CloudWatch Alarms

```bash
# Liệt kê alarms
aws cloudwatch describe-alarms --query 'MetricAlarms[?contains(AlarmName, `IdentityGovernance`) || contains(AlarmName, `Compliance`) || contains(AlarmName, `Risk`)].AlarmName' --output table

# Xóa alarms
aws cloudwatch delete-alarms --alarm-names HighFailedLoginAttempts PrivilegeEscalationDetected HighRiskEventsAlarm ComplianceViolationAlarm
```

## Bước 6: IAM Resources

### Xóa IAM Roles

```bash
# Liệt kê IAM roles được tạo cho workshop
aws iam list-roles --query 'Roles[?contains(RoleName, `IdentityGovernance`) || contains(RoleName, `Compliance`) || contains(RoleName, `Certification`)].RoleName' --output table

# Detach policies và xóa roles
aws iam detach-role-policy --role-name IdentityGovernanceLambdaRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam delete-role --role-name IdentityGovernanceLambdaRole

aws iam delete-role --role-name ComplianceValidationRole
aws iam delete-role --role-name CertificationWorkflowRole
```

## Bước 7: CloudFormation Stacks

```bash
# Liệt kê CloudFormation stacks
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE --query 'StackSummaries[?contains(StackName, `identity-governance`) || contains(StackName, `compliance`)].StackName' --output table

# Xóa CloudFormation stacks
aws cloudformation delete-stack --stack-name identity-governance-base
aws cloudformation delete-stack --stack-name identity-governance-monitoring
aws cloudformation delete-stack --stack-name identity-governance-compliance
```

## Script Dọn dẹp Tự động

Để thuận tiện, đây là script dọn dẹp toàn diện:

```bash
#!/bin/bash

echo "Bắt đầu dọn dẹp Identity Governance Workshop..."

# Lấy account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REGION=$(aws configure get region)

echo "Account ID: $ACCOUNT_ID"
echo "Region: $REGION"

# Function để kiểm tra resource tồn tại trước khi xóa
check_and_delete() {
    local resource_type=$1
    local resource_name=$2
    local delete_command=$3
    
    echo "Đang kiểm tra $resource_type: $resource_name"
    if eval "$delete_command" 2>/dev/null; then
        echo "✅ Đã xóa $resource_type: $resource_name"
    else
        echo "⚠️  $resource_type không tìm thấy hoặc đã được xóa: $resource_name"
    fi
}

# Xóa Lambda functions
echo "🧹 Đang dọn dẹp Lambda functions..."
LAMBDA_FUNCTIONS=("IdentityGovernanceMonitor" "AccessReviewGenerator" "ComplianceValidationEngine" "RiskAssessmentEngine")
for func in "${LAMBDA_FUNCTIONS[@]}"; do
    check_and_delete "Lambda function" "$func" "aws lambda delete-function --function-name $func"
done

# Xóa DynamoDB tables
echo "🧹 Đang dọn dẹp DynamoDB tables..."
DYNAMODB_TABLES=("CertificationTasks" "OperationsLog" "ComplianceEvidence" "RiskMonitoring" "AuditFindings")
for table in "${DYNAMODB_TABLES[@]}"; do
    check_and_delete "DynamoDB table" "$table" "aws dynamodb delete-table --table-name $table"
done

# Xóa S3 buckets
echo "🧹 Đang dọn dẹp S3 buckets..."
S3_BUCKETS=("privilege-analytics-${ACCOUNT_ID}" "compliance-reports-${ACCOUNT_ID}")
for bucket in "${S3_BUCKETS[@]}"; do
    echo "Đang làm trống S3 bucket: $bucket"
    aws s3 rm s3://$bucket --recursive 2>/dev/null || echo "Bucket $bucket không tìm thấy"
    check_and_delete "S3 bucket" "$bucket" "aws s3 rb s3://$bucket"
done

# Xóa CloudFormation stacks
echo "🧹 Đang dọn dẹp CloudFormation stacks..."
CF_STACKS=("identity-governance-base" "identity-governance-monitoring" "identity-governance-compliance")
for stack in "${CF_STACKS[@]}"; do
    check_and_delete "CloudFormation stack" "$stack" "aws cloudformation delete-stack --stack-name $stack"
done

echo "🎉 Dọn dẹp hoàn tất!"
echo "Lưu ý: Một số tài nguyên có thể mất vài phút để được xóa hoàn toàn."
echo "Vui lòng kiểm tra AWS Console để xác nhận tất cả tài nguyên đã được xóa."
```

## Xác minh

Sau khi chạy dọn dẹp, hãy xác minh rằng tất cả tài nguyên đã được xóa:

### Kiểm tra Tài nguyên Còn lại

```bash
# Kiểm tra Lambda functions
aws lambda list-functions --query 'Functions[?contains(FunctionName, `IdentityGovernance`)].FunctionName'

# Kiểm tra DynamoDB tables
aws dynamodb list-tables --query 'TableNames[?contains(@, `Certification`) || contains(@, `Compliance`)]'

# Kiểm tra S3 buckets
aws s3 ls | grep -E "(privilege-analytics|compliance-reports)"

# Kiểm tra CloudFormation stacks
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE --query 'StackSummaries[?contains(StackName, `identity-governance`)].StackName'
```

## Xác minh Chi phí

Sau khi dọn dẹp, hãy theo dõi dashboard billing AWS để đảm bảo không có phí phát sinh từ tài nguyên còn lại.

## Khắc phục Sự cố

### Vấn đề Thường gặp

1. **Lỗi Phụ thuộc**: Một số tài nguyên có thể có phụ thuộc. Hãy xóa tài nguyên phụ thuộc trước.
2. **Lỗi Quyền**: Đảm bảo bạn có đủ quyền để xóa tất cả tài nguyên.
3. **Vấn đề Region**: Đảm bảo bạn đang xóa tài nguyên ở đúng region.

### Dọn dẹp Thủ công

Nếu dọn dẹp tự động thất bại, hãy xóa tài nguyên thủ công qua AWS Console:

1. Truy cập console của từng service
2. Tìm kiếm tài nguyên có tên chứa "IdentityGovernance", "Compliance", hoặc "Certification"
3. Xóa từng tài nguyên một cách riêng lẻ

## Ghi chú Cuối

- Việc dọn dẹp này sẽ xóa TẤT CẢ tài nguyên workshop
- Kiểm tra kỹ trước khi chạy lệnh dọn dẹp
- Một số tài nguyên có thể có độ trễ ngắn trước khi bị xóa
- Theo dõi hóa đơn AWS để đảm bảo không có phí tiếp tục phát sinh

**Dọn dẹp workshop hoàn tất! 🎉**