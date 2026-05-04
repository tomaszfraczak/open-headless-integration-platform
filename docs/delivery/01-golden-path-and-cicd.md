# OCIP: Golden Path and GitOps Delivery Workflow

## Document Purpose
This document outlines the standard delivery lifecycle for integration developers using the Open Composable Integration Platform (OCIP). It explains how domain teams build, test, and deploy integrations without relying on a proprietary, closed-source UI portal.

---

## 1. Git as the Single Pane of Glass

### Philosophy
OCIP adopts a **"Lean Platform Engineering"** approach. We do not force developers into a custom-built "Integration Portal" UI. Instead, the version control system (e.g., GitHub/GitLab) serves as the primary developer interface. 

### Rationale
Developers are already comfortable with Git, Pull Requests, and IDEs. Building a custom web portal to click "Deploy" adds unnecessary abstraction. By using GitOps, every change—whether it is a line of Java code, a memory limit increase, or a new API route—is tracked, reviewable, and instantly reproducible.

---

## 2. The Delivery Workflow (Step-by-Step)

When a domain team needs to build a new integration (Tier C), they follow this standardized workflow:

### Step 1: Bootstrap via Template (The Golden Path)
Instead of starting from an empty repository and figuring out how to connect to the platform, the developer clicks **"Use this template"** on an official OCIP GitHub Template Repository (e.g., `ocip-camel-quarkus-template`).
* **What it does:** This instantly provisions a repository pre-configured with the correct Maven/Gradle build scripts, Dockerfiles, standard logging dependencies, and Helm charts. The developer immediately starts writing business logic.

### Step 2: Local Development and Testing
The developer writes their Apache Camel routes using their preferred IDE (IntelliJ, VS Code). 
* **Testing:** Utilizing Camel's native testing frameworks and Quarkus Testcontainers, the integration is verified locally without needing access to the production platform.

### Step 3: Pull Request and CI Validation
Once the logic is ready, the developer opens a Pull Request (PR).
* **Automated Checks:** GitHub Actions (or GitLab CI) automatically run unit tests, enforce code linting, scan for vulnerabilities (e.g., exposed secrets), and validate that the architectural contracts (e.g., OpenAPI specs) are unbroken.
* **Approval:** Senior engineers or domain architects approve the PR.

### Step 4: Merge and GitOps Sync (ArgoCD)
Once merged to the `main` branch, the CI pipeline builds the container image and pushes it to the registry. 
* **The Magic:** The developer does *not* run `kubectl apply`. ArgoCD (the GitOps controller residing in the OCIP Control Plane) detects the change in the Git repository and automatically synchronizes the cluster state with the code.

### Step 5: Operations via Native Tools
Post-deployment, developers use standard, best-in-class UI tools provided by the platform:
* **Deployment Status:** Viewed in the ArgoCD UI.
* **Logs & Traces:** Viewed in Grafana/Loki (or the client's compatible swapped component, like Azure Monitor).
* **API Metrics:** Viewed in the API Gateway dashboard.
