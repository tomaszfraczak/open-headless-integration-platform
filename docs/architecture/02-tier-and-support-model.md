# OCIP: Tiered Architecture and Support Model

## Document Purpose
This document defines the composability boundaries of the Open Composable Integration Platform (OCIP). It establishes the 80/20 architectural rule, categorizes platform components into strict tiers (Tier A, B, and C), and outlines the official Support Matrix. This model ensures that OCIP remains a scalable, productized platform rather than a fragmented consulting project.

---

## 1. The 80/20 Architecture Principle

### Statement
OCIP enforces an "Opinionated but Extensible" architecture based on the 80/20 rule: 80% of the platform consists of an opinionated, non-negotiable standard, while 20% allows for controlled customization and component replacement.

### Rationale
Total architectural freedom ("composable chaos") inevitably leads to unmaintainable infrastructure, broken governance, and exponentially increasing support costs. By locking down the operational standard (the 80%) while providing controlled escape hatches for specific integrations and infrastructure (the 20%), the platform scales efficiently across multiple tenants and business units.

### Implications
* Customization is allowed exclusively through predefined adapter frameworks and contracts, never through direct modification of the platform core.
* The platform provides a "Golden Path" with officially supported profiles; deviations from these profiles downgrade the support SLA from "Supported" to "Compatible" or "Unsupported".

---

## 2. The Three-Tier Composability Model

To safely manage extensibility, every component within the OCIP ecosystem is strictly classified into one of three tiers.

### Tier A: Non-Negotiable Core (Platform as a Product)
These components define the OCIP operating model and cannot be replaced or bypassed by the client under any circumstances.
* **Included Components:** Governance model, GitOps deployment model (ArgoCD), observability contracts, security baselines, API standards, CI/CD conventions, and the platform operating model.
* **Rule:** Modifying Tier A components breaks the platform contract and instantly voids operational support guarantees.

### Tier B: Replaceable Infrastructure Components
These are the underlying execution engines and storage mechanisms. Clients may replace these with their own corporate standards, provided the replacements fulfill the strict OCIP integration contracts (e.g., the *Event Backbone Contract*).
* **Included Components:** 
  * Message Broker: Kafka ↔ corporate broker (e.g., IBM MQ).
  * API Gateway: APISIX/Kong ↔ client gateway.
  * Identity Provider: Keycloak ↔ corporate IAM (e.g., Azure AD).
  * Secrets Management: HashiCorp Vault ↔ client secrets manager.
* **Rule:** Tier B replacements require mapping via the platform's Adapter Framework.

### Tier C: Customer-Owned Extensions
This tier belongs entirely to the domain teams (the users of the platform). It contains the actual business implementations built on top of OCIP.
* **Included Components:** Custom connectors, business domain logic, industry-specific workflows, and data transformations (e.g., Quarkus microservices and Camel routes).
* **Rule:** The platform guarantees the *execution* and *observability* of Tier C assets, but the business logic itself is owned and maintained by the client.

---

## 3. Supported Reference Profiles and Commercial Impact

Instead of attempting to support infinite combinations of tools, OCIP categorizes deployments into officially supported profiles. Crucially, the level of architectural flexibility chosen by the client directly impacts the commercial model, operational complexity, and time-to-production. Greater deviation from the standard equals higher operational cost.

### Profile A: Standard OSS Baseline (Highly Recommended)
The default, opinionated out-of-the-box experience using the full OCIP open-source stack (Kafka, Keycloak, Vault, APISIX).
* **Operational Impact:** Fastest time-to-production, lowest maintenance overhead, and seamless automated upgrades.
* **Commercial Impact:** Most cost-effective profile, included in the standard platform subscription.

### Profile B: Enterprise Hybrid
Combines the OCIP core (Control Plane) with existing enterprise assets replacing Tier B components (e.g., integrating the client's corporate IAM or an existing corporate message broker like IBM MQ).
* **Operational Impact:** Requires network peering, custom adapter maintenance, and joint troubleshooting during incidents.
* **Commercial Impact:** Subject to a premium pricing tier due to the increased complexity of support and integration.

### Profile C: Regulated & Air-Gapped Environment
A highly restricted profile designed for environments with extreme compliance requirements, operating entirely on-premise without external internet access.
* **Operational Impact:** Prevents the use of standard SaaS observability tools. Requires dedicated, isolated deployment pipelines and manual updates.
* **Commercial Impact:** Custom enterprise pricing model.

---

## 4. Official Support Matrix Definitions

To protect the platform from becoming a chaotic consulting project, OCIP establishes brutal clarity regarding what is supported, what is merely compatible, and what is strictly prohibited. 

### Support Level Definitions

1. **Supported (Out-of-the-Box):**
   * The platform provides native Infrastructure as Code (IaC) provisioning for this component.
   * The OCIP Platform Team maintains the integration adapter, guarantees Service Level Agreements (SLA), and actively monitors the component's health.

2. **Compatible (Bring Your Own Integration):**
   * The third-party component technically fulfills the OCIP integration contracts (e.g., it supports async pub/sub and DLQ).
   * **Crucial Boundary:** The client or a system integrator is fully responsible for building, configuring, and maintaining the adapter framework. The OCIP Platform Team will guarantee that the platform can send/receive data via the contract, but will not troubleshoot the internal workings of the compatible component.

3. **Unsupported (Prohibited):**
   * The component violates fundamental OCIP contracts (e.g., a legacy FTP queue that does not support replayability or dead-letter handling).
   * The platform's automated governance policies will block the deployment of any unsupported component to protect ecosystem stability.

### Component Support Matrix (Example)

| Component Category | Technology / Vendor | Status | Support Boundary & Ownership |
| :--- | :--- | :--- | :--- |
| **Event Backbone** | Apache Kafka | **Supported** | OCIP provides full IaC, monitoring, and SLA. |
| **Event Backbone** | RabbitMQ | **Supported** | OCIP provides full IaC, monitoring, and SLA. |
| **Event Backbone** | IBM MQ | **Compatible** | Client maintains the custom adapter framework. |
| **Event Backbone** | Legacy FTP Queues | **Unsupported** | Lacks async contracts; deployment blocked. |
| **Identity / IAM** | Keycloak | **Supported** | Native RBAC/OIDC integration managed by OCIP. |
| **Identity / IAM** | Custom Corporate Active Directory | **Compatible** | Client maps roles to OCIP RBAC contracts. |
