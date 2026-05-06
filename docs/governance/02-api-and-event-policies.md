# OCIP: API and Event Policy Governance

## Document Purpose
This document defines the centralized policy enforcement model for the Open Composable Integration Platform (OCIP). By offloading cross-cutting concerns—such as security, traffic shaping, and monitoring—to the Platform Control Plane (API Gateway and Mesh), OCIP ensures consistent governance across all integrations while keeping Tier C business logic "thin" and focused.

---

## 1. Centralized Policy Enforcement

### Statement
Cross-cutting integration concerns must be enforced at the platform level (Control Plane) rather than being hardcoded into individual integration routes.

### Rationale
Managing security, rate limiting, and logging logic within hundreds of individual Camel routes is an operational nightmare. It leads to inconsistent security postures and makes it impossible to update global policies without redeploying every service. Centralizing these policies at the API Gateway (for sync traffic) and the Event Backbone (for async traffic) ensures architectural integrity and operational agility.

### Implications
* **Zero-Trust Security:** No integration pod should trust an incoming request that has not been pre-validated by the platform's entry points.
* **Separation of Duties:** Domain teams focus on "what" the integration does; the Platform Team defines "how" it is protected and throttled.

---

## 2. Traffic Shaping: Rate Limiting and Throttling

### Statement
All exposed APIs and event consumers must have explicit traffic shaping policies to prevent resource exhaustion and ensure platform stability.

### Execution
* **Global Rate Limiting:** Applied at the Gateway level to protect the overall cluster from generic DDoS or misconfigured high-volume consumers.
* **Consumer-Specific Quotas:** Policies assigned to specific API Keys or OAuth2 Clients. If a specific business partner is contracted for 100 requests per second, the platform will strictly enforce this limit, returning `HTTP 429 Too Many Requests` upon violation.
* **Spike Arrest (Burst Control):** Defines the maximum number of requests allowed in a sub-second interval to prevent sudden traffic surges from crashing downstream legacy systems.

---

## 3. Security and Identity Policy

### Statement
All communication within OCIP must be authenticated and authorized using centralized identity standards.

### Execution
* **Authentication (Who are you?):** All incoming API traffic must present a valid JWT (JSON Web Token) issued by the platform's Identity Provider (Tier B - e.g., Keycloak). The API Gateway is responsible for validating the signature, issuer, and expiration of the token.
* **Authorization (What can you do?):** Access control is enforced via OAuth2 Scopes or Role-Based Access Control (RBAC). A request to `/finance/payments` will be rejected by the Gateway unless the token contains the `finance-admin` role.
* **mTLS (Internal Traffic):** For highly regulated environments (Profile C), the platform enforces mutual TLS (mTLS) between all internal pods, ensuring that even if an attacker enters the cluster, they cannot intercept internal data traffic.

---

## 4. Observability and Auditing Policies

### Statement
Every interaction with the platform must leave an immutable audit trail for compliance and troubleshooting.

### Execution
* **Access Logging:** The API Gateway automatically logs metadata for every request (Source IP, Latency, Status Code, Consumer ID).
* **Distributed Tracing Enforcement:** The platform automatically injects and propagates W3C Trace Context headers (Traceparent). Any Tier C extension must preserve these headers to ensure the full request path is visible in the observability stack.
* **Policy Auditing:** Changes to policies (e.g., changing a rate limit) are managed via GitOps. The Git history serves as the primary audit trail for who changed which policy and when.
