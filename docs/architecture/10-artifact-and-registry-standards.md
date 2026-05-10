# Artifact and Registry Standards

## Document Purpose
This document defines the management of software artifacts and container images. The Registry Layer (Tier B) ensures that only scanned, verified, and immutable artifacts are deployed to the Platform.

---

## 1. Registry Architecture (Tier B)

The Registry acts as the "Safe Harbor" for all containerized workloads. It enforces security gates before any code reaches the runtime environments.

### Core Functions:
* **Immutable Storage:** Versioning images using Git Commit SHAs to prevent configuration drift.
* **Vulnerability Scanning:** Automated CVE detection (e.g., via Trivy) for every pushed image.
* **Access Control:** Role-based access to private image projects per business domain.

---

## 2. Multi-Cloud Strategy and Component Swap

Clients can seamlessly swap the default registry for native cloud artifact repositories:

| OSS Component | Azure Equivalent | AWS Equivalent | GCP Equivalent |
| :--- | :--- | :--- | :--- |
| **Harbor** | Azure Container Registry (ACR) | Amazon ECR | Google Artifact Registry |
| **Helm Chart Repo** | Azure ACR (OCI support) | Amazon ECR | Google Artifact Registry |

---

## 3. Security Gates
* **Scan-on-Push:** Every image is scanned upon arrival.
* **Blocking Policy:** Deployments are automatically blocked if the image contains critical vulnerabilities or if the scan has not been completed.
