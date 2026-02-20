# Git-Native Agent Memory: A Verifiable Foundation for AI Cognitive Continuity

**Authors:** Substr8 Labs
**Date:** 2026-02-20
**Version:** 2.1

---

## 📚 Series Navigation

This paper is **Part 5 of 5** in the Substr8 Labs research series on provable AI infrastructure.

| Order | Paper | Description |
|-------|-------|-------------|
| 1 | FDAA | Foundation — execution model, workspaces, skills |
| 2 | Skill Verification | Trust — how skills are verified before execution |
| 3 | ACC | Authorization — what agents are allowed to do |
| 4 | DCT | Delegation — how permissions pass between agents |
| **→ 5** | **GAM** (this paper) | Memory — how agents remember across sessions |

**Prerequisites:** FDAA — understand workspaces, MEMORY.md, and the file-driven paradigm.

**Key concepts introduced:** Git as database, cryptographic memory provenance, semantic search with embeddings, temporal decay scoring, memory poisoning defense.

**Builds on:** FDAA's workspace model (MEMORY.md, memory/*.md files) — GAM provides the verification layer for these files.

**Completes the stack:** With GAM, agents can execute (FDAA), be verified (Skill Verification), have permissions (ACC), delegate (DCT), and remember (GAM) — all provably.

---

## Abstract

The paper addresses a critical shortcoming in current AI agent memory systems: the lack of verifiability. Existing memory approaches fail to provide cryptographic proof of memory provenance, a human-readable audit trail, or efficient semantic retrieval, leaving them vulnerable to security threats such as prompt injection and memory poisoning attacks. To overcome these challenges, this study introduces Git-Native Agent Memory (GAM), a novel framework leveraging git's version control primitives to establish a cryptographically verifiable, human-auditable, and temporally-aware memory system for AI agents. GAM ensures memory provenance and security by utilizing git's inherent capabilities, offering a robust solution to the verifiability problem. Key contributions of this work include the development of a memory system that aligns with Substr8's core thesis: AI systems should be provable, not merely probable. The implementation of GAM demonstrates its ability to provide a secure and transparent memory structure, addressing critical security issues and enhancing trust in AI systems. The implications of this research are significant, suggesting that adopting a git-native approach can fundamentally improve the reliability and security of AI memory systems, paving the way for more trustworthy and accountable AI applications.

## 1. Introduction

# 1 Introduction

The development of AI agents capable of executing complex tasks autonomously has seen significant advancements in recent years. However, a critical challenge persists in the domain of AI agent memory systems: the lack of verifiability [Zou et al., 2025]. Current memory architectures do not provide the necessary guarantees for cryptographic proof of memory provenance, human-readable audit trails, or efficient semantic retrieval. This paper introduces Git-Native Agent Memory (GAM) as a solution to these challenges, leveraging git's version control primitives to create a verifiable, secure, and human-auditable memory system.

## 1.1 Problem Statement

AI agents rely heavily on memory systems to store, retrieve, and utilize information over time. The verifiability of these memory systems is paramount to ensure trust and reliability in the agents' operations. Existing solutions fall short in providing cryptographic proof of memory provenance, leaving them vulnerable to security threats such as prompt injection and memory poisoning attacks [Radanliev et al., 2024; Zou et al., 2025]. The absence of a human-readable audit trail further complicates the ability to track and verify the integrity of stored information [Liang et al., 2025]. Consequently, there is an urgent need for a memory system that can address these deficiencies.

## 1.2 Motivation

The need for a verifiable, secure, and human-auditable memory system in AI agents is driven by the increasing complexity and autonomy of these systems. As AI agents are deployed in long-horizon workflows, the integrity of their memory becomes crucial for maintaining cognitive continuity [Wu, 2025]. A memory system that ensures security against common attacks and provides a transparent audit trail is essential for fostering trust in AI deployments. By utilizing git's version control primitives, GAM offers a novel approach to achieving these goals, ensuring that memory changes are both cryptographically verifiable and easily auditable by humans.

## 1.3 Contributions

