# NINE² Incubator — agent-registry.json Specification

**Playbook Reference:** Phase 4 · Step 4.2
**Branch:** 🔍 MARKET VISIBILITY
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document provides the formal specification for the `agent-registry.json` file, a public-facing governance artifact. While the `vnt03_naio_agent_registry.md` from Cycle 2 detailed the internal design, this specification focuses on the public JSON format as part of the **Niche Discoverability** phase. Its purpose is to provide a machine-readable, publicly accessible manifest of all AI agents operating within a track and their human stewards, enabling automated governance verification.

## 2. Schema and Example

The `agent-registry.json` file is hosted at the public URL defined in the `track-manifest.json`. It provides a detailed, accountable roster of all automated actors.

```json
{
  "schema_version": "1.0",
  "track_id": "VNT-03",
  "last_updated": "2026-03-05T14:05:00Z",
  "governance_anchor": {
    "name": "Robert Elvin Domondon",
    "naio_credential_level": "NAIO-A",
    "verification_uri": "https://api.nine2.ai/v1/credentials/verify?id=..."
  },
  "agents": [
    {
      "agent_id": "vnt03-intake-agent-v1.0.0",
      "role": "Intake Agent",
      "version": "1.0.0",
      "status": "ACTIVE",
      "description": "Validates inputs, gathers missing data, and routes tasks to the planner.",
      "image_uri": "ghcr.io/nine2-incubator/nurse-venture-builder-intake-agent:1.0.0"
    },
    {
      "agent_id": "vnt03-escalation-handler-v1.0.0",
      "role": "Escalation Handler",
      "version": "1.0.0",
      "status": "ACTIVE",
      "description": "Routes clinical and governance exceptions to the named steward.",
      "image_uri": "ghcr.io/nine2-incubator/nurse-venture-builder-escalation-handler:1.0.0"
    }
    // ... other agents listed here ...
  ]
}
```

## 3. Field Descriptions

| Field | Description |
| :--- | :--- |
| `schema_version` | The version of the agent registry schema. |
| `track_id` | The unique ID of the track. |
| `last_updated` | ISO 8601 timestamp of the last update. |
| `governance_anchor` | An object containing information about the human steward accountable for this track. |
| `governance_anchor.name` | The full name of the NAIO Governance Anchor. |
| `governance_anchor.naio_credential_level` | The NAIO credential level of the steward (e.g., NAIO-C, NAIO-A, NAIO-F). |
| `governance_anchor.verification_uri` | A link to the credential verification portal to confirm the steward's status. |
| `agents` | An array of objects, each representing a deployed AI agent. |
| `agents.agent_id` | A unique, versioned identifier for the agent. |
| `agents.role` | The human-readable role of the agent (e.g., "Planning Agent"). |
| `agents.version` | The semantic version of the agent's code. |
| `agents.status` | The current operational status of the agent (e.g., `ACTIVE`, `INACTIVE`, `DEPRECATED`). |
| `agents.description` | A brief description of the agent's function. |
| `agents.image_uri` | The full URI of the container image in the private registry. This provides a direct link to the exact build being run. |

## 4. Generation and Hosting

- **Generation:** This file is generated and updated by the same CI/CD process that manages the `track-manifest.json`, ensuring it is always in sync with the `cohort-config.yaml` and the deployed container images.
- **Hosting:** It is hosted at the public API endpoint specified in the `track-manifest.json`, making it discoverable and accessible for automated third-party auditing.

This public registry is a radical act of transparency. It makes the chain of command from AI to human explicit and verifiable, which is a prerequisite for earning institutional trust.

---

*AI draft; nurse reviews; nurse authorizes.*
