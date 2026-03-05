# NINE² Incubator — Billing Engine Integration Specification

**Playbook Reference:** Phase 7 · Step 7.2
**Branch:** 📊 REVENUE ARCHITECTURE
**EDENA Tier:** 🟢 GREEN
**Status:** COMPLETE

---

## 1. Purpose

This document specifies the design for integrating a billing engine to manage the NINE² pricing model. The engine must handle the initial one-time Cohort Access Fee and provide a mechanism for tracking and executing the Outcome-Based Fee. The primary goal is to automate the payment and tracking process in a secure and reliable manner.

## 2. Technology Selection

- **Payment Processor:** **Stripe** will be used as the primary payment processor due to its robust API, extensive documentation, and strong security posture.
- **Integration:** The integration will be handled by a dedicated `billing-service` container, which will act as the intermediary between the NINE² platform and the Stripe API.

## 3. System Architecture and Flow

### Stage 1: Cohort Access Fee Payment

1.  **Invoice Generation:** When a nurse founder is accepted into a cohort, the NAIO Governance Anchor triggers an action to create a new customer and invoice in Stripe.
2.  **Payment Link:** The `billing-service` generates a unique, secure Stripe payment link for the $5,000 Cohort Access Fee.
3.  **Notification:** The system sends an email to the founder with the payment link and instructions.
4.  **Payment Confirmation:** Upon successful payment, Stripe sends a webhook to the `billing-service`.
5.  **Activation:** The `billing-service` receives the webhook, verifies the payment, and updates the venture's status in the `cohort-config.yaml` to `active`. This automatically grants the founder access to the incubator's resources.

### Stage 2: Outcome-Based Fee Tracking

1.  **Milestone Tracking:** The platform will include a simple, internal-facing dashboard for the NAIO Governance Anchor to track the progress of each venture toward the outcome-based fee triggers (first institutional contract, funding round, acquisition).
2.  **Manual Trigger:** When a trigger event is confirmed, the NAIO Governance Anchor will manually update the venture's status in this dashboard.
3.  **Legal Workflow:** Updating the status to `outcome_fee_triggered` will initiate a **🟡 YELLOW** tier gated workflow. This workflow does not involve the billing engine but instead notifies the NINE² legal and finance team to begin the process of executing the 4% equity agreement. The billing engine's role is purely to track that the initial fee was paid.

## 4. API and Service Specification

### `billing-service`

-   **Responsibilities:**
    -   Create and manage customer objects in Stripe.
    -   Generate one-time invoices and payment links for the Cohort Access Fee.
    -   Listen for and process webhooks from Stripe for payment confirmation.
    -   Update the `cohort-config.yaml` or a central status database upon successful payment.
-   **API (Internal):**
    -   `POST /invoices`: Creates a new invoice for a specified cohort member.
    -   `POST /webhooks/stripe`: The public endpoint to receive webhooks from Stripe.

### Stripe API Usage

-   **Customers API:** To create a new customer record for each venture.
-   **Products & Prices API:** To define the "Cohort Access Fee" as a standard product.
-   **Invoices API:** To create a one-time invoice for the access fee.
-   **Payment Links API:** To generate the secure, shareable payment link.
-   **Webhooks:** To receive real-time notifications of payment events.

## 5. Security

-   All communication with the Stripe API will use secret keys stored securely in the environment variables of the `billing-service` container, injected from a secure vault.
-   The Stripe webhook endpoint will be protected by verifying the signature of each incoming request to ensure it originated from Stripe.
-   The `billing-service` will have no access to raw credit card information. All payment processing is handled directly by Stripe's secure infrastructure.

This design automates the initial revenue collection process and establishes a clear, auditable system for tracking the venture's lifecycle toward the outcome-based fee, ensuring the financial model is implemented reliably.

---

*AI draft; nurse reviews; nurse authorizes.*
