# AI and LLM Integration Standards

## Document Purpose
This document defines the reference architecture and standards for integrating Artificial Intelligence (AI) and Large Language Models (LLMs) within the Platform. It outlines how the Platform functions as an "AI Middleware," facilitating safe, governed, and scalable interactions between enterprise data, legacy systems, and AI models.

---

## 1. AI API Management (Edge Layer)

Directly connecting microservices or frontends to external LLM providers (e.g., OpenAI, Anthropic) is strictly prohibited. All AI traffic must be mediated by the Edge Layer (Tier B) to ensure security, compliance, and cost control.

### Key Capabilities:
* **Token-Based Rate Limiting:** Throttling policies must be based on Tokens Per Minute (TPM) rather than standard Requests Per Second (RPS) to prevent budget overruns.
* **LLM Routing and Fallback:** The Gateway must implement dynamic routing (Circuit Breaker). If the primary model (e.g., GPT-4) fails or rate-limits the request, the Gateway automatically reroutes the prompt to a fallback model (e.g., GPT-3.5 or an internal LLaMA model).
* **Semantic Caching:** To reduce latency and API costs, the Gateway should utilize the Persistence Layer (e.g., Redis) to cache frequent, semantically similar prompts.

---

## 2. RAG Data Ingestion Pipeline (Integration Layer)

Retrieval-Augmented Generation (RAG) requires continuous synchronization between enterprise systems and Vector Databases. 

### Ingestion Standard:
* **The "Thin Route" Data Loader:** Apache Camel must be used to extract data from source systems (APIs, databases, legacy FTPs), transform the data, and stream it via the Event Layer (Kafka).
* **Chunking and Embedding:** Event consumers process the raw text, apply chunking strategies, generate vector embeddings via an embedding model, and sink the data into the Vector Database (Persistence Layer).

---

## 3. Agentic Orchestration (Processing Layer)

While Camel handles the data pipelines, intelligent decision-making is delegated to the Processing Layer.

### Orchestration Standard:
* **Framework:** Complex LLM chains and Agentic workflows must be built within Quarkus using frameworks like **LangChain4j**.
* **Tool Calling:** When an LLM Agent requires enterprise data (e.g., "Check customer balance"), it must not connect to the database directly. It must invoke standard, governed internal APIs exposed on the Platform.

---

## 4. Observability and FinOps for AI

Traditional metrics are insufficient for AI operations. The Observability Layer must capture AI-specific telemetry.

### FinOps Standards:
* **Token Tracking:** Every request passing through the Gateway must log token usage, tagged with the `domain-owner` and `cost-center` for accurate monthly chargeback.
* **Prompt Logging Policy:** Full prompts and LLM responses must NOT be logged to standard observability stacks by default due to strict PII/GDPR risks. Only operational metadata (latency, tokens, model version, status) is logged unless a specific audit mode is explicitly approved.
