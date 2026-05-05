# OCIP: CI/CD Quality Gates and Pipeline Standards

## Document Purpose
This document defines the mandatory automated CI/CD pipeline stages and Quality Gates for the Open Composable Integration Platform (OCIP). To ensure delivery scalability and prevent human bottlenecks, all Tier C (Customer-Owned) extensions must pass strict automated validation before being deployed to the platform via the GitOps workflow.

---

## 1. The "Shift-Left" Quality Philosophy

### Statement
OCIP enforces a "Shift-Left" approach to security, compliance, and architectural governance. Every integration standard defined in the platform documentation must be automatically verifiable within the Continuous Integration (CI) pipeline.

### Rationale
Manual code reviews are error-prone and do not scale across hundreds of integrations and domain teams. By automating architectural checks (e.g., verifying if health probes exist, or if APIs match their contracts), the platform ensures ironclad enterprise governance without causing a bureaucracy explosion.

### Implications
* A Pull Request (PR) cannot be merged if any Quality Gate fails.
* The CI pipeline is centrally managed and provided by the OCIP Platform Team as part of the "Golden Path" repository templates. Domain teams consume these pipelines but cannot bypass or alter the core validation steps.

---

## 2. The Continuous Integration (CI) Quality Gates

When a domain developer pushes code or opens a Pull Request, the CI pipeline executes the following mandatory gates sequentially:

### Gate 1: Static Application Security Testing (SAST) and Linting
* **Execution:** The pipeline scans the application source code and dependencies for known vulnerabilities (e.g., CVEs) and code quality violations.
* **Checks:**
  * Secrets detection (blocking commits containing hardcoded API keys or passwords).
  * Dependency scanning (blocking outdated libraries with known vulnerabilities).
  * Code style linting to ensure maintainability of the Camel routes and Quarkus beans.

### Gate 2: Automated Testing and Coverage
* **Execution:** The pipeline executes all unit and integration tests using frameworks like Quarkus Testcontainers.
* **Checks:**
  * Minimum code coverage thresholds must be met (e.g., 80%).
  * Tests must run in a fully isolated, ephemeral environment without requiring access to production endpoints.

### Gate 3: Contract Validation
* **Execution:** The platform is API-first and event-driven by default, relying strictly on OpenAPI and AsyncAPI specifications.
* **Checks:**
  * **API Linting:** Validates that the provided OpenAPI/AsyncAPI specification adheres to enterprise standards (e.g., naming conventions, required headers).
  * **Contract Compatibility:** Ensures that the newly compiled code correctly implements the defined contract, and checks for backward-breaking changes (e.g., removing a mandatory field from an existing JSON payload) to protect downstream consumers.

### Gate 4: Policy-as-Code (Kubernetes Manifest Validation)
* **Execution:** Before generating the final deployment artifacts, the pipeline scans the Helm charts or Kubernetes manifests using Policy-as-Code tools (e.g., Open Policy Agent / Conftest).
* **Checks:**
  * Ensures `Liveness` and `Readiness` probes are defined.
  * Verifies that strict CPU and Memory limits/requests are configured to prevent "noisy neighbor" scenarios.
  * Validates the presence of mandatory billing and ownership tags/labels.

---

## 3. GitOps Handoff and Deployment

If all CI Quality Gates pass successfully, the pipeline transitions to the delivery phase.

### Immutable Build
The pipeline builds the container image and tags it with an immutable version (e.g., the Git commit hash). Using `latest` tags is strictly prohibited to guarantee deployment reproducibility.

### The GitOps Sync
1. The CI pipeline pushes the immutable container image to the central OCIP Container Registry.
2. The pipeline automatically updates the corresponding GitOps repository (the repository holding the target cluster state) with the new image tag.
3. The platform's native GitOps engine (ArgoCD) detects the change in the Git repository and safely synchronizes the Kubernetes cluster to match the new state, entirely eliminating manual `kubectl` interventions.
