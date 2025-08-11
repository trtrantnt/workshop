---
title: "3. Thiết lập Access Governance"
chapter: false
weight: 3
---

## Mục tiêu

Thiết lập nền tảng quản lý truy cập tập trung với AWS IAM Identity Center và IAM.

## Bước 1: Thiết lập IAM Foundation

### 1.1 Tạo IAM Groups

1. Điều hướng đến dịch vụ **IAM** trong AWS Console
2. Click **User groups** trong sidebar
3. Click **Create group**
4. Nhập thông tin group:
   - **Group name**: `SecurityAuditors`
5. Click **Create group**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/3/iamgr1.png?featherlight=false&width=90pc)

### 1.2 Tạo IAM Policies

1. Click **Policies** trong sidebar
2. Click **Create policy**
3. Sử dụng JSON editor để tạo custom policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:Get*",
                "iam:List*",
                "iam:Generate*"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "cloudtrail:Get*",
                "cloudtrail:List*",
                "cloudtrail:Describe*"
            ],
            "Resource": "*"
        }
    ]
}
```

4. Đặt tên policy: `SecurityAuditPolicy`
5. Click **Create policy**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/3/policy1.png?featherlight=false&width=90pc)

## Bước 2: Cấu hình IAM Identity Center

### 2.1 Kích hoạt IAM Identity Center

1. Tìm kiếm và mở **IAM Identity Center** trong AWS Console
2. Click **Enable** để kích hoạt IAM Identity Center
3. Chọn region để lưu trữ identity store
4. Chọn **Use IAM Identity Center as my identity source**
### 2.2 Tạo Permission Sets
1. Trong IAM Identity Center, click **Permission sets** ở sidebar
2. Click **Create permission set**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/3/centerEn1.png?featherlight=false&width=90pc)

3. Chọn **Predefined permission set**
4. Chọn **SecurityAudit** từ dropdown

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/3/centerEn2.png?featherlight=false&width=90pc)

5. Nhập thông tin:
   - **Name**: `SecurityAuditor`
   - **Description**: `Read-only access for security auditing`
   - **Session duration**: 8 hours

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/3/centerEn3.png?featherlight=false&width=90pc)

6. Click **Next** và **Create**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/3/centerEn4.png?featherlight=false&width=90pc)

## Bước 3: Thiết lập Identity Store

### 3.1 Tạo Users và Groups

1. Trong IAM Identity Center, click **Users** ở sidebar
2. Click **Add user**

3. Nhập thông tin user:
   - **Username**: `security-auditor`
   - **Email**: auditor@company.com
   - **First name**: `Security`
   - **Last name**: `Auditor`
4. Click **Next** và **Add user**
![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/3/is1.png?featherlight=false&width=90pc)
### 3.2 Tạo Groups

1. Click **Groups** ở sidebar
2. Click **Create group**
3. Nhập:
   - **Group name**: `SecurityAuditors`
   - **Description**: `Security auditing team`


4. Add users to group
    - Chọn user `security-auditor`
5. Click **Create group**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/3/is2.png?featherlight=false&width=90pc)

## Bước 4: Gán Quyền Truy cập

### 4.1 Gán Permission Sets cho Accounts

1. Click **AWS accounts** ở sidebar
2. Chọn account cần assign quyền
3. Click **Assign users or groups**
4. Chọn **Groups** tab
5. Chọn group **SecurityAuditors**
6. Click **Next**

7. Chọn permission set **SecurityAuditor**
8. Click **Next** và **Submit**

![Điều hướng đến S3](https://trtrantnt.github.io/workshop/images/3/is3.png?featherlight=false&width=90pc)

## Kết quả Mong đợi

Sau khi hoàn thành bước này, bạn sẽ có:

- ✅ IAM Groups và Policies được cấu hình
- ✅ IAM Identity Center được kích hoạt
- ✅ Permission Sets cho các vai trò governance
- ✅ Identity Store với groups và users
- ✅ Các assignment quyền được cấu hình

## Tiếp theo

Chuyển sang [4. Tự động hóa Certification](../4-tu-dong-hoa-certification) để thiết lập quy trình tự động hóa certification.