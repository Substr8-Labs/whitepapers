# The Runtime Integrity Layer: Deterministic Execution for Probabilistic Reasoning

**Authors:** Substr8 Labs (Rudi Heydra)  
**Date:** 2026-03-03  
**Version:** 1.0  

> A deterministic execution substrate on top of probabilistic reasoning.

---

## Abstract

Modern LLM systems operate as distributed, stateful runtimes — yet most implementations treat them as stateless chat APIs. This mismatch creates systemic fragility: orphaned tool calls corrupt sessions, context truncation silently breaks execution flow, crashes lose in-flight work, and memory capture depends on human discipline.

We introduce the **Runtime Integrity Layer (RIL)**, a middleware architecture that enforces structural correctness, execution continuity, and deterministic recovery across all agent interactions. RIL transforms LLM interaction from best-effort conversation into a governed execution system.

**Key contributions:**
1. Context Integrity Adapter (CIA) — structural validation before API interaction
2. GAM Trigger Architecture — automatic memory capture as a system primitive
3. Work Ledger — crash-resilient execution state tracking
4. Three-layer observability: behavioral (DCT), memory (GAM), work (Ledger)
5. Edge case analysis for parallel tools, mid-flight crash, truncation, and replay attacks

---

## 1. Introduction

### 1.1 The Problem

Modern LLM systems operate as distributed, stateful runtimes — yet most implementations treat them as stateless chat APIs. This mismatch creates systemic fragility:

- **Orphaned tool calls corrupt sessions** — tool_result references non-existent tool_use
- **Context truncation silently breaks execution flow** — compaction splits tool transactions
- **Crashes lose in-flight work** — no persistent execution state
- **Memory capture depends on human discipline** — if the agent forgets to log, context is lost
- **Replay or race conditions introduce non-determinism** — parallel tool calls interleave

When agents move from experimentation to production — especially in financial, operational, or regulated contexts — these fragilities become unacceptable.

### 1.2 The Insight

> LLM systems are not stateless chat APIs.  
> They are distributed transactional systems.

This reframing demands infrastructure that treats:
- **Tool calls as transactions** — atomic, logged, recoverable
- **Memory as automatic** — captured at execution time, not by human discipline
- **Execution state as persistent** — survives crashes, enables resume
- **Structural validity as guaranteed** — enforced before API interaction

### 1.3 The Solution: Runtime Integrity Layer

Substr8 introduces a **Runtime Integrity Layer (RIL)** within the FDAA Proxy. The RIL enforces structural correctness, execution continuity, and deterministic recovery across all agent interactions.

```
Request → CIA (validate/repair) → Triggers (capture) → Work Ledger (persist) → Forward
```

The Runtime Integrity Layer transforms LLM interaction from best-effort conversation into a **governed execution system**.

---

## 2. Architecture

### 2.1 Component Overview

| Component | Function | Output |
|-----------|----------|--------|
| Context Integrity Adapter (CIA) | Structural validation | Valid request or rejection |
| GAM Trigger Engine | Event-driven memory capture | Memories + commits |
| Work Ledger | Execution state persistence | Recovery context |

### 2.2 Architectural Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                     FDAA Proxy — Runtime Integrity Layer            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Request In                                                         │
│       ↓                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  1. CONTEXT INTEGRITY ADAPTER (CIA)                         │   │
│  │     • Validate tool_use ↔ tool_result pairing               │   │
│  │     • Prevent orphaned tool_result corruption               │   │
│  │     • Inject synthetic failures for unresolved tool_use     │   │
│  │     • Log all repairs to DCT audit trail                    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       ↓                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  2. GAM TRIGGER ENGINE                                      │   │
│  │     • message_received                                      │   │
│  │     • tool_invoked / tool_completed                         │   │
│  │     • turn_completed                                        │   │
│  │     • decision_point_detected                               │   │
│  │     • crash_recovery                                        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       ↓                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  3. WORK LEDGER                                             │   │
│  │     • Track in-flight execution state                       │   │
│  │     • Persist task context for recovery                     │   │
│  │     • Link to DCT entries and GAM commits                   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       ↓                                                             │
│  Forward to LLM API (guaranteed valid)                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.3 Three-Layer Observability

| Layer | Component | What It Captures |
|-------|-----------|------------------|
| **Behavioral** | DCT (Deterministic Capability Tokens) | Tool calls, API interactions, external state changes |
| **Memory** | GAM (Git-Native Agent Memory) | Decisions, artifacts, learned context |
| **Work** | Work Ledger | Task state, execution progress, recovery context |

Together: full trace from intent → execution → outcome.

---

## 3. Context Integrity Adapter (CIA)

### 3.1 Purpose

Ensures every LLM request satisfies protocol invariants. **No malformed request reaches the model.**

