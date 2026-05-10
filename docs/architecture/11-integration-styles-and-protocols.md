# Integration Styles and Protocols Standards

## Document Purpose
This document provides the architectural decision matrix for selecting the appropriate integration style (Synchronous vs. Asynchronous), communication protocol, and data model. It ensures that domain teams choose the most scalable, efficient, and appropriate pattern for their specific business use case when building on the Platform.

---

## 1. Synchronous vs. Asynchronous Decision Matrix

Choosing the right communication style is the most critical architectural decision. Domain teams must evaluate their use case against the following matrix before defining their contract.

### 1.1. Asynchronous Event-Driven (The Platform Default)
The Platform mandates asynchronous communication as the default pattern for system-to-system integration.
* **When to use:**
    * **Data Synchronization:** Keeping two or more systems in sync (e.g., updating customer profiles across the CRM, Billing, and Shipping systems).
    * **High Throughput/Batching:** Processing large volumes of records where immediate processing is not strictly required.
    * **Fire-and-Forget:** The source system needs to publish a state change but does not care who consumes it.
    * **Temporal Decoupling:** The target system might be temporarily unavailable, and messages need to be queued safely.
* **Prohibited for:** Real-time UI interactions requiring immediate feedback to the user.

### 1.2. Synchronous Request-Reply (API-Driven)
Synchronous communication blocks the caller's thread until a response is received. It should be used sparingly for system-to-system backend communication due to tight coupling and cascading failure risks.
* **When to use:**
    * **User Interfaces (UI):** A web or mobile frontend needs to query data to render a page immediately.
    * **Data Lookups/Queries:** Fetching a specific record (e.g., retrieving an account balance before processing a transaction).
    * **Strict Transactional Boundaries:** A process cannot proceed without a definitive success/fail response from a downstream service.
* **Prohibited for:** Mass data transfers or syncing databases.

---

## 2. Supported Protocols

Once the integration style is selected, teams must choose from the Platform's officially supported protocols.

### 2.1. Synchronous Protocols
* **REST (HTTP/1.1 or HTTP/2):** The universal standard. Best for public-facing APIs, CRUD operations, and integrations with standard SaaS applications.
* **gRPC (HTTP/2):** Highly performant, binary-based RPC framework. Best for high-speed, internal microservice-to-microservice communication where latency is critical.
* **GraphQL:** Supported specifically for Experience APIs (Edge Layer) where a frontend client needs to aggregate data from multiple downstream services in a single query.

### 2.2. Asynchronous Protocols
* **Kafka Protocol:** The absolute standard for event streaming, pub/sub, and log-tailing.
* **AMQP (Advanced Message Queuing Protocol):** Supported via message brokers (e.g., RabbitMQ/Service Bus) for traditional enterprise queuing, routing, and task distribution where strict message ordering per consumer is required.

---

## 3. Data Models and Serialization Formats

The format of the payload must align with the chosen protocol and performance requirements.

### 3.1. JSON (JavaScript Object Notation)
* **Usage:** Mandatory for all external REST APIs and general-purpose web integrations.
* **Pros:** Human-readable, universally supported.
* **Cons:** Verbose, lacks strict type enforcement out-of-the-box.
* **Governance:** Must be governed by strict **OpenAPI Specifications**.

### 3.2. Avro
* **Usage:** The mandatory serialization format for the Event/Messaging Layer (Kafka).
* **Pros:** Highly compressed binary format, embeds schema definitions, excellent for high-throughput data streams.
* **Governance:** All Avro schemas must be registered in the **Platform Schema Registry**. Producers and consumers must validate messages against the registry to prevent backward-breaking schema changes.

### 3.3. Protocol Buffers (Protobuf)
* **Usage:** Mandatory for all gRPC communication. Can also be used as an alternative to Avro for Kafka streams if required by specific domain standards.
* **Pros:** Strongly typed, generates highly efficient client/server code automatically.
* **Governance:** `.proto` files act as the undeniable contract and must be stored centrally.
