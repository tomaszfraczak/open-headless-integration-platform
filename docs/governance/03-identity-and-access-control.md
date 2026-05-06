# OCIP: Identity and Access Control

## Document Purpose
This document defines the Identity and Access Management (IAM) framework for the Open Composable Integration Platform (OCIP). It outlines how the platform secures access to administrative interfaces, observability tools, and deployment pipelines using a centralized Role-Based Access Control (RBAC) model federated with corporate identity providers.

---

## 1. Federated Identity (SSO First)

### Statement
OCIP prohibits the use of "local" platform accounts for human users. All access must be authenticated via a centralized Identity Provider (IdP).

### Rationale
Managing separate credentials for GitHub, ArgoCD, Grafana, and the API Gateway is insecure and unscalable. By federating OCIP with the enterprise's existing Single Sign-On (SSO) system (e.g., Microsoft Entra ID, Okta, or Ping Identity) via OIDC or SAML, we ensure that when an employee leaves the company, their access to the entire integration platform is revoked instantly.

### Execution
* **Tier B Integration:** The platform utilizes Keycloak (or a compatible swapped component) as the central Identity Broker.
* **Group Mapping:** Corporate directory groups (e.g., `AD-Group-Integrators-Finance`) are automatically mapped to internal OCIP roles and Kubernetes namespaces.

---

## 2. The RBAC Persona Model

OCIP enforces a strict "Least Privilege" principle. Access is granted based on predefined personas, ensuring that users only have the permissions necessary for their specific job functions.

| Persona | Description | Scope | Key Permissions |
| :--- | :--- | :--- | :--- |
| **Platform Admin** | Manages the global OCIP infrastructure (Tier A & B). | Global | Full cluster access, policy definition, infrastructure provisioning. |
| **Domain Owner** | Responsible for a specific business domain (e.g., "Logistics"). | Namespace | Resource quota management, approval of domain-specific PRs. |
| **Domain Developer** | Builds and deploys integration routes (Tier C). | Namespace | Read/Write access to domain Git repos, view logs/traces for their pods. |
| **Security Auditor** | Reviews platform compliance and security posture. | Global (Read) | View-only access to all audit logs, policies, and security scan results. |

---

## 3. Namespace Isolation as a Security Boundary

### Statement
The Kubernetes `Namespace` is the fundamental unit of isolation and security within OCIP. 

### Execution
* **Tenant Segregation:** Each business domain (Tenant) is confined to its own namespace. 
* **Network Policies:** By default, pods in the `finance` namespace cannot communicate with pods in the `hr` namespace unless an explicit global policy (Tier A) allows it.
* **Secret Isolation:** Secrets (passwords, tokens) stored in one namespace are physically and logically inaccessible to developers or applications running in another namespace.

---

## 4. Observability and Log Access Governance

### Statement
Access to telemetry data (logs and traces) is a security concern. Log data may contain sensitive business metadata and must be protected according to domain boundaries.

### Execution
* **Filtered Access:** Developers can only query logs and metrics filtered by their assigned namespace/domain.
* **Audit Trail:** Every login attempt, policy change, and deployment action is logged to an immutable audit stream, stored for a mandatory retention period (e.g., 365 days) for compliance purposes.
* **Masking Verification:** Security Auditors periodically review the effectiveness of automated log masking (defined in `delivery/02-integration-patterns.md`) to ensure no PII/PCI data is leaking into the observability stack.

---

## 5. Technical Access (Service Accounts)

### Statement
Non-human entities (CI/CD pipelines, automated scripts) must use dedicated Service Accounts with scoped permissions.

### Execution
* **Short-Lived Tokens:** Service accounts should utilize short-lived OIDC tokens or certificates rather than long-lived static API keys.
* **Workload Identity:** Applications running on the platform (e.g., a Camel route) use Kubernetes Service Accounts to fetch secrets from Vault or publish to Kafka, eliminating the need for hardcoded credentials.
