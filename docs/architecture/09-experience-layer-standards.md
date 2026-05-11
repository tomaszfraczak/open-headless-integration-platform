# Experience Layer Standards

## Document Purpose
This document defines the standards for the developer-facing interface of the Platform. The Experience Layer (Tier B) provides a centralized portal for API discovery, documentation, and self-service capabilities.

---

## 1. Developer Portal Architecture (Tier B)

The Experience Layer serves as the internal marketplace for integrations, ensuring that all contracts (APIs and Events) are discoverable and documented.

### Key Features:
* **Service Catalog:** A unified view of all microservices and integration routes.
* **Contract Visualization:** Rendering of OpenAPI and AsyncAPI specifications.
* **Software Templates:** Access to the "Golden Path" repository templates for bootstrapping new integrations.

---

## 2. Multi-Cloud Strategy and Component Swap

While the Experience Layer is typically centralized, it can be integrated with or replaced by cloud-native portals:

| OSS Component | Azure Equivalent | AWS Equivalent | GCP Equivalent |
| :--- | :--- | :--- | :--- |
| **Backstage.io** | Azure Developer CLI / Custom IDP | AWS Proton / Service Catalog | GCP Service Catalog |
| **Documentation Hub** | Azure API Management Portal | AWS API Gateway Portal | GCP API Hub |

---

## 3. Ownership and Governance
* **Platform Team:** Owns the portal infrastructure and template curation.
* **Domain Teams:** Responsible for keeping their service metadata and contract documentation up to date.
