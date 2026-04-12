# Nodus Protocol — Specification v0.2

> **Status:** Release Candidate — Reference Implementation Available  
> **Authors:** Quirze Salomó · Nodus Factory  
> **Date:** April 2026  
> **License:** Creative Commons Attribution 4.0 International (CC BY 4.0)  
> **Reference Implementation:** github.com/nodus-factory/nodus-os-adk (private)  
> **Previous version:** v0.1 (March 2026) — internal draft

---

## Abstract

The Nodus Protocol defines a minimal set of standards for the **identity, delegation, governance, verifiable human intervention, and audit of Digital Workers** in enterprise environments.

The protocol is **transport-agnostic**: it does not impose a single communication channel between agents, but rather a common layer of action legitimacy. A Digital Worker can communicate over A2A, ACP, HTTP, queues, or Nostr — but when it acts, the Protocol ensures that anyone can answer:

*Who performed this action? Did they have the authority to do so? Who authorized them? When? On what basis? And how can it be stopped?*

The Nodus Protocol is not a platform. It is not a product. It is the minimum infrastructure needed for a digital workforce to be governed, audited, and trusted — just as today's human workforce is.

**v0.2 extends v0.1 with:** A2A Nostr-native communication (kinds 10010–10013), multi-relay federation, a public DW marketplace, cross-enterprise Human-in-the-Loop, and verifiable employment contracts. All features are implemented in the Reference Implementation behind feature flags that default to `false`.

---

## Table of Contents