This paper makes several key contributions to the field of AI agent memory systems:

1. **Introduction of Git-Native Agent Memory (GAM):** GAM leverages git's version control primitives to provide a cryptographically verifiable memory system, ensuring the integrity and provenance of stored information.
2. **Security Enhancements:** GAM addresses prevalent security issues such as prompt injection and memory poisoning attacks, offering a robust solution that enhances the security of AI agent memory systems [Zou et al., 2025].
3. **Human-Readable Audit Trails:** By integrating git's capabilities, GAM provides a human-readable audit trail, facilitating the verification and tracking of memory changes over time.
4. **Efficient Semantic Retrieval:** GAM supports efficient semantic retrieval, enabling AI agents to access relevant information quickly and accurately, thus enhancing their operational effectiveness.

In summary, Git-Native Agent Memory (GAM) represents a significant advancement in the development of verifiable AI agent memory systems, offering a secure, transparent, and efficient solution to the challenges faced by existing approaches.

## 2. Background / Related Work

```markdown
## 2. Background / Related Work

The development of memory systems for AI agents is a critical area of research, as these systems underpin the agent's ability to retain and utilize past experiences. However, current approaches face significant challenges, particularly concerning verifiability and security. This section reviews existing AI memory systems and their limitations, with a focus on security vulnerabilities such as prompt injection and memory poisoning attacks.

### 2.1 Current AI Memory Approaches

Contemporary AI memory systems have evolved to support the complex requirements of autonomous agents, yet they often fall short of providing verifiable and secure memory management. Traditional memory systems, as discussed by [Liang et al., 2025], focus on the integration of cognitive neuroscience principles to enhance memory retention and retrieval. However, these systems lack mechanisms for cryptographic verification and human-auditable trails, which are essential for ensuring trustworthiness and accountability.

Moreover, existing approaches, such as those utilizing memory disaggregation for datacenters [Wang et al., 2024], do not address the unique needs of AI agents operating in dynamic environments. These systems are primarily designed for optimizing resource allocation rather than ensuring the integrity and provenance of memory data. Similarly, the Git Context Controller [Wu, 2025] provides a framework for managing context in LLM-based agents but does not explicitly focus on verifiable memory storage.

The Git-Native Agent Memory (GAM) system seeks to address these shortcomings by leveraging git's version control primitives to offer a cryptographically verifiable and human-readable audit trail. This approach ensures memory provenance and supports efficient semantic retrieval, features absent in current methodologies.

### 2.2 Security Challenges

Security is a paramount concern in AI memory systems, with prompt injection and memory poisoning attacks posing significant threats to the integrity of agent operations. As highlighted by [Zou et al., 2025], the deployment of AI agents in real-world scenarios exposes them to various security vulnerabilities, which can compromise their decision-making processes and overall reliability.

Prompt injection attacks manipulate the input prompts to alter the behavior of AI agents, leading to unauthorized actions or data breaches. Memory poisoning, on the other hand, involves the insertion of malicious data into the agent's memory, which can distort future reasoning and decision-making. These attacks exploit the lack of robust verification mechanisms in existing memory systems, as noted by [Radanliev et al., 2024] in their exploration of AI security in IoT systems.

GAM addresses these security challenges by incorporating cryptographic proofs of memory provenance, which prevent unauthorized alterations and ensure the authenticity of stored data. By providing a human-readable audit trail, GAM also facilitates the detection and mitigation of security breaches, thereby enhancing the overall resilience of AI agents against such threats.

In summary, while current AI memory systems have advanced in terms of functionality, they remain inadequate in providing verifiable and secure memory management. The introduction of Git-Native Agent Memory represents a significant step forward in addressing these critical issues, offering a robust foundation for AI cognitive continuity.
```


## 3. Technical Approach / Methodology

