---
title: "3. Thiết lập Access Governance"
chapter: false
weight: 3
---

## Mục tiêu

Thiết lập nền tảng quản lý truy cập tập trung với AWS IAM Identity Center và IAM.

## Kiến trúc

![Kiến trúc Access Governance](/images/3/access-governance-architecture.png)

## Bước 1: Thiết lập IAM Foundation

### 1.1 Tạo IAM Groups

1. Điều hướng đến dịch vụ **IAM** trong AWS Console
2. Click **User groups** trong sidebar
3. Click **Create group**

![Tạo IAM Group](/images/3/create-iam-group.png)

4. Nhập thông tin group:
   - **Group name**: SecurityAuditors
   - **Description**: Security auditing team
5. Click **Create group**

![Chi tiết IAM Group](/images/3/iam-group-details.png)

### 1.2 Tạo IAM Policies

1. Click **Policies** trong sidebar
2. Click **Create policy**

![Tạo IAM Policy](/images/3/create-iam-policy.png)

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

4. Đặt tên policy: **SecurityAuditPolicy**
5. Click **Create policy**

![IAM Policy JSON](/images/3/iam-policy-json.png)

## Bước 2: Cấu hình IAM Identity Center

### 2.1 Kích hoạt IAM Identity Center

1. Tìm kiếm và mở **IAM Identity Center** trong AWS Console
2. Click **Enable** để kích hoạt IAM Identity Center

![Kích hoạt IAM Identity Center](/images/3/enable-identity-center.png)

3. Chọn region để lưu trữ identity store
4. Chọn **Use IAM Identity Center as my identity source**

![Chọn Identity Source](/images/3/choose-identity-source.png)

### 2.2 Tạo Permission Sets

1. Trong IAM Identity Center, click **Permission sets** ở sidebar
2. Click **Create permission set**

![Tạo Permission Set](/images/3/create-permission-set.png?featherlight=false&width=90pc)

3. Chọn **Predefined permission set**
4. Chọn **SecurityAudit** từ dropdown

![Chọn SecurityAudit](/images/3/select-security-audit.png?featherlight=false&width=90pc)

5. Nhập thông tin:
   - **Name**: SecurityAuditor
   - **Description**: Read-only access for security auditing
   - **Session duration**: 8 hours

![Chi tiết Permission Set](/images/3/permission-set-details.png?featherlight=false&width=90pc)

6. Click **Next** và **Create**

## Bước 3: Thiết lập Identity Store

### 3.1 Tạo Users và Groups

1. Trong IAM Identity Center, click **Users** ở sidebar
2. Click **Add user**

![Thêm User](/images/3/add-user.png?featherlight=false&width=90pc)

3. Nhập thông tin user:
   - **Username**: security-auditor
   - **Email**: auditor@company.com
   - **First name**: Security
   - **Last name**: Auditor

![Chi tiết User](/images/3/user-details.png?featherlight=false&width=90pc)

4. Click **Next** và **Add user**

### 3.2 Tạo Groups

1. Click **Groups** ở sidebar
2. Click **Create group**

![Tạo Group](/images/3/create-group.png?featherlight=false&width=90pc)

3. Nhập:
   - **Group name**: SecurityAuditors
   - **Description**: Security auditing team

![Chi tiết Group](/images/3/group-details.png?featherlight=false&width=90pc)

4. Click **Create group**

### 3.3 Gán Users vào Groups

1. Chọn group **SecurityAuditors**
2. Click **Add users to group**

![Thêm Users vào Group](/images/3/add-users-to-group.png?featherlight=false&width=90pc)

3. Chọn user **security-auditor**
4. Click **Add users**

![Chọn Users](/images/3/select-users.png?featherlight=false&width=90pc)

## Bước 4: Gán Quyền Truy cập

### 4.1 Gán Permission Sets cho Accounts

1. Click **AWS accounts** ở sidebar
2. Chọn account cần assign quyền
3. Click **Assign users or groups**

![Gán Quyền](/images/3/assign-access.png?featherlight=false&width=90pc)

4. Chọn **Groups** tab
5. Chọn group **SecurityAuditors**
6. Click **Next**

![Chọn Group](/images/3/select-group-assign.png?featherlight=false&width=90pc)

7. Chọn permission set **SecurityAuditor**
8. Click **Next** và **Submit**

![Chọn Permission Set](/images/3/select-permission-set-assign.png?featherlight=false&width=90pc)

## Kết quả Mong đợi

Sau khi hoàn thành bước này, bạn sẽ có:

- ✅ IAM Groups và Policies được cấu hình
- ✅ IAM Identity Center được kích hoạt
- ✅ Permission Sets cho các vai trò governance
- ✅ Identity Store với groups và users
- ✅ Các assignment quyền được cấu hình

![Thiết lập Hoàn tất](/images/3/final-setup.png)

## Tiếp theo

Chuyển sang [4. Tự động hóa Certification](../4-tu-dong-hoa-certification) để thiết lập quy trình tự động hóa certification.