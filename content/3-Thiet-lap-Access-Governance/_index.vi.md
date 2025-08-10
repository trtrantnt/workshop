---
title: "3. Thiết lập Access Governance"
chapter: false
weight: 3
---

# 3. Thiết lập Access Governance

## Mục tiêu

Thiết lập nền tảng quản lý truy cập tập trung với AWS IAM Identity Center và Organizations.

## Kiến trúc

```mermaid
graph LR
    A[Management Account] --> B[AWS Organizations]
    B --> C[Member Accounts]
    C --> D[IAM Identity Center]
    D --> E[Permission Sets]
    E --> F[User Assignments]
```

## Bước 1: Thiết lập AWS Organizations

### 1.1 Tạo Organization

```bash
aws organizations create-organization --feature-set ALL
```

### 1.2 Tạo Organizational Units

```json
{
  "Name": "Security",
  "ParentId": "r-xxxx"
}
```

```bash
aws organizations create-organizational-unit \
  --parent-id r-xxxx \
  --name "Security"
```

## Bước 2: Cấu hình IAM Identity Center

### 2.1 Enable IAM Identity Center

```bash
aws sso-admin create-instance \
  --name "IdentityGovernance" \
  --description "Identity Governance Instance"
```

### 2.2 Tạo Permission Sets

```json
{
  "Name": "SecurityAuditor",
  "Description": "Read-only access for security auditing",
  "SessionDuration": "PT8H",
  "ManagedPolicies": [
    "arn:aws:iam::aws:policy/SecurityAudit",
    "arn:aws:iam::aws:policy/ReadOnlyAccess"
  ]
}
```

## Bước 3: Thiết lập Identity Store

### 3.1 Cấu hình External Identity Provider

```python
import boto3

def setup_external_idp():
    client = boto3.client('identitystore')
    
    # Configure SAML IdP
    response = client.create_external_id_provider(
        IdentityStoreId='d-xxxxxxxxxx',
        ExternalIdProvider={
            'Type': 'SAML',
            'Configuration': {
                'MetadataDocument': 'metadata_content',
                'LoginUrl': 'https://idp.company.com/login',
                'LogoutUrl': 'https://idp.company.com/logout'
            }
        }
    )
    return response
```

## Kết quả Mong đợi

Sau khi hoàn thành bước này, bạn sẽ có:

- ✅ AWS Organizations được cấu hình với OUs
- ✅ IAM Identity Center được kích hoạt
- ✅ Permission Sets cho các vai trò governance
- ✅ Identity Store với groups và users
- ✅ Automation scripts cho quản lý

## Tiếp theo

Chuyển sang [4. Tự động hóa Certification](../4-tu-dong-hoa-certification) để thiết lập quy trình tự động hóa certification.