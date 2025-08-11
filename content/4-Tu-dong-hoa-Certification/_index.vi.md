---
title: "4. Tự động hóa Certification"
chapter: false
weight: 4
---

## Mục tiêu

Tự động hóa quy trình access certification để đảm bảo quyền truy cập được xem xét định kỳ và tuân thủ các yêu cầu bảo mật.

## Bước 1: Xác minh DynamoDB Table

### 1.1 Kiểm tra Table đã tạo

1. Mở **Amazon DynamoDB** trong console
2. Xác minh table `AccessCertifications` đã được tạo trong chương 2
3. Table này sẽ được sử dụng để lưu certification data

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/dynamo1.png?featherlight=false&width=90pc)

## Bước 2: Tạo Lambda Function

### 2.1 Tạo Lambda Function

1. Mở **AWS Lambda** trong console
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
4. Click **Add permissions** → **Attach policies**
5. Tìm và attach policy **AmazonDynamoDBFullAccess**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/lambda4.png?featherlight=false&width=90pc)

## Bước 3: Thiết lập EventBridge Scheduler

### 3.1 Tạo Scheduled Rule

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
6. Click **Next**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb2.png?featherlight=false&width=90pc)

#### Bước 2: Define schedule
7. Trong **Occurrence**, chọn **Recurring schedule**
8. Trong **Schedule pattern**, chọn **Rate-based schedule**
9. Nhập **90** và chọn **Days**
10. Trong **Flexible time window**, nhập **15** minutes
11. Click **Next**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb3.png?featherlight=false&width=90pc)

#### Bước 3: Select target
12. Trong **Target API**, chọn **AWS Lambda**
13. Chọn API **Invoke**
14. Trong **Lambda function**, chọn **AccessCertificationTrigger**
15. Click **Next**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb4.png?featherlight=false&width=90pc)

#### Bước 4: Configure tags (Optional)
16. Bỏ qua phần tags, click **Next**

#### Bước 5: Review and create
17. Xem lại cấu hình và click **Create rule**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb5.png?featherlight=false&width=90pc)

## Bước 4: Kiểm tra Tự động hóa

### 4.1 Thực thi Kiểm tra Thủ công

1. Trong EventBridge, chọn rule của bạn
2. Click **Actions** → **Test rule**

![Kiểm tra EventBridge Rule](https://trtrantnt.github.io/workshop/images/4/test1.png?featherlight=false&width=90pc)

3. Giám sát thực thi Lambda function trong CloudWatch Logs

![CloudWatch Logs](https://trtrantnt.github.io/workshop/images/4/test2.png?featherlight=false&width=90pc)

## Kết quả Mong đợi

Sau khi hoàn thành:

- ✅ DynamoDB table để lưu certification data
- ✅ Lambda function xử lý certification logic
- ✅ EventBridge scheduled triggers hàng quý
- ✅ Tự động hóa đánh giá truy cập định kỳ
- ✅ Audit trail và giám sát

![Hoàn thành Tự động hóa Certification](https://trtrantnt.github.io/workshop/images/4/complete.png?featherlight=false&width=90pc)

## Tiếp theo

Chuyển sang [5. Phân tích Đặc quyền](../5-phan-tich-dac-quyen) để thiết lập phân tích đặc quyền.