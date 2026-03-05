# NINE² Incubator — Private Container Registry Plan

**Playbook Reference:** Phase 2 · Step 2.3
**Branch:** 🏗️ VENTURE ARCHITECTURE
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document outlines the plan for setting up and managing a private container registry for the NINE² Incubator. A private registry is a critical piece of infrastructure that provides a secure, centralized location to store and distribute the Docker images for all Base Incubator services and forked Niche Tracks. This ensures that our intellectual property is protected and that deployments are consistent and reliable.

## 2. Registry Technology Selection

We will use **GitHub Container Registry (GHCR)** as our primary private registry. This choice is based on several key advantages:

- **Integration:** It is tightly integrated with our existing GitHub repositories and CI/CD workflows (`.github/workflows`).
- **Security:** It provides fine-grained access control, allowing us to link package access directly to repository permissions.
- **Cost-Effectiveness:** It offers a generous free tier for private repositories, which is sufficient for our initial development and can scale as we grow.
- **Simplicity:** It eliminates the need to manage a separate, self-hosted registry infrastructure.

## 3. Naming and Versioning Convention

To maintain clarity and avoid conflicts, all images pushed to the registry will follow a strict naming and versioning convention.

### Image Naming Schema:

`ghcr.io/<OWNER>/<TRACK_SLUG>-<SERVICE_NAME>:<VERSION>`

- **`<OWNER>`:** The GitHub organization or user name (e.g., `NINE2-Incubator`).
- **`<TRACK_SLUG>`:** The URL-friendly slug from `cohort-config.yaml` (e.g., `nurse-venture-builder`).
- **`<SERVICE_NAME>`:** The name of the service from `docker-compose.yaml` (e.g., `orchestrator`, `intake-agent`).
- **`<VERSION>`:** The semantic version of the image.

**Example:** `ghcr.io/nine2-incubator/nurse-venture-builder-orchestrator:1.0.0`

### Semantic Versioning (SemVer):

All images will be versioned using the **Major.Minor.Patch** format (e.g., `1.0.0`).

- **MAJOR** version is incremented for incompatible API changes.
- **MINOR** version is incremented for adding functionality in a backward-compatible manner.
- **PATCH** version is incremented for backward-compatible bug fixes.

This ensures that we can deploy updates and bug fixes without breaking existing tracks, and it provides a clear system for managing dependencies.

## 4. CI/CD Workflow Integration

The `fork-track.sh` script will be responsible for the initial build and push of a new track's images. Subsequently, our CI/CD workflow (defined in `.github/workflows/publish.yml`) will automate the process of building, tagging, and pushing images to GHCR on every merge to the `main` branch. The workflow will automatically extract the version number from a `VERSION` file in the repository to tag the images correctly.

## 5. Security and Access Control

- All container images will be stored as **private** packages within the GitHub organization.
- Access to the packages will be granted based on repository roles. Only members with `write` access to the `nine2-base-incubator` repository will be able to push new images.
- Deployment environments (e.g., production servers) will use a fine-grained Personal Access Token (PAT) with only `read:packages` scope to pull images from the registry, adhering to the principle of least privilege.

This plan provides a secure, scalable, and maintainable solution for managing the containerized components of the NINE² ecosystem.

---

*AI draft; nurse reviews; nurse authorizes.*
