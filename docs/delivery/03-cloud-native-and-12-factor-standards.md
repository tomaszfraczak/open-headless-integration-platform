# OCIP: Cloud Native and 12-Factor App Standards

## Document Purpose
While Enterprise Integration Patterns (EIP) dictate how data flows through an integration, this document defines how the integration application itself must behave within the Kubernetes cluster. All Tier C (Customer-Owned) integration workloads deployed on the Open Composable Integration Platform (OCIP) must strictly adhere to the 12-Factor App methodology to guarantee zero-downtime deployments, horizontal scalability, and platform stability.

---

## 1. Stateless Execution and Concurrency

### Statement
Integration pods must be treated as ephemeral, disposable computing units. They must execute workloads without assuming that subsequent requests will be routed to the same pod.

### Execution
* **Strict Statelessness:** The core integration execution engines must remain completely stateless. Any integration pod can be terminated and recreated instantaneously by Kubernetes without losing business context.
* **Horizontal Scaling:** Applications must be designed to scale out (adding more pods via Horizontal Pod Autoscaler) rather than scaling up (allocating more CPU/RAM).
* **Shared State Cache:** If operational state is temporarily required (e.g., aggregation strategies, idempotency keys), it must be offloaded to an external backing service (e.g., Redis).

---

## 2. Externalized Configuration

### Statement
Hardcoding environment-specific configurations (URLs, credentials, thread pool limits) directly into the application code or repository is strictly prohibited.

### Execution
* **Environment Variables:** All configurations that vary between environments (Dev, Test, Prod) must be injected during runtime via Kubernetes `ConfigMaps` and exposed as Environment Variables.
* **Secret Injection:** Passwords, API keys, and certificates must be dynamically fetched from the platform's native secrets management system (e.g., HashiCorp Vault) and injected into the pod memory. They must never be committed to Git.

---

## 3. Container Lifecycle and Graceful Shutdown

### Statement
Kubernetes routinely evicts pods for node upgrades, scaling, or rebalancing. Integration applications must handle termination signals gracefully to prevent data loss or dropped API connections.

### Execution
* **SIGTERM Handling:** When Kubernetes sends a `SIGTERM` signal, the application must immediately stop accepting new incoming requests (e.g., unbinding REST endpoints, pausing Kafka consumers).
* **In-Flight Processing:** The integration runtime (Apache Camel / Quarkus) must be configured to allow currently executing routes (in-flight messages) to finish processing before shutting down the JVM.
* **Hard Kill (SIGKILL):** Only if the application fails to shut down within the platform's defined grace period (e.g., 30-60 seconds) will Kubernetes forcefully terminate the process.

---

## 4. Platform-Native Health Probes

### Statement
Kubernetes must maintain continuous awareness of the application's internal state to ensure traffic is only routed to healthy pods. 

### Execution
All integration workloads must expose standardized HTTP health endpoints:
* **Liveness Probe (`/q/health/live`):** Determines if the pod's main process is deadlocked or crashed. If this probe fails repeatedly, Kubernetes will automatically restart the pod.
* **Readiness Probe (`/q/health/ready`):** Determines if the application is ready to accept external traffic. This probe must verify downstream connectivity (e.g., verifying that the internal database connection pool is initialized and the Kafka broker is reachable). If this probe fails, the pod is temporarily removed from the platform's routing mechanisms.

---

## 5. Observability and Log Aggregation

### Statement
Applications running on OCIP must not manage their own log files, log rotation, or local storage. Telemetry collection is a built-in platform primitive.

### Execution
* **Standard Output:** Applications must write all logs directly to `stdout` and `stderr`.
* **Structured JSON Logging:** Logs must be formatted as structured JSON. This allows the OCIP Control Plane's log aggregators (e.g., Promtail/Fluent Bit) to automatically parse, tag, and ship the logs to the centralized observability stack (e.g., Loki or Elasticsearch) without custom parsing rules.
* **Correlation IDs:** Every log entry must include a unified `Trace-ID` and `Span-ID` to enable distributed tracing across multiple platform components and microservices.
