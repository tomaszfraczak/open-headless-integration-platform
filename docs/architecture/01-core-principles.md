# Core Architecture Principles

## Document Purpose
This document defines the foundational architecture principles for the Open Composable Integration Platform. These principles represent the non-negotiable core (Tier A) of the platform. They dictate how integration solutions are designed, deployed, and managed, ensuring scalability, resilience, and operational excellence across the enterprise.

---

## Principle 1: Separation of Control Plane and Data Plane

### Statement
The platform architecture must strictly separate the management and governance layer (Control Plane) from the execution and data processing layer (Data Plane).

### Rationale
In enterprise integration, mixing business data processing with platform management leads to performance bottlenecks, security vulnerabilities, and difficulties in scaling. By separating these concerns, we ensure that a failure or heavy load in the management layer does not impact the execution of business-critical integration flows, and vice versa. It also enables multi-tenancy and the ability to manage distributed execution environments from a central hub.

### Implications
* **Data Plane:** Components such as Apache Camel, API Gateways (APISIX/Kong), and message brokers (Kafka/RabbitMQ) are strictly dedicated to executing integration flows, mediating traffic, and transforming data.
* **Control Plane:** Components responsible for governance, CI/CD (ArgoCD), policies (Open Policy Agent), and observability (Prometheus/Grafana) exist independently and oversee the Data Plane.
* **Security:** The Data Plane processes business data and payloads; the Control Plane only processes metadata, telemetry, and configurations.

---

## Principle 2: Contract-Based Composability (Replaceable Infrastructure)

### Statement
The platform is built on stable integration contracts rather than hard dependencies on specific vendor technologies or open-source tools.

### Rationale
To truly eliminate vendor lock-in and enable long-term adaptability, the platform must allow for the controlled replacement of underlying infrastructure components (Tier B). If an organization decides to move from Apache Kafka to an existing corporate IBM MQ cluster, the platform must support this transition without requiring a complete rewrite, provided the new component adheres to the established contract.

### Implications
* Integration flows must not tightly couple with proprietary APIs of the underlying message broker or gateway.
* **Support Matrix Enforcement:** The platform will define strict contracts (e.g., *Event Backbone Contract* requiring pub/sub, DLQ, and replay support). Any infrastructure component meeting this contract is considered a "Compatible" or "Supported" profile.
* The 80/20 Rule: 80% of the platform enforces a strict, opinionated operational standard, while 20% allows for controlled customization and replacement of infrastructure blocks.

---

## Principle 3: Stateless Integration Runtime

### Statement
The core integration execution engines (e.g., Apache Camel routes) must remain completely stateless.

### Rationale
Stateful integration components are difficult to scale horizontally, prone to data loss during pod evictions, and complicate disaster recovery. By enforcing a stateless runtime, we ensure that any integration pod can be terminated and recreated instantaneously by Kubernetes without losing business context or disrupting workflows.

### Implications
* **No In-Memory State:** Business state, orchestration progress, or session data must never be stored in the memory of the integration route.
* **State Delegation:** Long-running processes and stateful sagas must be delegated to dedicated Workflow & State engines (e.g., Temporal Technologies).
* **Caching & Persistence:** Operational state (like idempotency keys) must be offloaded to external persistence layers such as Redis or PostgreSQL.

---

## Principle 4: API-First and Event-Driven by Default

### Statement
Every system integration must be designed as a consumable API product or a decoupled event contract, strictly avoiding point-to-point database links or hidden file transfers.

### Rationale
Point-to-point integrations create a brittle, tightly coupled architecture that cannot scale and is impossible to govern. Shifting to an event-driven and API-first paradigm promotes reusability, asynchronous decoupling, and independent scaling of services.

### Implications
* **Event-Driven Preference:** Asynchronous messaging (using Kafka/RabbitMQ) is preferred over synchronous REST orchestration for system-to-system data synchronization.
* **Contracts First:** OpenAPI (Swagger) or AsyncAPI specifications must be defined and registered before any implementation begins.
* REST is not a universal hammer; protocols like gRPC, AMQP, or GraphQL should be evaluated based on the specific use case.

---

## Principle 5: Infrastructure as Code (IaC) and GitOps Delivery

### Statement
Manual configuration of environments, deployments, or policies is strictly prohibited. The entire platform lifecycle must be managed declaratively through code.

### Rationale
Enterprise scale requires absolute repeatability and auditability. Manual interventions lead to configuration drift, security vulnerabilities, and unpredictable disaster recovery times. "Golden Path Engineering" dictates that the platform provides a paved road for delivery, eliminating operational guesswork.

### Implications
* **Provisioning:** Cloud infrastructure and core services must be provisioned via Terraform.
* **Deployment:** Application states and integration routes must be deployed exclusively via GitOps controllers (e.g., ArgoCD) reacting to changes in Git repositories.
* **Auditability:** Git acts as the single source of truth and the primary audit trail for all changes applied to the platform.

