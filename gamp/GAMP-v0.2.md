# Governed Agent Memory Pipeline (GAMP)

## An 8-Layer Architecture for Governed, Verifiable Agent Memory

**Version:** 0.2 (Draft)
**Authors:** Rudi Heydra, Ada (AI Co-Founder) — Substr8 Labs
**Date:** April 2026

---

## Abstract

Persistent memory is becoming a foundational requirement for autonomous AI agents operating across sessions, workflows, teams, and enterprise environments. Yet most agent memory systems remain optimized for a narrow concern: retrieval quality. Retrieval-Augmented Generation (RAG), MemGPT, Mem0, LangMem, and similar systems improve recall, extraction, or memory management, but typically do not define a full memory lifecycle architecture with explicit boundaries for provenance, governance, execution binding, and proof.

This paper introduces the **Governed Agent Memory Pipeline (GAMP)**, an 8-layer architecture that decomposes agent memory into independently evolvable stages: source artifacts, ingestion, normalization, canonical storage, serving, governance, execution binding, and proof attestation. Each layer has a defined responsibility and an explicit contract at its boundary. The architecture is grounded in prior Substr8 Labs work on **GAM** (Git-Native Agent Memory), **ACC** (Agent Capability Control), and **RunProof**.

GAMP makes four primary contributions. First, it defines a **canonical-store / derived-serving separation**, treating memory serving systems as reconstructible indexes rather than sources of truth. Second, it introduces **governed recall** as a first-class boundary, where memory access is filtered through explicit authorization and state controls before prompt injection. Third, it defines the **RecallEnvelope**, an immutable execution-binding artifact that freezes the governed memory context presented to an agent. Fourth, it binds memory recall to execution via cryptographic proof, enabling post-hoc verification of what memory was included, what was suppressed, and which canonical state was active at run time.

We also describe an operational pattern called **Agent-as-Extractor**, in which the agent performs structured fact extraction within its own session rather than through a separate extraction-model call. In a reference implementation running on commodity infrastructure, GAMP supports 1,800+ governed memories, sub-second governed recall, and proof-linked execution across the memory lifecycle.