### 3.2 Invariants Enforced

**Invariant 1: Tool Result Reference Integrity**
```
For every tool_result.tool_use_id in message[i]:
  ASSERT: tool_use_id EXISTS IN message[i-1].tool_use[].id
  ASSERT: message[i-1].role == "assistant"
```

**Invariant 2: Tool Transaction Completeness**
```
If message[i-1] contains tool_use blocks:
  ASSERT: message[i] contains matching tool_result for each tool_use
  ASSERT: No further assistant messages until tool_results resolved
```

**Invariant 3: Safe Truncation Boundaries**
```
History truncation MUST NOT:
  - Remove tool_use while retaining its tool_result
  - Split tool transaction boundaries
  - Leave dangling references
```

### 3.3 Repair Modes

| Mode | Behavior | Use Case |
|------|----------|----------|
| `STRICT` | Reject request if invariant fails | Regulated environments, audits |
| `PERMISSIVE` | Drop orphaned tool_result; inject synthetic failure for dangling tool_use | Default for development/production |
| `FORENSIC` | Halt execution; snapshot payload; log structural breach | Incident investigation |

### 3.4 Forensic Logging

Every repair operation produces an audit record:

```json
{
  "event": "context_integrity_repair",
  "timestamp": "2026-03-03T11:45:00Z",
  "session_id": "abc123",
  
  "original_payload_hash": "sha256:a1b2c3...",
  "repaired_payload_hash": "sha256:d4e5f6...",
  
  "repair_type": "PRUNE_ORPHAN_RESULT",
  "repairs": [
    {
      "action": "prune_tool_result",
      "tool_use_id": "toolu_015Wyv88HEqn3VwNzHCATJTN",
      "message_index": 96,
      "reason": "No matching tool_use in previous message"
    }
  ],
  
  "mode": "PERMISSIVE",
  "forwarded": true
}
```

### 3.5 Result

> "Our system guarantees structural validity before API interaction."

CIA transforms reactive error handling into proactive governance.

---

## 4. GAM Trigger Architecture

### 4.1 Purpose

Makes memory capture **systemic rather than optional**. Memory becomes an automatic property of execution.

### 4.2 Trigger Events

| Trigger | When | What to Capture |
|---------|------|-----------------|
| `message_received` | User message arrives | Intent, entities, urgency |
| `tool_invoked` | Tool call initiated | Tool name, params, intent |
| `tool_completed` | Tool returns result | Result summary, success/fail |
| `turn_completed` | Agent finishes responding | Turn summary, state delta |
| `decision_point_detected` | High-value moment identified | Decision, reasoning, context |
| `crash_recovery` | Session restored after failure | Last known state, incomplete work |

### 4.3 Decision Point Detection

High-value commit moments deserve first-class detection:

| Decision Point | Signal |
|----------------|--------|
| External state change | Tool in {exec, write, message, deploy} completed |
| Plan mutation | Agent expresses changed approach |
| Task fork | Sub-agent spawned |
| User intent pivot | Topic/goal change detected |
| Significant choice | Decision + reasoning expressed |

### 4.4 Significance Classification

Not all events warrant persistence. The significance classifier:

- **Filters noise** — acknowledgments, status checks, routine searches
- **Prioritizes decisions** — choices with reasoning
- **Flags artifacts** — files created, commits made
- **Boosts explicit requests** — user says "remember this"

Only high-significance events auto-commit to GAM.

### 4.5 Result

> Memory becomes an automatic property of execution, not a discipline requirement.

---

## 5. Work Ledger

### 5.1 Purpose

Tracks in-flight execution state. **Agents survive crashes without losing intent.**

### 5.2 Schema

```sql
CREATE TABLE work_ledger (
    task_id UUID PRIMARY KEY,
    session_id TEXT NOT NULL,
    
    -- What
    intent TEXT NOT NULL,
    plan JSONB,
    current_step TEXT,
    
    -- State
    status TEXT NOT NULL,  -- pending | in_progress | blocked | complete | failed
    
    -- Recovery
    context_snapshot_hash TEXT,  -- SHA256 of serialized context
    
    -- Linkage
    linked_dct_entries TEXT[],
    linked_gam_commits TEXT[],
    
    -- Timestamps
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### 5.3 Recovery Algorithm

```
On session start:
  1. Query Work Ledger: SELECT * WHERE status NOT IN ('complete', 'failed')
  2. For each active task:
     a. Validate structural integrity
     b. Inject summarized state into system context:
        "RECOVERY: You were working on {intent}. Last step: {current_step}."
     c. Resume execution OR prompt user for confirmation
