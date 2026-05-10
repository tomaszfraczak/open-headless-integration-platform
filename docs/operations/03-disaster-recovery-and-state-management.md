# Disaster Recovery and State Management

## Document Purpose
This document defines the Business Continuity Planning (BCP) and Disaster Recovery (DR) strategies for the platform. It outlines how the platform handles catastrophic failures (e.g., the loss of an entire Availability Zone or Region) and manages the critical state of in-flight messages during a data center outage.

---

## 1. RTO, RPO, and the GitOps Source of Truth

### Statement
Disaster Recovery on the platform is not a manual restoration process; it is a fully automated redeployment driven by Infrastructure as Code (IaC) via the **Automation / CI/CD Layer**.

### Execution
* **Control Plane Recovery:** The platform configuration, routing rules, and security policies are not backed up using traditional disk snapshots. The Git repository is the absolute Source of Truth.
* **RTO (Recovery Time Objective):** In the event of a total cluster loss, a new Kubernetes cluster can be provisioned via Terraform, and ArgoCD will automatically synchronize the entire platform state from Git within minutes.
* **RPO (Recovery Point Objective):** Because integration routes (Tier C) are stateless, the RPO for the application logic is strictly zero. The only variable is the state held within the Tier B message brokers.

---

## 2. Stateless Runtime Recovery (Tier C)

### Statement
Integration pods (Apache Camel / Quarkus) inside the **Integration and Processing Layers** are entirely ephemeral and hold no local business state. This directly enforces the "Stateless by Design" principle of the Microservices Manifesto.

### Execution
* **Local Failures (Node/Rack):** Handled natively by Kubernetes. If a physical server burns down, the integration pods are automatically rescheduled onto healthy nodes.
* **Regional Failures (Active-Passive):** In a Multi-Region deployment, global DNS load balancers automatically route incoming API traffic to the secondary region if the primary region goes offline. Because the Camel routes are stateless, the secondary region can begin processing API requests immediately.

---

## 3. Stateful Component Replication (Tier B)

### Statement
While APIs are stateless, the **Event / Messaging Layer** (e.g., Apache Kafka) holds critical, unacknowledged business events. The platform must guarantee message durability across geographical regions.

### Execution
* **Broker Replication:** For highly critical profiles, Tier B message brokers are deployed in an Active-Passive cross-region topology.
* **Data Mirroring:** The platform utilizes native replication tools (e.g., Kafka MirrorMaker 2 or native Tiered Storage cross-region replication) to continuously asynchronously copy topic data and consumer group offsets from the Primary Region to the Disaster Recovery Region.

---

## 4. The "Stuck Message" Dilemma (In-Flight State)

### Statement
When a primary data center fails abruptly, integration pods are killed without executing a graceful shutdown. This leaves a critical question: *What happens to a Kafka message that was halfway through processing when the power went out?*

### Execution
The solution relies strictly on the architectural patterns enforced in the Delivery layer regarding "Thin Routes" and idempotency:
1. **Offset Commitment:** The integration pod only commits the Kafka offset *after* the entire route finishes successfully. Because the pod died halfway, the offset was never committed.
2. **Failover Activation:** The secondary DR cluster spins up and connects to the replicated Kafka broker.
3. **Re-consumption:** The secondary consumers resume from the last known committed offset. They will pull the exact same message that was "stuck" during the fire.
4. **Idempotency Saves the State:** Because the platform enforces the *Idempotent Consumer Pattern*, if the previous pod managed to update a database before dying but failed to commit the offset, the new pod will detect the unique `Correlation-ID` in the Idempotency Repository (e.g., a globally replicated Redis inside the **Persistence Layer**) and safely discard the duplicate without triggering a double-billing scenario.

---

## 5. Continuous Resilience (Chaos Engineering)

### Statement
A Disaster Recovery plan that is not tested does not exist.

### Execution
The platform mandates regular "Game Days" where infrastructure failures (e.g., severing network ties to a database, or terminating random Kafka broker pods) are injected into the production or pre-production environment to mathematically prove the platform's self-healing capabilities and ensure the DR runbooks are accurate.