# NINE² Incubator — Base Repository Structure

**Playbook Reference:** Phase 1 · Step 1.1
**Branch:** 🏗️ VENTURE ARCHITECTURE
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document defines the canonical repository structure for the `nine2-base-incubator`. This structure is designed for maximum reusability and clear separation of concerns, enabling the rapid forking of new niche tracks via the `fork-track.sh` script. Every niche track, including VNT-03, will be a fork of this repository.

## 2. Repository Layout

```
/nine2-base-incubator
│
├── .github/                    # CI/CD workflows, issue templates, PR templates
│
├── /agents
│   ├── /base-agents/           # Core, non-niche-specific agents
│   │   ├── intake_agent.py
│   │   ├── planning_agent.py
│   │   ├── execution_agent.py
│   │   ├── qa_critic_agent.py
│   │   ├── compliance_agent.py
│   │   ├── escalation_handler.py
│   │   └── reporting_agent.py
│   └── /niche-agents/          # Placeholder for forked, niche-specific agents
│       └── .gitkeep
│
├── /core
│   ├── /orchestrator/          # Florence-X conductor engine
│   ├── /governance/            # NAIO policy engine & EDENA tier enforcement
│   ├── /memory/                # Cohort knowledge base, decision logs, exception registry
│   ├── /credentialing/         # NAIO-C/A/F verification & badge engine
│   ├── /billing/               # Stripe integration for cohort fees & outcome pricing
│   └── /audit/                 # Immutable NAIO event log (7-year retention)
│
├── /dashboard/                 # Mission control UI for cohort & steward (React/Vite)
│
├── /governance-artifacts
│   ├── /edena-framework/       # Risk tier classifier & ethical checkpoint engine
│   ├── /naio-registry/         # Manifests for registered agents and stewards
│   └── cohort-config.yaml      # THE fork configuration file (template)
│
├── /tools
│   ├── /ehr-adapters/          # Epic, Cerner, generic FHIR connectors
│   ├── /clinical-libraries/    # SBAR, protocol, teach-back template libraries
│   └── /communication/         # Secure messaging, LMS, CRM adapters
│
├── /scripts
│   └── fork-track.sh           # Automation script for forking a new niche track
│
├── docker-compose.yaml         # Base compose file for all services
├── Dockerfile                  # Dockerfile for the main application
├── README.md                   # High-level documentation
└── .gitignore
```

## 3. Directory Descriptions

| Directory | Purpose |
| :--- | :--- |
| `/agents` | Contains the Python code for all autonomous agents. `base-agents` are inherited by all forks, while `niche-agents` are populated during the fork process. |
| `/core` | The central nervous system of the incubator. Contains the non-negotiable, shared services like orchestration, governance, and memory. |
| `/dashboard` | The human-in-the-loop interface. Provides a web-based mission control for the nurse-founder and NAIO steward to monitor progress, approve gates, and review audits. |
| `/governance-artifacts` | The heart of the governance layer. Contains the EDENA framework, the NAIO registry manifests, and the master `cohort-config.yaml` template that defines a niche fork. |
| `/tools` | A collection of adapters and libraries that connect the agents to external systems (EHRs, CRMs, etc.). This directory allows for modular integration with various clinical and business environments. |
| `/scripts` | Contains operational scripts, with `fork-track.sh` being the most critical for implementing the horizontal scaling strategy. |

---

*AI draft; nurse reviews; nurse authorizes.*
