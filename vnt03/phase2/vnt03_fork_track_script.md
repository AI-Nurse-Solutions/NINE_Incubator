# NINE² Incubator — fork-track.sh Script Design

**Playbook Reference:** Phase 2 · Step 2.2
**Branch:** 🏗️ VENTURE ARCHITECTURE
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

The `fork-track.sh` script is the core automation engine for the NINE² horizontal scaling strategy. Its sole purpose is to take the `nine2-base-incubator` repository and a completed `cohort-config.yaml` file and, in a single, automated process, produce a new, fully configured, and ready-to-deploy niche incubator track. This script embodies **Foundational Doctrine 1: The Factory Beats the Curriculum** by turning the process of creating a new incubator from a months-long effort into a minutes-long automated task.

## 2. Script Logic & Workflow

The script will execute the following sequence of operations:

| Step | Action | Description |
| :--- | :--- | :--- |
| 1 | **Initialization** | The script takes one argument: the path to the `cohort-config.yaml` file. It validates the file exists and parses key variables (`track_slug`, `track_id`, etc.) using a YAML parser like `yq`. |
| 2 | **Clone Base Repo** | It performs a fresh `git clone` of the `nine2-base-incubator` repository into a new directory named after the `track_slug`. |
| 3 | **Inject Configuration** | It copies the provided `cohort-config.yaml` into the new repository, overwriting the template file. |
| 4 | **Generate Environment** | It reads all key-value pairs from the `cohort-config.yaml` and generates a `.env` file in the root of the new repository. This file will be used by `docker-compose` to configure the services. |
| 5 | **Build & Tag Images** | It iterates through all the services defined in the `docker-compose.yaml` (orchestrator, agents, dashboard) and builds the Docker images. Each image is tagged with the track-specific name and version (e.g., `nine2-registry/nurse-venture-builder-orchestrator:1.0.0`). |
| 6 | **Push to Registry** | It pushes all the newly built and tagged images to the designated private container registry (specified in the environment). |
| 7 | **Generate Manifests** | It runs a script to generate the initial governance artifacts, such as `track-manifest.json` and `agent-registry.json`, based on the contents of the `cohort-config.yaml`. |
| 8 | **Cleanup** | It removes the local cloned repository, leaving only the pushed Docker images and the generated artifacts as the final output. |

## 3. Pseudocode

```bash
#!/bin/bash

# Step 1: Initialization
CONFIG_FILE=$1
if [ ! -f "$CONFIG_FILE" ]; then
  echo "Error: Config file not found!" >&2
  exit 1
fi

TRACK_SLUG=$(yq e ".track.slug" $CONFIG_FILE)
TRACK_VERSION="1.0.0" # Or derive from config
DOCKER_REGISTRY="nine2-registry"

# Step 2: Clone Base Repo
REPO_DIR="./$TRACK_SLUG"
git clone https://github.com/NINE2/nine2-base-incubator.git "$REPO_DIR"

# Step 3: Inject Configuration
cp "$CONFIG_FILE" "$REPO_DIR/governance-artifacts/cohort-config.yaml"

# Step 4: Generate Environment
cd "$REPO_DIR"
yq e -o=props "." governance-artifacts/cohort-config.yaml > .env
# Add other necessary env vars
echo "DOCKER_REGISTRY=$DOCKER_REGISTRY" >> .env
echo "TRACK_VERSION=$TRACK_VERSION" >> .env

# Step 5 & 6: Build & Push Images
docker-compose build
docker-compose push

# Step 7: Generate Manifests
./scripts/generate-manifests.sh

# Step 8: Cleanup
cd ..
rm -rf "$REPO_DIR"

echo "Track '$TRACK_SLUG' forked and deployed to registry successfully."
```

## 4. Governance

The `fork-track.sh` script is a powerful tool and its execution is considered a **🟡 YELLOW** tier action. While the script itself is Green-tier to design and build, running it to create a new track requires nurse-founder approval via a Gate Card. This ensures that the creation of new incubator tracks remains a deliberate, governed process.

---

*AI draft; nurse reviews; nurse authorizes.*
