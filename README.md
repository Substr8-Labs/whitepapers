# Substr8 Labs Research Publications

**Mission:** Provable, auditable, deterministic agent infrastructure.

---

## 📚 Papers

| Paper | Version | Description |
|-------|---------|-------------|
| [**FDAA**](fdaa/WHITEPAPER.md) | v1.3 | File-Driven Agent Architecture — execution model, workspaces, skills |
| [**FDAA + MCP Governance**](fdaa/FDAA-MCP-GOVERNANCE.md) | v1.0 | Governing MCP servers with cryptographic audit trails |
| [**Skill Verification**](skill-verification/WHITEPAPER.md) | v2.0 | Trust pipeline — how skills are verified before execution |
| [**ACC**](acc/WHITEPAPER.md) | v2.0 | Agent Capability Control — declarative authorization framework |
| [**DCT**](dct/WHITEPAPER.md) | v1.0 | Delegation Capability Tokens — permission delegation between agents |
| [**GAM**](gam/WHITEPAPER.md) | **v3.1** | Git-Native Agent Memory — cryptographic verification + attention-weighted retrieval |

---

## 🔬 GAM v3.1 Highlights (Latest)

**Git-Native Agent Memory v3.1** introduces:

- **Typed Hints Architecture**: Separates specific hints (entities, dates) from intent hints (generic phrases)
- **Gated Retrieval**: Query tier classification (keyword/semantic/intent) with selective index participation
- **Benchmark Results**: MRR +32.5%, Intent Recall@5 5x improvement

```
┌─────────────────────────────────────────────────────────────┐
│                    WRITE PATH (Git)                         │
│  Agent stores memory → git commit → embed → index           │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    READ PATH (Hybrid Search)                │
│  Query → classify tier → gate indexes → RRF fusion          │
│  → attention rerank → return verified (git SHA)             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📦 DOI References

| Paper | DOI |
|-------|-----|
| FDAA v1.2 | [`10.5281/zenodo.18675147`](https://doi.org/10.5281/zenodo.18675147) |
| ACC v1.0 | [`10.5281/zenodo.18676278`](https://doi.org/10.5281/zenodo.18676278) |
| Skill Verification v1.0 | [`10.5281/zenodo.18676240`](https://doi.org/10.5281/zenodo.18676240) |
| GAM v1.0 | [`10.5281/zenodo.18704573`](https://doi.org/10.5281/zenodo.18704573) |
| GAM v3.0 | [`10.5281/zenodo.18758191`](https://doi.org/10.5281/zenodo.18758191) |

---

## 🔗 Related Repositories

- [fdaa-cli](https://github.com/Substr8-Labs/fdaa-cli) — Reference implementation
- [acc-spec](https://github.com/Substr8-Labs/acc-spec) — ACC specification
- [gam](https://github.com/Substr8-Labs/gam) — GAM implementation (substr8-gam)
- [fdaa-proxy](https://github.com/Substr8-Labs/fdaa-proxy) — Governed MCP Gateway

---

## 📖 Citation

```bibtex
@misc{substr8labs2026gam,
  title={Git-Native Agent Memory: A Verifiable Foundation for AI Cognitive Continuity},
  author={Heydra, Rudi},
  year={2026},
  publisher={Substr8 Labs},
  url={https://github.com/Substr8-Labs/whitepapers/blob/main/gam/WHITEPAPER.md}
}
```

---

**Substr8 Labs** — Building provable agent infrastructure.

- Website: [substr8labs.com](https://substr8labs.com)
- GitHub: [@Substr8-Labs](https://github.com/Substr8-Labs)
