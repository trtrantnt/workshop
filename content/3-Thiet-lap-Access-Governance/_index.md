---
title: "3. Access Governance Setup"
chapter: false
weight: 3
---

## Objective

Set up centralized access management foundation with AWS IAM Identity Center and Organizations.

## Architecture

```mermaid
graph LR
    A[Management Account] --> B[AWS Organizations]
    B --> C[Member Accounts]
    C --> D[IAM Identity Center]
    D --> E[Permission Sets]
    E --> F[User Assignments]
```

## Step 1: AWS Organizations Setup

### 1.1 Create Organization

1. Đăng nhập vào AWS Console với tài khoản root
2. Tìm kiếm và mở service **AWS Organizations**
3. Click **Create organization**

![Create Organization](/images/3/create-organization.png?featherlight=false&width=90pc)

4. Chọn **Enable all features** để có đầy đủ tính năng quản lý
5. Click **Create organization**

![Enable All Features](/images/3/enable-all-features.png?featherlight=false&width=90pc)

### 1.2 Create Organizational Units

1. Trong AWS Organizations console, click **AWS accounts** ở sidebar
2. Click **Actions** → **Create organizational unit**

![Create OU](/images/3/create-ou.png?featherlight=false&width=90pc)

3. Nhập tên OU: **Security**
4. Click **Create organizational unit**

![OU Name](/images/3/ou-name.png?featherlight=false&width=90pc)

## Step 2: Configure IAM Identity Center

### 2.1 Enable IAM Identity Center

1. Tìm kiếm và mở **IAM Identity Center** trong AWS Console
2. Click **Enable** để kích hoạt IAM Identity Center

![Enable IAM Identity Center](/images/3/enable-identity-center.png?featherlight=false&width=90pc)

3. Chọn region để lưu trữ identity store
4. Click **Create AWS organization** nếu chưa có

![Choose Region](/images/3/choose-region.png?featherlight=false&width=90pc)

### 2.2 Create Permission Sets

1. Trong IAM Identity Center, click **Permission sets** ở sidebar
2. Click **Create permission set**

![Create Permission Set](/images/3/create-permission-set.png?featherlight=false&width=90pc)

3. Chọn **Predefined permission set**
4. Chọn **SecurityAudit** từ dropdown

![Select SecurityAudit](/images/3/select-security-audit.png?featherlight=false&width=90pc)

5. Nhập thông tin:
   - **Name**: SecurityAuditor
   - **Description**: Read-only access for security auditing
   - **Session duration**: 8 hours

![Permission Set Details](/images/3/permission-set-details.png?featherlight=false&width=90pc)

6. Click **Next** và **Create**

## Step 3: Identity Store Setup

### 3.1 Create Users and Groups

1. Trong IAM Identity Center, click **Users** ở sidebar
2. Click **Add user**

![Add User](/images/3/add-user.png?featherlight=false&width=90pc)

3. Nhập thông tin user:
   - **Username**: security-auditor
   - **Email**: auditor@company.com
   - **First name**: Security
   - **Last name**: Auditor

![User Details](/images/3/user-details.png?featherlight=false&width=90pc)

4. Click **Next** và **Add user**

### 3.2 Create Groups

1. Click **Groups** ở sidebar
2. Click **Create group**

![Create Group](/images/3/create-group.png?featherlight=false&width=90pc)

3. Nhập:
   - **Group name**: SecurityAuditors
   - **Description**: Security auditing team

![Group Details](/images/3/group-details.png?featherlight=false&width=90pc)

4. Click **Create group**

### 3.3 Assign Users to Groups

1. Chọn group **SecurityAuditors**
2. Click **Add users to group**

![Add Users to Group](/images/3/add-users-to-group.png?featherlight=false&width=90pc)

3. Chọn user **security-auditor**
4. Click **Add users**

![Select Users](/images/3/select-users.png?featherlight=false&width=90pc)

## Step 4: Assign Access

### 4.1 Assign Permission Sets to Accounts

1. Click **AWS accounts** ở sidebar
2. Chọn account cần assign quyền
3. Click **Assign users or groups**

![Assign Access](/images/3/assign-access.png?featherlight=false&width=90pc)

4. Chọn **Groups** tab
5. Chọn group **SecurityAuditors**
6. Click **Next**

![Select Group](/images/3/select-group-assign.png?featherlight=false&width=90pc)

7. Chọn permission set **SecurityAuditor**
8. Click **Next** và **Submit**

![Select Permission Set](/images/3/select-permission-set-assign.png?featherlight=false&width=90pc)

## Expected Results

After completing this step, you will have:

- ✅ AWS Organizations configured with OUs
- ✅ IAM Identity Center activated
- ✅ Permission Sets for governance roles
- ✅ Identity Store with groups and users
- ✅ Access assignments configured

![Final Setup](/images/3/final-setup.png?featherlight=false&width=90pc)

## Next Steps

Continue to [4. Certification Automation](../4-tu-dong-hoa-certification) to set up automated certification processes.