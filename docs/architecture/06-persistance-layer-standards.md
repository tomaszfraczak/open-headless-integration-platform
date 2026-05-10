# Persistence Layer Standards

## Document Purpose
This document defines the architecture and standards for state management within the Platform. Adhering to the "Stateless by Design" principle, integration engines must not store data locally, making this layer critical for maintaining business process consistency and resilience.

---

## 1. The Role of the Persistence Layer (Tier B)

The Persistence Layer is classified as **Tier B (Replaceable Infrastructure)**. It is essential for implementing the following architectural requirements:

* **Ensuring Idempotency:** Storing unique transaction keys (e.g., `Correlation-ID`) to prevent duplicate processing in "at-least-once" delivery systems (e.g., Kafka).
* **State Management (Saga & Aggregation):** Handling long-running business flows where data from multiple systems must be correlated before further processing.
* **Operational Resilience:** Storing intermediate states, allowing processes to resume from the last known checkpoint after container restarts or infrastructure outages.

---

## 2. Recommended Open Source Stack

The Platform provides two default data stores optimized for different integration patterns:

1.  **Redis (In-memory Store):**
    * **Usage:** Ultra-fast idempotency checks, temporary caching, and simple message aggregation.
    * **Benefit:** Minimal latency with high throughput for transient state.
2.  **PostgreSQL (Relational Store):**
    * **Usage:** Durable storage for operational logs, Saga pattern states, and complex data correlations.
    * **Benefit:** Full ACID compliance and support for complex transactional integrity.

---

## 3. Multi-Cloud Strategy and Component Swap (Azure)

The Persistence Layer is designed to be cloud-agnostic. In an **Azure** environment, Tier B components can be swapped for native managed services without impacting the business logic (Tier C):

| Platform Component | Azure Equivalent (Component Swap) |
| :--- | :--- |
| **PostgreSQL (OSS)** | Azure Database for PostgreSQL |
| **Redis (OSS)** | Azure Cache for Redis |
| **Object Store (S3)** | Azure Blob Storage (for the Claim Check Pattern) |

---

## 4. Developer Guidelines

* **Protocol Neutrality:** Applications in the *Processing Layer* must connect to persistence stores using standard protocols (JDBC, RESP), avoiding vendor-specific lock-in.
* **Dynamic Configuration:** Database credentials and connection strings must be injected at runtime via the **Security & Identity Layer** (e.g., HashiCorp Vault or Azure Key Vault).
* **Stateless Manifesto:** No microservice is allowed to use local persistent volumes (PVs) for business data.