---
title: "3. Access Governance Setup"
chapter: false
weight: 3
---

## Objective

Set up centralized access management foundation with AWS IAM Identity Center and IAM.

## Step 1: IAM Foundation Setup

### 1.1 Create IAM Groups

1. Navigate to **IAM** service in AWS Console
2. Click **User groups** in the sidebar
3. Click **Create group**
4. Enter group information:
   - **Group name**: `SecurityAuditors`
5. Click **Create group**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/3/iamgr1.png?featherlight=false&width=90pc)

### 1.2 Create IAM Policies

1. Click **Policies** in the sidebar
2. Click **Create policy**
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

4. Name the policy: `SecurityAuditPolicy`
5. Click **Create policy**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/3/policy1.png?featherlight=false&width=90pc)

## Step 2: Configure IAM Identity Center

### 2.1 Enable IAM Identity Center

1. Search and open **IAM Identity Center** in AWS Console
2. Click **Enable** to activate IAM Identity Center
3. Choose region to store identity store
4. Select **Use IAM Identity Center as my identity source**

### 2.2 Create Permission Sets

1. In IAM Identity Center, click **Permission sets** in the sidebar
2. Click **Create permission set**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/3/centerEn1.png?featherlight=false&width=90pc)

3. Select **Predefined permission set**
4. Choose **SecurityAudit** from dropdown

![Navigate to S3](https://trtrantnt.github.io/workshop/images/3/centerEn2.png?featherlight=false&width=90pc)

5. Enter information:
   - **Name**: `SecurityAuditor`
   - **Description**: `Read-only access for security auditing`
   - **Session duration**: 8 hours

![Navigate to S3](https://trtrantnt.github.io/workshop/images/3/centerEn3.png?featherlight=false&width=90pc)

6. Click **Next** and **Create**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/3/centerEn4.png?featherlight=false&width=90pc)

## Step 3: Identity Store Setup

### 3.1 Create Users and Groups

1. In IAM Identity Center, click **Users** in the sidebar
2. Click **Add user**

3. Enter user information:
   - **Username**: `security-auditor`
   - **Email**: auditor@company.com
   - **First name**: `Security`
   - **Last name**: `Auditor`
4. Click **Next** and **Add user**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/3/is1.png?featherlight=false&width=90pc)

### 3.2 Create Groups

1. Click **Groups** in the sidebar
2. Click **Create group**
3. Enter:
   - **Group name**: `SecurityAuditors`
   - **Description**: `Security auditing team`

4. Add users to group
    - Select user `security-auditor`
5. Click **Create group**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/3/is2.png?featherlight=false&width=90pc)

## Step 4: Assign Access

### 4.1 Assign Permission Sets to Accounts

1. Click **AWS accounts** in the sidebar
2. Select the account to assign permissions
3. Click **Assign users or groups**
4. Select **Groups** tab
5. Select group **SecurityAuditors**
6. Click **Next**

7. Select permission set **SecurityAuditor**
8. Click **Next** and **Submit**

![Navigate to S3](https://trtrantnt.github.io/workshop/images/3/is3.png?featherlight=false&width=90pc)

## Expected Results

After completing this step, you will have:

- ✅ IAM Groups and Policies configured
- ✅ IAM Identity Center activated
- ✅ Permission Sets for governance roles
- ✅ Identity Store with groups and users
- ✅ Access assignments configured

## Next Steps

Continue to [4. Certification Automation](../4-tu-dong-hoa-certification) to set up automated certification processes.