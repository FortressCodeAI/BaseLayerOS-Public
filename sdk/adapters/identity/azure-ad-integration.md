---

# ⭐ `azure-ad-integration.md`

```markdown
# Azure AD / Entra ID Integration Guide

Integrating Azure AD with BaseLayerOS Identity Adapter Layer (IAL)

## Overview

This guide explains how to connect Azure AD (Entra ID) to BaseLayerOS using OIDC and SCIM. Azure AD is the primary identity provider for healthcare, government, and enterprise IT.

## 1. Prerequisites

- Azure AD tenant admin access
- Ability to register enterprise applications
- SCIM provisioning enabled (optional but recommended)

## 2. Register BaseLayerOS as an Enterprise Application

In Azure Portal:

1. Azure Active Directory → App Registrations → New Registration
2. Name: `BaseLayerOS`
3. Redirect URI: `https://<your-blos-endpoint>/oidc/callback`

Record:

- Client ID
- Tenant ID
- OpenID configuration URL

## 3. Configure OIDC Claims

BaseLayerOS expects the following claims:

| Canonical Field | Azure AD Claim                      |
| --------------- | ----------------------------------- |
| subject_id      | oid                                 |
| display_name    | name                                |
| roles           | roles                               |
| groups          | groups                              |
| attributes      | department, jobTitle, extension\_\* |

These mappings are defined in `identity-adapter-config.yaml`.

## 4. Enable SCIM Provisioning (Optional)

SCIM sync improves:

- role accuracy
- group propagation
- attribute freshness

Configure SCIM endpoint:
```