1. [The World Ahead](#1-the-world-ahead)
2. [Motivation](#2-motivation)
3. [Design Principles](#3-design-principles)
4. [Core Concepts](#4-core-concepts)
5. [Technical Specification](#5-technical-specification)
   - 5.1 Nostr NIPs as Governance Foundation
   - 5.2 Nodus Protocol Kinds (Session + Governance)
   - 5.3 Synchronous A2A Transport
   - 5.4 Persistent ACP Sessions
   - 5.5 MCP under Protocol Governance
   - 5.6 Nodus Policy Relay
   - 5.7 Enterprise Control over Digital Workers
   - 5.8 Panic Button and Revocation
   - 5.9 Constitutional Separation: Human / Digital Worker
   - 5.10 Graduated Governance Model
   - 5.11 Federation and Cross-Enterprise Communication
   - 5.12 A2A Nostr-Native Protocol (v0.2)
   - 5.13 Multi-Relay Federation (v0.2)
   - 5.14 Public DW Marketplace (v0.2)
   - 5.15 Cross-Enterprise HITL (v0.2)
   - 5.16 Verifiable Employment Contracts (v0.2)
6. [Reference Implementation](#6-reference-implementation)
   - 6.1 Reference Implementation Status
   - 6.2 Activation Flags
7. [Conformance and Certification](#7-conformance-and-certification)
8. [Appendix](#8-appendix)

---

## 1. The World Ahead

Companies are about to incorporate hundreds — and soon thousands — of artificial intelligence agents into their daily operations. These agents will make decisions, execute actions, access sensitive data, sign documents, manage money, hire suppliers, and interact with customers.

Some of these agents will have been created by the company itself. Others by a partner. Others contracted as a service. Some will be built on GPT. Others on Claude. Others on local models. Some will communicate with each other. Others will operate alone.

And no one — no executive, no auditor, no regulator — will be able to answer the most basic questions:

*Who made this decision? Did they have the authority to do so? Who authorized them? When? On what basis?*

This is already happening today, at small scale. In two years, it will be the central problem of corporate governance.

The Nodus Protocol is the answer to this problem.

---

## 2. Motivation

> *Voice of Quirze Salomó, founder*

I have been talking about AI governance for two or three years. I have always believed that the digital identity of agents — of Digital Workers — is the key to governing all of this.

Blockchain technology can contribute very interesting things to the world of AI governance and control. But in the enterprise world, that piece was missing.

The critical moment came when I saw that Digital Workers were starting to gain very serious power, that the acceleration was real, and that moving forward without governance would be chaos — for companies and for society.

The Nodus Protocol was born from a simple conviction:

> **"You cannot govern what you cannot identify."**

And you cannot identify anything that does not have a verifiable, independent, cryptographic identity.

This is a protocol designed for companies and institutions. But the problem it solves is a general social problem.

---

## 3. Design Principles

These principles are non-negotiable. An implementation that violates them is not compatible with the Nodus Protocol.

### P1 — Clear and Verifiable Identity

Every agent operating under the Nodus Protocol must have a unique cryptographic identity, independent of any central platform. Who is who must always be verifiable without consulting any third-party server.

### P2 — Native Interoperability

No implementation may create closed silos. Any certified Digital Worker must be able to communicate with any other certified DW, and with any authorized human. The protocol defines the common language; no implementation may speak an incompatible dialect.

### P3 — Sufficient Decentralization

The protocol must not depend on a central server, a single company, or a single jurisdiction. It must be able to survive the failure of any node, including Nodus Factory itself. No single entity may have the power to shut down the network.

### P4 — Immutable Auditability

Every significant action by a Digital Worker must be recorded in a way that cannot be modified or deleted. Not for surveillance, but for accountability. The audit log is append-only; no one can erase the trace of a past action.

### P5 — Balance Between Privacy and Transparency

The protocol must allow a company to internally audit all actions of its Digital Workers, without those actions being visible to unauthorized third parties. At the same time, it must ensure that certain facts — that an action occurred, that an agent had authority to perform it — can be verified externally without revealing the content.

This is neither total opacity nor total transparency. It is **selective governed transparency**.

### P6 — Ontological Separation: Human / Digital Worker

A human and a Digital Worker are not the same type of entity. Ever. A DW cannot escalate its privileges to the level of a human. A human cannot be impersonated by a DW. This separation is structural, not a configuration option, and cannot be bypassed by any implementation.

### P7 — Parallel and Additive (v0.2)

No component of the protocol may modify, degrade, or interfere with existing v1 systems. All protocol features run in parallel, protected by feature flags that default to `false`. The existing HTTP transport and session system is never touched.

---

## 4. Core Concepts

### 4.1 Digital Worker (DW)

A Digital Worker is an artificial intelligence agent that acts on behalf of an organization or person, with an **explicit mandate and defined limits**.

A DW is not a script. It is not a chatbot. It is an entity with:
- Its own verifiable identity
- A clear human or organizational owner
- A declared set of capabilities
- A set of limits it cannot exceed
- An auditable history of actions

The fundamental difference between a DW and a generic AI agent is **traceable accountability**. It is always known who created the DW, who authorized it to act, and what it has done.

### 4.2 Cryptographic Identity

Each entity operating under the Nodus Protocol — human or digital — has a cryptographic identity based on a key pair:

- **`npub`** (public key) — public identifier, visible to everyone on the network
- **`nsec`** (private key) — never shared, controlled exclusively by the owner

This identity is **self-sovereign**: it does not depend on any central server. A DW continues to have verifiable identity even if Nodus Factory ceases to exist.

> Technical basis: NIP-01 of the Nostr protocol. All events use BIP-340 Schnorr signatures over secp256k1.

### 4.3 Owner

Every Digital Worker has an owner: the person or organization that created it and is responsible for it. The owner–DW relationship is cryptographically verifiable.

An owner can:
- Create and revoke mandates
- Delegate authority to the DW
- Audit all DW actions
- Transfer ownership to a third party
- Stop the DW immediately (panic button)

### 4.4 Mandate (kind 34002)

The mandate is the document that defines **what a Digital Worker can and cannot do**. It is the contract between the owner and the DW. It includes:

- Authorized capabilities (which actions it may execute)
- Explicit limits (what it may never do)
- HITL-required actions (which require live human approval)
- Auto-approved actions (which bypass HITL)
- Scope (over which data and systems it may act)
- Validity period (from when to when it is valid)

The mandate is **immutable once signed** by the owner. If it needs to change, a new version must be signed. Relays must refuse DELETE and UPDATE on kind:34002.

### 4.5 Delegation (NIP-26)

Delegation is the mechanism by which an authorized human transfers authority to a Digital Worker to act on their behalf. When a DW acts with delegation:

- Its action carries the DW's signature (technical identity)
- And the cryptographic proof that the owner authorized this action (delegated authority)

Any third party can verify both without consulting any central server.

### 4.6 Agent-to-Agent Interaction (A2A)

Digital Workers communicate with each other. The Nodus Protocol supports two primary A2A patterns:

- **Synchronous A2A HTTP v1** (Google A2A over HTTP/JSON-RPC) — when an immediate response is needed, legacy transport
- **A2A Nostr-Native v0.2** (kinds 10010–10013) — direct DW-to-DW delegation via the relay, no HTTP intermediary

The protocol does not impose a single transport. What it imposes is that every governed interaction can demonstrate: who is acting, with what authority, under what mandate, and what trace it leaves.

### 4.7 Human-in-the-Loop (HITL)

Certain actions require real human intervention. The Nodus Protocol distinguishes between:

- **Operational HITL:** the human receives a notification and approves via any surface
- **Constitutional HITL:** when the human exercises authority, this act is materialized on the Nostr layer as a verifiable cryptographic proof (kind:10004, signed by the human's keypair)
- **Cross-Enterprise HITL (v0.2):** a human from another company can approve DW actions using their own keypair and relay

> Ordinary human↔agent conversation may pass through any channel. But when a human exercises authority, the Protocol records it permanently.

### 4.8 Audit Log (kind 34003)

The audit log is the immutable record of all significant actions by a Digital Worker. Each entry contains:

- Who performed the action (DW identity)
- With what authority (reference to mandate and delegation)
- When (verifiable timestamp)
- What was done (hash of the action and its result)
- In the context of which interaction (session and mandate references)

The audit log is **append-only**: no entry can be modified or deleted. Not by the DW's owner. Not by Nodus. Not by anyone.

---

## 5. Technical Specification

The Nodus Protocol is implemented over four complementary communication layers:

| Layer | Protocol | Role | When to use |
|-------|----------|------|-------------|
| **Cryptographic governance** | Nostr (NIPs 01/16/26/33/42/44/46/59/89) | Identity, delegation, revocation, immutable audit, constitutional HITL | When legitimizing, revoking, auditing, or involving a human with verifiable authority |
| **A2A Nostr-native (v0.2)** | Nostr kinds 10010–10013 | DW↔DW direct delegation via relay | Cross-DW coordination without HTTP server dependency |
| **Synchronous A2A (v1)** | Google A2A (JSON-RPC/HTTP + SSE) | Direct agent→agent synchronous calls | DW executes atomic task with blocking response |
| **Persistent sessions** | ACP (Agent Communication Protocol) | Stateful sessions, streaming, long context, multi-turn orchestration | LLM streaming between agents, long conversational sessions |
| **Agent↔Tools** | MCP (Model Context Protocol) | Access to tools, APIs, databases, external systems | DW accesses calendar, CRM, email, files, APIs |

**The Nodus Protocol is transport-agnostic but rooted in Nostr for governance.** Agents may communicate over many channels, but when an action requires protocol legitimacy, that legitimacy lives in the Nostr cryptographic layer.

---

### 5.1 Nostr NIPs as Governance Foundation

The protocol requires support for the following NIPs:

| NIP | Name | Role in Nodus | Status |
|-----|------|--------------|--------|
| NIP-01 | Basic protocol | Foundation. Event signing, basic kinds. Mandatory. | ✅ Implemented |
| NIP-16 | Event Treatment | Kinds 10000–19999 replaceable semantics | ✅ Implemented |
| NIP-19 | bech32-encoded entities | `nsec1…`, `npub1…` for all keys | ✅ Implemented |
| NIP-26 | Delegated event signing | DWs sign on behalf of the owner. Verifiable authority proofs. | ✅ Implemented |
| NIP-29 | Relay-based Groups | Work rooms (M6) | ✅ Implemented |
| NIP-33 | Parameterized Replaceable Events | All kinds 34000–34010 governance layer | ✅ Implemented |
| NIP-42 | Relay AUTH | Private relay per company. Only authenticated identities connect. | ⚠️ Partial |
| NIP-44 | Versioned Encryption | Encrypted P2P communication between agents (future) | 📋 Referenced |
| NIP-46 | Nostr Connect / Remote Signing | Technical basis for the Policy Relay (M8) | ✅ Implemented (variant) |
| NIP-59 | Gift Wrap | Maximum privacy for sensitive HITL decisions | 📋 Referenced |
| NIP-89 | App Handler Info | Agent Cards and discovery on the relay | ✅ Partial |
| NIP-90 | Data Vending Machines | Async A2A decoupled in time (future) | 📋 Referenced |

**NIP-01 implementation:** The reference implementation uses a pure-Python BIP-340 Schnorr signer with no external cryptographic dependencies. All event serialisation, hashing, and signature verification is self-contained.

```python
class _NostrSigner:
    def create_signed_event(self, kind: int, content: str, tags: list[list[str]]) -> dict:
        created_at = int(time.time())
        serialised = json.dumps(
            [0, self._pubkey_hex, created_at, kind, tags, content],
            separators=(",", ":"), ensure_ascii=False,
        )
        event_id_bytes = hashlib.sha256(serialised.encode("utf-8")).digest()
        sig = _schnorr_sign(event_id_bytes, self._sk_bytes, aux_rand)
        return {"id": event_id_bytes.hex(), "pubkey": self._pubkey_hex,
                "created_at": created_at, "kind": kind,
                "tags": tags, "content": content, "sig": sig.hex()}
```

---

### 5.2 Nodus Protocol Kinds (Session + Governance)

The Nodus Protocol defines two kind ranges. The complete reference is in [KINDS.md](KINDS.md).

#### Session Layer (kinds 10001–10021)

NIP-16 replaceable events. The DW uses a `since` filter to avoid processing stale messages.

| Kind | Name | Publisher | Description | Status |
|------|------|-----------|-------------|--------|
| `10001` | `MESSAGE_USER` | Human | User message to a DW | ✅ v0.1 |
| `10002` | `RESPONSE_DW` | DW | Final DW response | ✅ v0.1 |
| `10003` | `HITL_REQUEST` | DW | Human approval request | ✅ v0.1 |
| `10004` | `HITL_RESPONSE` | Human | Cryptographically signed human decision | ✅ v0.1 |
| `10005` | `RESPONSE_AGENT` | Agent | Internal agent-to-agent (HTTP v1) | ✅ v0.1 |
| `10006` | `STREAMING_CHUNK` | DW | Real-time streaming chunk | ✅ v0.1 |
| `10010` | `A2A_REQUEST` | DW | DW-to-DW task request (Nostr-native) | ✅ v0.2 |
| `10011` | `A2A_RESPONSE` | DW | DW-to-DW task response | ✅ v0.2 |
| `10012` | `A2A_STREAM` | DW | DW-to-DW streaming chunk | ✅ v0.2 |
| `10013` | `A2A_ERROR` | DW | DW-to-DW error | ✅ v0.2 |
| `10020` | `INBOX_ITEM` | DW/Cron | Async HITL request (inbox) | ✅ v0.1 |
| `10021` | `INBOX_RESOLVED` | Human | Async HITL resolution | ✅ v0.1 |

#### Governance Layer (kinds 34000–34010)

NIP-33 parameterized replaceable events. The `["d", "<identifier>"]` tag defines uniqueness within pubkey+kind.

| Kind | Name | `d` tag value | Mutability | Status |
|------|------|---------------|------------|--------|
| `34000` | `nodus:dw-profile` | `<dw_pubkey_hex>` | Replaceable | ✅ v0.1 |
| `34001` | `nodus:org-relation` | `<owner_hex>-<dw_hex>` | Replaceable | ✅ v0.1 |
| `34002` | `nodus:policy` | `<dw_pubkey_hex>` | **IMMUTABLE** | ✅ v0.1 |
| `34003` | `nodus:audit-event` | `sha256(dw+session+ts_ms)` | **Append-only** | ✅ v0.1 |
| `34004` | `nodus:mcp-server-profile` | `"nodus-mcp-gateway"` | Replaceable | ✅ v0.1 |
| `34005` | `nodus:emergency-stop` | `<tenant_id>` | Immutable | ✅ v0.1 |
| `34006` | `nodus:emergency-resume` | `<tenant_id>` | Immutable | ✅ v0.1 |
| `34010` | `nodus:kyc-corp-claim` | `"kyc-<tenant_id>"` | Replaceable | ✅ v0.2 |

> **Critical relay rules:**
> 1. Relays MUST reject DELETE or UPDATE on kind:34002 and kind:34003
> 2. DWs (`entity_type: "digital_worker"`) MUST NOT be permitted to sign kinds 34002, 34005
> 3. kind:34003 `d` uniqueness enforces append-only semantics

#### Kind 34000 — Digital Worker Profile (full structure)

```json
{
  "kind": 34000,
  "pubkey": "<dw_pubkey_hex>",
  "tags": [
    ["d", "<dw_pubkey_hex>"],
    ["p", "<owner_pubkey_hex>"]
  ],
  "content": "{\"name\":\"Athena\",\"description\":\"Root orchestrator agent\",\"owner\":\"<owner_hex>\",\"tenant\":\"default\",\"entity_type\":\"digital_worker\",\"capabilities\":[\"orchestrate\",\"email\",\"calendar\"],\"limits\":[\"no_financial_without_hitl\"],\"transports\":[{\"type\":\"nostr-session\",\"relay\":\"ws://nostr-relay:7777\",\"kinds\":[10001]},{\"type\":\"a2a\",\"url\":\"https://adk.nodus.local/agents/athena/a2a\"}],\"nodus_version\":\"0.2\"}"
}
```

#### Kind 34002 — Policy / Mandate (full structure)

```json
{
  "kind": 34002,
  "pubkey": "<owner_pubkey_hex>",
  "tags": [
    ["d", "<dw_pubkey_hex>"],
    ["p", "<dw_pubkey_hex>"]
  ],
  "content": "{\"dw\":\"<dw_pubkey_hex>\",\"tenant\":\"my-tenant\",\"capabilities\":[\"send_email\",\"read_calendar\",\"orchestrate\"],\"limits\":[\"no_delete_without_confirmation\"],\"hitl_required\":[\"send_email\",\"delete_*\",\"financial_*\"],\"auto_approve\":[\"read_calendar\",\"list_memory\"],\"max_auto_cost_eur\":0,\"valid_from\":1714000000,\"valid_until\":null,\"nodus_version\":\"0.1\"}"
}
```

#### Kind 34003 — Audit Event (full structure)

```json
{
  "kind": 34003,
  "pubkey": "<dw_pubkey_hex>",
  "tags": [
    ["d", "<sha256_of_dw+session+ts_ms>"],
    ["p", "<user_pubkey_hex>"],
    ["mandate", "<kind:34002_event_id>"],
    ["session", "<session_uuid>"],
    ["action", "agent_response"],
    ["tenant", "<tenant_id>"]
  ],
  "content": "{\"action\":\"agent_response\",\"result_hash\":\"<sha256>\",\"timestamp\":1714000010,\"dw\":\"<dw_pubkey_hex>\",\"user\":\"<user_hex>\",\"tenant\":\"default\",\"session_id\":\"<uuid>\",\"mandate_ref\":\"<event_id>\"}"
}
```

---

### 5.3 Synchronous A2A Transport

Google A2A (v0.3.0, 50+ partners: Salesforce, SAP, Atlassian, PayPal...) is the emerging standard for direct agent-to-agent communication. The Nodus Protocol adopts it as the primary synchronous transport and adds the governance layer it lacks.

**The gap Nodus fills over Google A2A:**

| Google A2A | + Nodus Protocol |
|-----------|-----------------|
| Agent Cards | Agent Cards + npub + signed mandate (34002) |
| Task lifecycle | Task lifecycle + immutable audit (34003) |
| HTTP/JSON-RPC | HTTP/JSON-RPC + cryptographic signature |
| Auth: OpenAPI keys | Auth: NIP-26 cryptographic delegation |
| No revocation | Revocation in seconds (kind 34002 revoked) |

**v0.2 note:** The Nodus Protocol v0.2 introduces A2A Nostr-Native (section 5.12) as an alternative to A2A HTTP. Both transports coexist; the Nostr-native transport eliminates server intermediaries and enables cross-tenant federation.

---

### 5.4 Persistent ACP Sessions

Google A2A is synchronous and stateless. The Nodus Protocol supports **ACP (Agent Communication Protocol)** for multi-turn orchestration: an orchestrating agent collaborating in a long conversation with subordinate agents, maintaining context across multiple interactions.

#### When to use ACP

- An orchestrating agent coordinating multiple DWs in a complex flow
- An agent maintaining a long working session with a specialized agent
- Workflows requiring shared context between steps
- Streaming partial responses between agents while work is in progress

#### Relationship between layers

```
Orchestrator (ACP session)
      │
      ├─→ DW Office   (A2A HTTP, direct call)        ← atomic task
      ├─→ DW Business (A2A Nostr, v0.2)              ← atomic task, no HTTP
      │
      └─→ Sub-orchestrator (ACP session)             ← complex multi-step flow
              │
              └─→ Audit → Nostr (kind 34003)         ← immutable record
```

---

### 5.5 MCP under Protocol Governance

#### The problem MCP does not solve

MCP (Model Context Protocol, Anthropic 2024) is the emerging standard for connecting agents with external tools and systems. But MCP has no identity. An MCP Server is a URL. There is no mechanism to know who manages it, whether it is legitimate, or how to revoke access.

> **"An MCP Server without verifiable identity is an attack vector. The Nodus Protocol gives it a passport."**

#### The solution: kind 34004 — MCP Server Profile

Just like Digital Workers, MCP Servers operating under the Nodus Protocol have a keypair, a registered profile (kind:34004), and an audit trail.

```json
{
  "kind": 34004,
  "pubkey": "<mcp_gateway_pubkey_hex>",
  "tags": [["d", "nodus-mcp-gateway"]],
  "content": "{\"name\":\"Nodus MCP Gateway\",\"url\":\"https://mcp.nodus.local\",\"tools\":[\"calendar\",\"email\",\"drive\",\"crm\"],\"owner\":\"<owner_pubkey_hex>\",\"authorized_dws\":[\"<athena_pubkey_hex>\"],\"valid_until\":null}"
}
```

#### Flow: DW uses an MCP tool under Nodus governance

```
DW wants to use "calendar" from an MCP Server
        │
        ├─→ Check relay: does kind 34004 exist for this MCP Server?
        │       ├── No → REJECT (MCP Server not certified)
        │       └── Yes → check: is this DW in authorized_dws?
        │                   ├── No → REJECT (not authorized)
        │                   └── Yes → CONNECT
        │
        ├─→ Execute MCP call (tool: "calendar", action: "create_event")
        │
        └─→ Record in kind 34003 (audit)
```

---

### 5.6 Nodus Policy Relay

> **Status: fully implemented (M8) — reference implementation available**

Standard Nostr relays distribute events but do not filter them against business policies. The **Nodus Policy Relay** solves this by turning the relay into an active **Signing Service**.

#### The core principle: DW nsec keys never leave the relay

```
v0.1 (DW has its own nsec):
  DW pod (has nsec) ──sign──► relay ──► network

v0.2 Policy Relay (M8):
  DW pod (no nsec) ──unsigned event──► Policy Relay ──signed event──► relay ──► network
```

**The DW does not hold the key. It cannot sign on its own.** Without a valid signature from the Policy Relay, no other Nodus DW accepts its requests. This is **cryptographic impossibility**, not a matter of good faith.

#### WebSocket Protocol

The Policy Relay exposes an internal WebSocket endpoint. The DW sends unsigned events; the Policy Relay signs them if the mandate permits.

**Request:**
```json
{
  "id": "<req_uuid>",
  "method": "sign_event",
  "params": {
    "event": {
      "pubkey": "<dw_pubkey_hex>",
      "created_at": 1714000000,
      "kind": 10002,
      "tags": [["p", "<user_hex>"], ["session", "<id>"]],
      "content": "DW response"
    },
    "dw_pubkey": "<dw_pubkey_hex>"
  }
}
```

**Success response:**
```json
{
  "id": "<req_uuid>",
  "result": {
    "id": "<event_id_sha256>",
    "pubkey": "<dw_pubkey_hex>",
    "created_at": 1714000000,
    "kind": 10002,
    "tags": [...],
    "content": "...",
    "sig": "<schnorr_sig_hex>"
  }
}
```

**Rejection response:**
```json
{
  "id": "<req_uuid>",
  "error": "mandate_violation: action not permitted"
}
```

#### Security model

| Property | v0.1 (direct nsec) | v0.2 (Policy Relay) |
|----------|-------------------|---------------------|
| nsec location | K8s Secret in DW pod | Policy Relay service (isolated) |
| Pod compromise | nsec exposed | Only unsigned events exposed |
| Mandate enforcement | Observational (log-only) | Hard enforcement (sign or reject) |
| Audit | kind:34003 per action | kind:34003 per action |

**Configuration:**
```bash
# DW (no nsec, points to Policy Relay)
NODUS_POLICY_RELAY_V1=true
NODUS_POLICY_RELAY_URL=ws://policy-relay:8080/sign

# Policy Relay (holds all DW nsecs)
NODUS_DW_NSEC_MAP='{"<dw_pubkey_hex>": "nsec1..."}'
# or per-DW:
POLICY_RELAY_NSEC_<PUBKEY_HEX_UPPER>=nsec1...
```

> Technical basis: NIP-46 (Nostr Connect / nsecbunker) — adapted for enterprise DWs with mandate semantics. The Nodus variant replaces NIP-46 metadata events with a direct WebSocket RPC protocol optimised for latency.

#### Relay enforcement: strfry writePolicy plugin

The private relay uses a JavaScript `writePolicy` plugin that enforces governance rules at write time:

| Phase | Rule |
|-------|------|
| M1.4 | Reject DELETE on kind:34002/34003 · DWs cannot sign kinds 34002, 34005 |
| M3.3 | Verify NIP-26 delegation in DW events |
| M5.2 | If kind:34005 active for tenant → reject all DW events (except owner's kind:34006) |

---

### 5.7 Enterprise Control over Digital Workers

A company must be able to know that:
- The DW acting **is theirs** and not an impersonator
- The DW **had authorization** for that specific action
- If someone **steals the DW's key**, it can be revoked immediately
- An **external DW** (partner/supplier) acts with the correct permissions

#### The 4 control layers

**Layer 1 — Governed creation**
The DW keypair is generated at the company's Policy Relay. The `nsec` never leaves the server. Without relay access, the DW cannot act.

**Layer 2 — Signed mandate**
The owner (CEO or admin) signs kind 34002: "may do X, may never do Y". Immutable, verifiable by anyone.

**Layer 3 — Verifiable delegation**
Every DW event carries NIP-26 proof that the owner authorized it. Any third party can verify without consulting any central server.

**Layer 4 — Immutable audit**
Every significant action → kind 34003. Cannot be deleted. Cannot be altered. The trace is permanent.

#### Cryptographic hierarchy

```
CEO / Company  (entity_type: "human")
      │
      │ kind 34002 — signs mandate
      │ NIP-26 — delegates authority
      ▼
DW  (entity_type: "digital_worker")
      │
      │ Each action carries:
      │   ① DW signature (Schnorr BIP-340)
      │   ② Delegation proof (NIP-26 tag)
      │   ③ Mandate reference (kind:34002 event_id)
      ▼
Action → kind 34003 (immutable audit)
```

---

### 5.8 Panic Button and Revocation

> *"A Digital Worker without a panic button is not an enterprise Digital Worker. It is a risk."*

#### 4 revocation scenarios

**1. Compromised DW**
→ Owner signs a new kind:34002 marking it revoked → relay propagates immediately → **Time: seconds**

**2. Employee offboarding**
→ Revoke DWs associated with that human → revoke NIP-26 delegations they issued

**3. Total security incident**
→ Owner publishes kind:34005 (`nodus:emergency-stop`) signed → relay rejects ALL DW events for the tenant → **Time: <30 seconds**

**4. External DW (partner/supplier)**
→ Revoke the NIP-26 delegation to the external DW → immediate effect, no coordination needed

#### kind:34005 — Emergency Stop

```json
{
  "kind": 34005,
  "pubkey": "<owner_pubkey_hex>",
  "tags": [
    ["d", "<tenant_id>"],
    ["tenant", "<tenant_id>"]
  ],
  "content": "{\"tenant\":\"1\",\"reason\":\"Security incident — precautionary suspension\",\"authorized_by\":\"<owner_npub>\"}"
}
```

The DW polls every 30 seconds for kind:34005/34006 events. Logic:
```python
# Active if: latest_stop_at > 0 and latest_stop_at > latest_resume_at
# Effect: if self._emergency_active: return  (first line of _handle_message)
```

---

### 5.9 Constitutional Separation: Human / Digital Worker

The Nodus Protocol closes privilege escalation vectors structurally.

#### The 4 safeguards

**Safeguard 1 — Entity type in the profile**

Kind 34000 carries `entity_type`: `"human"` vs `"digital_worker"`. Relay rule:
```
If event.author.entity_type == "digital_worker"
And event.kind IN [34002, 34005, NIP-26 delegation]
→ REJECT always, without exception
```

**Safeguard 2 — Hardware binding for humans**

Humans authenticate via NIP-07 (browser extension), NIP-46 (mobile app), or hardware tokens. A DW cannot have a phone or browser extension. The asymmetry is physical.

**Safeguard 3 — Non-escalation principle**

> A DW may not issue any event that modifies the limits of its own authority. None. Ever.

**Safeguard 4 — Cryptographic HITL for critical actions**

Actions marked `hitl_required` in the mandate require a live human signature (kind:10004). Without it, the action does not exist on the Nodus network.

---

### 5.10 Graduated Governance Model

The protocol defines 4 governance levels. A company may choose the level that suits it, and migrate upward over time.

| Level | Model | Suitable for |
|-------|-------|-------------|
| 0 | Total Owner | Most SMBs — one human controls everything |
| 1 | Owner + Auditor | 5–10 employees — external read-only audit |
| 2 | Multisig for critical actions | Mid-sized companies, regulated industries |
| 3 | Full governance | Enterprise — quorums, separation of powers, committees |

**All levels share one universal minimum:** the audit log (kind:34003) is always immutable. The owner can do anything, but cannot erase the past.

> *"It is like a permanent digital notary. The notary does not stop you from doing anything. But everything is recorded and verifiable forever."*

---

### 5.11 Federation and Cross-Enterprise Communication

Each company has its own **private relay** (Policy Relay) with its internal governance. For external communication, a **public relay** exists — a shared network where DWs publish what they want to be externally visible.

```
Company A (private relay A)
    │
    └──► publishes kind:34000 to public relay
                    │
    Company B ◄─────┘ (subscribed to public relay)
    (private relay B)
```

#### Cross-enterprise verification

When Company A's DW receives an event from Company B's DW:

1. Verify BIP-340 Schnorr signature → valid pubkey? ✅
2. Query public relay → valid kind:34000 profile? ✅
3. Check kind:34010 corporate KYC claim → verified legal entity? ✅
4. If all OK → accept communication and record in kind:34003

#### v0.2 federation mechanism

The v0.2 federation (section 5.13) extends this with direct relay-to-relay discovery via `relay_hint` tags on kind:34001, enabling DWs to discover and communicate with DWs at other companies' private relays without a central registry.

---

### 5.12 A2A Nostr-Native Protocol (v0.2)

**Feature flag:** `NODUS_A2A_NOSTR_V2`  
**Implementation:** `a2a_nostr_v2.py` in `nodus-adk-runtime`

The A2A Nostr-Native protocol eliminates the HTTP intermediary in agent-to-agent communication. DWs coordinate directly via the Nostr relay using kinds 10010–10013.

#### Motivation

| Aspect | A2A HTTP v1 | A2A Nostr-Native v0.2 |
|--------|-------------|----------------------|
| Server dependency | DW B must have an HTTP endpoint | None — relay only |
| Cross-tenant | Complex HTTP federation | Relay federation via `relay_hint` |
| Audit | Manual kind:34003 | Events are inherently signed and immutable |
| Mandate enforcement | Application-level | Policy Relay enforces at signature time |
| Offline resilience | DW B must be online | Events queue at relay |

#### Protocol kinds

```
kind:10010  A2A_REQUEST   DW A → DW B  (task, signed with optional NIP-26)
kind:10011  A2A_RESPONSE  DW B → DW A  (result)
kind:10012  A2A_STREAM    DW B → DW A  (streaming chunk, content.done flag)
kind:10013  A2A_ERROR     DW B → DW A  (error with context)
```

#### Request structure (kind:10010)

```json
{
  "kind": 10010,
  "pubkey": "<dw_a_pubkey_hex>",
  "tags": [
    ["p", "<dw_b_pubkey_hex>"],
    ["session", "<session_uuid>"],
    ["request_id", "<8char_uuid>"],
    ["action", "summarize_document"],
    ["mandate", "<kind:34002_event_id_optional>"]
  ],
  "content": "{\"action\":\"summarize_document\",\"params\":{\"doc_id\":\"abc123\",\"lang\":\"en\"}}"
}
```

#### Full A2A Nostr flow

```
DW A                         Relay                    DW B
  │  kind:10010 REQUEST        │                         │
  │  request_id = uuid8        │                         │  REQ kinds:[10010] #p=[DW_B]
  ├───────────────────────────►│                         │◄─────────────────────────────
  │                            │  EVENT kind:10010       │
  │  REQ kinds:[10011,10013]   ├────────────────────────►│  handler(action, params)
  │  #p=[DW_A], since=now-5    │                         │
  ├───────────────────────────►│                         │  kind:10011 RESPONSE
  │                            │◄────────────────────────┤
  │  EVENT kind:10011          │                         │
  │◄───────────────────────────┤                         │
```

#### Streaming variant (kind:10012)

```
DW B → kind:10012 (chunk, done=false) ×N
DW B → kind:10011 (final)
DW A → AsyncIterator receives all chunks
```

---

### 5.13 Multi-Relay Federation (v0.2)

**Feature flag:** `NODUS_FEDERATION_V2`  
**Implementation:** `relay_federation.py` in `nodus-adk-runtime`

Multi-relay federation enables DWs from different tenants (companies) to collaborate, each staying on their own private relay.

#### Discovery mechanism

The `relay_hint` tag on kind:34001 teaches any observer about a tenant's relay address:

```json
{
  "kind": 34001,
  "pubkey": "<owner_hex>",
  "tags": [
    ["d", "<owner_hex>-<external_dw_hex>"],
    ["p", "<external_dw_pubkey_hex>"],
    ["tenant", "tenant-b"],
    ["relay_hint", "wss://relay.tenant-b.example"],
    ["federation_scope", "delegate"]
  ],
  "content": "..."
}
```

**`federation_scope` values:**
- `read-only` — can subscribe to events at the remote relay
- `delegate` — can send A2A requests to DWs at the remote relay
- `full` — bidirectional, including HITL cross-tenant

#### Discovery flow

```
DW Tenant A                     Local Relay A
     │  REQ {kinds:[34001], limit:50}
     ├───────────────────────────────►│
     │                                │  returns all 34001 events
     │◄───────────────────────────────┤  (including ones with relay_hint)
     │                                │
     │  self._known_relays = {
     │    "tenant-b": "wss://relay.tenant-b.example",
     │    "tenant-c": "wss://relay.tenant-c.example"
     │  }
```

The `RelayFederation` component maintains a map of known federated relays and provides `publish_to_federation(event, tenant)` and `subscribe_from_federation(filter, tenant)` APIs.

---

### 5.14 Public DW Marketplace (v0.2)

**Feature flag:** `NODUS_MARKETPLACE_V2`  
**Implementation:** `nostr-marketplace-service.ts` in `nodus-backoffice`

The marketplace makes DWs publicly discoverable while keeping their operational data (mandates, audit logs, sessions) on the private relay.

#### Architecture

```
Private relay (operations)          Public relay (discovery)
──────────────────────────          ────────────────────────
kind:34002 mandate (private)        kind:34000 DW profile (public, t=nodus-dw)
kind:34003 audit (private)          kind:31990 NIP-89 Agent Card (public)
kind:10001-10021 sessions (private) kind:34010 KYC Claim (public)
```

#### Public DW profile (kind:34000 with `t` tag)

When `nostr_marketplace_opt_in` is enabled for a tenant, the DW profile is published to the public relay with an additional `["t", "nodus-dw"]` tag:

```json
{
  "kind": 34000,
  "tags": [
    ["d", "<dw_pubkey_hex>"],
    ["p", "<dw_pubkey_hex>"],
    ["t", "nodus-dw"]
  ],
  "content": "{\"name\":\"Athena\",\"about\":\"Root orchestrator\",\"nodus_version\":\"0.2\",\"dw_type\":\"adk\",\"capabilities\":[\"orchestrate\",\"email\"],\"limits\":[]}"
}
```

Any client subscribed to the public relay can discover all Nodus DWs by querying:
```json
{"kinds": [34000], "#t": ["nodus-dw"]}
```

#### kind:34010 — KYC Corp Claim

Links a legal entity to its cryptographic identity. Published to the public relay.

```json
{
  "kind": 34010,
  "pubkey": "<owner_pubkey_hex>",
  "tags": [
    ["d", "kyc-<tenant_id>"],
    ["t", "nodus-kyc"],
    ["legal_entity", "Nodus Factory SL"],
    ["jurisdiction", "ES"],
    ["registration", "B12345678"],
    ["p", "<verifier_pubkey_hex>", "", "verifier"]
  ],
  "content": "{\"legal_entity\":\"Nodus Factory SL\",\"jurisdiction\":\"ES\",\"registration\":\"B12345678\",\"tenant_id\":\"1\",\"nodus_version\":\"0.2\"}"
}
```

Any third party can verify a DW's legal identity without contacting Nodus by fetching the owner's kind:34010 from the public relay.

---

### 5.15 Cross-Enterprise HITL (v0.2)

**Feature flag:** `NODUS_CROSS_TENANT_HITL_V2`  
**Implementation:** `cross_tenant_hitl.py` in `nodus-adk-runtime`

Cross-Enterprise HITL allows a human from a client company to approve DW actions using their own keypair and their own relay — without needing a Nodus account.

#### Key design properties

- **No central dependency:** the human uses their own app, their own keypair, their own relay
- **Cryptographic validation:** the human's kind:10004 must be signed by a pubkey that appears in a cross-tenant kind:34001 at the provider's relay
- **No trust required:** the DW does not need to trust Nodus — it only trusts cryptographic signatures

#### Full flow (6 steps)

1. **Mandate defines scope** — the mandate (kind:34002) specifies which actions require cross-tenant HITL
2. **DW publishes HITL request** — kind:10003 published to the provider's relay
3. **Bridge copies to client relay** — `CrossTenantHitlBridge` publishes kind:10003 to the client's relay (discovered via `relay_hint` on kind:34001)
4. **Client human sees request** — the human sees kind:10003 in their app on their relay
5. **Human signs response** — signs kind:10004 with their own keypair, published to their relay
6. **DW validates and continues** — DW subscribes to client relay, receives kind:10004, validates that `event.pubkey` appears in a cross-tenant kind:34001 at the provider relay

#### Authorisation validation

```python
async def _is_authorized_human(self, pubkey_hex: str) -> bool:
    # REQ {kinds:[34001], "#p":[pubkey_hex], "limit":5}
    # Returns True if any 34001 event at the local relay mentions this pubkey
    # This proves the cross-tenant org-relation was established by the owner
```

**Security:** if the responder's pubkey does not appear in any kind:34001, the response is discarded. No cryptographic forgery can bypass this — the kind:34001 is signed by the owner and stored at the provider's relay.

---

### 5.16 Verifiable Employment Contracts (v0.2)

**Feature flag:** `NODUS_VERIFIABLE_CONTRACTS_V2`  
**Implementation:** `nostr-contract-service.ts` in `nodus-backoffice`

A verifiable employment contract binds a DW, its owner, and a legal entity into a single cryptographically verifiable document. Any third party — auditor, regulator, partner — can verify the contract without contacting Nodus.

#### Contract components

| Component | Kind | Role |
|-----------|------|------|
| Mandate | kind:34002 | Defines DW capabilities and limits |
| Org-relation | kind:34001 | Links owner to DW |
| KYC claim | kind:34010 | Links owner to legal entity |
| Contract event | kind:34002 (contract variant) | Hash binding all three |

#### Contract hash

```typescript
const contractHashInput = [
  mandateEventId,
  orgRelationEventId,
  kycClaimEventId ?? ""
].join(":");
const contractHash = crypto.createHash("sha256").update(contractHashInput).digest("hex");
```

#### Contract event (kind:34002 with `t=nodus-contract`)

```json
{
  "kind": 34002,
  "pubkey": "<owner_pubkey_hex>",
  "tags": [
    ["d", "<contract_hash>"],
    ["p", "<dw_pubkey_hex>"],
    ["e", "<mandate_event_id>", "", "mandate"],
    ["e", "<org_relation_event_id>", "", "org-relation"],
    ["e", "<kyc_claim_event_id>", "", "kyc-claim"],
    ["t", "nodus-contract"],
    ["expiration", "<unix_ts_optional>"]
  ],
  "content": "{\"contract_hash\":\"<hash>\",\"dw\":\"<dw_pubkey_hex>\",\"legal_entity\":\"Nodus Factory SL\",\"nodus_version\":\"0.2\"}"
}
```

#### Third-party verification (no trust in Nodus required)

1. Fetch kind:34002 `#d=<contract_hash>` from the relay
2. Verify the three `e` tag events exist and are correctly signed
3. Recompute: `sha256(mandate_id + ":" + org_relation_id + ":" + kyc_claim_id)` — must match `d` tag
4. Verify BIP-340 Schnorr signature on each referenced event
5. Check `expiration` tag if present

**Public verification endpoint:** `GET /api/contracts/verify/:hash`

#### Human-readable contract statement example

> *"Nodus Factory SL (reg. B12345678, ES) has authorised Digital Worker `npub1abc...xyz` (Athena) to perform [send_email, read_calendar, orchestrate] on behalf of Client SL, subject to HITL approval for all financial actions and email sends. Effective 2026-04-01. No expiry. Contract hash: `sha256:abcdef...`"*

Any observer with the contract hash and access to the relay can independently verify this statement.

---

## 6. Reference Implementation

**Nodus OS** is the first certified implementation of the Nodus Protocol. It implements all milestones M0–M13 additively behind feature flags.

**Reference stack:**
- Agent runtime: Google ADK + FastAPI
- Multi-step workflows: LangGraph
- Memory and knowledge: vector store (Qdrant) + graph (PostgreSQL)
- Private Nostr relay: strfry + writePolicy hooks
- Policy Relay: custom WebSocket server (NIP-46 variant)
- Deployment: Kubernetes (GitOps via ArgoCD)

### 6.1 Reference Implementation Status

| Milestone | Description | Status | Primary repo |
|-----------|-------------|--------|-------------|
| M0 | DW & Human Identities (kind:34000 + kind:34001) | ✅ Implemented | nodus-adk-runtime, nodus-llibreta-v2 |
| M1 | Mandates (kind:34002, immutable) | ✅ Implemented | nodus-backoffice, nodus-adk-runtime |
| M2 | Audit Log (kind:34003, append-only) | ✅ Implemented | nodus-adk-runtime |
| M3 | NIP-26 Delegation (authority proof per event) | ✅ Implemented | nodus-adk-runtime |
| M4 | Constitutional HITL (kind:10003/10004, NIP-07 + custodial) | ✅ Implemented | nodus-adk-runtime, nodus-llibreta-v2 |
| M5 | Emergency Stop/Resume (kind:34005/34006, <30s halt) | ✅ Implemented | nodus-backoffice, nodus-adk-runtime |
| M6 | Room UX + Async HITL Inbox (kind:10020/10021) | ✅ Implemented | nodus-llibreta-v2 |
| M7 | MCP Governance (kind:34004, DW verifies gateway) | ✅ Implemented | nodus-mcp-gateway, nodus-adk-runtime |
| M8 | Policy Relay (NIP-46 variant, nsec never leaves relay) | ✅ Implemented | nodus-adk-runtime |
| M9 | A2A Nostr-Native (kinds 10010–10013) | ✅ Implemented (v0.2) | nodus-adk-runtime |
| M10 | Multi-Relay Federation (relay_hint discovery) | ✅ Implemented (v0.2) | nodus-adk-runtime |
| M11 | Public DW Marketplace (kind:34000 + kind:34010 public) | ✅ Implemented (v0.2) | nodus-backoffice |
| M12 | Cross-Enterprise HITL (cross-tenant approval) | ✅ Implemented (v0.2) | nodus-adk-runtime, nodus-llibreta-v2 |
| M13 | Verifiable Employment Contracts (contract hash) | ✅ Implemented (v0.2) | nodus-backoffice |

**All flags default to `false`.** The v1 HTTP system is never touched. Enable incrementally.

### 6.2 Activation Flags

All protocol feature flags reside in `nodus-adk-runtime/src/nodus_adk_runtime/config/feature_flags.py`. Activation is via environment variable with the same name.

#### Recommended activation order

**Phase 1 — Base identities**
```bash
NODUS_PROTOCOL_IDENTITY_V1=true
NOSTR_ADK_TRANSPORT_V1=true
NOSTR_RELAY_URL=ws://nostr-relay:7777
NOSTR_AGENT_NSEC=nsec1...
```
*Verify:* DW profile appears on relay. Transport subscribes to kind:10001.

**Phase 2 — Mandates**
```bash
NODUS_PROTOCOL_MANDATES_V1=true
# Publish kind:34002 via Backoffice → POST /api/workers/:id/mandate
```
*Verify:* Logs show `mandate check permitted=True`.

**Phase 3 — Audit**
```bash
NODUS_PROTOCOL_AUDIT_V1=true
```
*Verify:* Logs show `audit: logged agent_response`.

**Phase 4 — NIP-26 Delegation**
```bash
NODUS_PROTOCOL_DELEGATION_V1=true
NOSTR_OWNER_NSEC=nsec1...
```
*Verify:* Events carry `delegation` tag. Token creation logged.

**Phase 5 — Constitutional HITL**
```bash
NODUS_PROTOCOL_CONSTITUTIONAL_HITL_V1=true
```
*Verify:* HITL requests appear as kind:10003. kind:10004 signed by user.

**Phase 6 — Emergency Controls**
```bash
NODUS_PROTOCOL_EMERGENCY_V1=true
NODUS_TENANT_ID=1
```
*Verify:* Panic button publishes kind:34005. DW detects in <30s.

**Phase 7 — Room UX**
```bash
ROOMS_UX_V2=true
```
*Verify:* Route `/room/:sessionId` accessible.

**Phase 8 — Policy Relay**
```bash
NODUS_POLICY_RELAY_V1=true
NODUS_POLICY_RELAY_URL=ws://policy-relay:8080/sign
# Move NOSTR_AGENT_NSEC from DW pod to Policy Relay (NODUS_DW_NSEC_MAP)
```
*Verify:* DW has no NSEC. Logs show `Policy Relay mode active`.

**Phase 9 — v0.2 features**
```bash
NODUS_A2A_NOSTR_V2=true          # M9: DW-to-DW via Nostr
NODUS_FEDERATION_V2=true         # M10: Cross-tenant relay discovery
NODUS_CROSS_TENANT_HITL_V2=true  # M12: HITL from external company
# M11/M13: via Backoffice UI (no flag in feature_flags.py)
```

#### Complete flag reference

| Flag | Milestone | Default | Description |
|------|-----------|---------|-------------|
| `NOSTR_ADK_TRANSPORT_V1` | M0 | `false` | Activates NostrAdkTransport — DW listens on kind:10001 |
| `NODUS_PROTOCOL_IDENTITY_V1` | M0 | `false` | Publishes kind:34000 DW profile to relay |
| `NODUS_PROTOCOL_MANDATES_V1` | M1 | `false` | DW consults kind:34002 mandate before acting |
| `NODUS_PROTOCOL_AUDIT_V1` | M2 | `false` | DW publishes kind:34003 per action |
| `NODUS_PROTOCOL_DELEGATION_V1` | M3 | `false` | Adds NIP-26 delegation tag to all events |
| `NODUS_PROTOCOL_CONSTITUTIONAL_HITL_V1` | M4 | `false` | Enables kind:10003/10004 constitutional HITL |
| `NODUS_PROTOCOL_EMERGENCY_V1` | M5 | `false` | Activates 30s emergency stop polling |
| `ROOMS_UX_V2` | M6 | `false` | Enables `/room/:sessionId` route in frontend |
| `NODUS_PROTOCOL_MCP_PROFILE_V1` | M7 | `false` | DW verifies kind:34004 before MCP calls |
| `NODUS_POLICY_RELAY_V1` | M8 | `false` | DW delegates all signing to Policy Relay |
| `NODUS_A2A_NOSTR_V2` | M9 | `false` | Enables kinds 10010–10013 A2A |
| `NODUS_FEDERATION_V2` | M10 | `false` | Enables cross-tenant relay discovery |
| `NODUS_CROSS_TENANT_HITL_V2` | M12 | `false` | Enables cross-tenant HITL bridge |

---

## 7. Conformance and Certification

An implementation is compatible with Nodus Protocol v0.2 if it meets:

### Minimum conformance checklist

**Identity (M0):**
- [ ] Each DW has a unique, securely generated keypair (npub/nsec)
- [ ] Each DW has a kind:34000 profile published on the relay
- [ ] The `entity_type` field is present and correct
- [ ] DW nsec keys are custodial (never leave the server, or reside in Policy Relay)

**Mandates (M1):**
- [ ] Every DW action references a valid, active kind:34002 mandate
- [ ] The relay rejects actions from DWs without a valid mandate
- [ ] The relay refuses DELETE and UPDATE on kinds 34002 and 34003

**Delegation (M3):**
- [ ] DW actions carry NIP-26 delegation proof from the owner
- [ ] The relay verifies delegation proofs

**Audit (M2):**
- [ ] Every significant action generates a kind:34003 event
- [ ] The audit log is append-only
- [ ] The relay refuses DELETE and UPDATE on kind:34003

**Constitutional HITL (M4):**
- [ ] Actions marked `hitl_required` in the mandate require a kind:10004 before execution
- [ ] kind:10004 is signed by the human's keypair (NIP-07 or custodial)

**Human/DW Separation:**
- [ ] The relay rejects kind:34002, 34005 events from `digital_worker` entities
- [ ] No DW can modify the limits of its own authority

**Revocation (M5):**
- [ ] kind:34005 stops all tenant DWs in <30 seconds
- [ ] Individual mandate revocation is effective in <10 seconds

**v0.2 additions (M9–M13):**
- [ ] A2A Nostr-Native (if implemented): kinds 10010–10013 with correct tag structure
- [ ] Cross-tenant federation (if implemented): kind:34001 `relay_hint` tag respected
- [ ] Verifiable contracts (if implemented): `contract_hash = sha256(mandate+org_relation+kyc)` verifiable

### Certification process

*(To be defined — including corporate KYC via kind:34010, audit by Nodus operator, and registration on the public Nodus Relay)*

---

## 8. Appendix

### A. Glossary

| Term | Definition |
|------|-----------|
| **A2A** | Agent-to-Agent. Direct communication between Digital Workers. |
| **A2A Nostr-Native** | v0.2 A2A transport using kinds 10010–10013 via relay, no HTTP server. |
| **ACP** | Agent Communication Protocol. Persistent stateful sessions between agents. |
| **Audit log** | Immutable record of all significant DW actions. Kind 34003. |
| **Constitutional HITL** | Human approval that produces a cryptographic signature (kind:10004). |
| **Cross-Enterprise HITL** | HITL approval from a human at a different company using their own keypair. |
| **Delegation** | NIP-26 mechanism by which a human authorizes a DW to act on their behalf. |
| **DW** | Digital Worker. AI agent with identity, mandate, and defined limits. |
| **Emergency-stop** | Kind 34005. Immediately stops all DWs in a tenant in <30 seconds. |
| **Federation** | Multi-relay architecture enabling DWs at different tenants to collaborate. |
| **Feature flag** | Environment variable controlling protocol feature activation. All default to `false`. |
| **HITL** | Human-in-the-Loop. Human intervention in an agent workflow. |
| **Corporate KYC** | Know Your Customer applied to companies. Kind 34010. |
| **Mandate** | Signed document defining what a DW can and cannot do. Kind 34002. |
| **MCP** | Model Context Protocol. Standard for connecting agents to external tools. |
| **npub** | Nostr public key. Public identifier of an entity. |
| **nsec** | Nostr private key. Never shared. For DWs, lives custodially at the relay or Policy Relay. |
| **NIP** | Nostr Implementation Proposal. Specification of Nostr protocol functionality. |
| **Nostr** | Decentralized protocol based on keypairs and relays. Foundation of the Nodus Protocol governance layer. |
| **Policy Relay** | Extended Nostr relay acting as a Signing Service with mandate enforcement. DW nsec never leaves it. |
| **relay_hint** | Tag on kind:34001 advertising another tenant's relay address for federation. |
| **Relay** | Nostr server that distributes events. Nodus uses private relays (per tenant) and a public relay (marketplace). |
| **Tenant** | Company or organization operating DWs under the Nodus Protocol. |
| **Verifiable contract** | kind:34002 event whose `d` tag is `sha256(mandate+org_relation+kyc)`, verifiable by any third party. |

### B. Referenced Nostr NIPs

| NIP | Title | URL |
|-----|-------|-----|
| NIP-01 | Basic protocol flow | https://github.com/nostr-protocol/nostr/blob/master/01.md |
| NIP-16 | Event Treatment | https://github.com/nostr-protocol/nostr/blob/master/16.md |
| NIP-19 | bech32-encoded entities | https://github.com/nostr-protocol/nostr/blob/master/19.md |
| NIP-26 | Delegated event signing | https://github.com/nostr-protocol/nostr/blob/master/26.md |
| NIP-29 | Relay-based Groups | https://github.com/nostr-protocol/nostr/blob/master/29.md |
| NIP-33 | Parameterized Replaceable Events | https://github.com/nostr-protocol/nostr/blob/master/33.md |
| NIP-42 | Authentication of clients | https://github.com/nostr-protocol/nostr/blob/master/42.md |
| NIP-44 | Versioned Encryption | https://github.com/nostr-protocol/nostr/blob/master/44.md |
| NIP-46 | Nostr Connect (remote signing) | https://github.com/nostr-protocol/nostr/blob/master/46.md |
| NIP-59 | Gift Wrap | https://github.com/nostr-protocol/nostr/blob/master/59.md |
| NIP-89 | Recommended Application Handlers | https://github.com/nostr-protocol/nostr/blob/master/89.md |
| NIP-90 | Data Vending Machines | https://github.com/nostr-protocol/nostr/blob/master/90.md |

### C. Referenced External Protocols

| Protocol | Version | URL |
|----------|---------|-----|
| Google A2A | v0.3.0 | https://google.github.io/A2A |
| MCP (Model Context Protocol) | 2024-11 | https://modelcontextprotocol.io |

### D. Changelog

| Version | Date | Description |
|---------|------|-------------|
| v0.1 | March 2026 | First internal draft. 4 layers, kinds 34000–34010, graduated governance, conceptual Policy Relay. |
| **v0.2** | **April 2026** | **Release Candidate. Reference implementation available (M0–M13). A2A Nostr-native, multi-relay federation, public marketplace, cross-enterprise HITL, verifiable contracts. KINDS.md and FLOWS.md added.** |

---

*Nodus Factory · © 2026 · CC BY 4.0*
