# NINE² Incubator — NAIO Governance Endpoint Specification

**Playbook Reference:** Phase 4 · Step 4.3
**Branch:** 🔍 MARKET VISIBILITY
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document specifies the design of the public-facing NAIO Governance Endpoint. This is a critical component of the Niche Discoverability architecture, providing a standardized, machine-readable API for third parties to query the governance status and history of a NINE² niche track. It transforms the internal, append-only audit log into a public utility for transparency and trust verification.

## 2. Endpoint Design

The Governance Endpoint will be a read-only API that exposes a summarized and non-sensitive view of the internal audit log. It will be hosted at the predictable URL defined in the `track-manifest.json` (e.g., `https://api.nine2.ai/v1/tracks/vnt-03/audit`).

### Key Design Principles:

- **Read-Only:** The endpoint will not allow any write operations.
- **Summarization:** It will not expose the full, detailed audit log. Instead, it will provide aggregated counts and summaries of key governance events.
- **No Sensitive Data:** The endpoint will not return any personally identifiable information (PII), protected health information (PHI), or proprietary business logic from the audit log details.
- **Standardized Format:** The API will use a standard JSON format for all responses.

## 3. API Specification

### `GET /v1/tracks/{track_id}/audit/summary`

This is the primary endpoint. It returns a high-level summary of governance activity for the specified track.

**Example Response:**

```json
{
  "track_id": "VNT-03",
  "summary_period_days": 30,
  "last_event_timestamp": "2026-03-05T14:10:00Z",
  "event_counts": {
    "green_actions_executed": 142,
    "yellow_actions_gated": 12,
    "yellow_actions_approved": 11,
    "red_actions_blocked": 1,
    "config_changes": 3
  },
  "gate_card_stats": {
    "open_gates": 1,
    "avg_approval_time_hours": 2.5
  },
  "last_integrity_check": {
    "timestamp": "2026-03-05T08:00:00Z",
    "status": "PASS"
  }
}
```

### `GET /v1/tracks/{track_id}/audit/events`

This endpoint returns a paginated list of recent, non-sensitive governance events.

**Query Parameters:**

- `limit` (integer, default: 100): The number of events to return.
- `page` (integer, default: 1): The page number for pagination.

**Example Response:**

```json
{
  "pagination": {
    "total_events": 58,
    "limit": 2,
    "page": 1,
    "total_pages": 29
  },
  "events": [
    {
      "event_id": "evt_12345",
      "timestamp": "2026-03-05T14:10:00Z",
      "event_type": "AGENT_ACTION_EXECUTED",
      "summary": "Agent vnt03-reporting-agent-v1.0.0 executed action: generate_cycle_report."
    },
    {
      "event_id": "evt_12344",
      "timestamp": "2026-03-05T13:45:00Z",
      "event_type": "GATE_APPROVED",
      "summary": "Gate Card GC-001 was approved by the nurse-founder."
    }
  ]
}
```

## 4. Implementation

- The API will be implemented as a separate, read-only service that has permission to query a materialized view of the main audit log database.
- This materialized view will be pre-aggregated and stripped of all sensitive `details` from the `audit_events` table, ensuring that the public-facing API cannot accidentally expose confidential information.
- Heavy caching will be used to ensure the endpoint is fast and does not place a significant load on the primary database.

This public endpoint allows any third party to programmatically answer the question, "Is this venture operating under the governance it claims?" without needing to access any internal systems, which is a foundational element of building trust in the open.

---

*AI draft; nurse reviews; nurse authorizes.*
