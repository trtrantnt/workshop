---
title: "9. Audit Procedures"
chapter: false
weight: 9
---

# 9. Audit Procedures

## Objective

Establish comprehensive audit procedures to ensure compliance with security, legal, and internal requirements in identity governance management.

## Audit Architecture

```mermaid
graph TB
    A[Audit Planning] --> B[Evidence Collection]
    B --> C[Analysis & Testing]
    C --> D[Findings Documentation]
    D --> E[Remediation Tracking]
    E --> F[Audit Reporting]
    
    G[Continuous Monitoring] --> H[Automated Controls Testing]
    H --> I[Exception Reporting]
    I --> J[Management Dashboard]
```

## Step 1: AWS Audit Manager Setup

### 1.1 Enable AWS Audit Manager

1. Open **AWS Audit Manager** in the console
2. Click **Get started**

![Enable Audit Manager](/images/9/enable-audit-manager.png?featherlight=false&width=90pc)

3. Configure service settings:
   - **Enable data collection**: Yes
   - **KMS encryption**: Use default key
   - **S3 bucket**: Create new bucket for audit evidence

![Audit Manager Settings](/images/9/audit-manager-settings.png?featherlight=false&width=90pc)

4. Click **Complete setup**

### 1.2 Create Audit Framework

1. Click **Assessment frameworks** in sidebar
2. Click **Create custom framework**

![Create Audit Framework](/images/9/create-audit-framework.png?featherlight=false&width=90pc)

3. Configure framework:
   - **Name**: Identity Governance Audit Framework
   - **Description**: Comprehensive identity governance audit
   - **Compliance type**: SOX, SOC2, ISO27001

![Framework Configuration](/images/9/framework-configuration.png?featherlight=false&width=90pc)

4. Add control sets for:
   - **Access Management**
   - **Privilege Governance**
   - **Compliance Monitoring**

![Control Sets](/images/9/control-sets.png?featherlight=false&width=90pc)

## Step 2: Create Assessment

### 2.1 Start New Assessment

1. Click **Assessments** in Audit Manager
2. Click **Create assessment**

![Create Assessment](/images/9/create-assessment.png?featherlight=false&width=90pc)

3. Configure assessment:
   - **Name**: Q1 Identity Governance Audit
   - **Framework**: Identity Governance Audit Framework
   - **Scope**: Select AWS accounts and services

![Assessment Configuration](/images/9/assessment-configuration.png?featherlight=false&width=90pc)

4. Assign roles:
   - **Assessment owner**: Security team lead
   - **Reviewers**: Compliance team

![Assessment Roles](/images/9/assessment-roles.png?featherlight=false&width=90pc)

### 2.2 Configure Evidence Collection

1. Set up **automatic evidence collection**:
   - **AWS Config**: Compliance data
   - **CloudTrail**: Activity logs
   - **Security Hub**: Security findings

![Evidence Collection](/images/9/evidence-collection.png?featherlight=false&width=90pc)

2. Configure **manual evidence** upload process

![Manual Evidence](/images/9/manual-evidence.png?featherlight=false&width=90pc)

## Step 3: Control Testing and Evidence Review

### 3.1 Review Control Evidence

1. Navigate to **Control sets** in your assessment
2. Review each control:
   - **Evidence status**
   - **Compliance rating**
   - **Comments and findings**

![Control Evidence Review](/images/9/control-evidence-review.png?featherlight=false&width=90pc)

3. Upload additional evidence if needed
4. Add control testing notes

![Control Testing Notes](/images/9/control-testing-notes.png?featherlight=false&width=90pc)

### 3.2 Document Findings

1. For each control, document:
   - **Testing procedures performed**
   - **Results and observations**
   - **Exceptions or deficiencies**
   - **Recommendations**

![Document Findings](/images/9/document-findings.png?featherlight=false&width=90pc)

2. Assign **risk ratings** to findings
3. Set **remediation timelines**

![Risk Ratings](/images/9/risk-ratings.png?featherlight=false&width=90pc)

## Step 4: Generate Audit Reports

### 4.1 Create Assessment Report

1. Click **Generate assessment report**
2. Select report format: **PDF** or **HTML**

![Generate Report](/images/9/generate-report.png?featherlight=false&width=90pc)

3. Configure report sections:
   - **Executive summary**
   - **Control testing results**
   - **Findings and recommendations**
   - **Evidence appendix**

![Report Sections](/images/9/report-sections.png?featherlight=false&width=90pc)

### 4.2 Share Report with Stakeholders

1. Download completed report
2. Share with:
   - **Executive leadership**
   - **Audit committee**
   - **Compliance team**

![Share Report](/images/9/share-report.png?featherlight=false&width=90pc)

## Expected Results

After completion:

- ✅ AWS Audit Manager configured
- ✅ Custom audit framework created
- ✅ Assessment with evidence collection
- ✅ Control testing documentation
- ✅ Comprehensive audit reports
- ✅ Stakeholder communication process

![Audit Procedures Complete](/images/9/audit-procedures-complete.png?featherlight=false&width=90pc)

## Next Steps

Continue to [10. Compliance Validation](../10-xac-thuc-tuan-thu) to complete the workshop.