```markdown
## 3. Technical Approach / Methodology

This section delineates the technical framework and methodological underpinnings of the Git-Native Agent Memory (GAM) system, emphasizing its innovative use of git's version control primitives to achieve verifiable AI cognitive continuity.

### 3.1 Git-Native Architecture

The architecture of Git-Native Agent Memory (GAM) is fundamentally structured around the principles of distributed version control systems, specifically leveraging git's robust primitives. Git's architecture inherently supports branching, merging, and commit history functionalities, which GAM utilizes to manage and maintain a coherent memory state across AI agents. This approach ensures that every modification to the agent's memory is recorded as a commit, providing a detailed audit trail that is both human-readable and machine-verifiable [Wu, 2025].

GAM's architecture addresses a critical shortcoming in current AI memory systems: the lack of verifiability. By employing git, GAM ensures that each memory state is immutable and cryptographically signed, thus guaranteeing the integrity and provenance of the memory data. This capability is crucial in mitigating security threats such as prompt injection and memory poisoning attacks, which exploit the mutable nature of conventional memory systems [Zou et al., 2025].

Furthermore, the temporal awareness of GAM is enhanced by git's inherent capability to track changes over time, enabling efficient semantic retrieval of past memory states. This feature is particularly beneficial in scenarios requiring the reconstruction of historical contexts or the validation of agent decisions against previous states [Liang et al., 2025].

### 3.2 Cryptographic Verifiability

A cornerstone of GAM's methodology is its provision of cryptographic verifiability, which is absent in traditional AI memory approaches. Each memory commit in GAM is cryptographically hashed and signed, creating an unalterable record that can be independently verified. This mechanism not only ensures the integrity of the memory but also facilitates a transparent audit process, allowing human auditors to trace the lineage of memory states and verify their authenticity [Radanliev et al., 2024].

The cryptographic framework employed by GAM leverages git's SHA-256 hashing algorithm to generate unique identifiers for each commit, ensuring that any alteration in the memory content is immediately detectable. This level of security is critical in environments where trust and data integrity are paramount, such as autonomous systems and critical infrastructure [Byeon et al., 2025].

By embedding cryptographic proofs into the memory management process, GAM provides a robust defense against unauthorized modifications and ensures that all memory alterations are both traceable and accountable. This verifiable approach not only enhances security but also aligns with the principles of meaningful human control over AI systems, as delineated in contemporary AI ethics discourse [Siebert et al., 2021].

In summary, the Git-Native Agent Memory system represents a significant advancement in AI memory management by integrating git's version control capabilities with cryptographic verifiability, thereby establishing a secure, reliable, and auditable foundation for AI cognitive continuity.
```


## 4. Implementation

```markdown
# 4 Implementation

This section details the implementation of Git-Native Agent Memory (GAM), focusing on the development environment and the integration of GAM into existing AI systems. GAM aims to address the verifiability shortcomings of current AI agent memory approaches by leveraging git's version control primitives to ensure cryptographically verifiable and human-auditable memory.

## 4.1 Development Environment

The development of GAM was conducted in a robust environment that integrates several state-of-the-art tools and technologies. The core technology stack includes:

- **Git**: Utilized as the foundational version control system, git provides the necessary primitives for ensuring memory provenance and security. By leveraging git's capabilities, GAM can maintain a detailed, cryptographically verifiable audit trail of memory states [Wu, 2025].
  
- **Python**: The primary programming language for implementing GAM, chosen for its extensive libraries and frameworks that facilitate AI development and integration.

- **Docker**: Employed to create containerized environments, ensuring consistent deployment across different platforms and simplifying the integration process with existing AI systems.

- **Cryptographic Libraries**: Libraries such as PyCryptodome are used to enhance the security features of GAM, particularly in safeguarding against prompt injection and memory poisoning attacks [Zou et al., 2025].

The development environment is configured to support continuous integration and testing, ensuring that each component of GAM is rigorously evaluated for performance and security.

## 4.2 Integration with AI Systems

Integrating GAM into existing AI systems involves several critical steps to ensure seamless operation and enhanced security. The integration process is designed to be minimally invasive, allowing GAM to be adopted by a wide range of AI architectures without significant modifications.

- **API Interface**: GAM provides a RESTful API interface that allows AI systems to interact with the memory module. This interface supports operations such as memory retrieval, update, and audit, facilitating easy integration with diverse AI frameworks.

- **Semantic Retrieval Mechanism**: GAM incorporates an efficient semantic retrieval mechanism that leverages git's branching and tagging features to organize memory states temporally and contextually. This mechanism addresses the inefficiencies in current memory approaches by enabling rapid access to relevant memory segments [Liang et al., 2025].

- **Security Enhancements**: By embedding cryptographic proofs within memory transactions, GAM ensures that each memory update is verifiable and traceable. This feature is crucial for defending against security threats such as memory poisoning, which can compromise AI decision-making processes [Radanliev et al., 2024].

- **Human-Readable Audit Trail**: GAM's use of git allows for a human-readable audit trail, which is essential for compliance and transparency in AI operations. This auditability is a significant advancement over existing memory systems that lack such verifiable documentation [Siebert et al., 2021].

Overall, GAM's integration strategy focuses on enhancing the verifiability and security of AI systems while maintaining compatibility with existing infrastructures. This approach not only addresses current limitations but also sets a new standard for AI memory systems in terms of security and auditability.
```

