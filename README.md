# Nodus Protocol

**Governance specification for Digital Workers** — identity, delegation, mandate, audit, emergency control, and verifiable human intervention.

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Status: Release Candidate](https://img.shields.io/badge/Status-Release%20Candidate-green)](SPEC.md)
[![Version: 0.2](https://img.shields.io/badge/Version-0.2-blue)](CHANGELOG.md)

## What is the Nodus Protocol?

The Nodus Protocol defines the **minimum infrastructure** for a digital workforce to be governed, audited, and trusted. It answers:

- *Who is this Digital Worker, and who authorised them?* → **Identity + Delegation**
- *What are they allowed to do?* → **Mandates**
- *What did they actually do?* → **Audit Log**
- *Can a human stop them immediately?* → **Emergency Controls**
- *Can humans from other companies authorise DW actions?* → **Cross-Enterprise HITL**

> *"You cannot govern what you cannot identify."*

## Built on Nostr

The protocol uses Nostr NIPs as its cryptographic governance layer. Nostr events are immutable, verifiable by any third party, and relay-agnostic. The governance layer uses kinds 34000–34010 (NIP-33 parameterized replaceable events).

## Status

| Version | Status | Date |
|---------|--------|------|
| v0.1 | Internal Draft | March 2026 |
| **v0.2** | **Release Candidate** | **April 2026** |

The reference implementation (**Nodus OS**) implements all v0.2 milestones (M0–M13) behind feature flags. All flags default to `false` — the v1 system remains untouched.

## Quick Overview

### Protocol Kinds

| Range | Layer | Purpose |
|-------|-------|---------|
| 10001–10006 | Session | DW↔Human chat, HITL, streaming |
| 10010–10013 | A2A | DW↔DW direct delegation (v0.2) |
| 10020–10021 | Async HITL | Inbox items (crons, graphs) |
| 34000–34010 | Governance | Identity, mandate, audit, emergency |

### Key Features

- **Cryptographic identity** for every Digital Worker (kind:34000)
- **Immutable mandates** defining exactly what each DW can do (kind:34002)
- **Permanent audit log** for every DW action (kind:34003)
- **Constitutional HITL** — human approvals are cryptographic signatures (kind:10004)
- **Emergency stop** — halt all DWs in < 30 seconds (kind:34005)
- **Policy Relay** — the DW's nsec never leaves the relay (NIP-46 variant)
- **Cross-enterprise** — DWs from company A can work for company B with full governance (v0.2)
- **Verifiable contracts** — any third party can verify DW authority without trusting Nodus (v0.2)

## Read the Spec

| Document | Description |
|----------|-------------|
| [SPEC.md](SPEC.md) | Full specification v0.2 |
| [KINDS.md](KINDS.md) | Event kinds reference — all 16 kinds with JSON examples |
| [FLOWS.md](FLOWS.md) | Protocol flows — 8 flows with sequence diagrams |
| [CHANGELOG.md](CHANGELOG.md) | Version history |

## Reference Implementation

The reference implementation is **Nodus OS** (private repository: `github.com/nodus-factory/nodus-os-adk`). All milestones M0–M13 are implemented additively behind feature flags.

### Milestone Status

| Milestone | Description | Status |
|-----------|-------------|--------|
| M0 | DW & Human Identities | ✅ Implemented |
| M1 | Mandates | ✅ Implemented |
| M2 | Audit Log | ✅ Implemented |
| M3 | NIP-26 Delegation | ✅ Implemented |
| M4 | Constitutional HITL | ✅ Implemented |
| M5 | Emergency Stop | ✅ Implemented |
| M6 | Room UX + Inbox | ✅ Implemented |
| M7 | MCP Governance | ✅ Implemented |
| M8 | Policy Relay | ✅ Implemented |
| M9 | A2A Nostr-Native | ✅ Implemented (v0.2) |
| M10 | Multi-Relay Federation | ✅ Implemented (v0.2) |
| M11 | Public DW Marketplace | ✅ Implemented (v0.2) |
| M12 | Cross-Enterprise HITL | ✅ Implemented (v0.2) |
| M13 | Verifiable Contracts | ✅ Implemented (v0.2) |

## License

[Creative Commons Attribution 4.0 International](LICENSE)

---

*Nodus Factory · April 2026*
