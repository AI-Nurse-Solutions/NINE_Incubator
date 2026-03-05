# NINE² Incubator — Base Agent Team Specification

**Playbook Reference:** Phase 1 · Step 1.3
**Branch:** 🏗️ VENTURE ARCHITECTURE
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document specifies the roles, functions, and EDENA risk zones for the seven core agents that form the NINE² Base Agent Team. Every niche track inherits this team. The agents are designed to be general-purpose and are parameterized by the `cohort-config.yaml` file at runtime, allowing them to adapt to the specific context of each niche.

## 2. Base Agent Team

| Agent Role | Clinical/Venture Function | EDENA Zone | Human Gate? | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Intake Agent** | Validates inputs, gathers missing data, routes to planner. | 🟢 **GREEN** | No | The front door of the incubator. It receives tasks, checks for completeness (e.g., "Does this research request have a clear objective?"), and gathers any missing context before passing the work unit to the Planning Agent. |
| **Planning Agent** | Decomposes tasks into steps, selects tools, estimates costs. | 🟢 **GREEN** | No | The project manager. It takes a validated task from the Intake Agent and breaks it down into atomic work units, mapping each to a Playbook step and assigning an initial EDENA tier. |
| **Execution Agent** | Performs mechanical workflow steps against tools/APIs. | 🟡 **YELLOW** | On Threshold Breach | The workhorse. It executes Green-tier tasks autonomously. For Yellow-tier tasks, it stages the action (e.g., drafts an email) and awaits approval via a Gate Card. It only executes Yellow-tier actions after receiving explicit nurse-founder approval. |
| **QA / Critic Agent** | Checks for completeness, policy fit, evidence currency. | 🟢 **GREEN** | No | The internal auditor. It reviews the artifacts produced by the Execution Agent to ensure they meet quality standards, align with Playbook doctrine, and (if applicable) cite evidence less than 5 years old. |
| **Compliance Agent** | Enforces NAIO rules, flags EDENA red-zone actions. | 🟡 **YELLOW** | Always | The governance enforcer. It runs a final check on all pending actions, ensuring they comply with NAIO standards and the track's `cohort-config.yaml`. It has the authority to halt any action that violates a governance rule and escalate it. |
| **Escalation Handler** | Routes clinical/governance exceptions to the named steward. | 🔴 **RED** | Always (No Exception) | The safety valve. When a Red-tier action is proposed or a critical governance failure is detected, this agent takes over. It halts all related work, creates a formal escalation record, and notifies the designated NAIO Governance Anchor. It does not proceed without human intervention. |
| **Reporting Agent** | Produces audit logs, nurse-facing summaries, NAIO reports. | 🟢 **GREEN** | No | The chronicler. It generates all the persistent state documents: Venture State Reports, Execution Records, audit logs for the NAIO registry, and human-readable summaries for the nurse-founder. |

## 3. Orchestration Flow

These agents do not operate in isolation. They are orchestrated by the Florence-X Conductor Engine in a predictable sequence:

1.  **Task Arrival** → `Intake Agent` (Validate & Enrich)
2.  **Validated Task** → `Planning Agent` (Decompose & Plan)
3.  **Execution Plan** → `Execution Agent` (Execute Green / Gate Yellow)
4.  **Artifact Produced** → `QA / Critic Agent` (Review & Validate)
5.  **Validated Action** → `Compliance Agent` (Final Governance Check)
6.  **Compliant Action** → `Execution Agent` (Commit / Send)
7.  **All Steps** → `Reporting Agent` (Log & Report)
8.  **Red Tier / Critical Error** → `Escalation Handler` (Halt & Notify)

This orchestrated workflow ensures that every action is planned, validated, and checked for compliance before execution, with clear gates for human authority.

---

*AI draft; nurse reviews; nurse authorizes.*