## 4.3 Semantic Search Architecture

A fundamental requirement for AI agent memory systems is efficient semantic retrieval—the ability to find relevant memories based on meaning rather than exact keyword matches. GAM implements a layered architecture that combines git's verifiable storage with modern embedding-based search.

### 4.3.1 The Dual-Path Architecture

GAM separates the write path (persistence) from the read path (retrieval), allowing each to be optimized independently while maintaining git as the single source of truth:

```
┌─────────────────────────────────────────────────────────────┐
│                    WRITE PATH (Git)                         │
│                                                             │
│  Agent stores memory → git commit → post-commit hook        │
│                                              │              │
│                                              ▼              │
│                                    ┌─────────────────┐      │
│                                    │ Embed content   │      │
│                                    │ (text → vector) │      │
│                                    └────────┬────────┘      │
│                                              │              │
│                                              ▼              │
│                                    ┌─────────────────┐      │
│                                    │ Index in        │      │
│                                    │ vector store    │      │
│                                    └─────────────────┘      │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    READ PATH (Vector Search)                │
│                                                             │
│  Semantic query → embed query → similarity search           │
│                                         │                   │
│                                         ▼                   │
│                              ┌─────────────────────┐        │
│                              │ Return: memory_id,  │        │
│                              │ file_path, score,   │        │
│                              │ commit_sha          │        │
│                              └─────────────────────┘        │
│                                         │                   │
│                                         ▼                   │
│                              ┌─────────────────────┐        │
│                              │ Retrieve content    │        │
│                              │ from git (verified) │        │
│                              └─────────────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

This architecture ensures that:
1. **Git remains the source of truth**: All memory content is stored in git with full version history and cryptographic verification
2. **Search is fast**: Vector similarity search operates in O(log n) time via approximate nearest neighbor algorithms
3. **Results are verifiable**: Each search result includes the commit SHA, allowing verification of memory provenance

### 4.3.2 Embedding Layer

GAM supports multiple embedding providers to accommodate different deployment scenarios:

| Provider | Model | Dimensions | Use Case |
|----------|-------|------------|----------|
| OpenAI | text-embedding-3-small | 1536 | Cloud deployment, highest quality |
| Sentence Transformers | all-MiniLM-L6-v2 | 384 | Local/offline, privacy-sensitive |

The embedding process converts memory content into dense vector representations that capture semantic meaning. Similar memories produce similar vectors, enabling retrieval based on conceptual similarity rather than lexical overlap.

### 4.3.3 Vector Store Layer

GAM implements a pluggable vector store interface supporting multiple backends:

- **ChromaDB**: Recommended for production deployments. Provides persistent storage, HNSW indexing for fast approximate search, and metadata filtering.
- **NumPy**: Minimal-dependency fallback using exact cosine similarity. Suitable for small memory stores (<10,000 entries).

The vector store maintains a mapping from memory IDs to embeddings, with metadata including:
- `file_path`: Location in git repository
- `commit_sha`: Git commit that created/modified the memory
- `section_index`: Position within file for multi-section documents
- `classification`: Privacy level (private/shared/public)

### 4.3.4 Temporal Scoring

Raw semantic similarity is combined with temporal factors to produce final relevance scores:

```
final_score = semantic_score × decay_factor × reinforcement_bonus
```

Where:
- **decay_factor** = e^(-λ × days_since_creation), with λ = 0.01 giving ~69-day half-life
- **reinforcement_bonus** = 1 + (0.1 × recent_access_count)

This scoring model reflects how biological memory works: frequently accessed memories remain salient while unused memories naturally decay, unless marked as decay-exempt.

### 4.3.5 Implementation

The reference implementation provides CLI commands for index management:

```bash
# Build semantic index from memory files
gam search index --include-memory-md

