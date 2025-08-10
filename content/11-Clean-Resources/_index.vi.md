---
title: "11. Dọn dẹp Tài nguyên"
chapter: false
weight: 11
---

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

1. Điều hướng đến dịch vụ **IAM** trong AWS Console
2. Click **Roles** trong sidebar
3. Tìm kiếm các workshop roles:
   - **IdentityGovernanceLambdaRole**
   - **ComplianceValidationRole**
   - **CertificationWorkflowRole**

![Danh sách IAM Roles](/images/11/iam-roles-list.png)

4. Chọn từng role và click **Delete**
5. Gõ tên role để xác nhận xóa

![Xóa IAM Role](/images/11/delete-iam-role.png)

### Xóa Custom IAM Policies

1. Click **Policies** trong sidebar
2. Lọc theo **Customer managed**
3. Tìm kiếm các workshop policies:
   - **SecurityAuditPolicy**
   - **IdentityGovernancePolicy**
   - **ComplianceValidationPolicy**

![Danh sách IAM Policies](/images/11/iam-policies-list.png)

4. Chọn từng policy và click **Actions** → **Delete**
5. Xác nhận xóa

![Xóa IAM Policy](/images/11/delete-iam-policy.png)

### Xóa IAM Users và Groups

1. Click **Users** trong sidebar
2. Chọn workshop users và click **Delete**

![Xóa IAM Users](/images/11/delete-iam-users.png)

3. Click **User groups** trong sidebar
4. Chọn workshop groups và click **Delete**

![Xóa IAM Groups](/images/11/delete-iam-groups.png)

## Bước 7: Dọn dẹp IAM Identity Center

### Xóa Permission Set Assignments

1. Điều hướng đến **IAM Identity Center**
2. Click **AWS accounts** trong sidebar
3. Chọn account của bạn và click **Remove access**

![Xóa SSO Access](/images/11/remove-sso-access.png)

### Xóa Permission Sets

1. Click **Permission sets** trong sidebar
2. Chọn các workshop permission sets:
   - **SecurityAuditor**
   - **ComplianceReviewer**
3. Click **Delete**

![Xóa Permission Sets](/images/11/delete-permission-sets.png)

### Xóa Users và Groups

1. Click **Users** trong sidebar
2. Chọn workshop users và click **Delete**

![Xóa SSO Users](/images/11/delete-sso-users.png)

3. Click **Groups** trong sidebar
4. Chọn workshop groups và click **Delete**

![Xóa SSO Groups](/images/11/delete-sso-groups.png)

## Bước 8: Dọn dẹp AWS Config

1. Điều hướng đến dịch vụ **AWS Config**
2. Click **Settings** trong sidebar
3. Click **Edit** và sau đó **Delete configuration recorder**

![Xóa Config Recorder](/images/11/delete-config-recorder.png)

4. Xác nhận xóa bằng cách gõ **delete**

## Bước 9: Dọn dẹp CloudTrail

1. Điều hướng đến dịch vụ **CloudTrail**
2. Click **Trails** trong sidebar
3. Chọn **IdentityGovernanceTrail**
4. Click **Delete**

![Xóa CloudTrail](/images/11/delete-cloudtrail.png)

5. Gõ tên trail để xác nhận xóa

## Checklist Dọn dẹp qua Console

Để dọn dẹp có hệ thống qua AWS Console, hãy làm theo checklist này:

### ✅ Checklist Dọn dẹp

**Lambda Functions:**
- [ ] IdentityGovernanceMonitor
- [ ] AccessReviewGenerator  
- [ ] ComplianceValidationEngine
- [ ] RiskAssessmentEngine
- [ ] CertificationNotifier

**EventBridge Rules:**
- [ ] AccessCertificationSchedule
- [ ] ComplianceValidationSchedule
- [ ] RiskAssessmentSchedule

**Step Functions:**
- [ ] AccessCertificationWorkflow
- [ ] ComplianceValidationWorkflow

**DynamoDB Tables:**
- [ ] CertificationTasks
- [ ] OperationsLog
- [ ] ComplianceEvidence
- [ ] RiskMonitoring
- [ ] AuditFindings

**S3 Buckets:**
- [ ] privilege-analytics-[ACCOUNT-ID]
- [ ] compliance-reports-[ACCOUNT-ID]

**CloudWatch Resources:**
- [ ] IdentityGovernanceRiskDashboard
- [ ] DailyOperationsDashboard
- [ ] Tất cả workshop alarms
- [ ] Tất cả workshop log groups

**SNS Topics:**
- [ ] IdentityGovernanceAlerts
- [ ] ComplianceAlerts

**IAM Resources:**
- [ ] Workshop IAM roles
- [ ] Workshop IAM policies
- [ ] Workshop IAM users
- [ ] Workshop IAM groups

**IAM Identity Center:**
- [ ] Permission set assignments
- [ ] Permission sets
- [ ] SSO users và groups

**Các Dịch vụ Khác:**
- [ ] AWS Config recorder
- [ ] CloudTrail trail
- [ ] GuardDuty detector (nếu không cần)

![Checklist Dọn dẹp](/images/11/cleanup-checklist.png)

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