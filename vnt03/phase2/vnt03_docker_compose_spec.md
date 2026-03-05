# NINE² Incubator — Base docker-compose.yaml Specification

**Playbook Reference:** Phase 2 · Step 2.1
**Branch:** 🏗️ VENTURE ARCHITECTURE
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document specifies the structure and design of the base `docker-compose.yaml` file for the `nine2-base-incubator`. This file is the cornerstone of the infrastructure, defining all the services that make up a niche track. It is designed to be heavily parameterized, with nearly all configurable values being injected at runtime from the `cohort-config.yaml` via environment variables. This ensures that the infrastructure is as reusable and forkable as the rest of the system.

## 2. Design Principles

- **Parameterization over Hardcoding:** No service, port, or configuration specific to a niche will be hardcoded. Everything will be derived from environment variables.
- **Service Discovery:** Services will communicate with each other using their service names, relying on Docker's internal DNS.
- **Stateless Services:** All core agent and application services will be designed to be stateless. State will be managed in dedicated services like the database and the audit log.

## 3. Base `docker-compose.yaml` Template

This template illustrates how the services are defined and how they consume environment variables that will be populated by the `fork-track.sh` script from the `cohort-config.yaml`.

```yaml
version: '3.8'

services:
  # ◈ CORE ORCHESTRATION & GOVERNANCE ◈
  orchestrator:
    image: ${DOCKER_REGISTRY}/${TRACK_SLUG}-orchestrator:${TRACK_VERSION}
    container_name: ${TRACK_SLUG}-orchestrator
    restart: always
    environment:
      - TRACK_ID=${TRACK_ID}
      - EDENA_DEFAULT=${GOVERNANCE_EDENA_DEFAULT}
      - STEWARD_EMAIL=${GOVERNANCE_STEWARD_EMAIL}
    ports:
      - "${ORCHESTRATOR_PORT}:8080"
    networks:
      - nine2-net

  governance-engine:
    image: ${DOCKER_REGISTRY}/${TRACK_SLUG}-governance:${TRACK_VERSION}
    container_name: ${TRACK_SLUG}-governance-engine
    restart: always
    environment:
      - AUTO_APPROVE_GREEN=${GOVERNANCE_AUTO_APPROVE_GREEN}
      - HUMAN_GATE_YELLOW=${GOVERNANCE_HUMAN_GATE_YELLOW}
      - BLOCK_RED=${GOVERNANCE_BLOCK_RED}
    networks:
      - nine2-net

  # ◈ AGENT SERVICES ◈
  # Example: Intake Agent. This pattern repeats for all enabled base and niche agents.
  intake-agent:
    image: ${DOCKER_REGISTRY}/${TRACK_SLUG}-intake-agent:${TRACK_VERSION}
    container_name: ${TRACK_SLUG}-intake-agent
    restart: always
    environment:
      - ORCHESTRATOR_URL=http://orchestrator:8080
    networks:
      - nine2-net

  # ◈ STATEFUL SERVICES & DATA STORES ◈
  audit-log:
    image: postgres:14-alpine # Using a standard, reliable image for the append-only log
    container_name: ${TRACK_SLUG}-audit-log
    restart: always
    environment:
      - POSTGRES_USER=audit
      - POSTGRES_PASSWORD=${AUDIT_LOG_DB_PASSWORD} # Secret injected at runtime
      - POSTGRES_DB=audit_log
    volumes:
      - audit-log-data:/var/lib/postgresql/data
    networks:
      - nine2-net

  # ◈ HUMAN INTERFACE ◈
  dashboard:
    image: ${DOCKER_REGISTRY}/${TRACK_SLUG}-dashboard:${TRACK_VERSION}
    container_name: ${TRACK_SLUG}-dashboard
    restart: always
    ports:
      - "${DASHBOARD_PORT}:80"
    environment:
      - REACT_APP_ORCHESTRATOR_API=http://localhost:${ORCHESTRATOR_PORT}
      - REACT_APP_TRACK_NAME=${TRACK_NAME}
    networks:
      - nine2-net

# ◈ NETWORKS & VOLUMES ◈
networks:
  nine2-net:
    driver: bridge

volumes:
  audit-log-data:
```

## 4. Environment Variable Mapping

The `fork-track.sh` script will be responsible for reading the `cohort-config.yaml` and generating a `.env` file to be used by `docker-compose`.

| `cohort-config.yaml` Path | Environment Variable |
| :--- | :--- |
| `track.track_id` | `TRACK_ID` |
| `track.name` | `TRACK_NAME` |
| `track.slug` | `TRACK_SLUG` |
| `governance.edena_default` | `GOVERNANCE_EDENA_DEFAULT` |
| `governance.steward_email` | `GOVERNANCE_STEWARD_EMAIL` |
| *User Defined* | `DOCKER_REGISTRY` |
| *User Defined* | `TRACK_VERSION` |

This design ensures that the `docker-compose.yaml` file remains generic and reusable, while the specific configuration of each niche track is managed entirely within the `cohort-config.yaml` and the environment, perfectly aligning with the "factory" model.

---

*AI draft; nurse reviews; nurse authorizes.*
