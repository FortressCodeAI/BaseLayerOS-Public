# AWS IAM Integration Guide

Integrating AWS IAM with BaseLayerOS Identity Adapter Layer (IAL)

## Overview

This guide explains how to connect AWS IAM identities, roles, and policies to the BaseLayerOS substrate. The goal is to normalize AWS identities into canonical `IdentityBinding` objects consumed by IBEL, CIIPL, CEL, RIE, and GDAL.

## 1. Prerequisites

- AWS account with IAM administrative access
- Ability to create IAM roles, policies, and trust relationships
- BaseLayerOS Identity Adapter configured with AWS IAM mapping script

## 2. Create an IAM Role for BaseLayerOS

Create a role that BaseLayerOS can assume to read IAM metadata.

Example trust policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::<ACCOUNT_ID>:role/BaseLayerOS" },
      "Action": "sts:AssumeRole"
    }
  ]
}

## 3. Map IAM attributes to Canonical Identity Fields

BaseLayerOS uses the following mappings:

Canonical Field - Subject_id, roles, attributes
IAM Source - ARN, Attached IAM Policies, AccountID & Region
**Mappings are defined in identity-adapter-config.yaml**.

## 4. Export IAM Events to BaseLayerOS

Enable CloudTrail and configure export to the Logging Adapter endpoint.
Recommended Events:
- 'iam:CreateUser'
- 'iam:AttachRolePolicy'
- 'iam:PutRolePolicy
- 'sts:AssumeRole'
- iam:UpdateAssumeRolePolicy

## 5. Governance Enforcement

IAM events are evaluated against:

- Abuse-prevention Predicates
- Privilege escalation rules
- Regulatory predicates (OSFI, SOC2, etc.)

Example:
- Agent attempts to modify its own IAM policy -> DENIED by 'abuse-prevention-001'

6. Verification
Run:
'blos identity verify --provider aws_iam'

## 7. Forensics
All IAM events are written to ITFS with:
- Identity Binding
- Governance Decision
- Cryptographic Hash
```
