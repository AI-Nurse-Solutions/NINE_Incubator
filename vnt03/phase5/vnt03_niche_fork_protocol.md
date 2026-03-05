# NINE² Incubator — Niche Fork Protocol

**Playbook Reference:** Phase 5 · Steps 5.1-5.7
**Branch:** 🏗️ VENTURE ARCHITECTURE
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document provides the detailed, 7-stage protocol for forking the `nine2-base-incubator` to create a new, governed niche incubator track. This is the master playbook for the `fork-track.sh` script and the human operators who oversee it. Following this protocol ensures that every new track is created in a consistent, secure, and governable manner.

## 2. The 7 Stages of the Fork Protocol

### Stage 1: Configuration Definition (Step 5.1)

**Objective:** To define the identity, rules, and parameters of the new niche track.

**Procedure:**
1.  Copy the master `cohort-config.yaml` template.
2.  Fill out every field in the template for the new niche (e.g., `ICU-01`, `Home-Health-07`). This includes:
    *   `track_id`, `name`, `slug`, `icp`, `anchor_workflow`.
    *   All `governance` settings, including the `edena_default` tier and the `steward_email`.
    *   The list of `niche_agents` to be developed and enabled.
    *   All `tools`, `pricing`, and `credentialing` requirements.
3.  Submit the completed `cohort-config.yaml` for review by the NAIO Governance Anchor.

**Gate:** This is a **🟡 YELLOW** tier action. The `cohort-config.yaml` must be approved before proceeding.

### Stage 2: Automated Forking (Step 5.2)

**Objective:** To create the new track's infrastructure and services.

**Procedure:**
1.  Execute the `fork-track.sh` script, providing the path to the approved `cohort-config.yaml`.
2.  The script will automatically:
    *   Clone the `nine2-base-incubator` repository.
    *   Inject the config file and generate the `.env` file.
    *   Build all required Docker images.
    *   Tag all images with the correct track slug and version.
    *   Push all images to the private container registry (GHCR).

### Stage 3: Niche Agent Development (Step 5.3)

**Objective:** To build and deploy the custom agents required for the new niche.

**Procedure:**
1.  Create a new development branch in the forked repository.
2.  Develop the Python code for the `niche_agents` listed in the `cohort-config.yaml`.
3.  Add the new agents to the `docker-compose.yaml` file.
4.  Build and test the new agent images locally.

### Stage 4: Governance Artifact Generation (Step 5.4)

**Objective:** To create the public-facing governance manifests for the new track.

**Procedure:**
1.  Run the `generate-manifests.sh` script.
2.  This script reads the `cohort-config.yaml` and the new agent definitions to produce:
    *   `track-manifest.json`
    *   `agent-registry.json`
3.  The CI/CD pipeline automatically deploys these files to the public API endpoints.

### Stage 5: Governance Discoverability Test (Step 5.5)

**Objective:** To verify that the new track is correctly exposing its governance artifacts.

**Procedure:**
1.  Trigger the Governance Discoverability Test Suite against the new track's public endpoints.
2.  The suite will automatically check for the existence and validity of the track manifest, agent registry, and audit summary endpoint.
3.  The deployment is only considered successful if all tests pass.

### Stage 6: Manual Workflow Validation (Step 5.6)

**Objective:** To run the anchor workflow manually with real users before enabling full automation.

**Procedure:**
1.  Onboard 1-3 nurses from the target niche who match the Ideal Cohort Profile (ICP).
2.  Manually walk them through the `anchor_workflow` defined in the `cohort-config.yaml`, using the AI agents as tools but with a human operator driving every step.
3.  Document every question, point of confusion, and exception encountered. This data is used to refine the workflow and train the agents.

### Stage 7: Go-Live (Step 5.7)

**Objective:** To switch the track to live, autonomous operation.

**Procedure:**
1.  Once the manual validation is complete and the workflow is stable, the NAIO Governance Anchor formally approves the track for go-live.
2.  The `docker-compose.yaml` is updated to enable the full, orchestrated workflow.
3.  The track is now considered live and begins accepting its first official cohort.

This rigorous, multi-stage protocol ensures that every new incubator track is not just technically functional, but also properly governed, validated, and ready to serve its target nurse community.

---

*AI draft; nurse reviews; nurse authorizes.*
