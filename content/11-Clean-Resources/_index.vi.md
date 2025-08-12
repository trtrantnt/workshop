---
title: "11. Dọn dẹp Tài nguyên"
chapter: false
weight: 11
---

## Tổng quan

Phần này cung cấp hướng dẫn toàn diện để dọn dẹp tất cả tài nguyên AWS được tạo trong workshop Identity Governance nhằm tránh phát sinh chi phí không cần thiết.

## Lưu ý quan trọng

⚠️ **Cảnh báo**: Thực hiện các bước dọn dẹp này sẽ xóa vĩnh viễn tất cả tài nguyên và dữ liệu được tạo trong workshop. Hãy đảm bảo bạn đã sao lưu các cấu hình hoặc dữ liệu quan trọng trước khi tiến hành.

## Thứ tự dọn dẹp

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

1. Mở **AWS Lambda** console
2. Lọc functions theo tên workshop:
   - **AccessCertificationTrigger**
   - **PrivilegeAnalyticsEngine**
   - **RiskAssessmentEngine**
   - **CustomMetricsPublisher**
   - **DailyOperationsEngine**
   - **AuditReportGenerator**
   - **E2EValidationTest**
3. Chọn từng function
4. Click **Actions** → **Delete**
5. Xác nhận xóa bằng cách gõ **confirm**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/11/del1.png?featherlight=false&width=90pc)

### Xóa EventBridge Rules

1. Mở **Amazon EventBridge** console
2. Vào **Schedules**
3. Chọn workshop rules:
   - **AccessCertificationSchedule**
   - **ComplianceValidationSchedule**
   - **RiskAssessmentSchedule**
   - **DailyOperationsSchedule**
   - **WeeklyOperationsReview**
4. Click **Delete** cho từng rule

![EventBridge Rules](https://trtrantnt.github.io/workshop/images/11/del2.png?featherlight=false&width=90pc)


## Bước 2: DynamoDB Tables

1. Mở **Amazon DynamoDB** console
2. Vào **Tables**
3. Chọn workshop tables:
   - **AccessCertifications**
   - **RiskAssessments**
   - **CertificationTasks**
   - **OperationsLog**
   - **ComplianceEvidence**
   - **RiskMonitoring**
   - **AuditFindings**
4. Click **Delete** cho từng table
5. Gõ **delete** để xác nhận

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/11/del3.png?featherlight=false&width=90pc)

## Bước 3: S3 Buckets

### Làm trống S3 Buckets

1. Mở **Amazon S3** console
2. Xác định workshop buckets:
   - **identity-governance-analytics**
   - **identity-governance-reports**
   - **aws-cloudtrail-logs-*** (CloudTrail bucket)
3. Chọn từng bucket và click **Empty**
4. Gõ `permanently delete` để xác nhận

### Xóa S3 Buckets

1. Sau khi làm trống, chọn từng bucket
2. Click **Delete**
3. Gõ tên bucket để xác nhận

