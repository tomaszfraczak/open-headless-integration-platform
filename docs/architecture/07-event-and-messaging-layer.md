# Event and Messaging Layer Standards

## Document Purpose
This document defines the architecture and standards for the asynchronous data backbone of the Platform. The Event and Messaging Layer (Tier B) enables decoupled communication, high throughput, and event-driven patterns between independent business domains.

---

## 1. The Event Backbone Architecture (Tier B)

The Event Layer serves as the central nervous system of the Platform. It is designed to handle high-velocity data streams while ensuring message durability.

### Key Requirements:
* **Asynchronous Decoupling:** Producers and consumers must operate independently without knowing each other's state.
* **Persistent Streaming:** Messages must be stored on disk to allow for consumer replays and recovery.
* **Guaranteed Delivery:** Support for "at-least-once" delivery semantics as a platform default.

---

## 2. Recommended Open Source Stack

The Platform utilizes **Apache Kafka** (via the Strimzi operator) as the primary engine for this layer:
* **Kafka Clusters:** High-availability deployments across multiple nodes/zones.
* **Schema Registry:** Mandatory use of a schema registry (e.g., Apicurio) to enforce contract versions (Avro/JSON Schema).
* **Connectors:** Usage of Kafka Connect for seamless data ingestion and egress (e.g., Debezium for the Outbox Pattern).

---

## 3. Multi-Cloud Strategy and Component Swap (Azure)

Following the Tier B swap model, the messaging infrastructure can be replaced by Azure-native services:

| Platform Component | Azure Equivalent (Component Swap) |
| :--- | :--- |
| **Apache Kafka (OSS)** | **Azure Event Hubs** (using the Kafka protocol head) |
| **RabbitMQ (OSS)** | **Azure Service Bus** |
| **Schema Registry** | **Azure Event Hubs Schema Registry** |

---

## 4. Governance and Naming Conventions

To ensure delivery scalability, the following rules apply:
* **Topic Naming:** `[domain].[entity].[event-type].[version]` (e.g., `logistics.shipment.created.v1`).
* **Retention Policy:** Default retention is set to 7 days. Critical topics requiring longer durability must use Tiered Storage (e.g., Azure Blob Storage integration).
* **Access Control:** Every tenant namespace is restricted to its own topics via Kafka ACLs. Cross-domain consumption requires an explicit contract approval.
