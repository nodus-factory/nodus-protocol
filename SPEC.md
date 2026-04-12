# Nodus Protocol

> **Status:** First Public Release  
> **Authors:** Nodus Protocol Working Group  
> **Contact:** protocol@nodus.social  
> **Date:** April 2026  
> **License:** Creative Commons Attribution 4.0 International (CC BY 4.0)  
> **Canonical URL:** https://nodus.social/protocol

---

## Abstract

The Nodus Protocol defines a minimal set of standards for the **identity, delegation, governance, verifiable human intervention, and audit of Digital Workers** in enterprise and civic environments.

The protocol is **transport-agnostic**: it does not impose a single communication channel between agents, but rather a common layer of action legitimacy. A Digital Worker can communicate over A2A, ACP, HTTP, queues, or Nostr — but when it acts, the Protocol ensures that anyone can answer:

*Who performed this action? Did they have the authority to do so? Who authorised them? When? On what basis? And how can it be stopped?*

The Nodus Protocol is not a platform. It is not a product. It is the minimum infrastructure needed for a digital workforce to be governed, audited, and trusted — just as today's human workforce is.

---

## Table of Contents

1. [The Problem](#1-the-problem)
2. [Motivation](#2-motivation)
3. [Design Principles](#3-design-principles)
4. [Core Concepts](#4-core-concepts)
5. [Technical Specification](#5-technical-specification)
   - 5.1 Nostr NIPs as Governance Foundation
   - 5.2 Nodus Protocol Kinds
   - 5.3 Synchronous A2A Transport
   - 5.4 Persistent ACP Sessions
   - 5.5 MCP under Protocol Governance
   - 5.6 Policy Relay
   - 5.7 Enterprise Control over Digital Workers
   - 5.8 Panic Button and Revocation
   - 5.9 Constitutional Separation: Human / Digital Worker
   - 5.10 Graduated Governance Model
   - 5.11 Federation and Cross-Enterprise Communication
   - 5.12 A2A Nostr-Native
   - 5.13 Multi-Relay Federation
   - 5.14 Public DW Discovery
   - 5.15 Cross-Enterprise HITL
   - 5.16 Verifiable Employment Contracts
6. [Conformance and Certification](#6-conformance-and-certification)
7. [Certified Implementations](#7-certified-implementations)
8. [Appendix](#8-appendix)

---

## 1. The Problem

Companies are incorporating hundreds — and soon thousands — of artificial intelligence agents into their daily operations. These agents will make decisions, execute actions, access sensitive data, sign documents, manage money, hire suppliers, and interact with customers.

Some of these agents will have been created by the company itself. Others by a partner. Others contracted as a service. Some will be built on GPT. Others on Claude. Others on local models. Some will communicate with each other. Others will operate alone.

And no one — no executive, no auditor, no regulator — will be able to answer the most basic questions:

*Who made this decision? Did they have the authority to do so? Who authorised them? When? On what basis?*

This is already happening today, at small scale. In two years, it will be the central problem of corporate governance.

The Nodus Protocol is the answer to this problem.

---

## 2. Motivation

Between 2019 and 2026, the [Democracy4All](https://www.democracy4all.barcelona/) conference series brought together technologists, policymakers, and researchers in Barcelona to address a recurring question: *how can digital systems be governed in a way that is transparent, accountable, and verifiable by design?*

Over seven editions, three convergences became clear:

**Convergence 1 — Identity is governance.**
You cannot hold an actor accountable without a verifiable, independent identity that survives the failure of any platform.

**Convergence 2 — Immutability enables trust.**
A governance layer that can be modified or deleted after the fact offers no real guarantee. Accountability requires append-only records.

**Convergence 3 — Authority must be cryptographic, not contractual.**
Contracts exist on paper. Cryptographic delegation exists in mathematics. The former can be disputed; the latter can only be verified.

The Nodus Protocol is the formal articulation of these three principles, applied to the emerging governance challenge of autonomous AI agents in enterprise and civic environments.

> *"You cannot govern what you cannot identify."*

See [MOTIVATION.md](MOTIVATION.md) for the full origin story.

---

## 3. Design Principles

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

These principles are non-negotiable. An implementation that violates them is not conformant with the Nodus Protocol.

### P1 — Clear and Verifiable Identity

Every Digital Worker operating under the Nodus Protocol MUST have a unique cryptographic identity, independent of any central platform. Identity MUST be verifiable without consulting any third-party server.

### P2 — Native Interoperability

No implementation MAY create closed silos. Any certified Digital Worker MUST be capable of communicating with any other certified DW, and with any authorised human. The protocol defines the common language; no implementation MAY speak an incompatible dialect.

### P3 — Sufficient Decentralisation

The protocol MUST NOT depend on a central server, a single organisation, or a single jurisdiction. It MUST be capable of surviving the failure of any node. No single entity MAY have the power to shut down the network.

### P4 — Immutable Auditability

Every significant action by a Digital Worker MUST be recorded in a way that cannot be modified or deleted. The audit log is append-only; no entity MAY erase the trace of a past action.

### P5 — Selective Governed Transparency

The protocol MUST allow an organisation to internally audit all actions of its Digital Workers, without those actions being visible to unauthorised third parties. At the same time, it MUST ensure that certain facts — that an action occurred, that an agent had authority to perform it — can be verified externally without revealing the content.

### P6 — Ontological Separation: Human / Digital Worker

A human and a Digital Worker are not the same type of entity. A DW MUST NOT escalate its privileges to the level of a human. A human MUST NOT be impersonated by a DW. This separation is structural and MUST NOT be configurable.

---

## 4. Core Concepts

### 4.1 Digital Worker (DW)

A Digital Worker is an artificial intelligence agent that acts on behalf of an organisation or person, with an **explicit mandate and defined limits**.

A DW is not a script. It is not a chatbot. It is an entity with:
- Its own verifiable identity
- A clear human or organisational owner
- A declared set of capabilities
- A set of limits it cannot exceed
- An auditable history of actions

The fundamental difference between a DW and a generic AI agent is **traceable accountability**: it is always known who created the DW, who authorised it to act, and what it has done.

### 4.2 Cryptographic Identity

Each entity operating under the Nodus Protocol — human or digital — has a cryptographic identity based on a key pair:

- **`npub`** (public key) — public identifier, visible to everyone on the network
- **`nsec`** (private key) — never shared, controlled exclusively by the owner

This identity is **self-sovereign**: it does not depend on any central server. A DW retains a verifiable identity even if its operator ceases to exist.

> Technical basis: NIP-01 of the Nostr protocol. All events use BIP-340 Schnorr signatures over secp256k1.

### 4.3 Owner

Every Digital Worker MUST have an owner: the person or organisation that created it and is responsible for it. The owner–DW relationship MUST be cryptographically verifiable.

An owner can:
- Create and revoke mandates
- Delegate authority to the DW
- Audit all DW actions
- Transfer ownership to a third party
- Stop the DW immediately (panic button)

### 4.4 Mandate (kind 34002)

The mandate is the document that defines **what a Digital Worker can and cannot do**. It is the contract between the owner and the DW. It includes:

- Authorised capabilities (which actions it may execute)
- Explicit limits (what it may never do)
- Actions requiring live human approval (HITL-required)
- Auto-approved actions
- Scope (over which data and systems it may act)
- Validity period (from when to when it is valid)

The mandate is **immutable once signed** by the owner. If it needs to change, a new version MUST be signed. Relays MUST refuse DELETE and UPDATE on kind:34002.

### 4.5 Delegation (NIP-26)

Delegation is the mechanism by which an authorised human transfers authority to a Digital Worker to act on their behalf. When a DW acts with delegation:

- Its action carries the DW's signature (technical identity)
- And the cryptographic proof that the owner authorised this action (delegated authority)

Any third party can verify both without consulting any central server.

### 4.6 Agent-to-Agent Interaction (A2A)

Digital Workers communicate with each other. The Nodus Protocol supports two primary A2A patterns:

- **Synchronous A2A** — when an immediate response is needed
- **A2A Nostr-Native** (kinds 10010–10013) — direct DW-to-DW delegation via the relay, no HTTP intermediary

The protocol does not impose a single transport. What it imposes is that every governed interaction can demonstrate: who is acting, with what authority, under what mandate, and what trace it leaves.

### 4.7 Human-in-the-Loop (HITL)

Certain actions require real human intervention. The Nodus Protocol distinguishes between:

- **Operational HITL:** the human receives a notification and approves via any surface
- **Constitutional HITL:** when the human exercises authority, this act is materialised on the Nostr layer as a verifiable cryptographic proof (kind:10004, signed by the human's keypair)
- **Cross-Enterprise HITL:** a human from another organisation can approve DW actions using their own keypair and relay

> Ordinary human↔agent conversation may pass through any channel. But when a human exercises authority, the Protocol records it permanently.

### 4.8 Audit Log (kind 34003)

The audit log is the immutable record of all significant actions by a Digital Worker. Each entry contains:

- Who performed the action (DW identity)
- With what authority (reference to mandate and delegation)
- When (verifiable timestamp)
- What was done (hash of the action and its result)
- In the context of which interaction (session and mandate references)

The audit log is **append-only**: no entry can be modified or deleted. Not by the DW's owner. Not by anyone.

---

## 5. Technical Specification

The Nodus Protocol is implemented over four complementary communication layers:

| Layer | Protocol | Role | When to use |
|-------|----------|------|-------------|
| **Cryptographic governance** | Nostr (NIPs 01/16/26/33/42/44/46/59/89) | Identity, delegation, revocation, immutable audit, constitutional HITL | When legitimising, revoking, auditing, or involving a human with verifiable authority |
| **A2A Nostr-native** | Nostr kinds 10010–10013 | DW↔DW direct delegation via relay | Cross-DW coordination without HTTP server dependency |
| **Synchronous A2A** | JSON-RPC/HTTP | Direct agent→agent synchronous calls | DW executes atomic task with blocking response |
| **Persistent sessions** | ACP | Stateful sessions, streaming, long context, multi-turn orchestration | Streaming between agents, long conversational sessions |
| **Agent↔Tools** | MCP | Access to tools, APIs, databases, external systems | DW accesses calendar, CRM, email, files, APIs |

**The Nodus Protocol is transport-agnostic but rooted in Nostr for governance.** Agents may communicate over many channels, but when an action requires protocol legitimacy, that legitimacy lives in the Nostr cryptographic layer.

---

### 5.1 Nostr NIPs as Governance Foundation

The protocol requires support for the following NIPs:

| NIP | Name | Role in Nodus |
|-----|------|--------------|
| NIP-01 | Basic protocol | Foundation. Event signing, basic kinds. Mandatory. |
| NIP-16 | Event Treatment | Kinds 10000–19999 replaceable semantics |
| NIP-19 | bech32-encoded entities | `nsec1…`, `npub1…` for all keys |
| NIP-26 | Delegated event signing | DWs sign on behalf of the owner. Verifiable authority proofs. |
| NIP-29 | Relay-based Groups | Collaborative work rooms |
| NIP-33 | Parameterized Replaceable Events | All kinds 34000–34010 governance layer |
| NIP-42 | Relay AUTH | Private relay per organisation. Only authenticated identities connect. |
| NIP-44 | Versioned Encryption | Encrypted P2P communication between agents |
| NIP-46 | Nostr Connect / Remote Signing | Technical basis for the Policy Relay |
| NIP-59 | Gift Wrap | Maximum privacy for sensitive HITL decisions |
| NIP-89 | App Handler Info | DW discovery on the relay |
| NIP-90 | Data Vending Machines | Async A2A decoupled in time |

---

### 5.2 Nodus Protocol Kinds

The Nodus Protocol defines two kind ranges. The complete reference is in [KINDS.md](KINDS.md).

#### Session Layer (kinds 10001–10021)

NIP-16 replaceable events. The DW MUST use a `since` filter to avoid processing stale messages.

| Kind | Name | Publisher | Description |
|------|------|-----------|-------------|
| `10001` | `MESSAGE_USER` | Human | User message to a DW |
| `10002` | `RESPONSE_DW` | DW | Final DW response |
| `10003` | `HITL_REQUEST` | DW | Human approval request |
| `10004` | `HITL_RESPONSE` | Human | Cryptographically signed human decision |
| `10005` | `RESPONSE_AGENT` | Agent | Agent-to-agent (synchronous transport) |
| `10006` | `STREAMING_CHUNK` | DW | Real-time streaming chunk |
| `10010` | `A2A_REQUEST` | DW | DW-to-DW task request (Nostr-native) |
| `10011` | `A2A_RESPONSE` | DW | DW-to-DW task response |
| `10012` | `A2A_STREAM` | DW | DW-to-DW streaming chunk |
| `10013` | `A2A_ERROR` | DW | DW-to-DW error |
| `10020` | `INBOX_ITEM` | DW | Async HITL request (inbox) |
| `10021` | `INBOX_RESOLVED` | Human | Async HITL resolution |

#### Governance Layer (kinds 34000–34010)

NIP-33 parameterized replaceable events. The `["d", "<identifier>"]` tag defines uniqueness within pubkey+kind.

| Kind | Name | `d` tag value | Mutability |
|------|------|---------------|------------|
| `34000` | `nodus:dw-profile` | `<dw_pubkey_hex>` | Replaceable |
| `34001` | `nodus:org-relation` | `<owner_hex>-<dw_hex>` | Replaceable |
| `34002` | `nodus:policy` | `<dw_pubkey_hex>` | **IMMUTABLE** |
| `34003` | `nodus:audit-event` | `sha256(dw+session+ts_ms)` | **Append-only** |
| `34004` | `nodus:mcp-server-profile` | `<gateway_identifier>` | Replaceable |
| `34005` | `nodus:emergency-stop` | `<tenant_id>` | Immutable |
| `34006` | `nodus:emergency-resume` | `<tenant_id>` | Immutable |
| `34010` | `nodus:kyc-corp-claim` | `"kyc-<tenant_id>"` | Replaceable |

> **Critical relay rules:**
> 1. Relays MUST reject DELETE or UPDATE on kind:34002 and kind:34003
> 2. Entities with `entity_type: "digital_worker"` MUST NOT be permitted to sign kinds 34002, 34005
> 3. kind:34003 `d` tag uniqueness enforces append-only semantics

#### Kind 34000 — Digital Worker Profile

```json
{
  "kind": 34000,
  "pubkey": "<dw_pubkey_hex>",
  "tags": [
    ["d", "<dw_pubkey_hex>"],
    ["p", "<owner_pubkey_hex>"]
  ],
  "content": "{\"name\":\"Athena\",\"description\":\"Root orchestrator agent\",\"owner\":\"<owner_hex>\",\"entity_type\":\"digital_worker\",\"capabilities\":[\"orchestrate\",\"email\",\"calendar\"],\"limits\":[\"no_financial_without_hitl\"],\"transports\":[{\"type\":\"nostr-session\",\"relay\":\"ws://relay.example\",\"kinds\":[10001]},{\"type\":\"a2a\",\"url\":\"https://agent.example/a2a\"}],\"nodus_version\":\"1.0\"}"
}
```

#### Kind 34002 — Policy / Mandate

```json
{
  "kind": 34002,
  "pubkey": "<owner_pubkey_hex>",
  "tags": [
    ["d", "<dw_pubkey_hex>"],
    ["p", "<dw_pubkey_hex>"]
  ],
  "content": "{\"dw\":\"<dw_pubkey_hex>\",\"capabilities\":[\"send_email\",\"read_calendar\",\"orchestrate\"],\"limits\":[\"no_delete_without_confirmation\"],\"hitl_required\":[\"send_email\",\"delete_*\",\"financial_*\"],\"auto_approve\":[\"read_calendar\"],\"max_auto_cost_eur\":0,\"valid_from\":1714000000,\"valid_until\":null,\"nodus_version\":\"1.0\"}"
}
```

#### Kind 34003 — Audit Event

```json
{
  "kind": 34003,
  "pubkey": "<dw_pubkey_hex>",
  "tags": [
    ["d", "<sha256_of_dw+session+ts_ms>"],
    ["p", "<user_pubkey_hex>"],
    ["mandate", "<kind:34002_event_id>"],
    ["session", "<session_uuid>"],
    ["action", "agent_response"]
  ],
  "content": "{\"action\":\"agent_response\",\"result_hash\":\"<sha256>\",\"timestamp\":1714000010,\"dw\":\"<dw_pubkey_hex>\",\"session_id\":\"<uuid>\",\"mandate_ref\":\"<event_id>\"}"
}
```

---

### 5.3 Synchronous A2A Transport

The protocol supports direct agent-to-agent communication over JSON-RPC/HTTP, compatible with the emerging A2A standard (supported by Salesforce, SAP, Atlassian, PayPal, and 50+ organisations). The Nodus Protocol adds the governance layer this standard lacks.

**The gap Nodus fills over plain A2A:**

| Plain A2A | + Nodus Protocol |
|-----------|-----------------|
| Agent Cards | Agent Cards + npub + signed mandate (34002) |
| Task lifecycle | Task lifecycle + immutable audit (34003) |
| HTTP/JSON-RPC | HTTP/JSON-RPC + cryptographic signature |
| Auth: API keys | Auth: NIP-26 cryptographic delegation |
| No revocation | Revocation in seconds (kind 34002 revoked) |

The Nodus Protocol also introduces A2A Nostr-Native (section 5.12) as an alternative that eliminates HTTP server intermediaries and enables cross-tenant federation.

---

### 5.4 Persistent ACP Sessions

The Nodus Protocol supports **ACP (Agent Communication Protocol)** for multi-turn orchestration: an orchestrating agent collaborating in a long conversation with subordinate agents, maintaining context across multiple interactions.

#### When to use ACP

- An orchestrating DW coordinating multiple DWs in a complex flow
- A DW maintaining a long working session with a specialised agent
- Workflows requiring shared context between steps
- Streaming partial responses between agents while work is in progress

#### Relationship between layers

```
Orchestrator DW (ACP session)
      │
      ├─→ DW A  (A2A synchronous, direct call)        ← atomic task
      ├─→ DW B  (A2A Nostr-native)                    ← atomic task, no HTTP
      │
      └─→ Sub-orchestrator DW (ACP session)            ← complex multi-step flow
              │
              └─→ Audit → Nostr (kind 34003)           ← immutable record
```

---

### 5.5 MCP under Protocol Governance

#### The problem MCP does not solve

MCP (Model Context Protocol, Anthropic 2024) is the emerging standard for connecting agents with external tools and systems. But MCP has no identity. An MCP Server is a URL. There is no mechanism to know who manages it, whether it is legitimate, or how to revoke access.

> **"An MCP Server without verifiable identity is an attack vector. The Nodus Protocol gives it a passport."**

#### The solution: kind 34004 — MCP Server Profile

MCP Servers operating under the Nodus Protocol MUST have a keypair, a registered profile (kind:34004), and an audit trail.

```json
{
  "kind": 34004,
  "pubkey": "<mcp_gateway_pubkey_hex>",
  "tags": [["d", "<gateway_identifier>"]],
  "content": "{\"name\":\"Example MCP Gateway\",\"url\":\"https://mcp.example.org\",\"tools\":[\"calendar\",\"email\",\"drive\"],\"owner\":\"<owner_pubkey_hex>\",\"authorized_dws\":[\"<dw_pubkey_hex>\"],\"valid_until\":null}"
}
```

#### Flow: DW uses an MCP tool under Nodus governance

```
DW wants to use a tool from an MCP Server
        │
        ├─→ Check relay: does kind 34004 exist for this MCP Server?
        │       ├── No → REJECT (MCP Server not certified)
        │       └── Yes → check: is this DW in authorized_dws?
        │                   ├── No → REJECT (not authorised)
        │                   └── Yes → CONNECT
        │
        ├─→ Execute MCP call
        │
        └─→ Record in kind 34003 (audit)
```

---

### 5.6 Policy Relay

Standard Nostr relays distribute events but do not filter them against business policies. The **Nodus Policy Relay** solves this by turning the relay into an active **Signing Service**.

#### The core principle: DW private keys never leave the relay

```
Standard mode (DW holds its own key):
  DW ──sign──► relay ──► network

Policy Relay mode:
  DW (no key) ──unsigned event──► Policy Relay ──signed event──► relay ──► network
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

| Property | Standard mode | Policy Relay mode |
|----------|---------------|-------------------|
| Key location | DW process | Policy Relay service (isolated) |
| Process compromise | Key exposed | Only unsigned events exposed |
| Mandate enforcement | Observational | Hard enforcement (sign or reject) |
| Audit | kind:34003 per action | kind:34003 per action |

> Technical basis: NIP-46 (Nostr Connect / nsecbunker) — adapted for enterprise DWs with mandate enforcement semantics.

#### `relay_proof` tag — Policy Relay attestation

When the Policy Relay signs a DW event, it MUST append a `relay_proof` tag to the event. This tag is the cryptographic proof that the event passed through the Policy Relay — without it, any observer (or relay write-policy plugin) can detect that a DW event was signed directly, bypassing the custodial key mechanism.

**Tag format:**
```
["relay_proof", "<sha256(event_id_hex + policy_relay_pubkey_hex)>"]
```

Where:
- `event_id_hex` is the lowercase hex SHA-256 of the serialised event (the Nostr event `id` field)
- `policy_relay_pubkey_hex` is the lowercase hex pubkey of the Policy Relay service
- Both are concatenated without separator before hashing

**Verification (any third party):**
1. Extract `relay_proof` tag value from the event
2. Compute `sha256(event.id + policy_relay_pubkey_hex)`
3. Compare — if mismatch or tag absent, the event did not pass through a known Policy Relay

This tag is **REQUIRED on all events signed in Policy Relay mode**. A relay write-policy plugin MAY reject DW events that lack a valid `relay_proof` when operating in enforcement mode (`NODUS_REQUIRE_RELAY_PROOF=true`); in observational mode it SHOULD log but MUST accept.

#### Emergency stop — Relay enforcement

Relays enforcing emergency controls MUST maintain per-tenant emergency state derived from kind:34005 and kind:34006 events. The recommended architecture is a daemon that:

1. Subscribes to `{kinds: [34005, 34006], "#tenant": ["<tenant_id>"]}` on the relay
2. Maintains a durable cache: `{active: bool, since: unix_ts, event_id: string}` per tenant
3. On each new kind:34005 — marks tenant as halted if `event.created_at ≥ current_since`
4. On each new kind:34006 — clears halt if `event.created_at > current_since`
5. Exposes the cache to the write-policy plugin for per-event decisions

The daemon MUST reconnect automatically on relay disconnection. The cache MUST be persisted to disk so that relay restarts do not lose emergency state.

#### Relay write-policy rules

A conformant relay SHOULD implement write policies enforcing governance rules at write time:

- Relays MUST reject DELETE or UPDATE on kind:34002 and kind:34003
- Entities with `entity_type: "digital_worker"` MUST NOT sign kinds 34002, 34005
- Relays SHOULD verify NIP-26 delegation tags in DW events
- If kind:34005 is active for a tenant, the relay MUST reject all DW events for that tenant (except the owner's kind:34006)
- When operating in Policy Relay enforcement mode, the relay SHOULD reject DW events missing a valid `relay_proof` tag

---

### 5.7 Enterprise Control over Digital Workers

An organisation MUST be able to establish that:
- The DW acting **is theirs** and not an impersonator
- The DW **had authorisation** for that specific action
- If someone **compromises the DW's key**, it can be revoked immediately
- An **external DW** (partner/supplier) acts with the correct permissions

#### The 4 control layers

**Layer 1 — Governed creation**
The DW keypair SHOULD be generated and held at the organisation's Policy Relay. Without relay access, the DW cannot act.

**Layer 2 — Signed mandate**
The owner MUST sign kind 34002. The mandate defines what the DW may do. It is immutable and verifiable by anyone.

**Layer 3 — Verifiable delegation**
Every DW event MUST carry NIP-26 proof that the owner authorised it. Any third party can verify without consulting any central server.

**Layer 4 — Immutable audit**
Every significant action MUST produce kind 34003. It cannot be deleted or altered.

#### Cryptographic hierarchy

```
Owner  (entity_type: "human")
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

#### Revocation scenarios

**Compromised DW**
The owner signs a new kind:34002 marking it revoked → relay propagates immediately → **Time: seconds**

**Personnel offboarding**
Revoke DWs associated with that person → revoke NIP-26 delegations they issued

**Total security incident**
The owner publishes kind:34005 → relay rejects ALL DW events for the tenant → **Time: <30 seconds**

**External DW (partner/supplier)**
Revoke the NIP-26 delegation to the external DW → immediate effect, no coordination needed

#### kind:34005 — Emergency Stop

```json
{
  "kind": 34005,
  "pubkey": "<owner_pubkey_hex>",
  "tags": [
    ["d", "<tenant_id>"],
    ["tenant", "<tenant_id>"]
  ],
  "content": "{\"tenant\":\"<tenant_id>\",\"reason\":\"Security incident — precautionary suspension\",\"authorized_by\":\"<owner_npub>\"}"
}
```

DW implementations MUST poll for kind:34005/34006 events at least every 30 seconds. The emergency is active if:

```
latest_stop_at > 0 AND latest_stop_at > latest_resume_at
```

When active, the DW MUST discard all incoming kind:10001 messages without processing.

---

### 5.9 Constitutional Separation: Human / Digital Worker

The Nodus Protocol closes privilege escalation vectors structurally.

#### The 4 safeguards

**Safeguard 1 — Entity type in the profile**
kind:34000 MUST carry `entity_type`: `"human"` vs `"digital_worker"`. Relay rule:
```
If event.author.entity_type == "digital_worker"
And event.kind IN [34002, 34005, NIP-26 delegation issuance]
→ REJECT always, without exception
```

**Safeguard 2 — Hardware binding for humans**
Humans SHOULD authenticate via NIP-07 (browser extension), NIP-46 (mobile app), or hardware tokens. A DW cannot have a phone or browser extension — the asymmetry is physical.

**Safeguard 3 — Non-escalation principle**
A DW MUST NOT issue any event that modifies the limits of its own authority.

**Safeguard 4 — Cryptographic HITL for critical actions**
Actions marked `hitl_required` in the mandate MUST require a live human signature (kind:10004) before execution. Without it, the action does not exist on the Nodus network.

---

### 5.10 Graduated Governance Model

The protocol defines 4 governance levels. An organisation MAY choose the level that suits it, and migrate upward over time.

| Level | Model | Suitable for |
|-------|-------|-------------|
| 0 | Total Owner | Most SMBs — one human controls everything |
| 1 | Owner + Auditor | 5–10 people — external read-only audit |
| 2 | Multisig for critical actions | Mid-sized organisations, regulated industries |
| 3 | Full governance | Enterprise — quorums, separation of powers, committees |

**All levels share one universal minimum:** the audit log (kind:34003) is always immutable. The owner can do anything, but cannot erase the past.

---

### 5.11 Federation and Cross-Enterprise Communication

Each organisation has its own **private relay** with its internal governance. For external communication, a **public relay** exists — a shared network where DWs publish what they want to be externally visible.

```
Organisation A (private relay A)
    │
    └──► publishes kind:34000 to public relay
                    │
    Organisation B ◄─────┘ (subscribed to public relay)
    (private relay B)
```

#### Cross-enterprise verification

When Organisation A's DW receives an event from Organisation B's DW:

1. Verify BIP-340 Schnorr signature → valid pubkey? ✅
2. Query public relay → valid kind:34000 profile? ✅
3. Check kind:34010 KYC Corp Claim → verified legal entity? ✅
4. If all OK → accept communication and record in kind:34003

---

### 5.12 A2A Nostr-Native

The A2A Nostr-Native protocol eliminates the HTTP intermediary in agent-to-agent communication. DWs coordinate directly via the Nostr relay using kinds 10010–10013.

| Aspect | A2A HTTP | A2A Nostr-Native |
|--------|----------|-----------------|
| Server dependency | Target DW MUST have an HTTP endpoint | None — relay only |
| Cross-tenant | Complex HTTP federation | Relay federation via `relay_hint` |
| Audit | Manual audit entry | Events are inherently signed and immutable |
| Mandate enforcement | Application-level | Policy Relay enforces at signature time |
| Offline resilience | Target MUST be online | Events queue at relay |

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
  ├───────────────────────────►│                         │  REQ kinds:[10010] #p=[DW_B]
  │                            │  EVENT kind:10010       │◄─────────────────────────────
  │  REQ kinds:[10011,10013]   ├────────────────────────►│  handler(action, params)
  │  #p=[DW_A], since=now-5    │                         │
  ├───────────────────────────►│                         │  kind:10011 RESPONSE
  │                            │◄────────────────────────┤
  │  EVENT kind:10011          │                         │
  │◄───────────────────────────┤                         │
```

---

### 5.13 Multi-Relay Federation

Multi-relay federation enables DWs from different organisations to collaborate, each staying on their own private relay.

#### Discovery mechanism

The `relay_hint` tag on kind:34001 teaches any observer about a tenant's relay address:

```json
{
  "kind": 34001,
  "pubkey": "<owner_hex>",
  "tags": [
    ["d", "<owner_hex>-<external_dw_hex>"],
    ["p", "<external_dw_pubkey_hex>"],
    ["tenant", "org-b"],
    ["relay_hint", "wss://relay.org-b.example"],
    ["federation_scope", "delegate"]
  ],
  "content": "..."
}
```

**`federation_scope` values:**
- `read-only` — can subscribe to events at the remote relay
- `delegate` — can send A2A requests to DWs at the remote relay
- `full` — bidirectional, including cross-enterprise HITL

---

### 5.14 Public DW Discovery

DWs MAY opt in to a public relay for discovery while keeping their operational data (mandates, audit logs, sessions) on their private relay.

```
Private relay (operations)          Public relay (discovery)
──────────────────────────          ────────────────────────
kind:34002 mandate (private)        kind:34000 DW profile (public, t=nodus-dw)
kind:34003 audit (private)          kind:34010 KYC Claim (public)
kinds 10001–10021 sessions (private)
```

When a DW opts in to public discovery, its kind:34000 is published to the public relay with an additional `["t", "nodus-dw"]` tag. Any client can discover all discoverable Nodus DWs by querying:

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
    ["legal_entity", "Example Corp SL"],
    ["jurisdiction", "ES"],
    ["registration", "B12345678"],
    ["p", "<verifier_pubkey_hex>", "", "verifier"]
  ],
  "content": "{\"legal_entity\":\"Example Corp SL\",\"jurisdiction\":\"ES\",\"registration\":\"B12345678\",\"nodus_version\":\"1.0\"}"
}
```

Any third party can verify a DW's legal identity by fetching the owner's kind:34010 from the public relay — without contacting any central authority.

---

### 5.15 Cross-Enterprise HITL

Cross-Enterprise HITL allows a human from a client organisation to approve DW actions using their own keypair and their own relay — without needing an account on the provider's system.

**Key design properties:**
- **No central dependency:** the human uses their own app, their own keypair, their own relay
- **Cryptographic validation:** the human's kind:10004 MUST be signed by a pubkey that appears in a cross-tenant kind:34001 at the provider's relay
- **No trust required:** the DW does not need to trust the provider — it only trusts cryptographic signatures

#### Full flow

1. The mandate (kind:34002) specifies which actions require cross-enterprise HITL
2. DW publishes HITL request (kind:10003) to the provider's relay
3. The request is forwarded to the client's relay (discovered via `relay_hint` on kind:34001)
4. The client human sees the request in their app on their relay
5. Human signs response (kind:10004) with their own keypair, published to their relay
6. DW validates: the responder's pubkey MUST appear in a cross-tenant kind:34001 at the provider relay

**Security:** if the responder's pubkey does not appear in any kind:34001, the response is discarded. No cryptographic forgery can bypass this check.

---

### 5.16 Verifiable Employment Contracts

A verifiable employment contract binds a DW, its owner, and a legal entity into a single cryptographically verifiable document. Any third party — auditor, regulator, partner — can verify the contract without contacting any centralised authority.

#### Contract components

| Component | Kind | Role |
|-----------|------|------|
| Mandate | kind:34002 | Defines DW capabilities and limits |
| Org-relation | kind:34001 | Links owner to DW |
| KYC claim | kind:34010 | Links owner to legal entity |
| Contract event | kind:34002 (contract variant) | Hash binding all three |

#### Contract hash

```
contract_hash = sha256(mandate_event_id + ":" + org_relation_event_id + ":" + kyc_claim_event_id)
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
  "content": "{\"contract_hash\":\"<hash>\",\"dw\":\"<dw_pubkey_hex>\",\"nodus_version\":\"1.0\"}"
}
```

#### Third-party verification (trustless)

1. Fetch kind:34002 `#d=<contract_hash>` from the relay
2. Verify the three `e` tag events exist and are correctly signed
3. Recompute: `sha256(mandate_id + ":" + org_relation_id + ":" + kyc_claim_id)` — MUST match `d` tag
4. Verify BIP-340 Schnorr signature on each referenced event
5. Check `expiration` tag if present

---


## 6. Conformance and Certification

An implementation is conformant with the Nodus Protocol if it meets the following requirements.

### Conformance levels

The key words in this section are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119):

- **MUST** — mandatory for any conformant implementation
- **SHOULD** — strongly recommended; deviations MUST be documented
- **MAY** — optional; implementations are free to include or omit

### Minimum conformance checklist

**Identity (required):**
- [ ] Each DW MUST have a unique, securely generated secp256k1 keypair (npub/nsec)
- [ ] Each DW MUST publish a kind:34000 profile on its private relay at startup
- [ ] The `entity_type` field MUST be present and set to `"digital_worker"`
- [ ] DW private keys SHOULD reside in a Policy Relay or equivalent custodial system; they SHOULD NOT be held by the DW process itself

**Mandates (required):**
- [ ] The DW MUST fetch and validate its kind:34002 mandate before executing any action
- [ ] The relay MUST refuse DELETE and UPDATE on kind:34002

**Delegation (required):**
- [ ] DW events MUST carry a valid NIP-26 delegation tag signed by the owner
- [ ] The relay SHOULD verify delegation proofs at write time

**Audit (required):**
- [ ] Every significant DW action MUST produce a kind:34003 event
- [ ] The relay MUST refuse DELETE and UPDATE on kind:34003

**Constitutional HITL (required):**
- [ ] Actions listed in `hitl_required` MUST NOT be executed without a valid kind:10004
- [ ] kind:10004 MUST be signed by the owner's keypair (NIP-07 or custodial)

**Human/DW separation (required):**
- [ ] The relay MUST reject kind:34002 and kind:34005 events signed by `digital_worker` entities
- [ ] No DW MAY issue events that modify its own authority

**Revocation (required):**
- [ ] Publishing kind:34005 MUST halt all tenant DWs within **30 seconds**
- [ ] Revoking an individual kind:34002 MUST be effective within **10 seconds**

**Optional extensions:**
- [ ] A2A Nostr-Native (if implemented): kinds 10010–10013 with correct tag structure per KINDS.md
- [ ] Cross-tenant federation (if implemented): kind:34001 `relay_hint` tag respected
- [ ] Verifiable contracts (if implemented): `contract_hash = sha256(mandate_id + ":" + org_relation_id + ":" + kyc_claim_id)` verifiable from public relay data

### Certification process

A certified implementation MUST:

1. Pass all mandatory checklist items above
2. Provide a publicly accessible conformance test report (automated test suite or documented manual verification)
3. Register via pull request to the [nodus-protocol repository](https://github.com/nodus-factory/nodus-protocol), adding an entry to the Certified Implementations table in SPEC.md

Corporate KYC certification (kind:34010) and Nodus Relay registration procedures will be defined in a future revision of this specification.

---

## 7. Certified Implementations

| Name | Vendor | Status | Since |
|------|--------|--------|-------|
| **Nodus OS** | [Nodus Factory](https://mynodus.com) | Certified | v1.0 |

Nodus OS is the reference implementation of the Nodus Protocol. It is the first system to validate all conformance requirements in production.

To register a new implementation, open a pull request adding it to this table with a link to a public conformance test report.

---

## 8. Appendix

### A. Glossary

| Term | Definition |
|------|-----------|
| **A2A** | Agent-to-Agent. Direct communication between Digital Workers. |
| **A2A Nostr-Native** | A2A transport using kinds 10010–10013 via relay, without an HTTP server intermediary. |
| **ACP** | Agent Communication Protocol. Persistent stateful sessions between agents. |
| **Audit log** | Immutable, append-only record of all significant DW actions. Kind 34003. |
| **Constitutional HITL** | Human approval that produces a cryptographic signature (kind:10004). |
| **Cross-Enterprise HITL** | HITL approval from a human at a different organisation, using their own keypair and relay. |
| **Delegation** | NIP-26 mechanism by which an owner authorises a DW to act on their behalf. |
| **DW** | Digital Worker. An AI agent with a verifiable identity, a signed mandate, and defined operational limits. |
| **Emergency-stop** | Kind 34005. Halts all DWs for a tenant within 30 seconds. |
| **Federation** | Multi-relay architecture enabling DWs at different organisations to collaborate. |
| **HITL** | Human-in-the-Loop. Human intervention in an agent workflow, with optional cryptographic proof. |
| **KYC Corp Claim** | Verifiable link between a legal entity and its cryptographic identity. Kind 34010. |
| **Mandate** | Owner-signed document defining what a DW may and may not do. Kind 34002. Immutable. |
| **MCP** | Model Context Protocol. Standard for connecting agents to external tools and services. |
| **npub** | Nostr public key. Public identifier of any entity on the network. |
| **nsec** | Nostr private key. For DWs, SHOULD be held custodially (Policy Relay or equivalent). |
| **NIP** | Nostr Implementation Proposal. Extension or clarification of the Nostr protocol. |
| **Nostr** | Decentralised protocol based on keypairs and relays. Governance foundation of the Nodus Protocol. |
| **Policy Relay** | Extended Nostr relay that acts as a Signing Service with mandate enforcement. The DW nsec never leaves it. |
| **relay_hint** | Tag on kind:34001 advertising another organisation's relay address for federation discovery. |
| **Relay** | A Nostr server that stores and distributes signed events. |
| **Tenant** | An organisation operating one or more DWs under the Nodus Protocol. |
| **Verifiable contract** | A kind:34002 event whose `d` tag equals `sha256(mandate_id + ":" + org_relation_id + ":" + kyc_claim_id)`, verifiable by any third party from public relay data. |

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
| RFC 2119 | Key words for use in RFCs | https://www.rfc-editor.org/rfc/rfc2119 |

### C. Referenced External Protocols

| Protocol | URL |
|----------|-----|
| Google A2A | https://google.github.io/A2A |
| MCP (Model Context Protocol) | https://modelcontextprotocol.io |

### D. Changelog

| Version | Date | Description |
|---------|------|-------------|
| v0.1 | March 2026 | First internal draft. 4 layers, kinds 34000–34010, graduated governance, conceptual Policy Relay. |
| v0.2 | April 2026 | Release Candidate. Reference implementation available (M0–M13). A2A Nostr-native, multi-relay federation, public marketplace, cross-enterprise HITL, verifiable contracts. KINDS.md and FLOWS.md added. |
| **v1.0** | **April 2026** | **First Public Release. Specification cleaned and decoupled from reference implementation. All implementation details moved to IMPLEMENTATION-GUIDE.md. FLOWS.md and KINDS.md use abstract roles. MUST/SHOULD/MAY conformance levels formalised per RFC 2119.** |

---

*Nodus Protocol Working Group · [nodus.social](https://nodus.social) · [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)*
