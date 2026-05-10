# Enterprise Observability and Monitoring

## Document Purpose
This document defines the observability architecture for the platform. In a highly distributed, event-driven architecture, traditional logging is insufficient. This standard ensures that platform operators and domain teams have real-time visibility into system health, performance bottlenecks, and cross-component transaction flows.

---

## 1. The Three Pillars of Observability

### Statement
The platform mandates the automated collection and correlation of the three pillars of observability: Metrics, Logs, and Distributed Traces, leveraging **OpenTelemetry (OTel)** as the vendor-neutral standard within the **Observability Layer**.

### Execution
* **Metrics (The "What"):** Aggregated numeric data. The platform automatically scrapes metrics from all Tier A/B components (e.g., Kafka consumer lag, API Gateway throughput) and Tier C integration pods using Prometheus.
* **Logs (The "Why"):** Highly contextual event records. Application logs are captured from `stdout/stderr` as structured JSON, tagged with Kubernetes metadata (namespace, pod name), and shipped to a central aggregator (e.g., Loki, Elasticsearch). Following the Microservices Manifesto, standard print commands are strictly prohibited.
* **Distributed Tracing (The "Where"):** End-to-end request tracking. Every incoming API request or Kafka event is injected with a W3C `Traceparent` header. This allows the platform to generate a visual topology map (via Jaeger or Tempo) showing exactly how much time a request spent in the Gateway (**Edge Layer**), the Message Broker (**Event / Messaging Layer**), and the backend Camel route (**Integration Layer**). Furthermore, applications must be sidecar-friendly to allow tracing agents to be injected without code changes.

---

## 2. The Four Golden Signals (SRE Standard)

### Statement
Dashboards and baseline monitoring must be organized around the "Four Golden Signals" of Site Reliability Engineering to rapidly identify user-impacting degradation.

### Execution
For every deployed integration (Tier C) and core component (Tier B), the platform automatically tracks:
1.  **Latency:** The time it takes to service a request (differentiating between successful and failed requests).
2.  **Traffic:** A measure of demand (e.g., HTTP requests per second, or Kafka messages consumed per minute).
3.  **Errors:** The rate of requests that fail (e.g., HTTP 5xx codes, DLQ routing events, or dropped connections).
4.  **Saturation:** How "full" the service is (e.g., CPU/Memory utilization, database connection pool exhaustion, Thread pool limits).

---

## 3. Dashboard Isolation and Self-Service

### Statement
Observability must respect the multi-tenant isolation boundaries defined in the Governance layer.

### Execution
* **Platform View (Global):** The Platform Team has access to overarching infrastructure dashboards (Kubernetes Node health, global Kafka cluster metrics, ArgoCD sync status). This aligns with the Responsibility Model, where the Platform Team ensures the underlying layers are stable.
* **Domain View (Namespace):** Domain teams are provided with out-of-the-box Grafana templates (or equivalent). They can only view metrics, logs, and traces originating from their assigned Kubernetes namespace (e.g., `platform-[domain]-[env]`).
* **No Manual Setup:** Developers do not need to configure log shippers or metric endpoints. The Golden Path templates automatically wire the Quarkus/Camel applications to expose `/q/metrics` and push OpenTelemetry spans.

---

## 4. Synthetic Monitoring (Active Probing)

### Statement
Passive monitoring (waiting for user traffic to fail) is insufficient for business-critical integrations. The platform must proactively verify component health.

### Execution
* **Blackbox Probing:** The platform utilizes Synthetic Monitoring (e.g., Prometheus Blackbox Exporter) to continuously ping external Tier C endpoints and internal APIs from the outside, verifying DNS resolution, TLS certificate validity, and endpoint response times even during periods of zero traffic.