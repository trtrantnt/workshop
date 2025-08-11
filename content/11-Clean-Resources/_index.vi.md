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

1. Mở **AWS Lambda** console
2. Lọc functions theo tên workshop:
   - **IdentityGovernance**
   - **AccessCertification**
   - **ComplianceValidation**

![Danh sách Lambda Functions](/images/11/lambda-functions-list.png)

3. Chọn workshop functions
4. Click **Actions** → **Delete**

![Xóa Lambda Functions](/images/11/delete-lambda-functions.png)

5. Xác nhận xóa bằng cách gõ **delete**

![Xác nhận Xóa Lambda](/images/11/confirm-lambda-deletion.png)

### Xóa EventBridge Rules

1. Mở **Amazon EventBridge** console
2. Vào **Rules**
3. Chọn workshop rules:
   - **AccessCertificationSchedule**
   - **ComplianceValidationSchedule**

![EventBridge Rules](/images/11/eventbridge-rules.png)

4. Click **Delete** cho từng rule

![Xóa EventBridge Rules](/images/11/delete-eventbridge-rules.png)

## Bước 2: Step Functions

1. Mở **AWS Step Functions** console
2. Chọn workshop state machines:
   - **AccessCertificationWorkflow**
   - **ComplianceValidationWorkflow**

![Danh sách Step Functions](/images/11/step-functions-list.png)

3. Click **Delete**
4. Xác nhận xóa

![Xóa Step Functions](/images/11/delete-step-functions.png)

## Bước 3: DynamoDB Tables

1. Mở **Amazon DynamoDB** console
2. Vào **Tables**
3. Chọn workshop tables:
   - **OperationalProcedures**
   - **ComplianceEvidence**
   - **AuditFindings**

![DynamoDB Tables](/images/11/dynamodb-tables.png)

4. Click **Delete** cho từng table
5. Gõ **delete** để xác nhận

![Xóa DynamoDB Tables](/images/11/delete-dynamodb-tables.png)

## Bước 4: S3 Buckets

### Làm trống S3 Buckets

1. Mở **Amazon S3** console
2. Xác định workshop buckets:
   - **privilege-analytics-***
   - **compliance-reports-***

![Danh sách S3 Buckets](/images/11/s3-buckets-list.png)

3. Chọn bucket và click **Empty**
4. Gõ **permanently delete** để xác nhận

![Làm trống S3 Bucket](/images/11/empty-s3-bucket.png)

### Xóa S3 Buckets

1. Sau khi làm trống, chọn bucket
2. Click **Delete**
3. Gõ tên bucket để xác nhận

![Xóa S3 Bucket](/images/11/delete-s3-bucket.png)

## Bước 5: CloudWatch Resources

### Xóa CloudWatch Dashboards

1. Mở **Amazon CloudWatch** console
2. Vào **Dashboards**
3. Chọn workshop dashboards:
   - **IdentityGovernanceRiskDashboard**
   - **DailyOperationsDashboard**

![CloudWatch Dashboards](/images/11/cloudwatch-dashboards.png)

4. Click **Delete** cho từng dashboard

![Xóa Dashboard](/images/11/delete-dashboard.png)

### Xóa CloudWatch Alarms

1. Vào **Alarms**
2. Chọn workshop alarms
3. Click **Actions** → **Delete**

![Xóa CloudWatch Alarms](/images/11/delete-cloudwatch-alarms.png)

### Xóa Log Groups

1. Vào **Log groups**
2. Chọn workshop log groups
3. Click **Actions** → **Delete log group**

![Xóa Log Groups](/images/11/delete-log-groups.png)

## Bước 6: SNS Topics

1. Mở **Amazon SNS** console
2. Vào **Topics**
3. Chọn workshop topics:
   - **IdentityGovernanceAlerts**
   - **ComplianceAlerts**

![SNS Topics](/images/11/sns-topics.png)

4. Click **Delete** cho từng topic
5. Xác nhận xóa

![Xóa SNS Topics](/images/11/delete-sns-topics.png)

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

## Bước 10: Xác minh Dọn dẹp

### Kiểm tra Tài nguyên Còn lại

1. Kiểm tra **AWS Cost Explorer** để xác nhận không còn phí phát sinh

![Xác minh Cost Explorer](/images/11/cost-explorer-verification.png)

2. Sử dụng **AWS Resource Groups** để tìm tagged resources
3. Tìm kiếm tag: **Project=IdentityGovernance**

![Kiểm tra Resource Groups](/images/11/resource-groups-check.png)

### Kiểm tra Dịch vụ Cuối cùng

1. **AWS Config**: Tắt configuration recorder nếu không cần
2. **AWS Security Hub**: Tắt nếu không sử dụng ở nơi khác
3. **Amazon GuardDuty**: Tắt nếu không cần
4. **AWS Audit Manager**: Tắt data collection

![Kiểm tra Dịch vụ Cuối](/images/11/final-service-checks.png)

### Báo cáo Xác minh Dọn dẹp

1. Tạo báo cáo tóm tắt dọn dẹp
2. Ghi chép các tài nguyên không thể xóa
3. Lưu ý các phí đang phát sinh

![Báo cáo Xác minh Dọn dẹp](/images/11/cleanup-verification-report.png)

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