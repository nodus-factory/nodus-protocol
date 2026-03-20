# Nodus Protocol

> **"You cannot govern what you cannot identify."**

The Nodus Protocol defines a minimal set of standards for the **identity, delegation, governance, verifiable human intervention, and audit of Digital Workers** in enterprise environments.

## The Problem

Companies are incorporating AI agents into their daily operations at scale. These agents make decisions, execute actions, access sensitive data, sign documents, and manage money.

But no executive, auditor, or regulator can answer the most basic questions:

*Who made this decision? Did they have the authority to do so? Who authorized them? When? And how can it be stopped?*

The Nodus Protocol is the answer.

## What It Is

A **transport-agnostic governance layer** for digital workforces:

- 🔑 **Cryptographic identity** for every Digital Worker (DW)
- 📋 **Signed mandates** defining what each DW can and cannot do
- 🔗 **Verifiable delegation** from human to agent
- 📜 **Immutable audit log** of every significant action
- 🚨 **Panic button** — revoke any DW in seconds
- 🏢 **Federation** — cross-enterprise DW collaboration with verified identity

## The 4 Layers

| Layer | Protocol | Role |
|-------|----------|------|
| Cryptographic governance | Nostr (NIPs 01/26/42/44/46/59/89/90) | Identity, delegation, audit, HITL |
| Synchronous A2A | Google A2A (JSON-RPC/HTTP + SSE) | Direct agent-to-agent calls |
| Persistent sessions | ACP | Multi-turn orchestration |
| Agent↔Tools | MCP | Governed access to external systems |

## Read the Specification

→ [SPEC.md](./SPEC.md) — Full technical specification v0.1

## Key Kinds (Nostr vocabulary)

| Kind | Name | Description |
|------|------|-------------|
| `34000` | `nodus:dw-profile` | Digital Worker identity and capabilities |
| `34001` | `nodus:org-relation` | Organizational graph arcs |
| `34002` | `nodus:policy` | DW mandate — immutable |
| `34003` | `nodus:audit-event` | Audit log — append-only |
| `34004` | `nodus:mcp-server-profile` | Certified MCP Server identity |
| `34005` | `nodus:emergency-stop` | Panic button |

## Reference Implementation

[Nodus OS](https://nodusos.com) is the first certified implementation of the Nodus Protocol.

## License

[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE)

---

*Nodus Factory · March 2026*
