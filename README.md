# Nodus Protocol

**Governance specification for Digital Workers** — identity, delegation, mandate, audit, emergency control, and verifiable human intervention.

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Status: First Public Release](https://img.shields.io/badge/Status-First%20Public%20Release-green)](SPEC.md)
[![Contact](https://img.shields.io/badge/Contact-protocol%40nodus.social-blue)](mailto:protocol@nodus.social)

---

## What is the Nodus Protocol?

The Nodus Protocol defines the minimum infrastructure for a digital workforce to be governed, audited, and trusted.

It answers:

- *Who is this Digital Worker, and who authorised them?* → **Identity + Delegation**
- *What are they allowed to do?* → **Mandates**
- *What did they actually do?* → **Audit Log**
- *Can a human stop them immediately?* → **Emergency Controls**
- *Can humans from other organisations approve DW actions?* → **Cross-Enterprise HITL**

> *"You cannot govern what you cannot identify."*

---

## Origin

The Nodus Protocol emerged from seven editions of [Democracy4All](https://www.democracy4all.barcelona/) (Barcelona, 2019–2026), a conference series exploring the governance of digital systems through blockchain, Web3, and AI.

Three principles crystallised over those seven years:

1. Identity is governance
2. Immutability enables trust
3. Authority must be cryptographic, not contractual

The protocol is the formal technical articulation of those principles, applied to autonomous AI agents.

See [MOTIVATION.md](MOTIVATION.md) for the full story.

---

## Built on Nostr

The governance layer uses [Nostr](https://github.com/nostr-protocol/nostr) NIPs for cryptographic identity and immutable records:

| Kind range | Layer | Purpose |
|------------|-------|---------|
| 10001–10006 | Session | DW↔Human messaging, HITL, streaming |
| 10010–10013 | A2A | DW↔DW direct delegation |
| 10020–10021 | Async HITL | Inbox items |
| 34000–34010 | Governance | Identity, mandate, audit, emergency |

---

## Documents

| Document | Description |
|----------|-------------|
| [SPEC.md](SPEC.md) | Full protocol specification |
| [KINDS.md](KINDS.md) | Event kinds reference with JSON examples |
| [FLOWS.md](FLOWS.md) | Protocol flows with sequence diagrams |
| [MOTIVATION.md](MOTIVATION.md) | Origin and context |
| [IMPLEMENTATION-GUIDE.md](IMPLEMENTATION-GUIDE.md) | Building a conformant implementation |

---

## Certified Implementations

| Name | Vendor | Status |
|------|--------|--------|
| **Nodus OS** | Nodus Factory | Certified — first production implementation |

To register a new implementation, open a pull request.

---

## License

[Creative Commons Attribution 4.0 International](LICENSE)

---

*Nodus Protocol Working Group · [protocol@nodus.social](mailto:protocol@nodus.social)*
