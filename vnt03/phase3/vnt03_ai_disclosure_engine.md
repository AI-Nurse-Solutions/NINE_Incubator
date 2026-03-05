# NINE² Incubator — AI Disclosure Engine Specification

**Playbook Reference:** Phase 3 · Step 3.4
**Branch:** 🏗️ VENTURE ARCHITECTURE
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document specifies the design of the AI Disclosure Engine. The purpose of this engine is to enforce **Standing Doctrine 2: Nurse authority is the product, not the user.** It ensures that every artifact produced by an AI agent is clearly and consistently marked as such before it is delivered to a human. The mandatory disclosure tag, **"AI draft; nurse reviews; nurse authorizes,"** is not a suggestion; it is a non-negotiable governance requirement.

This engine prevents the accidental or intentional misrepresentation of AI-generated content as human work, which is fundamental to building institutional trust and maintaining the integrity of the nurse-governed model.

## 2. Engine Architecture

The AI Disclosure Engine will be a simple, stateless service integrated into the final output stage of the Florence-X Orchestrator. Before any artifact (e.g., a markdown file, an email draft, a report) is saved to disk or sent to a user, it will be passed through this engine.

### Logical Flow:

1.  **Artifact Interception:** The Reporting Agent or Execution Agent finalizes an artifact.
2.  **Configuration Check:** The Orchestrator checks the `governance.ai_disclosure` setting in the `cohort-config.yaml`. If set to `required`, it passes the artifact to the Disclosure Engine.
3.  **Tag Injection:** The engine inspects the artifact's content. If the disclosure tag is not present, the engine appends it to the end of the document.
4.  **Output:** The engine returns the modified artifact, which is then saved or delivered.

## 3. Implementation Details

- **Implementation:** This can be a simple Python function within the Orchestrator's codebase.
- **Idempotency:** The engine must be idempotent. If the tag already exists in the document, it should not add it a second time. This prevents duplicate tags on artifacts that are modified multiple times.
- **Format Awareness:** The engine should be aware of the artifact's format to apply the tag correctly (e.g., as a footer in markdown, a signature in an email, or metadata in an image).

### Pseudocode:

```python
DISCLOSURE_TAG = "\n---\n\n*AI draft; nurse reviews; nurse authorizes.*"

def apply_disclosure(artifact_content, artifact_format="markdown"):
    """Applies the standard AI disclosure tag if it's not already present."""
    
    # Check if the tag already exists to ensure idempotency
    if DISCLOSURE_TAG in artifact_content:
        return artifact_content

    # Append the tag based on format (can be expanded)
    if artifact_format == "markdown":
        return artifact_content + DISCLOSURE_TAG
    elif artifact_format == "email":
        return artifact_content + "\n\n" + DISCLOSURE_TAG.replace("\n---\n\n", "")
    else:
        # For other formats, default to appending at the end
        return artifact_content + DISCLOSURE_TAG

```

## 4. Governance Enforcement

- The `governance.ai_disclosure: required` setting in `cohort-config.yaml` is **not optional** for any track involving clinical or professional content.
- The Compliance Agent will periodically sample artifacts from the memory store to verify that the disclosure tag is present, reporting any failures to the Audit Engine.
- Bypassing this engine is a **🔴 RED** tier violation and will trigger an immediate escalation.

This simple but rigid engine is a cornerstone of the NAIO framework. It makes the role of the AI explicit in every interaction, constantly reinforcing that the AI is a tool for the nurse, and the nurse remains the final, accountable authority.

---

*AI draft; nurse reviews; nurse authorizes.*
