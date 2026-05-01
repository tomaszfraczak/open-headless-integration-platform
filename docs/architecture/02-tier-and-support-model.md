# OCIP: Tiered Architecture and Support Model

## Document Purpose
This document defines the composability boundaries of the Open Composable Integration Platform (OCIP). It establishes the 80/20 architectural rule, categorizes platform components into strict tiers (Tier A, B, and C), and outlines the official Support Matrix. This model ensures that OCIP remains a scalable, productized platform rather than a fragmented consulting project[cite: 10].

---

## 1. The 80/20 Architecture Principle

### Statement
OCIP enforces an "Opinionated but Extensible" architecture based on the 80/20 rule: 80% of the platform consists of an opinionated, non-negotiable standard, while 20% allows for controlled customization and component replacement[cite: 10].

### Rationale
Total architectural freedom ("composable chaos") inevitably leads to unmaintainable infrastructure, broken governance, and exponentially increasing support costs[cite: 10]. By locking down the operational standard (the 80%) while providing controlled escape hatches for specific integrations and infrastructure (the 20%), the platform scales efficiently across multiple tenants and business units[cite: 10, 12].

### Implications
* Customization is allowed exclusively through predefined adapter frameworks and contracts, never through direct modification of the platform core[cite: 10].
* The platform provides a "Golden Path" with officially supported profiles; deviations from these profiles downgrade the support SLA from "Supported" to "Compatible" or "Unsupported"[cite: 10].

---

## 2. The Three-Tier Composability Model

To safely manage extensibility, every component within the OCIP ecosystem is strictly classified into one of three tiers[cite: 10].

### Tier A: Non-Negotiable Core (Platform as a Product)
These components define the OCIP operating model and cannot be replaced or bypassed by the client under any circumstances[cite: 10].
* **Included Components:** Governance model, GitOps deployment model (ArgoCD), observability contracts, security baselines, API standards, CI/CD conventions, and the platform operating model[cite: 10].
* **Rule:** Modifying Tier A components breaks the platform contract and instantly voids operational support guarantees.

### Tier B: Replaceable Infrastructure Components
These are the underlying execution engines and storage mechanisms. Clients may replace these with their own corporate standards, provided the replacements fulfill the strict OCIP integration contracts (e.g., the *Event Backbone Contract*)[cite: 10].
* **Included Components:** 
  * Message Broker: Kafka ↔ corporate broker (e.g., IBM MQ)[cite: 10].
  * API Gateway: APISIX/Kong ↔ client gateway[cite: 10].
  * Identity Provider: Keycloak ↔ corporate IAM (e.g., Azure AD)[cite: 10].
  * Secrets Management: HashiCorp Vault ↔ client secrets manager[cite: 10].
* **Rule:** Tier B replacements require mapping via the platform's Adapter Framework[cite: 10].

### Tier C: Customer-Owned Extensions
This tier belongs entirely to the domain teams (the users of the platform). It contains the actual business implementations built on top of OCIP[cite: 10].
* **Included Components:** Custom connectors, business domain logic, industry-specific workflows, and data transformations (e.g., Quarkus microservices and Camel routes)[cite: 10].
* **Rule:** The platform guarantees the *execution* and *observability* of Tier C assets, but the business logic itself is owned and maintained by the client[cite: 10].

---

## 3. Supported Reference Profiles

Instead of attempting to support infinite combinations of tools, OCIP offers predefined, certified deployment profiles[cite: 10].

1. **Profile A (Full OSS Stack):** The default, out-of-the-box experience. Uses Kafka, Keycloak, Vault, and APISIX. Guaranteed highest level of support and fastest time-to-production[cite: 10].
2. **Profile B (Enterprise Hybrid):** Combines OCIP core with existing enterprise assets (e.g., Corporate IAM + Corporate Message Broker + OCIP Governance)[cite: 10].
3. **Profile C (Regulated Environment):** Highly restricted profile designed for air-gapped, on-premise, and audit-heavy environments[cite: 10].
4. **Profile D (Cloud Native):** Replaces OSS infrastructure with managed cloud services (e.g., Azure Event Hubs instead of Kafka, managed Kubernetes delivery)[cite: 10].

---

## 4. Official Support Matrix

To establish clear operational boundaries, OCIP categorizes infrastructure integrations into three support levels[cite: 10].

| Component Category | Solution / Vendor | Status | Description |
| :--- | :--- | :--- | :--- |
| **Event Backbone** | Apache Kafka | **Supported** | Native integration, fully tested, full SLA[cite: 10]. |
| **Event Backbone** | RabbitMQ | **Supported** | Native integration, fully tested, full SLA[cite: 10]. |
| **Event Backbone** | IBM MQ | **Compatible** | Meets contracts; requires custom adapter. Best-effort support[cite: 10]. |
| **Event Backbone** | Legacy FTP Queues | **Unsupported** | Does not meet async/replay contracts. Will not execute on OCIP[cite: 10]. |
| **API Management** | APISIX / Kong | **Supported** | Native integration, automated policy enforcement. |
| **Identity / IAM** | Keycloak | **Supported** | Native RBAC and OIDC integration. |

*(Note: This matrix is actively maintained and updated as new technologies are evaluated against OCIP contracts.)*
