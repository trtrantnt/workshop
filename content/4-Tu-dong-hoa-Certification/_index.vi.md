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


## Bước 2: Tạo Lambda Function

### 2.1 Tạo Lambda Function

1. Mở **AWS Lambda** trong console
2. Click **Create function**
3. Chọn **Author from scratch**
4. Nhập thông tin function:
   - **Function name**: `AccessCertificationTrigger`
   - **Runtime**: Python 3.9
   - **Architecture**: x86_64

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/ld1.png?featherlight=false&width=90pc)

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

2. Click **Deploy** để lưu thay đổi

### 2.3 Cấu hình IAM Role cho Lambda
1. Chuyển đến tab **Configuration**
![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/ld2.png?featherlight=false&width=90pc)
2. Click **Permissions**
3. Click vào role name để mở IAM console
4. Click **Add permissions** → **Attach policies**
![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/ld3.png?featherlight=false&width=90pc)
5. Tìm và attach policy **AmazonDynamoDBFullAccess**
6. Chọn **Add permissions**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/ld4.png?featherlight=false&width=90pc)

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
   - **Enable the rule on the selected event bus**



5. Trong **Rule type**, chọn **Schedule**
6. Click **Next**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb1.png?featherlight=false&width=90pc)

#### Bước 2: Define schedule
7. Trong **Occurrence**, chọn **Recurring schedule**
8. Trong **Schedule pattern**, chọn **Rate-based schedule**
9. Nhập **90** và chọn **Days**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb2.png?featherlight=false&width=90pc)
10. Trong **Flexible time window**, nhập **15** minutes
11. Click **Next**



#### Bước 3: Select target
12. Trong **Target API**, chọn **AWS Lambda Invoke**
14. Trong **Lambda function**, chọn **AccessCertificationTrigger**
15. Click **Next**
![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb3.png?featherlight=false&width=90pc)

#### Bước 4: Configure tags (Optional)
16. Bỏ qua phần tags, click **Next**

#### Bước 5: Review and create
17. Xem lại cấu hình và click **Create rule**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/4/eb4.png?featherlight=false&width=90pc)
## Bước 4: Kiểm tra Tự động hóa

### 4.1 Kiểm tra EventBridge Schedule

1. Trong **Amazon EventBridge** console
2. Click **Schedules** ở sidebar (không phải Rules)
3. Xác minh schedule **AccessCertificationSchedule** đã được tạo và đang **Enabled**


### 4.2 Test Lambda Function thủ công

1. Vào **AWS Lambda** console
2. Chọn function **AccessCertificationTrigger**
3. Click **Test** để tạo test event
4. Sử dụng default test event và click **Test**
5. Kiểm tra kết quả thực thi

### 4.3 Xác minh DynamoDB Record

1. Vào **Amazon DynamoDB** console
2. Chọn table **AccessCertifications**
3. Click **Explore table items**
4. Xác minh có record mới được tạo bởi Lambda function


## Kết quả Mong đợi

Sau khi hoàn thành:

- ✅ DynamoDB table để lưu certification data
- ✅ Lambda function xử lý certification logic
- ✅ EventBridge scheduled triggers hàng quý
- ✅ Tự động hóa đánh giá truy cập định kỳ
- ✅ Audit trail và giám sát


## Tiếp theo

Chuyển sang [5. Phân tích Đặc quyền](../5-phan-tich-dac-quyen) để thiết lập phân tích đặc quyền.