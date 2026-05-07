# OCIP: Alerting and Incident Response Model

## Document Purpose
This document defines how the Open Composable Integration Platform (OCIP) detects anomalies, routes notifications, and manages incident response. The goal is to ensure rapid resolution of failures while preventing "alert fatigue" through strict routing rules and actionable notifications.

---

## 1. The Alerting Stack and Component Swap

### Statement
The platform provides a fully functional, zero-cost alerting mechanism out-of-the-box, while remaining 100% compatible with enterprise incident management tools.

### Execution
* **Default OSS Router (Tier B):** OCIP utilizes **Prometheus Alertmanager** as the native, open-source alert routing engine. It evaluates metrics against defined thresholds and groups, silences, and routes notifications.
* **Zero-Cost Delivery:** By default, Alertmanager routes notifications to standard corporate communication channels (e.g., MS Teams webhooks, Slack channels, or Email distribution lists).
* **Enterprise Component Swap:** If a client organization already utilizes paid enterprise Incident Management platforms (e.g., PagerDuty, Atlassian Opsgenie, ServiceNow), they execute a Component Swap. The platform team simply configures Alertmanager to forward categorized alerts to these external systems via standard API webhooks.

---

## 2. Alert Routing and Ownership (YBIYRI)

### Statement
OCIP strictly enforces the "You Build It, You Run It" (YBIYRI) model. Alerts must be routed exclusively to the team capable of fixing the underlying issue.

### Execution
Alert routing is determined dynamically based on the metadata (Kubernetes labels) attached to the failing component:
* **Infrastructure Alerts (Platform Team):** Alerts regarding Kafka cluster health, API Gateway latency, or Kubernetes node CPU exhaustion are routed directly to the Platform Engineering on-call rotation.
* **Domain Integration Alerts (Domain Teams):** If a specific Camel route deployed by the "Finance" team experiences a spike in `HTTP 500` errors, or its Dead Letter Queue (DLQ) overflows, the alert is routed exclusively to the Finance Domain Team's designated channel. The Platform Team is *not* paged for domain-specific logic failures.

---

## 3. Alerting as Code

### Statement
Manual creation of alerts via UI dashboards is strictly prohibited. All alert rules must be version-controlled and auditable.

### Execution
* **Declarative Rules:** Alerting thresholds (e.g., `rate(http_requests_total{status="500"}[5m]) > 0.05`) are defined as declarative Kubernetes Custom Resources (`PrometheusRule`).
* **Golden Path Injection:** When a domain team bootstraps a new integration using the OCIP GitOps templates, baseline alert rules (e.g., Pod CrashLoopBackOff, High Latency) are automatically injected into their repository, ensuring no integration goes to production unmonitored.

---

## 4. Actionable Alerts and Fatigue Prevention

### Statement
To prevent "alert fatigue" where operators begin ignoring notifications, every alert must be immediately actionable.

### Execution
* **Symptom-Based Alerting:** Alerts are triggered based on user-facing symptoms (e.g., "High Error Rate on API"), not raw causes (e.g., "CPU at 85%"). High CPU is not an incident unless it actually impacts API latency or success rates.
* **Mandatory Runbooks:** Every defined alert must include an annotation containing a URL to a specific **Runbook**. When an engineer is paged at 3:00 AM, the alert message will contain a direct link with step-by-step instructions on how to diagnose and mitigate that specific failure.

---

## 5. Standardized Incident Severity (SEV) Levels

When an alert requires human intervention, it is classified into one of four severity levels to dictate the response protocol:

| Level | Definition | Target Response | Primary Ownership |
| :--- | :--- | :--- | :--- |
| **SEV-1 (Critical)** | Core platform outage. No traffic is flowing (e.g., Kafka cluster down). | Immediate (24/7 Page) | Platform Team |
| **SEV-2 (Major)** | Complete failure of a critical business domain integration (e.g., Payments sync failing). | Immediate (24/7 Page) | Domain Team |
| **SEV-3 (Minor)** | Partial degradation, increased latency, or non-critical background jobs failing. | Next Business Day | Domain Team |
| **SEV-4 (Warning)** | Informational alert (e.g., DLQ contains 1 message, certificate expires in 30 days). | Handled during Sprint | Respective Team |
