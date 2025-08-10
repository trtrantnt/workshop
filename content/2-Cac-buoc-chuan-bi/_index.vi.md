---
title: "2. Các bước chuẩn bị"
chapter: false
weight: 2
---

## Yêu cầu Tiên quyết

### 1. AWS Account
- AWS Account với quyền Administrator
- Quyền tạo và quản lý AWS Organizations
- Quyền sử dụng IAM Identity Center

### 2. Kiến thức cần thiết
- Hiểu biết cơ bản về AWS IAM
- Kinh nghiệm với AWS Organizations
- Kiến thức về compliance frameworks (SOX, SOC2, ISO27001)
- Hiểu biết về Python và AWS CLI

### 3. Yêu cầu truy cập
- Trình duyệt web (Chrome, Firefox, Safari, hoặc Edge)
- Kết nối internet ổn định
- Quyền truy cập AWS Console
- Quyền quản trị trong AWS account

## Thiết lập môi trường

### 1. Truy cập AWS Console

1. Mở trình duyệt web và truy cập [AWS Console](https://console.aws.amazon.com/)
2. Đăng nhập bằng thông tin tài khoản AWS của bạn
3. Đảm bảo bạn có quyền Administrator

![AWS Console Login](images/aws-console-login.png)

### 2. Xác minh quyền tài khoản

1. Trong AWS Console, click vào tên tài khoản ở góc trên bên phải
2. Chọn **My Account** để xem chi tiết tài khoản
3. Xác minh bạn có quyền truy cập:
   - AWS Organizations
   - IAM Identity Center
   - Quyền quản trị

![Account Verification](images/account-verification.png)

### 3. Tạo S3 Bucket cho lưu trữ dữ liệu

1. Điều hướng đến dịch vụ **S3** trong AWS Console
2. Click **Create bucket**
3. Tạo bucket đầu tiên:
   - **Bucket name**: `privilege-analytics-[YOUR-ACCOUNT-ID]`
   - **Region**: Chọn region ưa thích
   - Giữ cài đặt mặc định
   - Click **Create bucket**

![S3 Bucket Creation 1](images/s3-bucket-analytics.png)

4. Tạo bucket thứ hai:
   - **Bucket name**: `compliance-reports-[YOUR-ACCOUNT-ID]`
   - **Region**: Giống bucket đầu tiên
   - Giữ cài đặt mặc định
   - Click **Create bucket**

![S3 Bucket Creation 2](images/s3-bucket-compliance.png)

## Chuẩn bị Infrastructure

### 1. Kích hoạt AWS CloudTrail

1. Điều hướng đến dịch vụ **CloudTrail**
2. Click **Create trail**
3. Cấu hình trail:
   - **Trail name**: `IdentityGovernanceTrail`
   - **S3 bucket**: Chọn bucket `privilege-analytics-[YOUR-ACCOUNT-ID]`
   - **Log file prefix**: `cloudtrail-logs/`
4. Click **Create trail**

![CloudTrail Setup](images/cloudtrail-setup.png)

### 2. Kích hoạt AWS Config

1. Điều hướng đến dịch vụ **AWS Config**
2. Click **Get started**
3. Cấu hình:
   - **Resource types**: Chọn **All resources**
   - **S3 bucket**: Tạo bucket mới hoặc sử dụng có sẵn
   - **SNS topic**: Tạo topic mới
4. Click **Next** và **Confirm**

![Config Setup](images/config-setup.png)

### 3. Kích hoạt Amazon GuardDuty

1. Điều hướng đến dịch vụ **GuardDuty**
2. Click **Get Started**
3. Click **Enable GuardDuty**
4. Xem lại quyền dịch vụ và click **Enable**

![GuardDuty Setup](images/guardduty-setup.png)

### 4. Tạo IAM Roles cần thiết

1. Điều hướng đến dịch vụ **IAM**
2. Click **Roles** trong sidebar
3. Click **Create role**
4. Tạo role cho Lambda:
   - **Trusted entity**: AWS service
   - **Service**: Lambda
   - **Role name**: `IdentityGovernanceLambdaRole`
   - **Policies**: Attach `AWSLambdaBasicExecutionRole`

![IAM Role Creation](images/iam-role-lambda.png)

### 5. Tạo DynamoDB Tables

1. Điều hướng đến dịch vụ **DynamoDB**
2. Click **Create table**
3. Tạo bảng đầu tiên:
   - **Table name**: `OperationsLog`
   - **Partition key**: `operation_id` (String)
   - **Billing mode**: On-demand
4. Click **Create table**

![DynamoDB Table 1](images/dynamodb-operations-log.png)

5. Tạo bảng thứ hai:
   - **Table name**: `CertificationTasks`
   - **Partition key**: `task_id` (String)
   - **Billing mode**: On-demand
6. Click **Create table**

![DynamoDB Table 2](images/dynamodb-certification-tasks.png)

## Xác thực thiết lập

### 1. Kiểm tra các dịch vụ đã kích hoạt

1. **CloudTrail**: Vào CloudTrail console, xác nhận trail đã được tạo và đang hoạt động
2. **S3**: Vào S3 console, xác nhận 2 bucket đã được tạo
3. **Config**: Vào Config console, xác nhận service đang ghi lại resources
4. **GuardDuty**: Vào GuardDuty console, xác nhận detector đang hoạt động
5. **DynamoDB**: Vào DynamoDB console, xác nhận 2 table đã được tạo

![Services Verification](images/services-verification.png)

### 2. Kiểm tra quyền truy cập

1. Vào **IAM** console
2. Click **Users** và xác nhận user hiện tại có quyền cần thiết
3. Click **Roles** và xác nhận các role đã được tạo
4. Kiểm tra **Organizations** service để đảm bảo có quyền quản lý

![Permissions Check](images/permissions-check.png)

## Kết quả Mong đợi

Sau khi hoàn thành các bước chuẩn bị:

- ✅ AWS Account được cấu hình đúng
- ✅ Các AWS services cần thiết đã được enable
- ✅ Base infrastructure đã được deploy
- ✅ Permissions đã được validate
- ✅ Workshop materials đã sẵn sàng

## Tiếp theo

Chuyển sang [3. Thiết lập Access Governance](../3-thiet-lap-access-governance) để bắt đầu triển khai hệ thống.