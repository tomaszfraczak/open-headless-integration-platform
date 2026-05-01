# OCIP: Core Architecture Principles

## Document Purpose
This document defines the foundational architecture principles for the Open Composable Integration Platform (OCIP). These principles represent the non-negotiable core (Tier A) of the platform[cite: 10]. They dictate how integration solutions are designed, deployed, and managed, ensuring scalability, resilience, and operational excellence across the enterprise.

---

## Principle 1: Separation of Control Plane and Data Plane

### Statement
The platform architecture must strictly separate the management and governance layer (Control Plane) from the execution and data processing layer (Data Plane)[cite: 9].

### Rationale
In enterprise integration, mixing business data processing with platform management leads to performance bottlenecks, security vulnerabilities, and difficulties in scaling. By separating these concerns, we ensure that a failure or heavy load in the management layer does not impact the execution of business-critical integration flows, and vice versa. It also enables multi-tenancy and the ability to manage distributed execution environments from a central hub[cite: 9, 12].

### Implications
* **Data Plane:** Components such as Apache Camel, API Gateways (APISIX/Kong), and message brokers (Kafka/RabbitMQ) are strictly dedicated to executing integration flows, mediating traffic, and transforming data[cite: 9].
* **Control Plane:** Components responsible for governance, CI/CD (ArgoCD), policies (Open Policy Agent), and observability (Prometheus/Grafana) exist independently and oversee the Data Plane[cite: 9].
* **Security:** The Data Plane processes business data and payloads; the Control Plane only processes metadata, telemetry, and configurations.

---

## Principle 2: Contract-Based Composability (Replaceable Infrastructure)

### Statement
OCIP is built on stable integration contracts rather than hard dependencies on specific vendor technologies or open-source tools[cite: 9, 10].

### Rationale
To truly eliminate vendor lock-in and enable long-term adaptability, the platform must allow for the controlled replacement of underlying infrastructure components (Tier B)[cite: 10]. If an organization decides to move from Apache Kafka to an existing corporate IBM MQ cluster, the platform must support this transition without requiring a complete rewrite, provided the new component adheres to the established contract[cite: 10].

### Implications
* Integration flows must not tightly couple with proprietary APIs of the underlying message broker or gateway.
* **Support Matrix Enforcement:** The platform will define strict contracts (e.g., *Event Backbone Contract* requiring pub/sub, DLQ, and replay support). Any infrastructure component meeting this contract is considered a "Compatible" or "Supported" profile[cite: 10].
* The 80/20 Rule: 80% of the platform enforces a strict, opinionated operational standard, while 20% allows for controlled customization and replacement of infrastructure blocks[cite: 10].

---

## Principle 3: Stateless Integration Runtime

### Statement
The core integration execution engines (e.g., Apache Camel routes) must remain completely stateless[cite: 9]. 

### Rationale
Stateful integration components are difficult to scale horizontally, prone to data loss during pod evictions, and complicate disaster recovery[cite: 12]. By enforcing a stateless runtime, we ensure that any integration pod can be terminated and recreated instantaneously by Kubernetes without losing business context or disrupting workflows.

### Implications
* **No In-Memory State:** Business state, orchestration progress, or session data must never be stored in the memory of the integration route[cite: 9].
* **State Delegation:** Long-running processes and stateful sagas must be delegated to dedicated Workflow & State engines (e.g., Temporal Technologies)[cite: 9].
* **Caching & Persistence:** Operational state (like idempotency keys) must be offloaded to external persistence layers such as Redis or PostgreSQL[cite: 9].

---

## Principle 4: API-First and Event-Driven by Default

### Statement
Every system integration must be designed as a consumable API product or a decoupled event contract, strictly avoiding point-to-point database links or hidden file transfers[cite: 9].

### Rationale
Point-to-point integrations create a brittle, tightly coupled architecture that cannot scale and is impossible to govern[cite: 7]. Shifting to an event-driven and API-first paradigm promotes reusability, asynchronous decoupling, and independent scaling of services.

### Implications
* **Event-Driven Preference:** Asynchronous messaging (using Kafka/RabbitMQ) is preferred over synchronous REST orchestration for system-to-system data synchronization[cite: 9].
* **Contracts First:** OpenAPI (Swagger) or AsyncAPI specifications must be defined and registered before any implementation begins.
* REST is not a universal hammer; protocols like gRPC, AMQP, or GraphQL should be evaluated based on the specific use case.

---

## Principle 5: Infrastructure as Code (IaC) and GitOps Delivery

### Statement
Manual configuration of environments, deployments, or policies is strictly prohibited. The entire platform lifecycle must be managed declaratively through code[cite: 9].

### Rationale
Enterprise scale requires absolute repeatability and auditability. Manual interventions lead to configuration drift, security vulnerabilities, and unpredictable disaster recovery times[cite: 12]. "Golden Path Engineering" dictates that the platform provides a paved road for delivery, eliminating operational guesswork[cite: 9, 12].

### Implications
* **Provisioning:** Cloud infrastructure and core services must be provisioned via Terraform[cite: 9].
* **Deployment:** Application states and integration routes must be deployed exclusively via GitOps controllers (e.g., ArgoCD) reacting to changes in Git repositories[cite: 9].
* **Auditability:** Git acts as the single source of truth and the primary audit trail for all changes applied to the platform[cite: 9].

---

## Principle 6: Native Observability and Security

### Statement
Security enforcement and telemetry collection are built-in platform primitives, not optional features to be added by developers[cite: 9].

### Rationale
In a distributed, headless integration architecture, operational visibility and strict access control are paramount. Developers should focus on business logic (thin routes), while the platform transparently handles logging, metrics extraction, and request authorization[cite: 8, 9].

### Implications
* **Security by Design:** Every entry point must be protected by the API Gateway with mandatory authentication (e.g., OAuth2/OIDC via Keycloak) and secrets must be injected dynamically via Vault (no hardcoded credentials)[cite: 9].
* **Observability by Default:** All integrations will automatically export distributed traces (OpenTelemetry), standardized metrics (Prometheus), and centralized logs (Loki) without requiring custom code from the integration developer[cite: 9].