![Làm trống S3 Bucket](https://trtrantnt.github.io/workshop/images/11/del5.png?featherlight=false&width=90pc)

## Bước 4: CloudWatch Resources

### Xóa CloudWatch Dashboards

1. Mở **Amazon CloudWatch** console
2. Vào **Dashboards**
3. Chọn workshop dashboards:
   - **IdentityGovernanceRiskDashboard**
   - **IdentityGovernanceOperations**
   - **DailyOperationsDashboard**
4. Click **Delete** cho từng dashboard

### Xóa CloudWatch Alarms

1. Vào **Alarms**
2. Chọn workshop alarms:
   - **Lambda-AccessCertification-Errors**
   - **HighRiskUserCount-Alarm**
   - **DynamoDB-ReadErrors**
   - **S3-AccessErrors**
3. Click **Actions** → **Delete**

### Xóa Log Groups

1. Vào **Log groups**
2. Chọn workshop log groups:
   - `/aws/lambda/AccessCertificationTrigger`
   - `/aws/lambda/PrivilegeAnalyticsEngine`
   - `/aws/lambda/RiskAssessmentEngine`
   - `/aws/lambda/CustomMetricsPublisher`
   - `/aws/lambda/DailyOperationsEngine`
   - `/aws/lambda/AuditReportGenerator`
   - `/aws/lambda/E2EValidationTest`
3. Click **Actions** → **Delete log group**

## Bước 5: SNS Topics

1. Mở **Amazon SNS** console
2. Vào **Topics**
3. Chọn workshop topics:
   - **IdentityGovernanceAlerts**
   - **ComplianceAlerts**
   - **RiskAssessmentAlerts**
   - **DailyOperationsAlerts**

4. Click **Delete** cho từng topic
5. Xác nhận xóa

## Bước 6: IAM Resources

### Xóa IAM Roles

1. Điều hướng đến dịch vụ **IAM** trong AWS Console
2. Click **Roles** trong sidebar
3. Tìm kiếm các workshop roles:
   - **IdentityGovernanceLambdaRole**
   - **ComplianceValidationRole**
   - **CertificationWorkflowRole**
4. Chọn từng role và click **Delete**
5. Gõ tên role để xác nhận xóa

![Làm trống S3 Bucket](https://trtrantnt.github.io/workshop/images/11/del6.png?featherlight=false&width=90pc)

### Xóa Custom IAM Policies

1. Click **Policies** trong sidebar
2. Lọc theo **Customer managed**
3. Tìm kiếm các workshop policies:
   - **SecurityAuditPolicy**
   - **IdentityGovernancePolicy**
   - **ComplianceValidationPolicy**

4. Chọn từng policy và click **Actions** → **Delete**
5. Xác nhận xóa

### Xóa IAM Users và Groups

1. Click **Users** trong sidebar
2. Chọn workshop users và click **Delete**
3. Click **User groups** trong sidebar


## Bước 7: Dọn dẹp IAM Identity Center

### Xóa Permission Set Assignments

1. Điều hướng đến **IAM Identity Center**
2. Click **AWS accounts** trong sidebar
3. Chọn account của bạn và click **Remove access**

### Xóa Permission Sets

1. Click **Permission sets** trong sidebar
2. Chọn các workshop permission sets:
   - **SecurityAuditor**
   - **ComplianceReviewer**
3. Click **Delete**

![Làm trống S3 Bucket](https://trtrantnt.github.io/workshop/images/11/del7.png?featherlight=false&width=90pc)

### Xóa Users và Groups

1. Click **Users** trong sidebar
2. Chọn workshop users và click **Delete**

3. Click **Groups** trong sidebar
4. Chọn workshop groups và click **Delete**


## Bước 8: Dọn dẹp CloudTrail

1. Điều hướng đến dịch vụ **CloudTrail**
2. Click **Trails** trong sidebar
3. Chọn **IdentityGovernanceTrail**
4. Click **Delete**

![Làm trống S3 Bucket](https://trtrantnt.github.io/workshop/images/11/del8.png?featherlight=false&width=90pc)

## Checklist dọn dẹp qua Console

Để dọn dẹp có hệ thống qua AWS Console, hãy làm theo checklist này:

### ✅ Checklist dọn dẹp

**Lambda Functions:**

- [ ] AccessCertificationTrigger
- [ ] PrivilegeAnalyticsEngine
- [ ] RiskAssessmentEngine
- [ ] CustomMetricsPublisher
- [ ] DailyOperationsEngine
- [ ] AuditReportGenerator
- [ ] E2EValidationTest

**EventBridge Rules:**

- [ ] AccessCertificationSchedule
- [ ] ComplianceValidationSchedule
- [ ] RiskAssessmentSchedule
- [ ] DailyOperationsSchedule
- [ ] WeeklyOperationsReview

**DynamoDB Tables:**

- [ ] AccessCertifications
- [ ] RiskAssessments
- [ ] CertificationTasks
- [ ] OperationsLog
- [ ] ComplianceEvidence
- [ ] RiskMonitoring
- [ ] AuditFindings

**S3 Buckets:**

- [ ] identity-governance-analytics
- [ ] identity-governance-reports
- [ ] aws-cloudtrail-logs-[ACCOUNT-ID]-[HASH]

**CloudWatch Resources:**

- [ ] IdentityGovernanceRiskDashboard
- [ ] IdentityGovernanceOperations
- [ ] DailyOperationsDashboard
- [ ] Tất cả workshop alarms
- [ ] Tất cả workshop log groups

**SNS Topics:**

- [ ] IdentityGovernanceAlerts
- [ ] ComplianceAlerts
- [ ] RiskAssessmentAlerts
- [ ] DailyOperationsAlerts

**IAM Resources:**

- [ ] Workshop IAM roles
- [ ] Workshop custom policies (nếu có)

**Optional Resources:**

- [ ] AWS Security Hub (nếu không cần)
- [ ] AWS CloudTrail (nếu không cần)


## Bước 9: Xác minh dọn dẹp

### Kiểm tra tài nguyên còn lại

1. Kiểm tra **AWS Cost Explorer** để xác nhận không còn phí phát sinh
2. Sử dụng **AWS Resource Groups** để tìm tagged resources
3. Tìm kiếm tag: **Project=IdentityGovernance**

### Kiểm tra dịch vụ cuối cùng

1. **AWS Config**: Tắt configuration recorder nếu không cần
2. **AWS Security Hub**: Tắt nếu không sử dụng ở nơi khác
3. **Amazon GuardDuty**: Tắt nếu không cần
4. **AWS Audit Manager**: Tắt data collection

## Hoàn thành dọn dẹp

Dọn dẹp workshop hoàn tất! 🎉