```

### 5.4 Result

> Execution state persists across crashes. Work is never silently lost.

---

## 6. Security Analysis: Edge Cases

### 6.1 Case 1: Parallel Tool Calls

**Risk:** `tool_use_id` A and B interleave; `tool_result` B arrives before A; history misordered.

**CIA Defense:**
- Enforce single-flight tool execution per session, OR
- Maintain strict ordering buffer
- Reject if `tool_result` references id not in immediate previous message

**Work Ledger Defense:**
- Track tool state per task
- Do not advance plan until all `tool_result` resolved

**Verdict:** Safe if serialized or buffered.

### 6.2 Case 2: Mid-Flight Crash

**Risk:** `tool_use` emitted; `tool_result` never recorded; session restored.

**CIA Defense:**
- On next message, detect dangling `tool_use`
- Inject synthetic failure OR flag for re-execution

**Work Ledger Defense:**
- `task.status = in_progress`
- `crash_recovery` trigger fires
- Resume step or prompt user

**Verdict:** Recoverable.

### 6.3 Case 3: History Truncation

**Risk:** Compaction removes `tool_use` but keeps `tool_result`; API rejection.

**CIA Defense:**
- Detect orphan `tool_result`
- Drop or fail-fast based on mode

**Prevention:**
- Safe truncation boundaries only
- Drop entire tool transaction if needed

**Verdict:** Structural protection present.

### 6.4 Case 4: Replay Attack

**Risk:** Malicious actor replays `tool_result` with valid id, but no matching `tool_use` in session.

**CIA Defense:**
- Reject due to invariant violation

**Optional Hardening:**
- Hash `tool_use` input + store in ledger
- Validate `tool_result` signature against input hash

**Verdict:** Detectable; hardening available.

### 6.5 Case 5: Crash During Memory Commit

**Risk:** GAM commit incomplete; ledger updated but Git commit missing.

**Mitigation:**
- Use atomic commit pattern
- Store pending memory event in ledger until Git confirms
- On recovery, check for uncommitted memories

**Verdict:** Requires atomic pattern implementation.

---

## 7. Discussion

### 7.1 From Tooling to Systems Engineering

The Runtime Integrity Layer represents a transition from "clever tooling" to "systems engineering." The key insight:

> A deterministic execution substrate on top of probabilistic reasoning.

LLMs provide probabilistic reasoning. RIL provides deterministic execution guarantees around that reasoning.

### 7.2 Positioning: Agent Runtime Governance

RIL enables a new category: **Agent Runtime Governance**.

| Capability | Traditional | With RIL |
|------------|-------------|----------|
| Structural validity | Hope API accepts | Guaranteed before send |
| Memory capture | Manual discipline | Automatic system primitive |
| Crash recovery | Lost context | Deterministic resume |
| Audit trail | Optional logging | Three-layer observability |
| Failure analysis | Debug from scratch | Forensic reproducibility |

### 7.3 Relationship to FDAA

RIL is the runtime enforcement layer for FDAA (Functional Determinism for Autonomous Agents):

- **FDAA** defines what an agent *should* do (capabilities, constraints, identity)
- **RIL** ensures what an agent *actually* does is valid, captured, and recoverable

Together: verifiable agent definition + governed agent execution.

---

## 8. Conclusion

The Runtime Integrity Layer transforms LLM-based agents from fragile conversational interfaces into governed execution systems. By enforcing structural validity (CIA), automating memory capture (GAM Triggers), and persisting execution state (Work Ledger), RIL provides the infrastructure necessary for production-grade autonomous agents.

This is not defensive infrastructure. This is the beginning of a **verifiable runtime substrate** for AI agents.

---

## References

1. Substr8 Labs. "FDAA: Functional Determinism for Autonomous Agents." 2026.
2. Substr8 Labs. "Git-Native Agent Memory (GAM)." 2026.
3. Substr8 Labs. "Deterministic Capability Tokens (DCT)." 2026.
4. Anthropic. "Tool Use Documentation." 2024.

---

## Appendix A: Implementation Status

| Component | Status | Location |
|-----------|--------|----------|
| CIA | Implemented | `fdaa-proxy/src/adapters/context_integrity.py` |
| GAM Triggers | Specified | `docs/architecture/GAM-TRIGGERS.md` |
| Work Ledger | Specified | `docs/architecture/RUNTIME-INTEGRITY-LAYER.md` |
| DCT Integration | Stub | Callback interface ready |

---

## Appendix B: Origin

**2026-03-03:** A tool pairing corruption bricked a production session for 6+ hours. The user received raw API errors with no automatic recovery path.

Response: Built CIA, GAM Triggers, and Work Ledger in one session.

Mentor validation: *"That's not a patch. That's an operating model."*

Lesson: Failures expose fragility. Guardrails become features. This is how platforms evolve.
