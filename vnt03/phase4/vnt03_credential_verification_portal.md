# NINE² Incubator — Credential Verification Portal Specification

**Playbook Reference:** Phase 4 · Step 4.4
**Branch:** 🔍 MARKET VISIBILITY
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document specifies the design for the public-facing NAIO Credential Verification Portal. The portal provides a simple, secure, and trustworthy mechanism for anyone (e.g., a hospital administrator, a patient, a regulator) to verify the authenticity and status of a NAIO credential held by a nurse founder or a governance steward. This is the final link in the chain of discoverable governance, making our credentials as transparent and verifiable as our agent registries.

## 2. Portal Design and User Flow

The portal will be a simple, single-page web application with one function: verify a credential ID.

**User Flow:**

1.  A user navigates to the verification portal (e.g., `https://naio.org/verify`).
2.  They are presented with a single input field labeled "Enter Credential ID."
3.  The user enters the unique ID from a NAIO credential (e.g., `NAIOA-R-DOMONDON-2025-001`).
4.  They click "Verify."
5.  The portal calls the backend API to look up the credential.
6.  The result is displayed clearly on the screen.

**Result Display:**

-   **On Success:** A digital card is displayed with the following information:
    *   Credential Holder Name
    *   Credential Level (e.g., "NAIO Architect - NAIO-A")
    *   Issue Date
    *   Expiration Date
    *   Status (e.g., `ACTIVE`, `EXPIRED`, `REVOKED`)
    *   A green checkmark and the text "Verified Authentic Credential."
-   **On Failure:** A simple message is displayed: "Invalid or Not Found. Please check the Credential ID and try again."

## 3. API Endpoint Specification

The portal will be powered by a single, public, read-only API endpoint.

### `GET /v1/credentials/verify?id={credential_id}`

**Example Successful Response (200 OK):**

```json
{
  "credential_id": "NAIOA-R-DOMONDON-2025-001",
  "holder_name": "Robert Elvin Domondon",
  "credential_level": "NAIO-A",
  "credential_name": "NAIO Architect",
  "issue_date": "2025-01-15",
  "expiration_date": "2027-01-14",
  "status": "ACTIVE"
}
```

**Example Failure Response (404 Not Found):**

```json
{
  "error": "Credential not found."
}
```

## 4. Security and Integrity

-   **Rate Limiting:** The API endpoint will be rate-limited to prevent brute-force attacks or scraping of credential data.
-   **No Search/List Functionality:** The portal will only allow direct lookup by ID. There will be no way to search for or list credential holders.
-   **Database:** The backend will be a dedicated, read-only replica of the master credentialing database, ensuring the public-facing service cannot affect the source of truth.

This portal closes the loop on public accountability. It allows anyone to take a credential found in an `agent-registry.json` or on a founder's profile and instantly verify its legitimacy, providing a tangible and powerful tool for building trust.

---

*AI draft; nurse reviews; nurse authorizes.*
