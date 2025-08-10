---
title: "5. Privilege Analytics"
chapter: false
weight: 5
---

# 5. Privilege Analytics

## Objective

Analyze and monitor privilege usage to detect security risks, excessive permissions, and anomalous patterns.

## Analytics Architecture

```mermaid
graph TB
    A[CloudTrail Logs] --> B[S3 Data Lake]
    C[Config Data] --> B
    D[IAM Data] --> B
    B --> E[Athena Queries]
    E --> F[Analytics Engine]
    F --> G[Risk Scoring]
    G --> H[QuickSight Dashboard]
    F --> I[Anomaly Detection]
    I --> J[Alerts]
```

## Step 1: Data Collection Setup

### 1.1 CloudTrail Configuration

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Privilege Analytics Data Collection'

Resources:
  PrivilegeAnalyticsTrail:
    Type: AWS::CloudTrail::Trail
    Properties:
      TrailName: PrivilegeAnalyticsTrail
      S3BucketName: !Ref AnalyticsDataBucket
      S3KeyPrefix: 'cloudtrail-logs/'
      IncludeGlobalServiceEvents: true
      IsMultiRegionTrail: true
      EnableLogFileValidation: true
```

## Step 2: Athena Queries for Analytics

### 2.1 Create Athena Tables

```sql
-- Create table for privilege analytics data
CREATE EXTERNAL TABLE privilege_analytics (
  collection_timestamp string,
  iam_data struct<
    users: array<struct<
      username: string,
      user_id: string,
      arn: string,
      create_date: string,
      risk_score: double
    >>
  >
)
STORED AS JSON
LOCATION 's3://privilege-analytics-REGION/analytics-data/'
```

### 2.2 Analytics Queries

```sql
-- Query 1: High-risk users
SELECT 
  user.username,
  user.risk_score,
  cardinality(user.attached_policies) as policy_count,
  user.last_activity
FROM privilege_analytics
CROSS JOIN UNNEST(iam_data.users) AS t(user)
WHERE user.risk_score > 7.0
ORDER BY user.risk_score DESC;
```

## Step 3: Risk Scoring Engine

### 3.1 Risk Calculation Lambda

```python
import boto3
import json
import math
from datetime import datetime, timedelta

class RiskScoringEngine:
    def __init__(self):
        self.weights = {
            'privilege_level': 0.3,
            'usage_frequency': 0.2,
            'last_activity': 0.2,
            'policy_violations': 0.15,
            'external_access': 0.15
        }
    
    def calculate_user_risk_score(self, user_data, policies, groups, activity):
        """Calculate risk score for a user (0-10 scale)"""
        
        scores = {
            'privilege_level': self.score_privilege_level(policies, groups),
            'usage_frequency': self.score_usage_frequency(activity),
            'last_activity': self.score_last_activity(user_data.get('PasswordLastUsed')),
            'policy_violations': self.score_policy_violations(policies),
            'external_access': self.score_external_access(activity)
        }
        
        # Calculate weighted score
        total_score = sum(scores[factor] * self.weights[factor] for factor in scores)
        
        return {
            'total_score': round(total_score, 2),
            'factor_scores': scores,
            'risk_level': self.get_risk_level(total_score)
        }
```

## Expected Results

After completion:

- ✅ Automated privilege data collection
- ✅ Risk scoring for users and roles
- ✅ Athena queries for analytics
- ✅ QuickSight dashboard for visualization
- ✅ Anomaly detection and alerting

## Next Steps

Continue to [6. Risk Assessment](../6-danh-gia-rui-ro) to set up comprehensive risk assessment.