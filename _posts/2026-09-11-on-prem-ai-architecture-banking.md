---
title: "On-Prem AI Architecture for Banking: Privacy, RAG and Operational Control"
description: "Designing enterprise AI systems that keep sensitive data inside the institution while remaining searchable, observable and governable."
categories: [AI Architecture, Banking]
tags: [on-prem, banking, kvkk, rag, llm, security, keycloak, observability]
---

In regulated environments, the first AI architecture decision is often not the model. It is the data boundary.

A bank may want meeting intelligence, document extraction, semantic search or an internal assistant, but customer information, employee conversations and operational records cannot be sent freely to external services. An on-premises design gives the institution greater control, yet it also transfers responsibility for model serving, identity, data retention, monitoring and incident response to the internal platform.

The real objective is not “run an LLM locally.” It is to build a governed information system in which AI is one replaceable component.

## Begin with data classification

Before choosing a vector database or model, identify the information that will enter the system:

- audio and transcripts;
- customer or employee identifiers;
- documents and attachments;
- generated summaries and action items;
- embeddings;
- prompts, responses and feedback;
- operational logs and traces.

Embeddings are sometimes treated as harmless numeric data. They are derived from source content and should remain within the same protection model unless a formal assessment concludes otherwise.

Define where each data type is stored, how long it is retained, who can access it and how deletion propagates across primary storage, search indexes, vector indexes and backups.

## Build a layered service architecture

An enterprise AI assistant benefits from service boundaries that separate deterministic platform concerns from model behavior.

A typical architecture may include:

- an API Gateway for external policy enforcement;
- an application API for domain workflows;
- asynchronous workers for transcription and enrichment;
- an ASR service for speech-to-text;
- speaker diarization;
- an LLM gateway that abstracts model providers;
- PostgreSQL for transactional state;
- object storage for recordings and documents;
- OpenSearch for keyword and filtered search;
- pgvector or another vector store for semantic retrieval;
- RabbitMQ for durable background processing;
- Keycloak for identity and access management.

These services do not need to become independent deployment units on day one. The boundaries matter more than the count. Split components when scaling, security isolation, release ownership or failure containment justifies it.

## Treat model access as an internal platform

Applications should not integrate directly with a specific model endpoint. An internal LLM gateway can provide a stable contract for model routing, timeouts, token limits, auditing and fallback.

This layer also makes model replacement practical. A summarization workload, a retrieval answer and a document classification task may require different models. Routing can consider sensitivity, latency, quality and available compute without exposing those details to every product team.

Model prompts should be versioned like source code. A production result must be traceable to the model, prompt version, retrieval configuration and application release that produced it.

## RAG is an authorization problem

Retrieval-Augmented Generation is often described as search followed by generation. In an enterprise system, retrieval must be authorization-aware.

The user must never receive a document through semantic search that they could not open through the source application. Apply access filters before candidate content reaches the model. Do not rely on the LLM to ignore unauthorized context.

A hybrid retrieval flow can combine:

- lexical search for names, identifiers and exact phrases;
- vector similarity for semantic meaning;
- metadata filters for organization, project, date and classification;
- reranking for relevance;
- citations back to the authorized source.

OpenSearch and pgvector can complement each other when each has a deliberate role. The correct combination depends on scale, filtering needs, language behavior and operational maturity.

## Use asynchronous processing deliberately

Audio transcription, document OCR, embedding generation and batch enrichment can take longer than an interactive request. A durable queue separates user-facing APIs from these workloads and allows controlled retries.

Each job should be idempotent. Retries must not duplicate meeting records, notifications or extracted actions. Store job state and error categories so operators can distinguish transient infrastructure failure from invalid input or model failure.

Backpressure is critical. If GPU capacity falls behind incoming work, the system should expose queue delay, prioritize important workloads and protect interactive services rather than allowing every dependency to time out.

## Identity and tenant boundaries

Keycloak can provide OIDC-based authentication and centralized role management, but application services still enforce domain authorization. A role such as administrator may apply only to one organization or business unit.

Tenant identity should be carried consistently through APIs, queues, storage keys, search documents and audit events. Background workers must not lose authorization context simply because no human request is active at processing time.

For sensitive actions, record who initiated the job, which service processed it and which protected resources were read.

## Design for model uncertainty

Traditional services are expected to return deterministic results for the same input. AI services may produce incomplete, inconsistent or unsupported answers. The product must make that uncertainty visible and controllable.

Useful controls include:

- source citations for retrieval-based answers;
- confidence or quality thresholds for extraction;
- schema validation for structured output;
- human review for high-impact decisions;
- refusal when authorized evidence is insufficient;
- regression datasets that represent real language and document types.

An LLM should assist a banking workflow, not silently become the system of record. Deterministic services remain responsible for balances, eligibility, payments and formal approvals.

## Observe business outcomes, not only infrastructure

CPU, GPU memory and request latency are necessary metrics. They do not reveal whether the assistant produces useful and safe results.

Measure:

- transcription and speaker-separation quality;
- retrieval relevance;
- unsupported-answer rate;
- processing delay by workflow stage;
- queue depth and retry frequency;
- model and prompt version distribution;
- user corrections and rejected outputs;
- access-denied events and unusual retrieval patterns.

Logs must support diagnosis without becoming a second uncontrolled copy of sensitive content. Prefer identifiers, timings and classified error details over complete prompts and documents.

## Operate with controlled change

Model upgrades can change behavior without an API contract change. Evaluate candidate models against a versioned test set before promotion. Use canary traffic or shadow evaluation where policy permits, and maintain a rollback path for the model and prompt configuration.

Deployment automation should also understand business context. If an active meeting or critical processing job cannot tolerate interruption, postpone a rollout or drain work safely before replacing services.

Every production change should identify the application version, model version, prompt version and migration state. This is essential for auditability and incident analysis.

## The architectural principle

On-premises AI is not automatically private, compliant or reliable. It becomes governable when the data boundary, identity model, retrieval permissions, model lifecycle and operational controls are designed together.

The strongest architecture keeps AI replaceable. Models will change quickly; authorization rules, audit requirements and business accountability will not. Build the durable platform around those responsibilities, and allow the models to evolve inside it.
