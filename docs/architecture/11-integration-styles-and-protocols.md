# Integration Styles, Protocols, and Data Models

## Document Purpose
This document provides the architectural decision matrix for selecting the appropriate integration style (Synchronous vs. Asynchronous vs. File-based), communication protocol, and data model. It ensures that domain teams choose the most scalable and efficient pattern for their specific use case.

---

## 1. Integration Styles Decision Matrix

Choosing the right communication style is the most critical architectural decision. Domain teams must evaluate their requirements against the following criteria:

### 1.1. Asynchronous Event-Driven (Platform Default)
* **When to use:** Data synchronization across multiple systems, high-throughput processing, fire-and-forget notifications, and scenarios requiring high decoupling.
* **Implications:** Requires idempotency handling and manages state via the Persistence Layer.

### 1.2. Synchronous Request-Reply (API-Driven)
* **When to use:** Real-time UI interactions, direct data lookups, and processes requiring immediate transactional confirmation.
* **Implications:** Increases coupling and requires strict implementation of Circuit Breakers and Timeouts.

### 1.3. Managed File Transfer (MFT / File-based)
* **When to use:** Integration with legacy systems (Mainframes, ERPs) that do not support APIs/Events, or bulk transfers of non-structured data (e.g., daily batch exports).
* **Implications:** Must follow the "Claim Check Pattern" for large files to avoid platform memory exhaustion.

---

## 2. Supported Protocols and Cloud Equivalents (Tier B Swap)

The Platform remains agnostic by supporting standard protocols. Underlying engines can be swapped for native cloud services:

| Integration Style | Protocol | Azure Equivalent | AWS Equivalent | GCP Equivalent |
| :--- | :--- | :--- | :--- | :--- |
| **Synchronous** | REST / gRPC | Azure API Management | AWS API Gateway | Cloud API Gateway |
| **Asynchronous** | Kafka / AMQP | Azure Event Hubs | Amazon MSK / MQ | Cloud Pub/Sub |
| **File-based** | SFTP / AS2 | Azure Data Factory | AWS Transfer Family | Cloud Storage Transfer |

---

## 3. Data Models and Serialization Standards

To ensure long-term maintainability and performance, the following data formats are mandated:

### 3.1. JSON (REST Standard)
* **Usage:** Default for all external REST APIs.
* **Requirement:** Must be ustructured and documented via OpenAPI specifications.

### 3.2. Avro (Event Standard)
* **Usage:** Mandatory for all high-throughput Kafka streams.
* **Requirement:** All schemas must be registered in the Schema Registry to ensure backward compatibility.

### 3.3. Protobuf (High Performance)
* **Usage:** Mandatory for internal gRPC microservice communication in the Processing Layer.
