# Operating Model and Team Topologies

## Document Purpose
This document defines the organizational framework and interaction patterns for teams using the Platform. Inspired by the "Team Topologies" methodology, this model ensures that the platform accelerates delivery rather than becoming a centralized bureaucratic bottleneck.

---

## 1. Team Definitions and Responsibilities

The platform distinguishes between two primary types of teams to ensure a clear **Separation of Concerns** mapping directly to the platform's architectural layers:

### 1.1. The Platform Team (Platform as a Product)
* **Role:** Acts as a product team that builds and maintains the core infrastructure (Tier A and B), including the *Automation*, *Edge*, *Security*, and *Observability Layers*.
* **Focus:** Reliability, scalability, developer experience (DevEx), and the "Golden Path."
* **Mission:** To provide a self-service platform that requires minimal human intervention from the platform team to enable domain delivery.

### 1.2. Domain Teams (Stream-Aligned Teams)
* **Role:** Business-focused teams (e.g., Finance, Logistics, HR) that build integrations (Tier C) acting within the *Integration Layer* and *Processing Layer*.
* **Focus:** Business logic, data mapping, and meeting project deadlines.
* **Mission:** To own the full lifecycle of their integrations, from design to production support.

---

## 2. Interaction Patterns (Self-Service First)

### Statement
The interaction between Domain Teams and the Platform Team must be "X-as-a-Service" by default.

### Rationale
Traditional "ticket-based" operating models, where a developer must wait days for a new queue or a firewall rule, are the primary cause of integration project delays. The platform eliminates this via automation.

### Execution
* **No Tickets for Resources:** Domain teams provision their own integration namespaces, Kafka topics, and API routes using the platform's GitOps templates and strict naming conventions.
* **Automated Governance:** Compliance and security checks are baked into the CI/CD pipelines (Quality Gates). If the code passes the gates, it is automatically approved for deployment.
* **Consultancy Mode:** The Platform Team only engages in direct "collaboration" for highly complex architectural shifts or during the onboarding of entirely new business units.

---

## 3. Incident Management and Handoff

### Statement
The platform enforces a "You Build It, You Run It" (YBIYRI) model for Tier C business logic, while the Platform Team guarantees the underlying infrastructure.

### Execution
* **Infrastructure Alerts:** If a core component (e.g., the Kafka cluster or the API Gateway) fails, the **Platform Team** is paged and responsible for the resolution.
* **Integration Alerts:** If a specific business route fails (e.g., an ERP-to-CRM sync error), the **Domain Team** is responsible for troubleshooting and fixing the logic.
* **Centralized Visibility:** Both teams share the same observability dashboards to ensure a "Single Source of Truth" during cross-team incidents.