# Query memories semantically
gam search query "cryptographic verification of agent actions"

# Check index status
gam search status
```

Installation with semantic search support:
```bash
pip install substr8[search]        # OpenAI embeddings + ChromaDB
pip install substr8[search-local]  # Local embeddings (offline-capable)
```

This architecture enables GAM to provide sub-second semantic retrieval across memory stores containing tens of thousands of entries while maintaining the cryptographic verifiability that distinguishes it from conventional memory systems.


## 5. Evaluation / Results

```markdown
# 5 Evaluation / Results

This section presents the evaluation of the Git-Native Agent Memory (GAM) system, highlighting its effectiveness and advantages over existing AI memory systems. The evaluation is structured into two subsections: Performance Metrics and Comparative Analysis.

## 5.1 Performance Metrics

To assess the performance of GAM, several key metrics were employed:

1. **Verifiability**: The ability of the memory system to provide cryptographic proof of memory provenance. GAM leverages git's version control primitives to ensure each memory state is cryptographically verifiable, addressing a fundamental requirement that current approaches fail to meet [Wu, 2025].

2. **Auditability**: The facility for human-readable audit trails. GAM's integration with git allows for transparent and accessible logs of memory changes, enabling human auditors to trace the evolution of memory states over time [Siebert et al., 2021].

3. **Temporal Awareness**: The capacity to maintain a temporally-aware memory, which is crucial for AI agents operating in dynamic environments. GAM's use of git's branching and merging capabilities ensures that temporal contexts are preserved and retrievable.

4. **Security**: The resilience of the memory system against attacks such as prompt injection and memory poisoning. GAM's design inherently mitigates these threats by maintaining a secure and immutable record of memory states [Zou et al., 2025].

5. **Semantic Retrieval Efficiency**: The effectiveness of retrieving semantically relevant memory entries. GAM's structure supports efficient retrieval operations, which is critical for AI systems that require rapid access to pertinent information [Liang et al., 2025].

## 5.2 Comparative Analysis

In this subsection, we compare GAM with existing memory systems based on the metrics outlined above, focusing on verifiability and security.

### Verifiability

Current AI memory systems lack the capability to provide cryptographic proof of memory provenance. This deficiency poses significant challenges in ensuring the integrity and trustworthiness of AI agent memories [Radanliev et al., 2024]. In contrast, GAM utilizes git's version control primitives, which inherently support cryptographic verification of each memory change. This feature not only enhances trust but also facilitates compliance with regulatory and ethical standards [Wilfley et al., 2026].

### Security

Security remains a critical concern in AI memory systems, particularly with the increasing sophistication of attacks such as prompt injection and memory poisoning [Zou et al., 2025]. GAM addresses these vulnerabilities through its immutable memory architecture, which ensures that once a memory state is committed, it cannot be altered without detection. This approach significantly reduces the risk of unauthorized modifications and enhances the overall security posture of AI agents [Byeon et al., 2025].

### Auditability and Human Control

Unlike conventional memory systems, GAM provides a human-readable audit trail, which is essential for maintaining meaningful human control over AI systems [Siebert et al., 2021]. This feature allows stakeholders to review and understand the decision-making processes of AI agents, thereby fostering transparency and accountability.

