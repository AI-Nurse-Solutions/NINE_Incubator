# NINE² Incubator — cohort-config.yaml Schema

**Playbook Reference:** Phase 1 · Step 1.2
**Branch:** 🏗️ VENTURE ARCHITECTURE
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document defines the schema for `cohort-config.yaml`. This file is the most critical design component of the NINE² factory model. It is the **only** file an operator modifies when forking the Base Incubator to create a new niche track. The `fork-track.sh` script uses this file to parameterize and configure every aspect of the new track, from agent deployment to governance policy.

## 2. Schema Definition

```yaml
# ◈ TRACK CONFIGURATION ◈
# Defines the unique identity and focus of this incubator track.
track:
  track_id: "VNT-03"  # Unique identifier for the track (e.g., ICU-01, EDU-02).
  name: "Nurse Venture Builder" # Human-readable name of the niche track.
  slug: "nurse-venture-builder" # URL-friendly slug used for service naming and domains.
  clinical_vertical: "venture" # The primary clinical or professional vertical.
    # Enum: [icu, periop, educator, venture, governance, community, home-health, staffing]
  icp: "Entrepreneurial nurses launching AI-assisted healthcare products, consulting practices, or advocacy organizations." # Ideal Cohort Profile: A detailed description of the target nurse for this track.
  anchor_workflow: "Business model validation -> pitch deck assembly -> stakeholder communication system" # The first, primary workflow this track is designed to automate and govern.

# ◈ COHORT PARAMETERS ◈
# Defines the structure and rules for the cohort participating in this track.
cohort:
  size: 3-9 # The minimum and maximum number of ventures per cohort, per NINE² Charter.
  duration_months: 6-24 # The expected duration of the incubation program.
  governance_anchor: required-before-month-2 # NINE² Charter mandate: a named NAIO steward must be assigned before the second month.
  naio_credential_minimum: "NAIO-A" # The minimum NAIO credential level required for the Governance Anchor.
    # Enum: [NAIO-C, NAIO-A, NAIO-F]

# ◈ AGENT & GOVERNANCE CONFIGURATION ◈
# Defines the agent team and the rules of engagement (EDENA Tiers).
agents:
  enabled: # List of base agents to be deployed for this track.
    - "intake"
    - "planner"
    - "executor"
    - "qa_critic"
    - "compliance"
    - "escalation"
    - "reporter"
  niche_agents: # List of custom, niche-specific agents to be deployed.
    - "market_intelligence_agent"
    - "pitch_draft_agent"
    - "outreach_sequencer_agent"
  disabled: [] # List of any base agents to explicitly disable for this track.

governance:
  edena_default: "yellow" # The default EDENA risk tier for actions in this track if not otherwise specified.
    # Enum: [green, yellow, red]
  auto_approve_green: true # Whether to execute Green-tier actions autonomously.
  human_gate_yellow: true # ALWAYS true for clinical tracks. Requires nurse-founder approval for Yellow-tier actions.
  block_red: true # ALWAYS true. Red-tier actions are halted and escalated, never executed by the agent.
  ai_disclosure: required # Enforces the 'AI draft; nurse reviews; nurse authorizes' tag on all outputs.
  steward_email: "steward@nine2.ai" # The designated email for governance escalations.
  phi_at_edge: false # NEVER true per NINE² Charter. Prohibits handling of PHI at the edge.

# ◈ TOOL & INTEGRATION CONFIGURATION ◈
# Defines the external systems this track will connect to.
tools:
  ehr: "none" # The target EHR system for integration.
    # Enum: [epic, cerner, fhir-generic, none]
  lms: "none" # The target Learning Management System.
    # Enum: [canvas, blackboard, none]
  crm: "hubspot" # The target Customer Relationship Management system.
    # Enum: [hubspot, none]
  billing: "stripe" # The designated billing engine.
  secure_messaging: "none" # The target secure messaging platform.
    # Enum: [vocera, tigerconnect, none]
  clinical_libraries: # List of clinical knowledge libraries to enable.
    - "protocol-lookup"

# ◈ PRICING & MONETIZATION ◈
# Defines the business model for the track.
pricing:
  model: "per_cohort" # The pricing model.
    # Enum: [per_cohort, per_seat, outcome_based]
  currency: "USD"

# ◈ CREDENTIALING REQUIREMENTS ◈
# Defines the NAIO credentialing requirements for cohort members.
credentialing:
  naio_c_required: true # Is the NAIO-C (Credentialed) level required for cohort members?
  naio_a_required: false # Is the NAIO-A (Architect) level required?
  naio_f_required: false # Is the NAIO-F (Fellow) level required?
```

## 3. Usage

When the `fork-track.sh` script is executed, it reads this `cohort-config.yaml` file and uses its values to:

1.  **Set Environment Variables:** All values are injected as environment variables into the Docker containers for the new track.
2.  **Configure Services:** The orchestrator, governance engine, and other core services adapt their behavior based on these settings (e.g., enforcing the `edena_default` tier).
3.  **Deploy Agents:** The script enables, disables, or deploys agents based on the `agents` configuration.
4.  **Scaffold Artifacts:** It generates the initial `track-manifest.json` and other governance artifacts based on the track identity and configuration.

This single source of truth ensures that every niche track is a consistent, governed, and predictable fork of the base incubator.

---

*AI draft; nurse reviews; nurse authorizes.*
