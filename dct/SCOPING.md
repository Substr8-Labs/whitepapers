# DCT Whitepaper Scoping Document

**Working Title:** Delegation Capability Tokens: Cryptographic Permission Delegation for Autonomous AI Agents

**Target:** Academic whitepaper, Zenodo publication, ~15-20 pages

**Authors:** Ada, Rudi Heydra, Substr8 Labs

---

## 1. Problem Statement

AI agents increasingly operate with broad permissions: file access, API calls, code execution, network requests. When Agent A spawns Agent B for a subtask, there's no standard mechanism to:

1. **Limit** what permissions B receives
2. **Prove** what permissions were delegated
3. **Audit** the chain of delegation
4. **Enforce** that B cannot escalate beyond A's permissions

Current approaches:
- Implicit trust (dangerous)
- Hardcoded permission lists (inflexible)
- OAuth-style scopes (not designed for agent-to-agent)

**The gap:** No cryptographic primitive exists for capability delegation between autonomous AI agents.

---

## 2. Prior Art (Real, Verifiable)

### Macaroons (Google, 2014)
- Birgisson, Politz, Erlingsson, Taly, Vrable, Lentczner
- "Macaroons: Cookies with Contextual Caveats for Decentralized Authorization"
- IEEE S&P 2014
- Key insight: Bearer credentials with embedded caveats, attenuation via HMAC chaining

### Biscuit (Clever Cloud)
- Geoffroy Couprie et al.
- https://www.biscuitsec.org/
- Bearer tokens with offline attenuation
- Datalog-based authorization policies
- Ed25519 signatures

### SPIFFE/SPIRE
- CNCF project for workload identity
- X.509 SVIDs for service-to-service auth
- Not designed for capability delegation

### OAuth 2.0 Scopes
- RFC 6749
- Coarse-grained permission categories
- No cryptographic attenuation
- Not designed for autonomous agents

### Capability-Based Security
- Dennis & Van Horn (1966)
- "Programming Semantics for Multiprogrammed Computations"
- Foundational theory of capabilities
- Object-capability model (ocaps)

---

## 3. Core Contribution

**DCT (Delegation Capability Tokens):** A cryptographic token format and protocol for delegating fine-grained permissions between AI agents with:

1. **Monotonic Attenuation** — Delegated tokens can only have fewer permissions than parent
2. **Cryptographic Binding** — Ed25519 signatures prevent forgery
3. **Time Bounding** — Tokens expire, limiting exposure window
4. **Chain Tracking** — Parent token IDs create audit trail
5. **Re-delegation Limits** — Control depth of delegation chains

**Differentiation from prior art:**
- Unlike Macaroons: Uses Ed25519 (not HMAC), explicit permission model for agent actions
- Unlike Biscuits: Simpler (no Datalog), focused on agent-to-agent delegation
- Unlike OAuth: Cryptographic attenuation, not just scope strings

---

## 4. Technical Specification

### Token Structure
```json
{
  "version": "1.0",
  "token_id": "uuid",
  "delegator": "public_key_hex",
  "delegate": "public_key_hex | *",
  "permissions": [
    {"type": "file:read", "resource": "/path/*", "conditions": {}}
  ],
  "constraints": {
    "expires_at": "ISO8601",
    "max_delegations": 2,
    "allowed_hosts": []
  },
  "parent_token": "uuid | null",
  "issued_at": "ISO8601",
  "signature": "ed25519_hex"
}
```

### Permission Model
- Type: Action category (file:read, file:write, api:call, exec, network, spawn)
- Resource: Target pattern with glob support
- Conditions: Additional constraints (time-of-day, rate limits)

### Signature Construction
1. Canonical JSON payload (sorted keys, no signature field)
2. SHA-256 hash of payload
3. Ed25519 signature of hash
4. Signature hex-encoded in token

### Attenuation Rules
1. Child permissions must be subset of parent
2. Child expiry cannot exceed parent
3. Child max_delegations = parent - 1
4. Child inherits parent_token reference

---

## 5. Security Analysis

### Threat Model
- Malicious sub-agent attempts privilege escalation
- Token theft and replay
- Delegation chain manipulation
- Resource pattern bypass

### Security Properties
1. **Unforgeability** — Cannot create valid token without private key
2. **Non-escalation** — Attenuation is mathematically enforced
3. **Freshness** — Expiry prevents indefinite token reuse
4. **Auditability** — Chain provides complete delegation history

### Attack Surface
- Key compromise (mitigated: short-lived tokens)
- Pattern matching bugs (mitigated: conservative glob semantics)
- Clock skew (mitigated: grace period on expiry)

---

## 6. Implementation

### Reference Implementation
- fdaa-cli v0.5.0
- Python with cryptography library
- ~600 lines of code

### CLI Commands
```bash
fdaa dct create "file:read:/path/*" --expires 60
fdaa dct verify ./token.json
fdaa dct check ./token.json "file:read:/path/file.txt"
fdaa dct attenuate ./parent.json "file:read:/path/subdir/*"
```

### Performance
- Token creation: <10ms
- Signature verification: <1ms
- Permission check: O(n) where n = number of permissions

---

## 7. Evaluation

### Benchmarks
- Create 1000 tokens: Xms
- Verify 1000 tokens: Xms
- Attenuation depth impact

### Case Studies
1. PDF summarization agent with read-only access
2. Code review agent with repo-scoped permissions
3. Multi-hop delegation in agent swarm

### Comparison
| Feature | DCT | Macaroons | Biscuits | OAuth |
|---------|-----|-----------|----------|-------|
| Agent-native | ✓ | | | |
| Offline attenuation | ✓ | ✓ | ✓ | |
| Explicit permissions | ✓ | | ✓ | |
| Ed25519 | ✓ | | ✓ | |
| Chain tracking | ✓ | | ✓ | |

---

## 8. Integration with FDAA/ACC

- DCT implements ACC's capability token layer
- Tokens reference RBAC.md permission definitions
- W^X enforcement: tokens cannot grant write to executable paths
- Sandbox integration: tokens checked before skill execution

---

## 9. Future Work

1. **Token Revocation** — Revocation lists or short-lived + refresh pattern
2. **Hardware Binding** — TPM/HSM integration for high-security deployments
3. **Federated Delegation** — Cross-organization token acceptance
4. **Formal Verification** — Prove attenuation properties in Coq/Lean

---

## 10. Conclusion

DCT provides the missing cryptographic primitive for AI agent permission delegation. By combining capability-based security with modern cryptography, DCT enables:

- Least-privilege execution for agent tasks
- Auditable delegation chains
- Enforceable permission boundaries

As AI agents become more autonomous, DCT offers a foundation for trustworthy multi-agent systems.

---

## References (to verify/expand)

1. Birgisson et al. "Macaroons: Cookies with Contextual Caveats" IEEE S&P 2014
2. Biscuit specification: https://www.biscuitsec.org/
3. Dennis & Van Horn "Programming Semantics for Multiprogrammed Computations" 1966
4. RFC 6749 - OAuth 2.0
5. SPIFFE/SPIRE documentation
6. ACC Spec v1.0.0 (Substr8 Labs)
7. FDAA Spec v1.2.0 (Substr8 Labs)

---

*Scoping document for whitepaper-producer pipeline v2.0*
