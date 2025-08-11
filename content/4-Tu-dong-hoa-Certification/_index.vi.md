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
   - **Enable the rule on the selected event bus**

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
10. Trong **Flexible time window**, nhập **15** minutes
    - Cho phép schedule chạy trong vòng 15 phút sau thời gian bắt đầu
    - Giúp giảm tải hệ thống và tăng tính linh hoạt
11. Click **Next**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb3.png?featherlight=false&width=90pc)

#### Bước 3: Select target
12. Trong **Target API**, chọn **AWS Lambda**
13. Chọn API **Invoke**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb4.png?featherlight=false&width=90pc)

14. Bây giờ chúng ta cần tạo Lambda function trước. Click **Create a new Lambda function** hoặc mở tab mới để tạo Lambda function.

## Bước 2: Tạo Lambda Function cho EventBridge

### 2.1 Tạo Lambda Function

1. Mở tab mới và đi đến **AWS Lambda** trong console
2. Click **Create function**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/lambda1.png?featherlight=false&width=90pc)

3. Chọn **Author from scratch**
4. Nhập thông tin function:
   - **Function name**: `AccessCertificationTrigger`
   - **Runtime**: Python 3.9
   - **Architecture**: x86_64

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/lambda2.png?featherlight=false&width=90pc)

5. Click **Create function**

### 2.2 Cấu hình Code cho Lambda Function

1. Trong tab **Code**, thay thế code mặc định bằng code sau:

```python
import json
import boto3
from datetime import datetime

def lambda_handler(event, context):
    print("Access Certification Trigger Started")
    
    # Initialize AWS clients
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('AccessCertifications')
    
    # Create certification record
    response = table.put_item(
        Item={
            'UserId': 'system',
            'CertificationDate': datetime.now().isoformat(),
            'Status': 'Triggered',
            'Type': 'Quarterly Review'
        }
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps('Certification process triggered successfully')
    }
```

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/lambda3.png?featherlight=false&width=90pc)

2. Click **Deploy** để lưu thay đổi

### 2.3 Cấu hình IAM Role cho Lambda

1. Chuyển đến tab **Configuration**
2. Click **Permissions**
3. Click vào role name để mở IAM console
4. Thêm policy cho DynamoDB access

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/lambda4.png?featherlight=false&width=90pc)

## Bước 3: Hoàn thành EventBridge Configuration

### 3.1 Quay lại EventBridge và chọn Lambda function

1. Quay lại tab EventBridge
2. Trong **Lambda function**, chọn **AccessCertificationTrigger**

3. Click **Next**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb5.png?featherlight=false&width=90pc)

#### Bước 4: Configure tags (Optional)
4. Bỏ qua phần tags, click **Next**

#### Bước 5: Review and create
5. Xem lại cấu hình:
   - Rule name: AccessCertificationSchedule
   - Schedule: Rate(90 days)
   - Target: Lambda function (AccessCertificationTrigger)
6. Click **Create rule**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb6.png?featherlight=false&width=90pc)

## Bước 4: Kiểm tra Tự động hóa

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