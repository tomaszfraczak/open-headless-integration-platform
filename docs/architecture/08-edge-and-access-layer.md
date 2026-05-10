# Edge and Access Layer Standards

## Document Purpose
This document defines the standards for the entry point of the Platform. The Edge and Access Layer (Tier B) is responsible for traffic routing, edge security, and protocol mediation for all incoming synchronous requests.

---

## 1. Edge Gateway Architecture (Tier B)

The Edge Layer acts as the "Front Door" of the Platform. It centralizes cross-cutting concerns that would otherwise pollute individual integration routes.

### Primary Responsibilities:
* **SSL Termination:** Handling certificates and encryption at the edge.
* **API Gateway Functions:** Routing, rate limiting, and request transformation.
* **Edge Security:** Protecting internal workloads from DDoS, SQL injection, and unauthorized access.

---

## 2. Recommended Open Source Stack

The Platform utilizes **Apache APISIX** as the cloud-native API Gateway:
* **Dynamic Routing:** Real-time configuration changes without restarting the gateway.
* **Plugin System:** Native support for OIDC authentication, masking, and observability.
* **Ingress Controller:** Full integration with Kubernetes Ingress resources.

---

## 3. Multi-Cloud Strategy and Component Swap (Azure)

This layer is highly interchangeable to fit the client's existing network topology:

| Platform Component | Azure Equivalent (Component Swap) |
| :--- | :--- |
| **Apache APISIX / Kong** | **Azure API Management (APIM)** |
| **Ingress NGINX** | **Azure Application Gateway** (with WAF) |
| **Cert-Manager** | **Azure Key Vault (Certificate integration)** |

---

## 4. Operational Policies

* **Zero-Trust Ingress:** No internal service (Tier C) may expose a public LoadBalancer. All traffic must pass through the Edge Layer for validation.
* **Rate Limiting:** Every entry point must have a defined burst and average limit to protect the Data Plane.
* **Observability:** The Edge Layer must propagate `Trace-ID` headers to ensure end-to-end visibility in the **Observability Layer**.
