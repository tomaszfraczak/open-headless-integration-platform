# Developer Onboarding Guide

## Welcome to the Platform
This guide will help you get started with building modern, resilient integrations following the "Golden Path" delivery model.

---

## 1. Core Philosophy: "Thin Routes"
Your primary goal is to deliver business value through **Thin Routes**. 
* Use **Apache Camel** for routing, mediation, and data transformation (**Integration Layer**).
* Use **Quarkus** for complex business rules and domain logic (**Processing Layer**).
* Infrastructure (Kafka, Databases, API Gateway) is provided to you as a managed service.

---

## 2. Your Development Stack on Azure

When operating on Azure, the Platform utilizes the following Tier B components:
* **Runtime:** Your services run on **Azure Kubernetes Service (AKS)**.
* **Messaging:** Your events flow through **Azure Event Hubs** (Kafka-compatible).
* **Persistence:** Technical data is stored in **Azure Cache for Redis** and **Azure Database for PostgreSQL**.
* **Secrets:** All credentials are dynamically injected from **Azure Key Vault**.

---

## 3. The Lifecycle of an Integration (Step-by-Step)

### Step 1: Bootstrap (The Golden Path)
Do not create a repository from scratch. Use the official `platform-template-quarkus-camel` Git template.
* Follow the naming convention: `platform-[domain]-[service-name]-src`.

### Step 2: Local Development
Use `quarkus dev` mode. Thanks to **Testcontainers**, the Platform will automatically spin up local instances of Kafka and PostgreSQL in Docker. You do not need Azure access to test your integration locally.

### Step 3: Deployment (GitOps)
Once your code is merged to the `main` branch, the automated workflow takes over:
1.  **CI Pipeline:** Runs unit tests and verifies **Quality Gates** (Security, Coverage, Contracts).
2.  **Container Registry:** Builds your image and pushes it to **Harbor**, where it is scanned for vulnerabilities.
3.  **GitOps Sync:** **ArgoCD** detects the change and synchronizes the AKS cluster state with your code.

---

## 4. Pre-Deployment Checklist (Definition of Done)
* [ ] **Statelessness:** Is the service completely stateless?
* [ ] **Health Probes:** Are `/q/health/live` and `/q/health/ready` endpoints exposed?
* [ ] **Logging:** Are logs structured in JSON with a valid `Trace-ID`?
* [ ] **Contract:** Does the API match the OpenAPI/AsyncAPI specification?
* [ ] **Resources:** Are CPU/Memory limits and requests defined in the manifests?
