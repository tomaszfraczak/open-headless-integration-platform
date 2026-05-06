# OCIP: Contract and Lifecycle Governance

## Document Purpose
This document establishes the governance model for the creation, versioning, and lifecycle management of all integration contracts (APIs and Event Streams) on the Open Composable Integration Platform (OCIP). Strict lifecycle governance prevents the proliferation of undocumented, unmaintained, and permanently legacy integrations, ensuring long-term platform agility.

---

## 1. Design-First Contract Governance

### Statement
No integration code may be written or deployed without a formally defined and approved architectural contract. 

### Rationale
In a decentralized, composable architecture, domain teams must rely on the stability of interfaces provided by other teams. Writing code before defining the contract leads to tight coupling, undocumented behaviors, and integration failures. The contract acts as the absolute source of truth.

### Execution
* **Synchronous Interfaces:** Must be defined using the **OpenAPI Specification (OAS)**.
* **Asynchronous Interfaces:** Must be defined using the **AsyncAPI Specification**.
* **Git Repository as Registry:** These specifications must be committed to the designated central Git repository. The CI/CD pipelines will automatically publish these contracts to the platform's API/Event Developer Portal or documentation hub.

---

## 2. Strict Semantic Versioning (SemVer)

### Statement
All integration contracts and routing assets must strictly adhere to Semantic Versioning (MAJOR.MINOR.PATCH).

### Rationale
Consumers of an API or event stream need absolute predictability. They must know if an update is safe to adopt or if it requires them to rewrite their consuming applications.

### Execution
* **MAJOR (e.g., v1 to v2):** Introduced only for breaking changes (e.g., removing a mandatory field, changing a data type, modifying the URI structure). Major versions must be exposed as distinct endpoints (e.g., `/api/v1/orders` vs. `/api/v2/orders`).
* **MINOR (e.g., v1.1 to v1.2):** Introduced for backward-compatible additions (e.g., adding a new optional field to a JSON payload or a new endpoint).
* **PATCH (e.g., v1.1.0 to v1.1.1):** Introduced for backward-compatible bug fixes within the integration logic that do not affect the contract interface.
* **Event Streams:** For Kafka/RabbitMQ topics, a major breaking change requires publishing to an entirely new topic (e.g., `domain.entity.event.v2`).

---

## 3. The Contract Lifecycle Stages

Every integration endpoint and event topic published on OCIP must operate within one of the following explicitly defined lifecycle stages:

1. **Draft:** The contract is being designed and negotiated. No production deployment exists.
2. **Active:** The contract is live in production. It is the recommended version for all new consumers. Full SLA applies.
3. **Deprecated:** The contract is still live and supported for existing consumers, but is explicitly marked as deprecated in the API Gateway and documentation. **New consumers are strictly blocked from subscribing.**
4. **Retired:** The integration route is deleted from the platform, and the endpoints/topics are permanently disabled.

---

## 4. The Deprecation and Sunset Policy

### Statement
OCIP enforces a strict "Sunset Policy" to force the retirement of legacy integrations. The Platform Team will not support multiple legacy versions of the same integration indefinitely.

### Rationale
Without a forced sunset policy, domain teams will never migrate to newer versions, forcing the platform to maintain legacy code, outdated security protocols, and redundant compute resources. This directly degrades the platform's ROI and operational scalability.

### Execution
* **Notification:** When a domain team releases a new MAJOR version (e.g., v2), the previous version (v1) automatically enters the **Deprecated** stage.
* **Grace Period:** The Deprecated version will remain operational for a strictly enforced Grace Period (e.g., 6 months).
* **Sunset Execution:** Once the Grace Period expires, the API Gateway will begin returning `HTTP 410 Gone` for synchronous APIs, and access to legacy event topics will be revoked. It is the responsibility of the consuming teams to migrate within the provided window, not the responsibility of the platform to keep the legacy route alive.
