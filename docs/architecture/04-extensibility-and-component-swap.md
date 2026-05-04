# OCIP: Extensibility and Component Swap Model

## Document Purpose
This document outlines the extensibility framework for the Open Composable Integration Platform (OCIP). It defines how organizations can safely replace core infrastructure components (Tier B) and build custom business integrations (Tier C) without relying on proprietary vendor SDKs. The OCIP extensibility model is driven entirely by open-source standards, declarative configuration, and strict integration contracts.

---

## 1. The "No-SDK" Extensibility Philosophy

### Statement
OCIP explicitly rejects the use of proprietary Software Development Kits (SDKs) or custom plugin wrappers for platform extensibility. All platform extensions, infrastructure swaps, and custom connectors must be built using native open-source standards, primarily Apache Camel, and integrated via declarative Infrastructure as Code (IaC).

### Rationale
Proprietary SDKs create a secondary form of vendor lock-in and introduce significant technical debt during platform upgrades. By relying on native Apache Camel components and Kubernetes-native deployment mechanisms, the platform ensures that all integration assets remain portable, standard-compliant, and fully owned by the client organization.

### Implications
* Integration developers do not learn an "OCIP Framework"; they use standard Apache Camel and Java/Quarkus paradigms.
* Infrastructure replacements are handled as configuration swaps rather than custom code deployments.

---

## 2. Tier B: Replaceable Infrastructure (Component Swap)

When an enterprise client chooses to replace a native OCIP infrastructure component with an existing corporate standard (e.g., swapping out Apache Kafka for Azure Service Bus or IBM MQ), this is executed as a **Component Swap** rather than an adapter development project.

### Swap Execution Lifecycle

1. **Contract Validation:**
   The proposed replacement technology must strictly fulfill the established OCIP architectural contract. For example, if replacing the message broker, the new system must meet the *Event Backbone Contract*, which requires support for asynchronous pub/sub, message replay, Dead Letter Queues (DLQ), and retry orchestration. If the system does not meet these criteria (e.g., legacy FTP queues), it is rejected.

2. **Native Component Reconfiguration:**
   Because the execution engine is built on Apache Camel, the platform already supports hundreds of protocols out-of-the-box. Swapping a broker does not require writing an OCIP-specific plugin. The deployment team simply updates the GitOps templates to use the native Camel component for the target system (e.g., changing the route configuration from `kafka:` to `azure-servicebus:` or `sjms2:` for IBM MQ).

3. **Operational Boundary Shift:**
   Upon swapping a Tier B component, its support status changes to *Compatible*. The OCIP Control Plane continues to govern the routing, policies, and CI/CD pipelines, but the physical maintenance, scaling, and patching of the swapped infrastructure (e.g., the corporate IAM or broker) become the responsibility of the client's internal IT teams.

---

## 3. Tier C: Customer-Owned Extensions

Tier C represents the actual business implementations, custom connectors, and domain logic built on top of the OCIP foundation. Domain teams are fully empowered to build complex integrations without being restricted by the platform provider.

### Development Stack
Domain teams must build business services and custom connectors using **Quarkus + Apache Camel** or **Camel K** for lighter, event-driven integrations. 

### The "Thin Routes" Principle
To maintain platform stability, domain teams must adhere to the rule of thin routes:
* **Apache Camel's Role:** Strictly limited to routing, mediation, data transformation, orchestration, and retry/compensation logic.
* **Quarkus's Role:** Complex business rules, domain validations, and heavy domain logic must be encapsulated within dedicated Quarkus microservices, not embedded directly within the Camel Route DSL. Placing thousands of lines of business logic into a Camel route is classified as a severe anti-pattern that degrades platform maintainability

### Golden Path Delivery over SDKs
Instead of providing an SDK, the OCIP Platform Team provides a **Golden Path** for delivery. 
* **Templates:** Domain teams clone standardized repository templates.
* **Automated Injection:** These templates automatically inject the required security baselines (Vault secrets), enterprise observability configurations (metrics, tracing), and deployment manifests (Helm/ArgoCD).
* **Ownership:** While the platform provides the infrastructure, templates, and governance, the specific integration flows, custom APIs, and mapping logic remain 100% owned by the domain teams.
