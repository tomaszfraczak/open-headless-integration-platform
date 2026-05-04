# OCIP: Plugin and Adapter Framework

## Document Purpose
This document provides the technical blueprint for the OCIP Plugin and Adapter Framework. It explains the mechanisms by which Tier B (Replaceable Infrastructure) components and Tier C (Customer-Owned Extensions) are safely integrated into the platform. This framework ensures that custom infrastructure replacements do not compromise the stability, upgradability, or governance of the core Open Composable Integration Platform.

---

## 1. The Adapter Framework Concept

### Statement
To prevent "composable architecture" from degrading into unmaintainable custom consulting projects, OCIP strictly prohibits direct modification of its core codebase. Instead, all external, non-native infrastructure components must interface with the platform through the predefined Adapter Framework.

### Rationale
Enterprise environments frequently mandate the use of existing corporate assets (e.g., a legacy IAM system, a proprietary message broker, or a specific monitoring stack). Allowing these systems to directly hook into the integration execution engine creates tight coupling. The Adapter Framework acts as an anti-corruption layer, translating the proprietary APIs of the client's infrastructure into the standardized contracts expected by the OCIP Control Plane.

### Implications
* **Standardized Interfaces:** The platform exposes strict interfaces (Contracts) that any custom component must implement.
* **Isolated Execution:** Adapters run as isolated microservices, sidecars, or standalone Kubernetes deployments. If an adapter crashes, it does not bring down the core integration runtime.
* **Client Ownership:** While OCIP defines the contract, the implementation, maintenance, and lifecycle of the custom adapter are entirely owned by the client.

---

## 2. Core Adapter Categories

The Plugin Architecture defines several specific extension points where corporate infrastructure can be securely plugged in. 

### 2.1. Message Broker Adapter
* **Purpose:** Connects the OCIP Integration Runtime (Apache Camel) to a non-standard message broker (e.g., IBM MQ, proprietary corporate queues).
* **Contract Requirements:** The adapter must translate native broker messages into the OCIP standard event envelope. It must support asynchronous publish/subscribe mechanisms, implement dead-letter queue (DLQ) routing, and handle consumer backpressure.

### 2.2. Identity Provider Adapter
* **Purpose:** Integrates the platform's security and API layers with a corporate Identity and Access Management (IAM) system other than the native Keycloak.
* **Contract Requirements:** Must expose standard OpenID Connect (OIDC) or OAuth2 endpoints. The adapter is responsible for mapping internal corporate Active Directory groups to the strict Role-Based Access Control (RBAC) roles required by OCIP governance.

### 2.3. Secret Provider Adapter
* **Purpose:** Connects the Kubernetes `external-secrets-operator` to a proprietary corporate vault.
* **Contract Requirements:** Must securely fetch, decrypt, and rotate credentials without exposing them in plaintext within the platform's logging or deployment layers.

### 2.4. API Gateway Adapter
* **Purpose:** Allows the use of a client-owned, external API Gateway instead of the natively managed APISIX or Kong.
* **Contract Requirements:** The external gateway must support dynamic route provisioning via API or GitOps, enforce OCIP rate-limiting policies, and forward standardized correlation IDs for distributed tracing.

### 2.5. Monitoring Provider Adapter
* **Purpose:** Bridges OCIP's native observability stack with the client's enterprise monitoring tools (e.g., Splunk, Datadog).
* **Contract Requirements:** Must be capable of ingesting OpenTelemetry traces and Prometheus-formatted metrics, ensuring that the OCIP Control Plane dashboards still receive the required SLA visibility data.

---

## 3. The Connector SDK (Tier C Extensions)

While Adapters are used to replace Tier B infrastructure, the **Connector SDK** is provided for domain teams building Tier C business logic.

### Statement
Domain teams must use the official OCIP Connector SDK to build reusable integrations with proprietary business systems (e.g., legacy ERPs, niche manufacturing systems). 

### Functionality
* **Boilerplate Reduction:** The SDK provides pre-configured templates for Apache Camel and Quarkus, embedding the required security, logging, and retry mechanisms by default.
* **Certification Model:** Connectors built with the SDK can be submitted to the internal OCIP Connector Registry. Once approved by the Platform Team, they become reusable assets for other domain teams, promoting DRY (Don't Repeat Yourself) principles across the enterprise.

---

## 4. Implementation and Registration Lifecycle

When a client decides to build a custom adapter or connector, the following strict lifecycle applies:

1. **Contract Validation:** The client reviews the OCIP Interface Definition (e.g., the Swagger/AsyncAPI spec for the required adapter).
2. **Implementation:** The client develops the adapter wrapper using their preferred technology stack, ensuring it is containerized.
3. **Registration via GitOps:** The adapter's Kubernetes manifests (Deployments, Services, ConfigMaps) are committed to the designated Tier B folder in the GitOps repository.
4. **Automated Governance:** OCIP CI/CD pipelines run automated compliance checks against the adapter to verify contract adherence.
5. **Runtime Binding:** ArgoCD deploys the adapter, and the OCIP Control Plane dynamically binds to it via internal DNS routing.
