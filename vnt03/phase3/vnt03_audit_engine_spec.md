# NINE² Incubator — Append-Only Audit Engine Specification

**Playbook Reference:** Phase 3 · Step 3.3
**Branch:** 🏗️ VENTURE ARCHITECTURE
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document specifies the design of the Append-Only Audit Engine, a mission-critical component of the NAIO governance framework. The engine's purpose is to create a **permanent, immutable, and verifiable record** of every significant action taken within a NINE² niche track. This log is not for debugging; it is a legal and ethical record designed to provide complete transparency and traceability for all automated and human-approved actions, with a mandated retention period of 7 years.

## 2. Engine Architecture

The Audit Engine will be a dedicated service running in the `audit-log` container, using a PostgreSQL database configured for append-only operations. Direct write access to this database will be strictly limited to the Audit Engine service itself. All other services (Orchestrator, Agents, etc.) will send events to the Audit Engine via a dedicated, secure API endpoint.

### Key Architectural Features:

- **Append-Only Data Model:** The primary audit table will be configured with permissions that prevent `UPDATE` and `DELETE` operations. Data can only be inserted.
- **Cryptographic Hashing:** Each log entry will contain a cryptographic hash of the previous entry's contents, creating a blockchain-like chain of custody. Any attempt to tamper with a past entry would invalidate the entire chain from that point forward.
- **Secure Timestamping:** The engine will use a trusted, synchronized time source (NTP) for all timestamps, independent of the calling service's clock.
- **7-Year Retention:** The underlying database volume will be configured with backup and retention policies to ensure data is preserved for the mandated 7-year period.

## 3. Logged Events

The engine will be configured to log the following critical events:

| Event Type | Trigger | Data Captured |
| :--- | :--- | :--- |
| `AGENT_ACTION_PROPOSED` | An agent proposes an action. | Agent ID, Action Type, Parameters, EDENA Tier Classification |
| `GATE_CARD_ISSUED` | A Yellow-tier action is gated. | Gate Card ID, Work Unit, Action Requested, Founder Impact |
| `GATE_APPROVED` | Nurse-founder approves a Gate Card. | Gate Card ID, Approver ID, Timestamp |
| `GATE_REJECTED` | Nurse-founder rejects a Gate Card. | Gate Card ID, Rejecter ID, Reason |
| `AGENT_ACTION_EXECUTED` | A Green or approved-Yellow action is executed. | Agent ID, Action Type, Result (Success/Fail), Artifact Link |
| `RED_TIER_BLOCKED` | A Red-tier action is blocked. | Agent ID, Action Type, Escalation Record ID |
| `CONFIG_CHANGE` | `cohort-config.yaml` is modified. | Full copy of the new config file, Change Author ID |
| `REGISTRY_UPDATE` | `agent-registry.json` is updated. | Full copy of the new registry file, Change Author ID |

## 4. Audit Log Schema

The primary `audit_events` table in the PostgreSQL database will have the following schema:

```sql
CREATE TABLE audit_events (
    id BIGSERIAL PRIMARY KEY,                      -- Auto-incrementing unique ID
    event_timestamp TIMESTAMPTZ NOT NULL,          -- Secure, timezone-aware timestamp
    event_type VARCHAR(50) NOT NULL,               -- e.g., 'AGENT_ACTION_EXECUTED'
    track_id VARCHAR(50) NOT NULL,                 -- e.g., 'VNT-03'
    actor_id VARCHAR(100) NOT NULL,                -- e.g., 'vnt03-execution-agent-v1.0.0' or 'user:robert.domondon'
    details JSONB NOT NULL,                        -- A JSON blob containing all event-specific data
    previous_hash CHAR(64) NOT NULL,               -- SHA-256 hash of the previous log entry
    current_hash CHAR(64) NOT NULL UNIQUE          -- SHA-256 hash of this entire entry (excluding this field)
);

-- Prevent updates and deletes
REVOKE UPDATE, DELETE ON audit_events FROM public;
```

## 5. Verification

A separate, scheduled process will run periodically to verify the integrity of the audit log by recalculating the hash chain. If any discrepancy is found, it will immediately trigger a Red-tier escalation to the NAIO Governance Anchor, signaling a critical governance failure.

This engine ensures that every action is recorded, every decision is traceable, and the entire history of the venture is preserved in a verifiable, tamper-evident format, providing the highest level of assurance to founders, partners, and regulators.

---

*AI draft; nurse reviews; nurse authorizes.*
