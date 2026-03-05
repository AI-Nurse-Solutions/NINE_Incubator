# NINE² Incubator — Governance Discoverability Test Suite Design

**Playbook Reference:** Phase 4 · Step 4.5
**Branch:** 🔍 MARKET VISIBILITY
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document designs the Governance Discoverability Test Suite. This is an automated test suite that programmatically verifies that a forked niche track is correctly exposing all its required public governance artifacts. Its purpose is to provide a concrete, pass/fail mechanism to ensure that our commitment to transparency is being met technically. This suite will be run as part of the CI/CD pipeline for every new track deployment.

## 2. Test Suite Design

The test suite will be a script that simulates the actions of a skeptical third party attempting to verify a track's governance from the outside-in, using only public endpoints.

### Test Sequence:

| Test # | Action | Expected Outcome | Pass/Fail Criteria |
| :--- | :--- | :--- | :--- |
| 1 | **Fetch Track Manifest** | HTTP 200 OK | The script successfully fetches `track-manifest.json` from the public API endpoint. The file is valid JSON and contains the expected `track_id`. |
| 2 | **Parse Manifest Endpoints** | Extract `agent_registry` and `governance_log_summary` URLs. | The manifest contains valid, well-formed URLs for both required endpoints. |
| 3 | **Fetch Agent Registry** | HTTP 200 OK | The script successfully fetches the `agent-registry.json` file from the URL provided in the manifest. The file is valid JSON. |
| 4 | **Verify Steward Assignment** | The `governance_anchor` object exists and contains a name and credential level. | The `name` field is not empty, and the `naio_credential_level` is one of `NAIO-C`, `NAIO-A`, or `NAIO-F`. |
| 5 | **Verify Agent Accountability** | Every agent in the `agents` array has a non-empty `agent_id` and `role`. | The script iterates through all agents and confirms that required fields are present and populated. |
| 6 | **Fetch Governance Summary** | HTTP 200 OK | The script successfully fetches the audit summary from the `governance_log_summary` endpoint. The file is valid JSON. |
| 7 | **Check for Recent Activity** | The `last_event_timestamp` is within the last 24 hours. | The timestamp is parsed and compared against the current time to ensure the audit log is live. |
| 8 | **Verify Credential Link** | The `verification_uri` for the steward in the agent registry is a valid URL. | The script confirms the URI format. (A full end-to-end test would require a valid credential ID to query the portal). |

## 3. Implementation

-   **Technology:** The test suite will be written in Python using the `requests` and `jsonschema` libraries.
-   **Execution:** It will be configured as a GitHub Actions workflow that runs automatically on a schedule (e.g., nightly) against all deployed tracks and can be triggered manually.
-   **Reporting:** The test suite will output its results in a standard format (e.g., JUnit XML), which can be easily integrated with the GitHub Actions UI for clear pass/fail reporting.

### Pseudocode:

```python
import requests

def run_discoverability_test(track_id):
    # Test 1: Fetch Track Manifest
    manifest = fetch_json(f"https://api.nine2.ai/v1/tracks/{track_id}/manifest.json")
    assert manifest["track_id"] == track_id, "Manifest ID mismatch"

    # Test 2: Parse Endpoints
    registry_url = manifest["endpoints"]["agent_registry"]
    audit_url = manifest["endpoints"]["governance_log_summary"]
    assert registry_url and audit_url, "Endpoint URLs missing"

    # Test 3 & 4: Fetch Agent Registry and Verify Steward
    registry = fetch_json(registry_url)
    assert registry["governance_anchor"]["name"], "Steward name missing"

    # Test 5: Verify Agent Accountability
    for agent in registry["agents"]:
        assert agent["agent_id"] and agent["role"], "Agent details missing"

    # Test 6 & 7: Fetch Governance Summary and Check Activity
    audit_summary = fetch_json(audit_url)
    assert is_recent(audit_summary["last_event_timestamp"]), "Audit log is stale"

    print(f"All discoverability tests PASSED for track {track_id}")

```

This test suite provides an automated, objective measure of our commitment to transparency. A passing suite is a verifiable signal that a NINE² track is operating in accordance with its stated governance principles.

---

*AI draft; nurse reviews; nurse authorizes.*
