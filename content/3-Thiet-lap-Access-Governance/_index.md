---
title: "3. Access Governance Setup"
chapter: false
weight: 3
---

## Objective

Set up centralized access management foundation with AWS IAM Identity Center and IAM.

## Architecture

![Access Governance Architecture](/images/3/access-governance-architecture.png)

## Step 1: IAM Foundation Setup

### 1.1 Create IAM Groups

1. Navigate to **IAM** service in AWS Console
2. Click **User groups** in the sidebar
3. Click **Create group**

![Create IAM Group](/images/3/create-iam-group.png)

4. Enter group details:
   - **Group name**: SecurityAuditors
   - **Description**: Security auditing team
5. Click **Create group**

![IAM Group Details](/images/3/iam-group-details.png)

### 1.2 Create IAM Policies

1. Click **Policies** in the sidebar
2. Click **Create policy**

![Create IAM Policy](/images/3/create-iam-policy.png)

3. Use JSON editor to create custom policy:

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

4. Name the policy: **SecurityAuditPolicy**
5. Click **Create policy**

![IAM Policy JSON](/images/3/iam-policy-json.png)

## Step 2: Configure IAM Identity Center

### 2.1 Enable IAM Identity Center

1. Search and open **IAM Identity Center** in AWS Console
2. Click **Enable** to activate IAM Identity Center

![Enable IAM Identity Center](/images/3/enable-identity-center.png)

3. Choose region to store identity store
4. Select **Use IAM Identity Center as my identity source**

![Choose Identity Source](/images/3/choose-identity-source.png)

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

- ✅ IAM Groups and Policies configured
- ✅ IAM Identity Center activated
- ✅ Permission Sets for governance roles
- ✅ Identity Store with groups and users
- ✅ Access assignments configured

![Final Setup](/images/3/final-setup.png)

## Next Steps

Continue to [4. Certification Automation](../4-tu-dong-hoa-certification) to set up automated certification processes.