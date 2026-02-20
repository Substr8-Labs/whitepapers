# Substr8 Labs — Whitepapers

**Mission:** Provable, auditable, deterministic agent infrastructure.

This repository contains all research publications and specifications from Substr8 Labs.

---

## 📚 Papers

| Paper | Version | Status | Description |
|-------|---------|--------|-------------|
| [**FDAA**](./fdaa/WHITEPAPER.md) | v1.2.0 | ✅ Published | File-Driven Agent Architecture — Verifiable execution model for AI agents |
| [**ACC**](./acc/WHITEPAPER.md) | v1.0.0 | ✅ Published | Agent Capability Control — Declarative authorization for autonomous AI |
| [**GAM**](./gam/WHITEPAPER.md) | v2.1.0 | ✅ Published | Git-Native Agent Memory — Versioned, verifiable memory for AI agents |
| [**Skill Verification**](./skill-verification/WHITEPAPER.md) | v1.0.0 | ✅ Published | Pipeline for cryptographic skill verification |
| [**DCT**](./dct/WHITEPAPER.md) | v1.0.0 | ✅ Published | Delegation Capability Tokens — Cryptographic permission delegation |

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     SUBSTR8 STACK                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │  FDAA   │  │   ACC   │  │   GAM   │  │   DCT   │        │
│  │         │  │         │  │         │  │         │        │
│  │ Execute │  │ Authorize│  │ Remember│  │ Delegate│        │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │
│       │            │            │            │              │
│       └────────────┴────────────┴────────────┘              │
│                         │                                   │
│              ┌──────────┴──────────┐                        │
│              │  Skill Verification │                        │
│              │     Pipeline        │                        │
│              └─────────────────────┘                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**FDAA** — How agents execute tasks (file-driven, deterministic)  
**ACC** — What agents are allowed to do (capability-based auth)  
**GAM** — What agents remember (git-native, verifiable memory)  
**DCT** — How agents delegate authority (offline-verifiable tokens)  
**Skill Verification** — How we trust agent skills (cryptographic pipeline)

---

## 📖 Reading Order

**New to Substr8?** Start here:

1. **FDAA** — Core execution model (the foundation)
2. **ACC** — Authorization layer (what controls the agent)
3. **GAM** — Memory layer (how agents persist knowledge)
4. **Skill Verification** — Trust pipeline (how skills are verified)
5. **DCT** — Delegation (advanced: offline token attenuation)

---

## 🔗 External Links

| Resource | Link |
|----------|------|
| Website | [substr8labs.com](https://substr8labs.com) |
| GitHub | [github.com/Substr8-Labs](https://github.com/Substr8-Labs) |
| PyPI | [pypi.org/project/substr8](https://pypi.org/project/substr8/) |

---

## 📄 Citation

If you use our work, please cite:

```bibtex
@misc{substr8labs2026,
  author = {Substr8 Labs},
  title = {Provable Agent Infrastructure},
  year = {2026},
  publisher = {GitHub},
  url = {https://github.com/Substr8-Labs/whitepapers}
}
```

---

## 📜 License

All whitepapers are released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

Code examples within papers are released under [MIT License](https://opensource.org/licenses/MIT).

---

*This is the intellectual foundation of Substr8 Labs. Every paper represents a step toward provable agent infrastructure.*
