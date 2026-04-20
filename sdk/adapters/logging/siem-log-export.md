# SIEM Log Export Integration Guide

Exporting SIEM Events into the BaseLayerOS Logging Adapter (FLA)

## Overview

This guide explains how to export logs from enterprise SIEM platforms into the BaseLayerOS Logging Adapter. Supported SIEMs include:

- Microsoft Sentinel
- Splunk
- IBM QRadar
- Elastic SIEM
- Chronicle
- ArcSight

All exported events are normalized into substrate-native forensic events and written into ITFS (Immutable Temporal Forensics Substrate).

---

## 1. Prerequisites

- Network access from SIEM → BaseLayerOS Logging Adapter endpoint
- Ability to configure log forwarding or webhook export
- API key or mutual TLS certificate for ingestion endpoint
- Logging Adapter deployed and reachable

---

## 2. BaseLayerOS Logging Adapter Endpoint

### HTTPS Ingestion Endpoint

POST https://<your-blos-endpoint>/logging/ingest
Content-Type: application/json
Authorization: Bearer <API_KEY>

### Required Fields

- `id` — unique event ID
- `timestamp` — UTC timestamp
- `source` — SIEM system metadata
- `identity` — identity context (if available)
- `action` — normalized verb + target
- `governance` — predicates + enforcement decision

Events missing identity will be enriched by IAL where possible.

---

## 3. Microsoft Sentinel Integration

### Option A — Logic App Export

Use a Logic App to forward Sentinel alerts to the Logging Adapter.

Example payload mapping:

```json
{
  "id": "@{triggerBody()?['SystemAlertId']}",
  "timestamp": "@{triggerBody()?['TimeGenerated']}",
  "source": {
    "system": "sentinel",
    "event_type": "@{triggerBody()?['AlertName']}",
    "severity": "@{triggerBody()?['Severity']}"
  },
  "identity": {
    "subject_id": "@{triggerBody()?['AccountCustomEntity']}"
  },
  "action": {
    "verb": "alert",
    "target": "@{triggerBody()?['ResourceId']}"
  }
}
### Option B - Sentinel Data Connector
Forward logs via the “HTTP Data Collector” connector.

## 4. Splunk Integration.

### Option A — HTTP Event Collector (HEC)
Enable HEC and configure an output to the Logging Adapter.

Example Splunk transform:
TRANSFORMS-blos = blos_forwarder
blos_forwarder = DEST_KEY=_HTTP_ROUTING blos

Example Payload:
{
  "id": "splunk-evt-{{guid}}",
  "timestamp": "{{_time}}",
  "source": {
    "system": "splunk",
    "event_type": "{{event_type}}"
  },
  "identity": {
    "subject_id": "{{user}}"
  },
  "action": {
    "verb": "{{action}}",
    "target": "{{object}}"
  }
}

## 5. QRadar Integration
Use QRadar’s “Universal Cloud REST API” to forward events.

Steps:
1. Admin → Extensions → Create New Extension
2. Configure REST destination
3. Map QRadar fields → BaseLayerOS schema

Recommended fields:
- username
- sourceip
- eventname
- logsourceid
- qideventcategory

## 6. Elastic SIEM Integration
Use Elastic’s webhook connector:

PUT _watcher/watch/blos_export
{
  "trigger": { "schedule": { "interval": "30s" }},
  "actions": {
    "export": {
      "webhook": {
        "method": "POST",
        "url": "https://<your-blos-endpoint>/logging/ingest",
        "body": "{{#toJson}}ctx.payload.hits.hits{{/toJson}}"
      }
    }
  }
}

## 7. Identity Binding
If SIEM logs contain:
- username
- UPN
- ARN
- service account
- agent ID

…IAL will resolve them into canonical IdentityBinding.

If identity is missing:
- event is still ingested
- marked as identity: null
- flagged for enrichment
- governance rules requiring identity will trigger deny or escalate

8. Governance Enforcement
All SIEM events are evaluated against:
- abuse-prevention predicates
- regulatory predicates (OSFI, PHIPA, HIPAA, GDPR)
- workflow predicates
- data access predicates

### Example:
- CloudTrail IAM policy change → triggers abuse-prevention-001
- EHR access event → triggers phipa-access-001

9. Forensics & Explainability
Every SIEM event is written to ITFS with:
- cryptographic hash
- identity binding
- governance decision
- full replay context

This enables:
- forensic replay
- explainability narratives
- regulatory audit trails
- incident response

## 10. Verification
Run:
'blos logging verify --provider siem'

If successful, events will appear in:
'blos forensics tail'


```
