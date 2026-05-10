# Tiered Architecture and Support Model

## Document Purpose
This document defines the composability boundaries of the Open Composable Integration Platform. It establishes the 80/20 architectural rule, categorizes platform components into strict tiers (Tier A, B, and C), and outlines the official Support Matrix. This model ensures that the platform remains a scalable, productized platform rather than a fragmented consulting project.

---

## 1. The 80/20 Architecture Principle

### Statement
The platform enforces an "Opinionated but Extensible" architecture based on the 80/20 rule: 80% of the platform consists of an opinionated, non-negotiable standard, while 20% allows for controlled customization and component replacement.

### Rationale
Total architectural freedom ("composable architecture") inevitably leads to chaos, unmaintainable infrastructure, broken governance, and exponentially increasing support costs. By locking down the operational standard while providing controlled escape hatches for specific infrastructure components, the platform scales efficiently across multiple tenants and business units.

### Implications
* Customization is allowed exclusively through predefined contracts and native **Component Swaps**, rejecting the use of proprietary SDKs, and never through direct modification of the platform core.
* The platform provides a "Golden Path" with officially supported profiles; deviations from these profiles downgrade the support SLA.

---

## 2. The Three-Tier Composability Model

To safely manage extensibility, every component within the ecosystem is strictly classified into one of three tiers.

### Tier A: Non-Negotiable Core (Platform as a Product)
These components define the operating model and cannot be replaced or bypassed by the client under any circumstances.
* **Included Components:** Governance model, GitOps deployment model (ArgoCD), observability contracts, security baselines, API standards, CI/CD conventions, and the platform operating model
* **Rule:** Modifying Tier A components breaks the platform contract and instantly voids operational support guarantees.

### Tier B: Replaceable Infrastructure Components
These are the underlying execution engines and storage mechanisms. Clients may replace these with their own corporate standards, provided the replacements fulfill the strict integration contracts.
* **Included Components:** Message brokers (e.g., Kafka), API Gateways (e.g., APISIX/Kong), Identity Providers (e.g., Keycloak), Secrets Management (e.g., Vault), and Monitoring Stacks (e.g., Prometheus).
* **Rule:** Tier B replacements are executed as a **Component Swap** rather than an adapter development project. The deployment team simply updates the declarative Infrastructure as Code (IaC) templates to use the target system's native component.

### Tier C: Customer-Owned Extensions
This tier belongs entirely to the domain teams (the users of the platform). It contains the actual business implementations built on top of the platform.
* **Included Components:** Custom connectors, domain logic, industry-specific workflows, custom flows, and compliance-specific additions.
* **Rule:** The platform guarantees the execution and observability of Tier C assets, but the business logic itself is owned and maintained by the client.

---

## 3. Supported Reference Profiles and Commercial Impact

Instead of attempting to support infinite combinations of tools, the platform categorizes deployments into officially supported profiles. The level of architectural flexibility chosen directly impacts the commercial model, operational complexity, and time-to-production.

### Profile A: Full OSS Stack
The default, opinionated out-of-the-box experience using the full open-source stack (Kafka + Keycloak + Vault + APISIX).
* **Operational Impact:** Fastest time-to-production, lowest maintenance overhead, and seamless automated upgrades.
* **Commercial Impact:** Most cost-effective profile.

### Profile B: Enterprise Hybrid
Combines the platform core (Control Plane) with existing enterprise assets replacing Tier B components (e.g., client IAM + client broker + platform governance).
* **Operational Impact:** Requires the client to assume maintenance for the swapped infrastructure and participate in joint troubleshooting during incidents.
* **Commercial Impact:** Subject to a higher pricing tier due to the increased complexity of support and integration.

### Profile C: Regulated Environment
A highly restricted profile designed for environments with extreme compliance requirements, operating entirely air-gapped and on-premise.
* **Operational Impact:** Audit-heavy environments requiring dedicated, isolated deployment pipelines.

### Profile D: Cloud Native
Replaces OSS infrastructure with managed cloud services alongside Kubernetes-native delivery.
* **Operational Impact:** Offloads infrastructure maintenance to the cloud vendor while keeping the Control Plane intact.

---

## 4. Official Support Matrix Definitions

To protect the platform from becoming a custom consulting project, brutal clarity is established regarding what is supported, what is compatible, and what is strictly prohibited.

### Support Level Definitions

1. **Supported:** The platform provides native integration, automated provisioning, and the Platform Team guarantees Service Level Agreements (SLA).
2. **Compatible:** The third-party component fulfills the integration contracts. The client executes a **Component Swap** via native reconfigurations, assuming operational responsibility for their physical infrastructure (e.g., maintaining the corporate IAM or broker). Best-effort support is provided for platform reachability.
3. **Unsupported:** The component violates fundamental contracts and its deployment is blocked by platform governance.

### Component Support Matrix

| Component Category | Technology / Vendor | Status | Support Boundary & Ownership |
| :--- | :--- | :--- | :--- |
| **Event Backbone** | Apache Kafka | **Supported** | Platform provides full IaC, monitoring, and SLA. |
| **Event Backbone** | RabbitMQ | **Supported** | Platform provides full IaC, monitoring, and SLA. |
| **Event Backbone** | Corporate Broker (e.g., IBM MQ) | **Compatible** | **Component Swap.** Client reconfigures IaC templates; assumes infrastructure maintenance. |
| **Event Backbone** | Legacy FTP Queues | **Unsupported** | Violates async contracts; deployment blocked. |
| **API Management** | Apache APISIX | **Supported** | Native integration managed by Platform. |
| **API Management** | Kong OSS | **Supported** | Native integration managed by Platform. |
| **API Management** | Client-Owned Gateway | **Compatible** | **Component Swap.** Client reconfigures routing IaC; assumes infrastructure maintenance. |
| **Identity & Access** | Keycloak | **Supported** | Native RBAC/OIDC integration managed by Platform. |
| **Identity & Access** | Corporate IAM (e.g., Entra ID) | **Compatible** | **Component Swap.** Client manages OIDC/SAML federation and role mapping. |
| **Secrets Management** | HashiCorp Vault | **Supported** | Native integration managed by Platform. |
| **Secrets Management** | Client Secrets Manager | **Compatible** | **Component Swap.** Client configures native external-secrets integration. |
| **Observability** | Prometheus Stack | **Supported** | Native integration; full dashboards and alerting. |
| **Observability** | Client Monitoring Stack | **Compatible** | **Component Swap.** Client configures standard exporters to their stack. |
| **Deployment Tooling** | ArgoCD | **Supported** | Native GitOps engine; Platform provides deployment governance. |