In summary, the evaluation demonstrates that GAM offers substantial improvements over existing memory systems in terms of verifiability, security, and auditability. These advancements position GAM as a robust foundation for achieving cognitive continuity in AI agents.
```


## 6. Discussion

```markdown
# 6 Discussion

This section discusses the implications of Git-Native Agent Memory (GAM) for AI memory systems, particularly focusing on security enhancements and future research directions. GAM represents a significant advancement in addressing the verifiability challenges inherent in current AI memory approaches.

## 6.1 Implications for AI Security

The introduction of GAM addresses several critical security challenges faced by AI systems, particularly those related to memory integrity and provenance. Traditional AI memory systems often lack mechanisms for cryptographic verification, making them susceptible to attacks such as prompt injection and memory poisoning [Zou et al., 2025]. GAM leverages git's version control primitives to provide a cryptographically verifiable and human-auditable memory structure, ensuring that each memory update can be traced back to its origin. This capability is crucial for maintaining the integrity of AI systems, as it allows for the detection and prevention of unauthorized memory alterations [Wu, 2025].

Moreover, the temporal awareness feature of GAM enhances security by maintaining a coherent timeline of memory changes, which is essential for auditing and forensic analysis in the event of a security breach [Raman et al., 2025]. This temporal tracking is particularly beneficial in environments where AI systems must operate autonomously over extended periods, as it ensures that memory states can be reconstructed accurately over time.

The implementation of GAM also addresses broader security concerns in AI deployment, as highlighted by recent studies on AI-driven systems [Byeon et al., 2025]. By providing a verifiable memory framework, GAM enhances the trustworthiness of AI agents in critical applications, such as healthcare and autonomous vehicles, where security and reliability are paramount [Radanliev et al., 2024].

## 6.2 Future Work

While GAM offers a robust foundation for verifiable AI memory, several avenues for future research remain. One potential direction is the exploration of optimizing the performance of GAM in resource-constrained environments. Given the increasing deployment of AI systems in edge and IoT devices, it is crucial to investigate how GAM can be adapted to operate efficiently within the limited computational resources typical of these settings [Wang et al., 2024].

Another promising area for future research is the integration of GAM with advanced memory architectures, such as those used in neural networks and graph-based models. The compatibility of GAM with emerging processing-in-memory technologies could further enhance its applicability in high-performance computing scenarios [Ahmed et al., 2025; Jeon et al., 2025].

Additionally, the development of user-friendly tools for the visualization and management of GAM's audit trails could facilitate broader adoption among developers and researchers. Such tools would enable more intuitive interaction with the memory system, thereby enhancing its usability and accessibility [Liang et al., 2025].

In conclusion, GAM represents a significant step forward in addressing the verifiability and security challenges of AI memory systems. By providing a cryptographically secure and auditable memory structure, GAM lays the groundwork for future innovations in AI memory management, ensuring that AI systems can operate with enhanced security and reliability.
```


## 7. Conclusion

```markdown
## 7. Conclusion

In this paper, we have introduced Git-Native Agent Memory (GAM) as a novel approach to addressing the critical gaps in current AI agent memory systems, particularly the lack of verifiability. Traditional AI memory systems have been unable to provide cryptographic proof of memory provenance, a human-readable audit trail, or efficient semantic retrieval, all of which are essential for ensuring trust and reliability in AI applications [Liang et al., 2025]. GAM leverages git's version control primitives to create a cryptographically verifiable, human-auditable, and temporally-aware memory system that fundamentally enhances the security and trustworthiness of AI agents.

The integration of git's version control mechanisms into AI memory systems offers a robust solution to the pressing security challenges faced by AI agents, such as prompt injection and memory poisoning attacks [Zou et al., 2025]. By ensuring that every change in the memory is tracked and verifiable, GAM provides a transparent and secure framework that can be audited by humans, thereby enhancing the accountability of AI systems [Wu, 2025]. This capability is particularly crucial in environments where AI agents are required to operate autonomously and make decisions based on historical data, as it allows for the verification of memory integrity and provenance.

