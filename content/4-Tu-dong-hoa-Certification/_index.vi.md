---
title: "4. Tự động hóa Certification"
chapter: false
weight: 4
---

## Mục tiêu

Tự động hóa quy trình access certification để đảm bảo quyền truy cập được xem xét định kỳ và tuân thủ các yêu cầu bảo mật.

## Bước 1: Thiết lập EventBridge Scheduler

### 1.1 Tạo Scheduled Rule

1. Mở **Amazon EventBridge** trong AWS Console
2. Click **Rules** ở sidebar
3. Click **Create rule**

#### Bước 1: Define rule detail
4. Nhập thông tin rule:
   - **Name**: `AccessCertificationSchedule`
   - **Description**: `Quarterly access certification review`
   - **Event bus**: default
   - **Enable the rule on the selected event bus**: ✅ Checked

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb1.png?featherlight=false&width=90pc)

5. Trong **Rule type**, chọn **Schedule**
   - Chọn "A rule that runs on a schedule"
6. Click **Next**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb2.png?featherlight=false&width=90pc)

#### Bước 2: Define schedule
7. Trong **Occurrence**, chọn **Recurring schedule**
   - Chọn "Recurring schedule" vì chúng ta muốn chạy định kỳ hàng quý
8. Trong **Schedule pattern**, chọn **Rate-based schedule**
9. Nhập **90** và chọn **Days**
10. Click **Next**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb3.png?featherlight=false&width=90pc)

## Bước 2: Thiết lập Lambda Function

### 2.1 Tạo Lambda Function

1. Mở **AWS Lambda** trong console
2. Click **Create function**

![Tạo Lambda Function](/images/4/create-lambda-function.png?featherlight=false&width=90pc)

3. Chọn **Author from scratch**
4. Nhập thông tin function:
   - **Function name**: AccessCertificationTrigger
   - **Runtime**: Python 3.9
   - **Architecture**: x86_64

![Chi tiết Lambda Function](/images/4/lambda-function-details.png?featherlight=false&width=90pc)

5. Click **Create function**

### 2.2 Cấu hình Code cho Lambda Function

1. Trong tab **Code**, thay thế code mặc định
2. Upload logic code cho certification

![Lambda Code Editor](/images/4/lambda-code-editor.png?featherlight=false&width=90pc)

3. Click **Deploy** để lưu thay đổi

### 2.3 Thiết lập Environment Variables

1. Chuyển đến tab **Configuration**
2. Click **Environment variables**
3. Click **Edit**

![Biến Môi trường](/images/4/environment-variables.png?featherlight=false&width=90pc)

4. Thêm các biến cần thiết:
   - **S3_BUCKET**: certification-data-bucket
   - **SNS_TOPIC**: certification-notifications

![Thêm Biến Môi trường](/images/4/add-env-variables.png?featherlight=false&width=90pc)

## Bước 3: Thiết lập DynamoDB cho Certification Data

### 3.1 Tạo DynamoDB Table

1. Mở **Amazon DynamoDB** trong console
2. Click **Create table**

![Tạo DynamoDB Table](/images/4/create-dynamodb-table.png?featherlight=false&width=90pc)

3. Nhập thông tin table:
   - **Table name**: AccessCertifications
   - **Partition key**: UserId (String)
   - **Sort key**: CertificationDate (String)

![Chi tiết DynamoDB Table](/images/4/dynamodb-table-details.png?featherlight=false&width=90pc)

4. Click **Create table**

### 3.2 Cấu hình Lambda để ghi DynamoDB

1. Quay lại Lambda function **AccessCertificationTrigger**
2. Thêm DynamoDB permissions vào IAM role
3. Cập nhật code để ghi certification records

![Lambda DynamoDB Integration](/images/4/lambda-dynamodb-integration.png?featherlight=false&width=90pc)

## Bước 4: Kết nối EventBridge với Lambda

### 4.1 Thêm Lambda Target vào EventBridge Rule

#### Bước 3: Select target(s)
1. Trong **Target types**, chọn **AWS service**
2. Trong **Select a target**, chọn **Lambda function**
3. Trong **Function**, chọn **AccessCertificationTrigger**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb4.png?featherlight=false&width=90pc)

4. Click **Next**

#### Bước 4: Configure tags (Optional)
5. Bỏ qua phần tags, click **Next**

#### Bước 5: Review and create
6. Xem lại cấu hình:
   - Rule name: AccessCertificationSchedule
   - Schedule: Rate(90 days)
   - Target: Lambda function
7. Click **Create rule**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb5.png?featherlight=false&width=90pc)

## Bước 5: Kiểm tra Tự động hóa

### 5.1 Thực thi Kiểm tra Thủ công

1. Trong EventBridge, chọn rule của bạn
2. Click **Actions** → **Test rule**

![Kiểm tra EventBridge Rule](/images/4/test-eventbridge-rule.png?featherlight=false&width=90pc)

3. Giám sát thực thi Lambda function trong CloudWatch Logs

![CloudWatch Logs](/images/4/cloudwatch-logs.png?featherlight=false&width=90pc)

## Kết quả Mong đợi

Sau khi hoàn thành:

- ✅ Tự động hóa đánh giá truy cập hàng quý
- ✅ EventBridge scheduled triggers
- ✅ Lambda function xử lý
- ✅ Step Functions workflow orchestration
- ✅ Audit trail và giám sát

![Hoàn thành Tự động hóa Certification](/images/4/automation-complete.png?featherlight=false&width=90pc)

## Tiếp theo

Chuyển sang [5. Phân tích Đặc quyền](../5-phan-tich-dac-quyen) để thiết lập phân tích đặc quyền.