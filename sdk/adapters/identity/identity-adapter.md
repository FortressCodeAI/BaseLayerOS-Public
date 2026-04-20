# Identity Adapter Layer (IAL) – Specification

The Identity Adapter Layer (IAL) is the bridge between external identity providers
and the BaseLayerOS substrate. It normalizes identity, roles, attributes, and
entitlements into a substrate-native representation consumed by:

- IBEL (Identity-Bound Execution Layer)
- CIIPL (Cross-Industry Identity & Permissions Layer)
- CEL (Constitutional Enforcement Layer)
- RIE (Regulatory Interpretation Engine)
- WCF (Workflow Compliance Fabric)
- GDAL (Governed Data Access Layer)

---

## 1. Responsibilities

The Identity Adapter is responsible for:

- **Ingestion:** Connecting to external IdPs (Azure AD, Okta, AWS IAM, on-prem LDAP, etc.).
- **Normalization:** Mapping provider-specific identities, groups, and claims into a canonical identity model.
- **Enrichment:** Attaching contextual attributes (jurisdiction, department, license, clearance, etc.).
- **Binding:** Producing an `IdentityBinding` object for each execution context (agent, workflow, human).
- **Verification:** Ensuring identities are valid, non-expired, and policy-compliant at time of use.
- **Traceability:** Emitting identity metadata into ITFS for forensic replay and EDE for explainability.

---

## 2. Canonical Identity Model

All external identities are normalized into a canonical `IdentityBinding` structure:

```json
{
  "subject_id": "string", // Stable, substrate-level identifier
  "external_ids": [
    {
      "provider": "azure_ad",
      "id": "guid-or-upn"
    },
    {
      "provider": "okta",
      "id": "okta-user-id"
    }
  ],
  "type": "human | service | agent | workflow",
  "display_name": "string",
  "roles": ["clinician", "claims_adjuster", "llm_agent", "system_admin"],
  "groups": ["finance_ops", "icu_staff", "tier1_support"],
  "attributes": {
    "jurisdiction": "CA-ON",
    "organization": "Hospital-123",
    "department": "Cardiology",
    "license_number": "CPSO-XXXX",
    "clearance_level": "high",
    "employment_status": "active"
  },
  "entitlements": [
    "ehr.read",
    "ehr.write",
    "payments.view",
    "workflow.escalate"
  ],
  "session": {
    "session_id": "string",
    "issued_at": "2026-04-19T22:31:00Z",
    "expires_at": "2026-04-19T23:31:00Z",
    "auth_method": "mfa | cert | oidc | saml",
    "ip_address": "x.x.x.x",
    "device_id": "string"
  }
}
```