---

## Principle 6: Native Observability and Security

### Statement
Security enforcement and telemetry collection are built-in platform primitives, not optional features to be added by developers.

### Rationale
In a distributed, headless integration architecture, operational visibility and strict access control are paramount. Developers should focus on business logic (thin routes), while the platform transparently handles logging, metrics extraction, and request authorization.

### Implications
* **Security by Design:** Every entry point must be protected by the API Gateway with mandatory authentication (e.g., OAuth2/OIDC via Keycloak) and secrets must be injected dynamically via Vault (no hardcoded credentials).
* **Observability by Default:** All integrations will automatically export distributed traces (OpenTelemetry), standardized metrics (Prometheus), and centralized logs (Loki) without requiring custom code from the integration developer.

---

## Principle 7: Layered Technology Stack and Open Source Tooling

To ensure strict separation of concerns, the Platform is divided into specialized layers utilizing enterprise-grade Open Source products. 

| Layer | Function | Recommended Open Source Product |
| :--- | :--- | :--- |
| **Edge / Access Layer** | Entry point, edge security, and API routing. | **Apache APISIX** (or Kong OSS) |
| **Experience Layer** | Developer Portal, API catalog, and documentation (OAS/AsyncAPI). | **Backstage.io** |
| **Integration Layer** | Mediation engine, data transformation, and business routing. | **Apache Camel (Kamel)** |
| **Processing Layer** | Runtime for microservices requiring high performance. | **Quarkus** |
| **Event / Messaging Layer** | Asynchronous data backbone (Event Backbone). | **Apache Kafka (Strimzi)** |
| **Security & Identity** | Single Sign-On (SSO), RBAC, and secrets management. | **Keycloak** & **HashiCorp Vault** |
| **Persistence Layer** | State storage, caching, and operational data. | **PostgreSQL** & **Redis** |
| **Registry & Artifacts** | Storage for container images and software packages. | **Harbor (Container Registry)** |
| **Automation / CI/CD** | GitOps and continuous software delivery. | **ArgoCD** & **Tekton** |
| **Observability Layer** | Monitoring, log aggregation, and distributed tracing. | **Prometheus, Grafana, Loki, Tempo** |

### The Container Registry
* **Platform Container Registry (Harbor):** Serves as the central repository for all container images. Before any image is registered, it undergoes automated scanning (e.g., via Trivy) for vulnerabilities (CVEs). Images with critical security flaws are automatically blocked from being deployed to production environments.

---

## Principle 8: Responsibility Model

The layered architecture enforces a clear **Separation of Concerns**:
1. **Platform Team (Infrastructure):** This team is responsible for the availability, updates, and security of the "layers" themselves (e.g., ensuring the Kafka cluster is scalable and the API Gateway is secure). They do not interfere with or manage the business logic flowing through these layers.
2. **Domain Team (Business):** Domain teams focus solely on implementing business logic within the *Integration* and *Processing* layers. They consume the other platform layers as out-of-the-box services (Service-as-a-Product).
3. **Swapability:** By strictly adhering to layers, the organization can swap out a specific layer (e.g., changing the *Edge Layer* provider) without requiring a rewrite of the microservices living in the *Processing* layer.

---

## Principle 9: Microservices Manifesto

For a microservice to be deployed onto the Platform, it must adhere to the following architectural rules:
* **Stateless by Design:** A microservice must not store state in the local file system. All state must be offloaded to the *Persistence Layer* (Redis/Postgres).
* **Protocol Neutrality:** External communication must always route through the API Gateway (REST/gRPC). Internal communication (between domains) should default to the *Event Layer* (Kafka).
* **Self-Describing:** Every service must expose a `/q/dev` (or equivalent) endpoint returning its current OpenAPI/AsyncAPI specification.
* **Observability First:** The use of standard print commands (e.g., `System.out.println`) is strictly prohibited. Logging must occur in structured JSON format to `stdout` and must always include a unique `Trace-ID`.
* **Sidecar Friendly:** The application architecture must allow for infrastructure and monitoring agents (e.g., OpenTelemetry agents) to be injected without modifying the source code.

---

## Principle 10: Naming Conventions

Uniformity is the foundation of GitOps automation. The platform enforces the following conventions:
* **Namespaces:** `platform-[domain]-[env]` (e.g., `platform-finance-prod`).
* **Container Images:** `registry.platform.io/[domain]/[service-name]:[tag]` (The image tag must always be the Git Commit SHA).
* **Git Repositories:** `platform-[domain]-[service-type]-src` (for source code) and `platform-[domain]-gitops` (for deployment manifests).
* **Kafka Topics:** Follow the pattern `[domain].[entity].[event-type].[version]` (e.g., `finance.invoice.created.v1`).