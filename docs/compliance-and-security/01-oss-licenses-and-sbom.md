# OCIP: OSS Licensing and Software Bill of Materials (SBOM)

## Document Purpose
This document outlines the strategy for managing Open Source Software (OSS) licenses and maintaining a Software Bill of Materials (SBOM) for the Open Composable Integration Platform (OCIP). It ensures legal compliance, supply chain security, and transparency for enterprise clients regarding the third-party components integrated into the platform.

---

## 1. Open Source Licensing Strategy

### Statement
OCIP is built on a foundation of "Permissive" open-source components. The platform explicitly prioritizes tools licensed under business-friendly terms to eliminate legal risks for the end client.

### Preferred Licenses
The platform core and its recommended stack primarily utilize the following licenses:
* **Apache License 2.0:** (e.g., Apache Camel, Kafka, APISIX, Keycloak). Allows for commercial use, modification, and distribution without forcing the disclosure of the platform's proprietary Control Plane code.
* **MIT / BSD:** Highly permissive licenses used for various supporting libraries.

### Restricted Licenses (The "Copyleft" Policy)
OCIP strictly limits the use of "Strong Copyleft" licenses (e.g., GPL v3, AGPL) within the platform's distributed binaries. Any use of such components must be isolated (e.g., running as a separate microservice) to prevent the "viral" effect from impacting the proprietary parts of the platform's Control Plane.

---

## 2. Software Bill of Materials (SBOM)

### Statement
OCIP maintains a real-time, automated SBOM for every official release. An SBOM is a formal, machine-readable inventory of software components, dependencies, and hierarchical relationships.

### Rationale
In the event of a critical zero-day vulnerability (e.g., Log4Shell), the SBOM allows both the Platform Team and the Client's Security Team to instantly identify if and where a vulnerable library is used across the entire integration ecosystem.

### Execution
* **Standard Format:** OCIP generates SBOMs in the **CycloneDX** or **SPDX** standard.
* **Automated Generation:** The CI/CD pipeline (Quality Gate 1) automatically regenerates the SBOM during every build, capturing not only direct dependencies but also transitive ones.
* **Transparency:** The SBOM is provided to the client as part of the Release Notes for every platform upgrade.

---

## 3. Vulnerability Management and Patching

### Statement
The platform's security posture is only as strong as its oldest dependency. OCIP treats dependency updates as a critical operational task.

### Execution
* **Continuous Scanning:** The platform uses automated tools (e.g., Snyk, Trivy) to cross-reference the SBOM against the National Vulnerability Database (NVD).
* **SLA for Patches:**
    * **Critical Vulnerabilities:** Patch or mitigation must be available within 7 business days.
    * **High Vulnerabilities:** Patch provided in the next minor platform release.
* **Zero-Downtime Patching:** Leveraging the GitOps delivery model, security patches for Tier A and B components are rolled out as automated updates without interrupting Tier C business integrations.

---

## 4. Proprietary vs. Open Source Boundaries

### Statement
The platform maintainer (the vendor) retains full ownership of the **OCIP Control Plane** (management UI, proprietary governance logic, and automation scripts), while the **Data Plane** (execution engines) remains standard open-source.

### Implications for Clients
* Clients pay for the value-add of the Control Plane (automation, governance, support).
* Clients retain 100% ownership of their **Tier C Integration Logic** (Camel routes, mapping code), which is built using native, portable open-source standards.
* This separation ensures that if a client chooses to stop using the OCIP Control Plane, their core business integration logic remains functional and portable to any standard Kubernetes/Camel environment.