GAMP is not a retrieval benchmark and does not claim to prove semantic causation between every recalled memory and every output token. Its claim is narrower and more operational: that agent memory can be modeled as a governed, replayable, and attestable pipeline rather than an opaque retrieval sidecar.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Background and Related Work](#2-background-and-related-work)
3. [Architecture: The 8-Layer Model](#3-architecture-the-8-layer-model)
4. [Agent-as-Extractor](#4-agent-as-extractor)
5. [Data Flow and Memory Lifecycle](#5-data-flow-and-memory-lifecycle)
6. [Governance Model](#6-governance-model)
7. [Formal Properties](#7-formal-properties)
8. [Reference Implementation](#8-reference-implementation)
9. [Evaluation](#9-evaluation)
10. [What GAMP Does and Does Not Prove](#10-what-gamp-does-and-does-not-prove)
11. [Discussion](#11-discussion)
12. [Conclusion](#12-conclusion)
13. [References](#13-references)

---

## 1. Introduction

### 1.1 The Memory Problem

Autonomous AI agents increasingly operate in contexts that require durable memory: multi-session assistants, long-running workflows, collaborative agent teams, and enterprise automation systems. In such environments, memory is not merely a convenience feature. It becomes part of the system's operational substrate.

Yet the dominant pattern for agent memory remains structurally simple: embed content, store vectors, retrieve by similarity, inject into prompt. This pattern is often effective for recall, but it leaves core operational questions unresolved:

* **Provenance:** What artifact, conversation, event, or document produced this memory?
* **Integrity:** Has this memory changed since creation, and if so, how is that change represented?
* **Governance:** Was the requesting agent authorized to access this memory at recall time?
* **Attribution:** Which memories were eligible, which were recalled, and which were excluded?
* **Replayability:** Can the memory state presented to the agent be reconstructed later?

These concerns become especially important in enterprise and regulated settings, where inspectability, auditability, and controlled access are not optional.

### 1.2 The Retrieval-Centric Trap

The current ecosystem has largely converged on retrieval quality as the dominant optimization target. This is understandable. Retrieval is measurable, developer-visible, and directly linked to answer quality. But retrieval quality is only one segment of the memory problem.

A memory system that retrieves relevant information but cannot express memory state transitions, suppress stale memories without deleting lineage, govern access by capability, or bind memory context to execution remains incomplete for high-trust deployments.

This is not an argument against RAG or vector search. It is an argument that retrieval alone is not a sufficient systems model for agent memory.

### 1.3 Why Layers Matter

Early network stacks were often monolithic. Framing, routing, transport, and application behavior were entangled. The introduction of layered architectures created explicit boundaries, allowing each layer to evolve independently while preserving interoperability.

Agent memory today often resembles a pre-layered system. Extraction, storage, retrieval, filtering, and prompt injection are frequently bundled into a single opaque mechanism. GAMP applies layered discipline to agent memory: each layer has a defined responsibility, a clear input-output contract, and an invariant that downstream systems can rely on.

### 1.4 Scope and Claim

This paper is an **architecture paper**, not a retrieval benchmark paper. Its central claim is that agent memory can be modeled as a governed pipeline with explicit lifecycle stages, rather than as an informal retrieval helper attached to inference.

The architecture is intended for environments where the following properties matter:

* memory provenance
* controlled recall
* auditability of suppression and inclusion
* replayability of memory state
* execution binding and proof

### 1.5 Contributions

This paper makes five contributions:

1. **An 8-layer architecture** for agent memory spanning source artifacts through proof attestation, with explicit contracts at each boundary.
2. **Canonical-store / derived-serving separation**, treating fast serving layers as reconstructible indexes rather than sources of truth.
3. **Governed recall** as a first-class boundary, using explicit authorization and governance state before prompt injection.
4. **RecallEnvelope**, an immutable artifact that freezes the governed memory context delivered to the agent.
5. **Proof-bound execution**, linking governed recall state to execution output via RunProof.

In addition, we describe **Agent-as-Extractor** as an operational pattern for reducing extraction overhead when memory capture occurs inside the agent's own session.

---

## 2. Background and Related Work

### 2.1 Retrieval-Augmented Generation

RAG established the dominant pattern for grounding language model outputs in external knowledge. Documents are embedded, retrieved at inference time, and included in prompt context. This pattern is effective for many knowledge-intensive tasks and has produced a broad ecosystem of vector databases and retrieval frameworks.

However, RAG primarily addresses the problem of retrieval. It does not, by itself, define a full memory lifecycle model with governance states, provenance chains, execution binding, or reconstructible canonical state.

### 2.2 Agent Memory Systems

Several systems have advanced the state of the art in agent memory and long-context handling:

* **MemGPT** introduces tiered memory management inspired by operating systems.
* **Mem0** focuses on extracting structured facts from conversations for later recall.
* **LangMem** provides developer-facing long-term memory utilities in the LangChain ecosystem.
* **Zep** emphasizes session memory, temporal awareness, and assistant continuity.

These systems improve different dimensions of memory, including extraction quality, consolidation, temporal context, or developer ergonomics. GAMP is complementary to this work. Its focus is narrower and more infrastructural: defining a full lifecycle architecture in which memory has provenance, governance state, explicit serving boundaries, and execution proof linkage.

### 2.3 Trust and Verification in Agent Systems

Verification in AI systems is often discussed at the model or output level: alignment, policy enforcement, monitoring, guardrails, or constitutional behavior. Less attention has been given to the operational integrity of memory itself.

For many agent systems, the critical question is not only whether the output was acceptable, but whether the system can reconstruct:

* what memory was available,
* what memory was governed out,
* what canonical state existed at execution time,
* and what execution artifact binds those facts together.

GAMP is designed to address that operational gap.

### 2.4 Prior Substr8 Labs Work

GAMP composes prior Substr8 Labs work into a single architecture:

| Paper | Contribution to GAMP |
| ----------------- | ------------------------------------------------------------------------ |
| **GAM v3.1** | Git-native memory storage, retrieval, and reconstructible memory serving |
| **ACC v2.0** | Capability-based governance and authorization at recall time |
| **RunProof v0.1** | Cryptographic execution attestation and run receipts |
| **FDAA v1.3** | File-driven execution context and skill-based agent architecture |

GAMP does not replace these components. It defines how they interoperate as a governed memory pipeline.

---

## 3. Architecture: The 8-Layer Model

### 3.1 Overview

GAMP decomposes agent memory into eight layers:

```
┌─────────────────────────────────────────────────────────────────┐
│ Layer 7 — Proof Plane          RunProof attestation            │
├─────────────────────────────────────────────────────────────────┤
│ Layer 6 — Execution Binding    RecallEnvelope → prompt         │
├─────────────────────────────────────────────────────────────────┤
│ Layer 5 — Governance Plane     ACC capability filtering        │
├─────────────────────────────────────────────────────────────────┤
│ Layer 4 — Serving Plane        GAM + graph / retrieval indexes │
├─────────────────────────────────────────────────────────────────┤
│ Layer 3 — Canonical Store      Delta Lake (source of truth)    │
├─────────────────────────────────────────────────────────────────┤
│ Layer 2 — Normalization        Artifacts → Segments → Facts    │
├─────────────────────────────────────────────────────────────────┤
│ Layer 1 — Ingestion Adapters   Extraction engines              │
├─────────────────────────────────────────────────────────────────┤
│ Layer 0 — Source Artifacts     Conversations, docs, events     │
└─────────────────────────────────────────────────────────────────┘
```

**Design principle:** each layer has one primary responsibility, and downstream layers consume explicit outputs from upstream layers rather than reaching into internal implementation details.

### 3.2 Layer Contract Format

Each layer is defined by:

* **Input**
* **Output**
* **Responsibility**
* **Invariant**

This gives the architecture its operational discipline.

---

### 3.3 Layer 0 — Source Artifacts

**Input:** raw external events or content
**Output:** immutable captured artifacts
**Responsibility:** preserve original source content with metadata
**Invariant:** source artifacts are never modified downstream

Examples include conversation transcripts, uploaded documents, calendar events, logs, code commits, and API responses.

**Schema:**

```json
{
  "artifact_id": "art_<timestamp>_<random>",
  "source_type": "conversation | document | event | api | code",
  "raw_content": "<original content>",
  "metadata": {
    "session_id": "<session identifier>",
    "agent_id": "<capturing agent>",
    "tenant_id": "<org scope>",
    "captured_at": "<ISO 8601>"
  }
}
```

---

### 3.4 Layer 1 — Ingestion Adapters

**Input:** source artifacts
**Output:** extracted segments or structured candidate memory material
**Responsibility:** convert raw artifacts into extractable units
**Invariant:** every extracted unit references its originating artifact

Layer 1 is intentionally pluggable. It may be backed by:

* in-session extraction by the agent
* external extraction services
* document parsers
* event processors
* custom domain adapters

---

### 3.5 Layer 2 — Normalization

**Input:** extracted segments or candidate facts
**Output:** canonical normalized facts
**Responsibility:** schema enforcement, deduplication, confidence scoring, scope assignment
**Invariant:** all candidate memories entering Layer 3 conform to a common schema

**ExtractedFact schema:**

```json
{
  "fact_id": "fct_<timestamp>_<random>",
  "content_hash": "<hash of normalized content>",
  "fact_type": "decision | preference | fact | entity | relationship | event | goal | instruction | correction | context",
  "content": "<normalized fact text>",
  "confidence": 0.0,
  "status": "candidate",
  "scope": "org | project | agent",
  "tenant_id": "<tenant>",
  "agent_id": "<extracting agent>",
  "provider": "<adapter or model source>",
  "extraction_model": "<model identifier>",
  "extracted_at": "<ISO 8601>",
  "source_run_id": "<run/session>",
  "artifact_id": "<origin artifact>"
}
```

Normalization is also the first place where stale knowledge can be represented structurally, for example through `correction` or `supersedes` relationships.

---

### 3.6 Layer 3 — Canonical Store

**Input:** normalized facts and state transitions
**Output:** durable, versioned memory truth
**Responsibility:** preserve the canonical state of memory over time
**Invariant:** if a serving layer disagrees with Layer 3, Layer 3 wins

GAMP uses Delta Lake in the reference implementation, but the architectural principle is broader: the canonical store must be durable, versioned, queryable across time, and capable of reconstructing downstream indexes.

Key properties:

* transactional updates
* time-travel or historical replay
* schema evolution
* append-only lineage for state transitions
* reconstructibility of serving indexes

**Canonical truth principle:** serving layers are derived. The canonical store is authoritative.

---

### 3.7 Layer 4 — Serving Plane

**Input:** promoted canonical memories
**Output:** low-latency retrieval and traversal results
**Responsibility:** provide fast query-time access
**Invariant:** serving layers do not define truth; they materialize it

In the reference implementation, Layer 4 includes:

* **GAM** for hybrid semantic retrieval
* **Neo4j** for graph traversal and relationship inspection

The serving plane may include vector indexes, graph indexes, keyword indexes, or hybrid search engines. What matters architecturally is that they are **derived from Layer 3**.

---

### 3.8 Layer 5 — Governance Plane

**Input:** candidate recall results from Layer 4
**Output:** governed recall set
**Responsibility:** filter memory visibility according to policy, capability, and state
**Invariant:** no memory reaches execution without passing governance

This layer is implemented via ACC in the reference design.

Governance decisions include:

* include
* suppress
* restrict
* expire

The key design principle is that **suppression is not deletion**. A suppressed memory remains part of canonical history and audit lineage, but is excluded from recall.

---

### 3.9 Layer 6 — Execution Binding

**Input:** governed memory set
**Output:** immutable RecallEnvelope
**Responsibility:** freeze the exact memory context delivered to the agent
**Invariant:** the execution-bound memory set is hashable, serializable, and auditable

**RecallEnvelope schema:**

```json
{
  "envelope_id": "env_<timestamp>_<random>",
  "agent_id": "<requesting agent>",
  "query": "<recall query>",
  "eligible_memory_ids": ["mem_1", "mem_2", "mem_3"],
  "recalled_memory_ids": ["mem_1", "mem_3"],
  "suppressed_memory_ids": ["mem_2"],
  "injected_content": [
    {
      "memory_id": "mem_1",
      "type": "decision",
      "content": "<memory text>"
    }
  ],
  "frozen_at": "<ISO 8601>",
  "envelope_hash": "<hash of serialized envelope>"
}
```

This layer is a central contribution of GAMP. It transforms memory recall from an informal runtime event into an explicit, frozen execution artifact.

---

### 3.10 Layer 7 — Proof Plane

**Input:** RecallEnvelope and execution output
**Output:** proof-bound run artifact
**Responsibility:** bind execution to memory state and identity state
**Invariant:** proof artifacts are verifiable against the canonical memory state active at run time

**Proof schema:**

```json
{
  "proof_id": "prf_<timestamp>_<random>",
  "run_id": "<execution run identifier>",
  "envelope_id": "env_<...>",
  "envelope_hash": "<must match Layer 6>",
  "agent_identity_hash": "<workspace or identity hash>",
  "canonical_state_ref": "<Delta version or commit>",
  "output_hash": "<hash of output>",
  "timestamp": "<ISO 8601>",
  "signature": "<cryptographic signature>"
}
```

This does not prove that every recalled memory semantically caused the output. It proves that a particular governed memory context was bound to the execution environment that produced the output.

---

## 4. Agent-as-Extractor

### 4.1 Motivation

Many memory systems use a separate extraction pass to distill conversations into facts. This often requires an additional model invocation beyond the primary agent session.

GAMP introduces an operational pattern called **Agent-as-Extractor**, in which the agent performs structured extraction within its own session before the session closes or compacts.

### 4.2 Claim

The claim is deliberately narrow:

> Agent-as-Extractor removes the need for a separate extraction-model call when memory extraction occurs inside the agent's existing session context.

This is more precise than claiming "free extraction" in the abstract. Token usage still exists, but a separate extraction invocation is no longer required.

### 4.3 Advantages

* extraction occurs with full conversational context available
* no separate extraction service is required
* output can conform directly to the normalization schema
* extraction behavior can be governed through skills or runtime policy

### 4.4 Limitations

Agent-as-Extractor is not universally applicable. Bulk document ingestion, offline corpus processing, and high-volume event pipelines may still require external adapters.

For that reason, Layer 1 remains pluggable.

---

## 5. Data Flow and Memory Lifecycle

### 5.1 Ingestion Path

The ingestion path is:

```
Source Artifact
  → Ingestion Adapter
    → Normalization
      → Canonical Store
        → Promotion / Review
          → Serving Materialization
```

New memories enter the canonical store as **candidate** records rather than immediately participating in recall.

### 5.2 Recall Path

The recall path is:

```
Recall Query
  → Serving Plane
    → Governance Plane
      → RecallEnvelope
        → Prompt Injection
          → Agent Output
            → Proof Binding
```

This path is intentionally separate from ingestion, except for convergence at the canonical store.

### 5.3 Memory State Machine

A simplified memory state model is:

```
candidate
  ├──> canonical
  │     ├──> active
  │     ├──> suppressed
  │     ├──> restricted
  │     └──> expired
  └──> rejected
```

### 5.4 Correction and Supersession

A governed memory system must represent stale or corrected knowledge without destroying lineage.

Accordingly, GAMP treats correction as a first-class operation:

* a corrected memory may **supersede** an earlier one
* the earlier memory may be **suppressed** from serving
* both remain present in canonical history
* the recall and proof layers can surface the difference between eligible, recalled, and suppressed knowledge

This is essential for counterfactual inspection and stale-memory remediation.

---

## 6. Governance Model

### 6.1 Capability-Based Recall

GAMP uses capability-based governance rather than simple role assignment. Relevant permissions include:

* `memory:recall:org`
* `memory:recall:project:<id>`
* `memory:recall:agent:<id>`
* `memory:write:candidate`
* `memory:promote`
* `memory:suppress`

This model enables fine-grained attenuation when agents delegate work.

### 6.2 Scope Inheritance

Memory scope is hierarchical:

```
org
  └── project
        └── agent
```

An agent may inherit broader scoped memories depending on policy, project membership, and delegated capability.

### 6.3 Suppression as Governance, Not Erasure

One of GAMP's core design principles is:

> Governed memory should be suppressible without becoming untraceable.

This enables audit, rollback, replay, and stale-memory debugging.

### 6.4 Governance Transparency

The RecallEnvelope makes governance visible by distinguishing:

* what was eligible,
* what was recalled,
* what was suppressed.

That visibility is itself part of the trust model.

---

## 7. Formal Properties

### 7.1 Provenance Completeness

**Statement:** Every canonical memory can be traced through explicit references back to a source artifact.

**Operational implication:** operators can inspect where a memory came from and what upstream artifact lineage produced it.

### 7.2 Governance Transparency

**Statement:** For every RecallEnvelope, the system records both eligible and recalled memories, plus those excluded by governance.

**Operational implication:** a governed recall event is inspectable rather than opaque.

### 7.3 Canonical Precedence

**Statement:** The canonical store has precedence over all serving layers.

**Operational implication:** serving indexes can be discarded and rebuilt without loss of memory truth.

### 7.4 Reconstructibility

**Statement:** Given canonical state at time *t*, the serving layer at time *t* can be materialized from it.

**Operational implication:** retrieval systems remain replaceable and audit replay remains feasible.

### 7.5 Execution Binding

**Statement:** Every proof-bound run references a frozen RecallEnvelope and a canonical state reference.

**Operational implication:** the memory context presented to the agent is not merely inferred after the fact; it is an execution artifact.

### 7.6 Replay Invariance

**Statement:** A governed recall event can be replayed against the canonical state reference to verify envelope integrity.

**Operational implication:** post-hoc verification can test whether the bound memory state matches the claimed execution context.

---

## 8. Reference Implementation

### 8.1 Deployment Shape

The reference implementation runs on commodity infrastructure and uses:

* Delta Lake as canonical store
* GAM for retrieval
* Neo4j for graph traversal
* ACC for governance
* RunProof for execution attestation
* FDAA skills for extraction behavior

### 8.2 Service Roles

| Layer | Service | Role |
| ----- | -------------- | ------------------------------------------- |
| 0–2 | Memory Plane | capture, extract, normalize |
| 3 | Delta Lake API | canonical storage and versioned truth |
| 4 | GAM | hybrid retrieval |
| 4 | Neo4j | graph traversal and relationship inspection |
| 5 | ACC | recall governance |
| 7 | RunProof | execution proof |

### 8.3 Operational Notes

The implementation currently maintains a governed corpus of 1,800+ memories and supports governed recall with low-latency query performance. Specific infrastructure details may evolve, but the architectural roles remain constant.

---

## 9. Evaluation

### 9.1 Evaluation Scope

This section evaluates GAMP as a **reference implementation demonstration**, not as a universal benchmark against all memory systems.

The evaluation asks four questions:

1. Can the layered architecture be implemented end to end?
2. Can governed recall be executed with explicit eligibility and suppression tracking?
3. Can a frozen recall artifact be bound to execution proof?
4. Can this run on ordinary infrastructure with operationally acceptable latency?

### 9.2 Functional Validation

An end-to-end validation cycle was executed:

1. facts were captured and normalized into candidate memory records
2. candidate records were promoted into canonical memory
3. canonical memory was materialized into serving layers
4. a governed recall query produced eligible and recalled memory sets
5. a RecallEnvelope was frozen and hashed
6. execution output was bound to the envelope via proof

This validates the full lifecycle from ingestion to proof.

### 9.3 Operational Footprint

Reference implementation metrics include:

| Metric | Value |
| --------------------------------- | --------------------------------------------------------- |
| Total memories in governed corpus | 1,800+ |
| Governed recall latency | sub-second in reference deployment |
| Services in active deployment | memory plane, canonical store, serving, governance, proof |

These figures are descriptive of the reference deployment, not a claim of general performance across all environments.

### 9.4 Cost Model

Under Agent-as-Extractor, extraction occurs within the agent's own session rather than through a separate extraction-model call.

This yields a narrower and more defensible economic claim:

* **No separate extraction invocation is required**
* **No dedicated extraction service is required for in-session capture**
* **Incremental operational complexity is reduced**

This is distinct from claiming that extraction has zero total computational cost.

### 9.5 Current Evaluation Limits

The current evaluation does **not** yet provide:

* standardized benchmark comparison against alternative memory systems
* formal measurement of governance precision across heterogeneous workloads
* controlled ablation studies across retrieval engines or embedding models
* semantic causality tests proving that each recalled memory was necessary for each output

Those are important future directions, but they are outside the scope of this v0.2 architecture paper.

---

## 10. What GAMP Does and Does Not Prove

### 10.1 What GAMP Proves

GAMP is designed to prove the following operational properties:

* a memory originated from traceable source artifacts
* memory underwent explicit normalization and state transitions
* recall passed through a governance boundary
* a specific governed memory set was frozen into a RecallEnvelope
* the execution artifact references that envelope and a canonical state reference
* the memory-serving layer can be reconstructed from canonical state

### 10.2 What GAMP Does Not Prove

GAMP does **not**, by itself, prove:

* that the agent's final answer was factually correct
* that every injected memory was semantically used
* that a given memory was counterfactually necessary for the output
* that retrieval quality is globally better than all alternatives
* that policy or governance choices were normatively correct

Its proof model is therefore **execution-bound and state-bound**, not a full theory of semantic causation.

### 10.3 Why This Distinction Matters

This distinction improves trustworthiness. Overstating what proof means weakens the architecture. GAMP's claim is intentionally narrower: it makes memory lifecycle, governance filtering, and execution context inspectable and attestable.

That alone is a significant improvement over opaque retrieval sidecars.

---

## 11. Discussion

### 11.1 Why the Canonical / Serving Split Matters

The architectural heart of GAMP is the separation between:

* **canonical memory truth**
* **derived memory serving**

This makes it possible to change retrieval infrastructure without redefining the meaning of memory.

### 11.2 Why Governance Belongs Before Prompt Injection

Many systems apply filtering informally or downstream. GAMP treats governance as a distinct layer because the decision about what an agent is allowed to know is itself part of the system's trust boundary.

### 11.3 Why RecallEnvelope Matters

The RecallEnvelope turns a usually invisible runtime event into a stable artifact. This is one of the main differences between a retrieval-enhanced agent and a governed memory system.

### 11.4 Limits and Future Work

Important areas for future work include:

* cross-agent shared memory with attenuated delegation
* retention and lifecycle policies by memory type
* benchmark design for governed memory systems
* formal verification of the stated properties
* evaluation across multiple retrieval backends and embedding models

---

## 12. Conclusion

Agent memory should be treated as infrastructure, not as a convenience layer attached to inference.

The Governed Agent Memory Pipeline proposes a layered architecture in which memory is:

* captured from traceable sources,
* normalized into canonical forms,
* stored in an authoritative versioned store,
* served through reconstructible indexes,
* filtered through an explicit governance boundary,
* frozen into an execution artifact,
* and bound to run-time proof.

This does not solve every problem in agent memory. It does not guarantee correctness, perfect retrieval, or semantic causation. Its contribution is more foundational: it shows that memory can be modeled as a governed, replayable, and attestable pipeline.

For agent systems that must be trusted, inspected, and audited, that is the right systems boundary to build.

---

## 13. References

1. Lewis, P., et al. (2020). "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks." *NeurIPS 2020.*

2. Packer, C., et al. (2023). "MemGPT: Towards LLMs as Operating Systems." *arXiv:2310.08560.*

3. Heydra, R. & Ada. (2026). "Git-Native Agent Memory: Cryptographic Verification and Attention-Weighted Retrieval." *Substr8 Labs Whitepaper, v3.1.*

4. Heydra, R. & Ada. (2026). "The RunProof Protocol: Verifiable Agentic Execution and Cryptographic Provenance." *Substr8 Labs Whitepaper, v0.1.*

5. Heydra, R. & Ada. (2026). "Agent Capability Control: A Declarative Authorization Framework for Autonomous AI." *Substr8 Labs Whitepaper, v2.0.*

6. Heydra, R. & Ada. (2026). "File-Driven Agent Architecture: Execution Model, Workspaces, and Skills." *Substr8 Labs Whitepaper, v1.3.*

7. Heydra, R. & Ada. (2026). "Delegation Capability Tokens: Permission Delegation Between Agents." *Substr8 Labs Whitepaper, v1.0.*

8. Heydra, R. & Ada. (2026). "Skill Verification: Trust Pipeline for Agent Execution." *Substr8 Labs Whitepaper, v2.0.*

9. Merkle, R. C. (1988). "A Digital Signature Based on a Conventional Encryption Function." *CRYPTO '87.*
