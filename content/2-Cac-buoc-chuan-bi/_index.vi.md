---
title: "2. Các bước chuẩn bị"
chapter: false
weight: 2
---

## Thiết lập môi trường

### 1. Tạo S3 Bucket cho lưu trữ dữ liệu

1. Điều hướng đến dịch vụ **Amazon S3** trong AWS Console

![Điều hướng đến S3](/images/2/s3b1.png?featherlight=false&width=90pc)

2. Click **Create bucket**

![Điều hướng đến S3](/images/2/s3b2.png?featherlight=false&width=90pc)

3. Tạo bucket đầu tiên cho dữ liệu phân tích:
   - **Bucket name**: `identity-governance-analytics`
   - **AWS Region**: Chọn region ưa thích (ví dụ: us-east-1)
   - **Object Ownership**: ACLs disabled (khuyến nghị)

![Điều hướng đến S3](/images/2/s3b3.png?featherlight=false&width=90pc)

   - **Block Public Access settings**: Giữ tất cả đều bị chặn (khuyến nghị)
   - **Bucket Versioning**: Kích hoạt
   - **Default encryption**: Server-side encryption với Amazon S3 managed keys (SSE-S3)
   - **Bucket Key**: Kích hoạt

![Điều hướng đến S3](/images/2/s3b4.png?featherlight=false&width=90pc)

4. Click **Create bucket**

![Điều hướng đến S3](/images/2/s3b5.png?featherlight=false&width=90pc)

5. Tạo bucket thứ hai cho báo cáo tuân thủ:
   - **Bucket name**: `identity-governance-reports`
   - **AWS Region**: Giống bucket đầu tiên
   - **Object Ownership**: ACLs disabled (khuyến nghị)
   - **Block Public Access settings**: Giữ tất cả đều bị chặn (khuyến nghị)
   - **Bucket Versioning**: Kích hoạt
   - **Default encryption**: Server-side encryption với Amazon S3 managed keys (SSE-S3)
   - **Bucket Key**: Kích hoạt
   - **Object Lock**: Kích hoạt cho compliance retention
6. Click **Create bucket**

![Điều hướng đến S3](/images/2/s3b6.png?featherlight=false&width=90pc)

7. Xác minh cả hai bucket đã được tạo thành công:

![Điều hướng đến S3](/images/2/s3b7.png?featherlight=false&width=90pc)


## Chuẩn bị Infrastructure

### 1. Kích hoạt AWS CloudTrail

1. Điều hướng đến dịch vụ **CloudTrail** trong AWS Console
2. Click **Create trail**

![Mở CloudTrail Console](/images/2/cloudtrailb1.png?featherlight=false&width=90pc)

![Mở CloudTrail Console](/images/2/cloudtrailb2.png?featherlight=false&width=90pc)

#### Bước 1: General details

3. Nhập thông tin cơ bản:
   - **Trail name**: `IdentityGovernanceTrail`
   - **Enable for all accounts in my organization**: Để trống (unchecked)

![Mở CloudTrail Console](/images/2/cloudtrailb3.png?featherlight=false&width=90pc)

#### Bước 2: S3 bucket configuration

4. Cấu hình S3 storage:
   - **Create new S3 bucket**: Chọn option này (ĐỂ TRỐNG - KHÔNG chọn "Use existing S3 bucket")
   - **S3 bucket name**: CloudTrail sẽ tự động tạo tên (ví dụ: aws-cloudtrail-logs-123456789012-abc12345)

5. Cấu hình bảo mật:
   - **Log file SSE-KMS encryption**: Unchecked (giữ mặc định)
   - **Log file validation**: Checked (khuyến nghị)

#### Bước 3: CloudWatch Logs (Optional)

6. CloudWatch Logs configuration:
   - **CloudWatch Logs**: Unchecked (bỏ qua cho bây giờ)

7. Click **Next**
#### Bước 4: Choose log events
8. Chọn loại events để log:
   - **Management events**: Checked
     - **Read**: Checked
     - **Write**: Checked
   - **Data events**: Unchecked (bỏ qua)
   - **Insight events**: Unchecked (bỏ qua)
9. Click **Next**

#### Bước 5: Review and create

10. Kiểm tra lại cấu hình:
    - Xác nhận trail name
    - Xác nhận S3 bucket sẽ được tạo mới
    - Xác nhận management events được bật
11. Click **Create trail**
**QUAN TRỌNG**: 
- **KHÔNG BAO GIỜ** chọn bucket `identity-governance-analytics` hoặc bất kỳ bucket nào bạn đã tạo trước đó
- **BẮT BUỘC** chọn "Create new S3 bucket" để CloudTrail tự tạo bucket riêng
- CloudTrail sẽ tự động cấu hình bucket policy đúng, tránh lỗi `InsufficientS3BucketPolicyException`