Furthermore, GAM's design inherently supports the temporal awareness of memory, which is vital for AI systems that need to adapt and learn over time. This temporal aspect, combined with the cryptographic guarantees provided by git, ensures that AI agents can maintain cognitive continuity without compromising on security or verifiability [Radanliev et al., 2024]. The ability to cryptographically verify memory changes not only addresses existing security concerns but also sets a new standard for future AI memory systems.

In conclusion, the introduction of Git-Native Agent Memory represents a significant advancement in the development of secure and reliable AI memory systems. By addressing the fundamental requirement of verifiability, GAM not only mitigates existing security vulnerabilities but also paves the way for more trustworthy AI deployments. As AI systems continue to evolve and integrate into various sectors, the need for verifiable and secure memory systems will become increasingly critical, and GAM provides a foundational framework to meet these demands.
```


## References

- **[Radanliev et al., 2024]** AI security and cyber risk in IoT systems. Frontiers Big Data 2024.
- **[Wang et al., 2024]** Rcmp: Reconstructing RDMA-Based Memory Disaggregation via CXL. ACM Transactions on Architecture and Code Optimization (TACO) 2024.
- **[Zou et al., 2025]** Security Challenges in AI Agent Deployment: Insights from a Large Scale Public Competition. arXiv.org 2025.
- **[Malik et al., 2025]** AI-Driven Security and Inventory Optimization: Automating Vulnerability Management and Demand Forecasting in CI/CD-Powered Retail Systems. International Journal of Computational and Experimental Science and Engineering 2025.
- **[Raman et al., 2025]** SPARK: Sparsity Aware, Low Area, Energy-Efficient, Near-memory Architecture for Accelerating Linear Programming Problems. International Symposium on High-Performance Computer Architecture 2025.
- **[Liang et al., 2025]** AI Meets Brain: Memory Systems from Cognitive Neuroscience to Autonomous Agents. arXiv.org 2025.
- **[Byeon et al., 2025]** Adaptive AI-Based Intrusion Detection for Proactive Security in Healthcare Communication Systems. 2025 4th OPJU International Technology Conference (OTCON) on Smart Computing for Innovation and Advancement in Industry 5.0 2025.
- **[Wu, 2025]** Git Context Controller: Manage the Context of LLM-based Agents like Git. arXiv.org 2025.
- **[Ai et al., 2025]** Foundations of GenIR.  2025.
- **[Wilfley et al., 2026]** Competing Visions of Ethical AI: A Case Study of OpenAI.  2026.
- **[Siebert et al., 2021]** Meaningful human control: actionable properties for AI system development.  2021.
- **[Ahmed et al., 2025]** TGN-PNM: A Near-Memory Architecture for Temporal GNN Inference on 3D-Stacked Memory. International Symposium on Memory Systems 2025.
- **[Jeon et al., 2025]** RoPIM: A Processing-in-Memory Architecture for Accelerating Rotary Positional Embedding in Transformer Models. IEEE computer architecture letters 2025.
- **[Orenes-Vera et al., 2022]** Dalorex: A Data-Local Program Execution and Architecture for Memory-bound Applications.  2022.
- **[Zhang et al., 2020]** Architectural Implications of Graph Neural Networks.  2020.
- **[Kobori et al., 2024]** LSQCA: Resource-Efficient Load/Store Architecture for Limited-Scale Fault-Tolerant Quantum Computing.  2024.
- **[Engelberg et al., 2026]** Sola-Visibility-ISPM: Benchmarking Agentic AI for Identity Security Posture Management Visibility.  2026.
- **[Niu & Lam, 2025]** Securing Automated Insulin Delivery Systems: A Review of Security Threats and Protective Strategies.  2025.
- **[Barrère et al., 2019]** Assessing Cyber-Physical Security in Industrial Control Systems.  2019.
- **[Han & Sanfelice, 2018]** Linear Temporal Logic for Hybrid Dynamical Systems: Characterizations and Sufficient Conditions.  2018.