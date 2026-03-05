# NINE² Incubator — EDENA Risk Tier Enforcement Engine Specification

**Playbook Reference:** Phase 3 · Step 3.1
**Branch:** 🏗️ VENTURE ARCHITECTURE
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document specifies the design of the EDENA (Ethical Decision Engine for Nurse-led AI) Risk Tier Enforcement Engine. This engine is the core of the NINE² governance model. Its primary function is to intercept every action proposed by an agent, classify its risk, and enforce the appropriate governance protocol (execute, gate for approval, or block) based on its EDENA tier. This ensures that **human (nurse) authority is maintained at every critical decision point**.

## 2. Engine Architecture

The EDENA Engine will be implemented as a middleware service within the `governance-engine` container. The Florence-X Orchestrator will route every planned agent action through this engine before it is dispatched to an Execution Agent.

### Logical Flow:

1.  **Action Interception:** The Orchestrator sends a proposed action (e.g., `{agent: "outreach_agent", action: "send_email", params: {...}}`) to the EDENA Engine.
2.  **Tier Classification:** The engine classifies the action into a risk tier (Green, Yellow, or Red). Classification is based on a set of configurable rules derived from the `cohort-config.yaml`.
3.  **Protocol Enforcement:** Based on the tier, the engine returns a directive to the Orchestrator:
    *   `{disposition: "PROCEED"}` for 🟢 **Green** actions.
    *   `{disposition: "GATE"}` for 🟡 **Yellow** actions.
    *   `{disposition: "BLOCK"}` for 🔴 **Red** actions.
4.  **Orchestrator Execution:** The Orchestrator acts on the disposition. It executes `PROCEED` actions, creates a Gate Card for `GATE` actions, and halts the workflow for `BLOCK` actions, triggering the Escalation Handler agent.

## 3. Tier Classification Ruleset

The engine's classification logic will be based on a cascading ruleset. The first rule that matches an incoming action determines its tier.

```python
# Pseudocode for the classification logic

def classify_action(action, config):
    # Rule 1: Check for explicit Red-tier actions (highest priority)
    if action.type in config.red_tier_actions:
        return "RED"

    # Rule 2: Check for explicit Yellow-tier actions
    if action.type in config.yellow_tier_actions:
        return "YELLOW"

    # Rule 3: Check for keywords in action parameters
    if contains_keyword(action.params, config.red_keywords):
        return "RED"

    # Rule 4: Apply the track's default EDENA tier
    return config.edena_default # from cohort-config.yaml
```

**Configuration in `cohort-config.yaml`:**

The rules will be defined in a new `edena_rules` section within the `governance` block of the `cohort-config.yaml` file, allowing each niche track to have a custom risk profile.

```yaml
governance:
  edena_default: "yellow"
  edena_rules:
    red_tier_actions: ["modify_medication_order", "update_ventilator_setting"]
    yellow_tier_actions: ["send_external_email", "publish_blog_post", "create_financial_projection"]
    red_keywords: ["PHI", "patient_identifier", "social_security_number"]
```

## 4. Governance Actions

| EDENA Tier | Engine Action | Orchestrator Response |
| :--- | :--- | :--- |
| 🟢 **GREEN** | Returns `PROCEED`. | Immediately dispatches the action to the appropriate Execution Agent. |
| 🟡 **YELLOW** | Returns `GATE`. | Halts the workflow. Creates a **Gate Card** and places it in the Open Gate Queue. Notifies the nurse-founder for approval. |
| 🔴 **RED** | Returns `BLOCK`. | Halts the workflow. Triggers the **Escalation Handler** agent, which creates an escalation record and notifies the NAIO Governance Anchor. |

This engine is the technical implementation of the NINE² commitment to safe, nurse-governed AI. It ensures that no high-risk action can ever be taken without explicit human review and approval, making governance an automated, inescapable part of the venture-building process.

---

*AI draft; nurse reviews; nurse authorizes.*