![Mở CloudTrail Console](/images/2/cloudtrailb4.png?featherlight=false&width=90pc)

### 2. Kích hoạt AWS Security Hub

1. Điều hướng đến dịch vụ **AWS Security Hub** trong AWS Console

![Security Hub Onboard Page](/images/2/secuHub1.png?featherlight=false&width=90pc)

2. Bạn sẽ thấy trang **Security Hub Onboard**

#### Bước 1: Configure Security Hub
3. Trong phần **Configure Security Hub**:
   - Đọc thông tin về Service Linked Roles (SLRs)
   - Để mặc định các cài đặt
#### Bước 2: Delegated Administrator Account
4. Trong phần **Delegated administrator account**:
   - Chọn **Do not select an account** (cho single account setup)
#### Bước 3: Account Enablement
5. Trong phần **Account enablement**:
   - ☑️ **Enable Security Hub for my account** (giữ checked)
#### Bước 4: Delegated Administrator Policy
6. Trong phần **Delegated administrator policy**:
   - Đọc policy details
   - Giữ cài đặt mặc định
7. Click **Onboard** ở cuối trang

![Security Hub Onboard Page](/images/2/secuHub2.png?featherlight=false&width=90pc)

#### Bước 5: Xác minh kích hoạt thành công
8. Sau khi onboard thành công, bạn sẽ thấy Security Hub dashboard:
   - **Security score** hiển thị
   - **Findings** bắt đầu được thu thập
   - **Standards** tự động kích hoạt

![Security Hub Onboard Page](/images/2/secuHub3.png?featherlight=false&width=90pc)

### 3. Tạo DynamoDB Tables

1. Điều hướng đến dịch vụ **DynamoDB**

![DynamoDB Console](/images/2/dynab1.png?featherlight=false&width=90pc)

2. Click **Create table**
3. Tạo bảng đầu tiên:
   - **Table name**: `AccessCertifications`
   - **Partition key**: `UserId` (String)
   - **Sort key**: `CertificationDate` (String)
   - **Billing mode**: On-demand

![DynamoDB Console](/images/2/dynab2.png?featherlight=false&width=90pc)

4. Click **Create table**

5. Tạo bảng thứ hai:
   - **Table name**: `RiskAssessments`
   - **Partition key**: `AssessmentId` (String)
   - **Billing mode**: On-demand
6. Click **Create table**

![DynamoDB Console](/images/2/dynab3.png?featherlight=false&width=90pc)

### 4. Tạo IAM Roles cần thiết

1. Điều hướng đến dịch vụ **IAM**
2. Click **Roles** trong sidebar

![DynamoDB Console](/images/2/IAMb1.png?featherlight=false&width=90pc)

3. Click **Create role**
4. Tạo role cho Lambda:
   - **Trusted entity**: AWS service
   - **Service**: Lambda
   - **Role name**: `IdentityGovernanceLambdaRole`
   - **Policies**: Attach `AWSLambdaBasicExecutionRole`

![DynamoDB Console](/images/2/IAMb2.png?featherlight=false&width=90pc)

## Xác thực thiết lập

### 1. Kiểm tra các dịch vụ đã kích hoạt

1. **CloudTrail**: Vào CloudTrail console, xác nhận trail đã được tạo và đang hoạt động
2. **S3**: Vào S3 console, xác nhận 3 bucket đã được tạo (2 bucket của bạn + 1 CloudTrail bucket)
3. **Security Hub**: Vào Security Hub console, xác nhận service đã được kích hoạt và có security score
4. **DynamoDB**: Vào DynamoDB console, xác nhận 2 table đã được tạo
5. **IAM**: Vào IAM console, xác nhận Lambda role đã được tạo

![Services Verification](/images/2/services-verification.png?featherlight=false&width=90pc)

### 2. Kiểm tra quyền truy cập

1. Vào **IAM** console
2. Click **Users** và xác nhận user hiện tại có quyền cần thiết
3. Click **Roles** và xác nhận các role đã được tạo

![Permissions Check](/images/2/permissions-check.png?featherlight=false&width=90pc)

## Kết quả Mong đợi

Sau khi hoàn thành các bước chuẩn bị:

- ✅ AWS Account được cấu hình đúng
- ✅ Các AWS services cần thiết đã được enable
- ✅ Base infrastructure đã được deploy
- ✅ Permissions đã được validate
- ✅ Workshop materials đã sẵn sàng

## Tiếp theo

Chuyển sang [3. Thiết lập Access Governance](../3-thiet-lap-access-governance) để bắt đầu triển khai hệ thống.