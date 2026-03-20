# Nodus Protocol — Specification v0.1

> **Status:** Public Draft  
> **Authors:** Quirze Salomó · Nodus Factory  
> **Date:** March 2026  
> **License:** Creative Commons Attribution 4.0 International (CC BY 4.0)  
> **Repository:** github.com/nodus-factory/nodus-protocol *(coming soon)*

---

## Abstract

The Nodus Protocol defines a minimal set of standards for the **identity, delegation, governance, verifiable human intervention, and audit of Digital Workers** in enterprise environments.

The protocol is **transport-agnostic**: it does not impose a single communication channel between agents, but rather a common layer of action legitimacy. A Digital Worker can communicate over A2A, ACP, HTTP, queues, or Nostr — but when it acts, the Protocol ensures that anyone can answer:

*Who performed this action? Did they have the authority to do so? Who authorized them? When? On what basis? And how can it be stopped?*

The Nodus Protocol is not a platform. It is not a product. It is the minimum infrastructure needed for a digital workforce to be governed, audited, and trusted — just as today's human workforce is.

---

## Table of Contents

1. [The World Ahead](#1-the-world-ahead)
2. [Motivation](#2-motivation)
3. [Design Principles](#3-design-principles)
4. [Core Concepts](#4-core-concepts)
5. [Technical Specification](#5-technical-specification)
   - 5.1 Nostr NIPs as Governance Foundation
   - 5.2 Nodus Protocol Kinds (34000+)
   - 5.3 Synchronous A2A Transport
   - 5.4 Persistent ACP Sessions
   - 5.5 MCP under Protocol Governance
   - 5.6 Nodus Policy Relay
   - 5.7 Enterprise Control over Digital Workers
   - 5.8 Panic Button and Revocation
   - 5.9 Constitutional Separation: Human / Digital Worker
   - 5.10 Graduated Governance Model
   - 5.11 Federation and Cross-Enterprise Communication
6. [Reference Implementation](#6-reference-implementation)
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

> Technical basis: NIP-01 of the Nostr protocol.

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
- Scope (over which data and systems it may act)
- Validity period (from when to when it is valid)

The mandate is **immutable once signed** by the owner. If it needs to change, it must be revoked and a new one created.

### 4.5 Delegation (NIP-26)

Delegation is the mechanism by which an authorized human transfers authority to a Digital Worker to act on their behalf. When a DW acts with delegation:

- Its action carries the DW's signature (technical identity)
- And the cryptographic proof that the owner authorized this action (delegated authority)

Any third party can verify both without consulting any central server.

### 4.6 Agent-to-Agent Interaction (A2A)

Digital Workers communicate with each other. The Nodus Protocol recognizes two primary patterns:

- **Synchronous A2A** (Google A2A over HTTP/JSON-RPC) — when an immediate response is needed
- **Asynchronous A2A** (NIP-90 / DVMs over Nostr) — when the task can be decoupled in time

The protocol does not impose a single transport. What it imposes is that every governed interaction can demonstrate: who is acting, with what authority, under what mandate, and what trace it leaves.

### 4.7 Human-in-the-Loop (HITL)

Certain actions require real human intervention. The Nodus Protocol distinguishes between:

- **Operational HITL:** the human receives a notification and approves via any surface (messaging app, web interface, email...)
- **Constitutional HITL:** when the human exercises authority — approves, rejects, revokes, authorizes an exception — this act is ideally materialized on the Nostr layer as a verifiable and permanent cryptographic proof

> Ordinary human↔agent conversation may pass through any channel. But when a human exercises authority, the Protocol records it.

### 4.8 Audit Log (kind 34003)

The audit log is the immutable record of all significant actions by a Digital Worker. Each entry contains:

- Who performed the action (DW identity)
- With what authority (reference to mandate and delegation)
- When (verifiable timestamp)
- What was done (hash of the action and its result)
- In the context of which interaction (A2A job reference if applicable)

The audit log is **append-only**: no entry can be modified or deleted. Not by the DW's owner. Not by Nodus. Not by anyone.

---

## 5. Technical Specification

The Nodus Protocol is implemented over four complementary communication layers:

| Layer | Protocol | Role | When to use |
|-------|----------|------|-------------|
| **Cryptographic governance** | Nostr (NIPs 01/26/42/44/46/59/89/90) | Identity, delegation, revocation, immutable audit, discovery, federation, constitutional HITL | When legitimizing, revoking, auditing, or involving a human with verifiable authority |
| **Synchronous A2A transport** | Google A2A (JSON-RPC/HTTP + SSE) | Direct agent→agent synchronous calls | DW executes atomic task with blocking response |
| **Persistent sessions** | ACP (Agent Communication Protocol) | Stateful sessions, streaming, long context, multi-turn orchestration | LLM streaming between agents, long conversational sessions |
| **Agent↔Tools** | MCP (Model Context Protocol) | Access to tools, APIs, databases, external systems | DW accesses calendar, CRM, email, files, APIs |

**The Nodus Protocol is transport-agnostic but rooted in Nostr for governance.** Agents may communicate over many channels, but when an action requires protocol legitimacy, that legitimacy lives in the Nostr cryptographic layer.

---

### 5.1 Nostr NIPs as Governance Foundation

The protocol requires support for the following NIPs:

| NIP | Name | Role in Nodus |
|-----|------|--------------|
| NIP-01 | Basic protocol | Foundation. Event signing, basic kinds. Mandatory. |
| NIP-26 | Delegated event signing | DWs sign on behalf of the owner. Verifiable authority proofs. |
| NIP-42 | Relay AUTH | Private relay per company. Only authenticated identities connect. |
| NIP-44 | Versioned Encryption | Encrypted P2P communication between agents or human and agent. |
| NIP-46 | Nostr Connect / Remote Signing | Technical basis for the Policy Relay and constitutional HITL. |
| NIP-59 | Gift Wrap / Sealed DMs | Maximum privacy channel for sensitive HITL and critical decisions. |
| NIP-89 | App Handler Info | Agent Cards and discovery on the relay. Each DW declares capabilities and transports. |
| NIP-90 | Data Vending Machines | Async pattern for A2A. Jobs decoupled in time. |

---

### 5.2 Nodus Protocol Kinds (34000+)

Nostr allows defining custom vocabulary. The Nodus Protocol defines:

| Kind | Name | Description | Mutability |
|------|------|-------------|------------|
| `34000` | `nodus:dw-profile` | Full DW profile: capabilities, limits, owner, tenant, accepted transports. | Updatable |
| `34001` | `nodus:org-relation` | Organizational graph arc: OwnerOf, DelegatesTo, Supervises, ReportsTo, Approves, Audits. | Updatable |
| `34002` | `nodus:policy` | DW mandate. Signed by owner. **Cannot be deleted.** | Immutable |
| `34003` | `nodus:audit-event` | Audit log entry. Append-only, no DELETE or UPDATE. | Append-only |
| `34004` | `nodus:mcp-server-profile` | Certified MCP Server profile: exposed tools, owner, authorized tenants/DWs. | Updatable |
| `34005` | `nodus:emergency-stop` | Immediately stops all DWs in a tenant. Signed by CEO/owner. | Immutable |
| `34006` | `nodus:emergency-resume` | Resumes DW activity after an emergency-stop. | Immutable |
| `34010` | `nodus:kyc-corp-claim` | Link between legal entity (tax ID) and cryptographic identity (npub). | Updatable |

> **Critical implementation rule:** Relays MUST reject any DELETE or UPDATE attempt on kinds `34002` and `34003`. Without this rule, the audit log has no legal or compliance value.

#### Kind 34000 — Digital Worker Profile

```json
{
  "kind": 34000,
  "pubkey": "<DW npub>",
  "content": {
    "name": "DW Renewals",
    "description": "Manages permit renewal tracking and client notifications",
    "owner": "<owner npub>",
    "tenant": "<tenant npub>",
    "entity_type": "digital_worker",
    "capabilities": ["renewals", "client-notifications", "record-read"],
    "limits": ["no-denials", "no-red-category-clients"],
    "transports": [
      { "type": "nostr-nip90", "relay": "wss://relay.company.com" },
      { "type": "a2a", "url": "https://agents.company.com/dw-renewals/a2a" },
      { "type": "acp", "url": "https://agents.company.com/dw-renewals/acp" }
    ],
    "nodus_version": "0.1",
    "certified": true
  },
  "sig": "<owner signature>"
}
```

**`transports` field:** declares which communication protocols the DW accepts. Agents wishing to interact consult this field and choose the appropriate transport. Order indicates preference.

**`entity_type` field:** mandatory. Values: `"human"` | `"digital_worker"` | `"mcp_server"` | `"committee"`. The relay rejects mandates and delegations from non-human entities.

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
| No economic model | Staking + marketplace (Nodus Relay) |
| No revocation | Revocation in seconds (kind 34002 revoked) |

#### Basic A2A flow with Nodus governance

```
Agent A                          Agent B
   │                                │
   │── POST /a2a (JSON-RPC) ───────►│
   │   {method: "tasks/send",       │
   │    params: {task, mandate_ref, │
   │    delegation_proof}}          │
   │                                │ verify signature
   │                                │ verify mandate 34002
   │                                │ execute
   │◄── {result, audit_ref} ────────│
   │                                │
   └── Record 34003 (audit) ───────►│
```

#### Async A2A flow via Nostr (NIP-90)

```
Agent A                    Relay                    Agent B
   │                          │                          │
   │── kind 5050 (job req) ──►│                          │
   │                          │── kind 5050 ────────────►│
   │                          │                          │ (processing)
   │                          │◄── kind 7000 (progress) ─│
   │◄── kind 7000 ────────────│                          │
   │                          │◄── kind 6050 (result) ───│
   │◄── kind 6050 ────────────│                          │
   │                          │                          │
   │── kind 34003 (audit) ───►│                          │
```

- **kinds 5000–5999** → task requests (job request)
- **kinds 6000–6999** → results (job result)
- **kind 7000** → progress, errors, HITL requests

---

### 5.4 Persistent ACP Sessions

Google A2A is synchronous and stateless. Nostr is asynchronous but is not a conversational session protocol. Neither covers **multi-turn orchestration**: an orchestrating agent collaborating in a long conversation with subordinate agents, maintaining context across multiple interactions.

The Nodus Protocol supports **ACP (Agent Communication Protocol)** to fill this gap.

#### When to use ACP

- An orchestrating agent coordinating multiple DWs in a complex flow
- An agent maintaining a long working session with a specialized agent
- Workflows requiring shared context between steps
- Streaming partial responses between agents while work is in progress

#### When to use A2A instead of ACP

- Atomic, independent tasks ("send this email", "check the calendar")
- Fire-and-forget calls where context doesn't matter
- Interactions between agents from different tenants (use Nostr for the public layer)

#### Relationship between layers

```
Orchestrator (ACP session)
      │
      ├─→ DW Office   (A2A, direct call)        ← atomic task
      ├─→ DW Business (A2A, direct call)        ← atomic task
      │
      └─→ Sub-orchestrator (ACP session)        ← complex multi-step flow
              │
              └─→ Audit → Nostr (kind 34003)    ← immutable record
```

The identity of agents participating in an ACP session is established via the same cryptographic keys of the Nodus Protocol (npub/nsec), ensuring that ACP sessions are auditable and verifiable.

---

### 5.5 MCP under Protocol Governance

#### The problem MCP does not solve

MCP (Model Context Protocol, Anthropic 2024) is the emerging standard for connecting agents with external tools and systems. But MCP has no identity. An MCP Server is a URL. There is no mechanism to know:

- Who manages this MCP Server
- Whether it is legitimate or has been compromised
- Which tools it exposes and for whom it has authorization
- Whether a connection should be audited
- How to revoke a DW's access if there is an incident

> **"An MCP Server without verifiable identity is an attack vector. The Nodus Protocol gives it a passport."**

#### The solution: identity for MCP Servers

Just like Digital Workers, MCP Servers operating under the Nodus Protocol have:

1. **Their own keypair** (`npub`/`nsec`) — independent cryptographic identity
2. **Registered profile** (kind `34004`) — declares tools, owner, authorized tenants/DWs
3. **Signed mandate** — the owner signs which tools it may serve and to whom
4. **Audit trail** — every DW call is recorded in kind `34003`
5. **Revocation** — if an MCP Server is compromised, the owner publishes an immediate revocation

#### Kind 34004 — MCP Server Profile

```json
{
  "kind": 34004,
  "pubkey": "<MCP Server npub>",
  "content": {
    "name": "Company MCP Gateway",
    "url": "https://mcp.company.com",
    "tools": ["calendar", "email", "drive", "crm"],
    "owner": "<owner npub>",
    "authorized_tenants": ["<tenant A npub>", "<tenant B npub>"],
    "authorized_dws": ["<DW 1 npub>", "<DW 2 npub>"],
    "valid_from": "<timestamp>",
    "valid_until": null
  },
  "sig": "<owner signature>"
}
```

#### Flow: DW uses an MCP tool under Nodus governance

```
DW wants to use "calendar" from an MCP Server
        │
        ├─→ Check relay: does kind 34004 exist for this MCP Server?
        │       ├── No → REJECT connection (MCP Server not certified)
        │       └── Yes → check: is this DW in authorized_dws?
        │                   ├── No → REJECT (not authorized)
        │                   └── Yes → CONNECT
        │
        ├─→ Execute MCP call (tool: "calendar", action: "create_event")
        │
        └─→ Record in kind 34003 (audit):
              - DW: <npub>
              - MCP Server: <npub>
              - Tool used: "calendar/create_event"
              - Timestamp + result hash
```

---

### 5.6 Nodus Policy Relay

> ⚠️ **Status: conceptual specification — full implementation in v0.2**

Standard Nostr relays distribute events but do not filter them against business policies. The **Nodus Policy Relay** solves this by turning the relay into an active **Signing Service**.

#### The core principle: DW nsec keys never leave the relay

```
When creating a DW → keypair generated at the relay
  nsec → stored encrypted at the relay (never leaves)
  npub → public, visible to everyone

When the DW wants to act:
  DW → "relay, sign this event for me"
  Relay → checks: valid mandate 34002 for this event type?
         → checks: authorized recipient?
         → Yes → signs with nsec → returns signed event
         → No → rejects → the action does not exist
```

**The DW does not hold the key. It cannot sign on its own.** Without a valid signature, no other Nodus DW accepts its requests. This is not a matter of good faith — it is **cryptographic impossibility**.

> Technical basis: NIP-46 (Nostr Connect / nsecbunker) — already implemented for humans. The Nodus Protocol adapts it for DWs, adding the semantics of enterprise mandates.

#### Technical implementation

- **`writePolicy` plugin** for `strfry` — reads mandates (kind 34002) in memory and rejects non-compliant events
- ~500 lines of code on top of a standard strfry relay
- Scalable: one relay per tenant, or multi-tenant relay with NIP-42 partitioning

#### Trust model: two levels

| Level | Mechanism | Effect |
|-------|-----------|--------|
| **Hard enforcement** | Cryptographic — without the relay's signing, the DW cannot act | The legitimate action physically does not exist |
| **Soft enforcement** | Economic — without Nodus certification, no access to marketplace | No global network access, no reputation, no partners |

#### Relay strategy: two types

**Company private relay (Policy Relay)**
- Authenticated via NIP-42 — only tenant identities
- Write policies per kind (who may write what)
- Kind 34003: append-only
- Hosted by the company or a certified Nodus operator

**Public network relay (Nodus Relay)**
- Public DW profiles (kind 34000)
- Discovery and marketplace of certified agents
- Partner reputation and staking

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
CEO / Company  (npub type: "human")
      │
      │ kind 34002 — signs mandate
      │ NIP-26 — delegates authority
      ▼
DW  (npub type: "digital_worker")
      │
      │ Each action carries:
      │   ① DW signature
      │   ② Delegation proof (NIP-26)
      │   ③ Mandate reference (34002)
      ▼
Action → kind 34003 (immutable audit)
```

---

### 5.8 Panic Button and Revocation

> *"A Digital Worker without a panic button is not an enterprise Digital Worker. It is a risk."*

#### 4 revocation scenarios

**1. Compromised DW (suspicious key, anomalous behavior)**
→ CEO publishes a revoked kind 34002 for that DW
→ Relay propagates immediately → all agents reject events from that DW
→ Audit 34003: exact timestamp, who, reason
→ **Time: seconds**

**2. Employee offboarding (person who controlled DWs leaves the company)**
→ Revoke DWs associated with that human
→ Revoke NIP-26 delegations they had issued
→ Effect: none of the DWs reporting to them can act until new delegation

**3. Total security incident (stop ALL DWs now)**
→ CEO publishes **kind 34005** (`nodus:emergency-stop`) signed
→ Relay rejects ALL DW events for the tenant
→ The network enters "read-only mode" for the tenant's DWs
→ To resume: CEO publishes kind 34006 (`nodus:emergency-resume`)

**4. External DW from a partner or supplier**
→ Company A publishes revocation of the NIP-26 delegation to Company B's DW
→ Immediate effect, without calling anyone, without waiting for the supplier
→ The external DW is no longer authorized to act on Company A's data

#### Technical implementation (kind 34005)

```json
{
  "kind": 34005,
  "pubkey": "<CEO / owner npub>",
  "content": {
    "tenant": "<tenant npub>",
    "reason": "Security incident detected — precautionary suspension",
    "timestamp": "<unix timestamp>",
    "authorized_by": "<CEO npub>"
  },
  "sig": "<CEO signature>"
}
```

The relay, upon receiving a valid kind 34005 signed by the tenant CEO, activates a total block policy until it receives a kind 34006 from the same authority.

---

### 5.9 Constitutional Separation: Human / Digital Worker

#### The problem

A DW could, in theory, attempt to:
- Sign its own mandates (privilege escalation)
- Publish kind 34005 (stop other DWs)
- Impersonate a human to obtain approval from another DW
- Create new subordinate DWs without authorization

The Nodus Protocol closes these vectors structurally.

#### The 4 safeguards

**Safeguard 1 — Entity type in the keypair**

Kind 34000 (profile) carries an `entity_type` field: `"human"` vs `"digital_worker"`. The relay applies rules based on this field:

```
Relay rule:
  If event.author.entity_type == "digital_worker"
  And event.kind IN [34002, 34005, NIP-26 delegation]
  → REJECT always, without exception
```

A DW cannot issue mandates or stop other DWs. Structurally impossible.

**Safeguard 2 — Hardware binding for humans**

Humans authenticate via:
- NIP-07 (browser extension): private key protected by hardware
- NIP-46 (mobile app): physical approval on the phone
- YubiKey or similar hardware device

A DW cannot have a phone, a YubiKey, or a browser extension. The asymmetry is physical.

**Safeguard 3 — The non-escalation principle**

Immutable relay rule:
> A DW may not issue any event that modifies the limits of its own authority. None. Ever.

Not creating new mandates, not revoking others' mandates, not issuing emergency-stop, not creating new DWs.

**Safeguard 4 — Cryptographic HITL for critical actions**

A class of actions — those the mandate marks as `hitl_required: true` — require a human signature in real time. Without the human's physical signature (via NIP-46 or NIP-07), the action does not exist on the Nodus network, regardless of what the DW says.

#### Constitutional rule

> *"A Digital Worker can never be more than what its human owner has explicitly granted, nor can it modify the limits of that grant."*

---

### 5.10 Graduated Governance Model

The Nodus Protocol does not impose a single governance model. Most SMBs are **owner-centric**: one human controls everything. Imposing a complex governance model would be an unacceptable adoption barrier.

The protocol defines 4 governance levels. A company may choose the level that suits it, and migrate upward over time.

#### Level 0 — Total Owner *(default, most SMBs)*

One human controls everything. Simple. No additional approvals.

**Risk:** if the owner is malicious or compromised, they can do anything.
**Mitigation:** everything is recorded in kind 34003. The trace is immutable. They cannot erase the past.

> *"We don't stop you. But everything is recorded forever."*

#### Level 1 — Owner + Auditor

The owner does as they wish. An Auditor (external accountant, partner, compliance officer) has verifiable read-only access to the audit log. Any irregularity is detectable after the fact.

Easy to adopt. Suitable for most companies from 5–10 employees.

#### Level 2 — Multisig for Critical Actions

The owner acts normally. But certain acts require a second signature:
- Deleting a DW or changing its mandate
- Economic actions above a threshold (e.g., >€10,000)
- Changes to access lists for critical systems

Suitable for mid-sized companies, regulated industries, or where insider risk is real.

#### Level 3 — Full Governance *(Enterprise / Institutional)*

Large companies, listed companies, public institutions. Quorums for critical decisions. Separation of powers between CEO, CFO, CTO roles. External auditor with certified access. Multi-signature approval committees.

> *"The protocol does not impose a governance level. But it guarantees that the chosen level is verifiable, auditable, and cannot be secretly bypassed."*

#### Immutability as universal minimum

**All levels** share one thing: the audit log (kind 34003) is always immutable. Even at Level 0, everything is recorded. The owner can do anything, but cannot erase the past. This is sufficient for most compliance cases.

#### Analogy for SMBs

> *"It is like a permanent digital notary. The notary does not stop you from doing anything. But everything is recorded and verifiable forever."*

---

### 5.11 Federation and Cross-Enterprise Communication

The previous sections describe the internal governance of a single company. But the real use case requires DWs from different companies to collaborate verifiably.

#### The federated model

Each company has its own **private relay** (Policy Relay) with its internal governance. For external communication, the **Nodus public relay** exists — a shared network where DWs publish what they want to be externally visible.

```
Company A (private relay A)
    │
    └──► publishes event to Nodus public relay
                    │
    Company B ◄─────┘ (subscribed to public relay)
    (private relay B)
```

Each company's governance remains internal and private. External communication passes through the public relay. No company imposes its governance on another.

#### Cross-enterprise verification

When Company A's DW receives an event from Company B's DW:

1. Verify signature → is it from the npub it claims to be? ✅
2. Query public relay → does that npub have a valid kind 34000 profile? ✅
3. Check if Company B is a certified Nodus tenant → kind 34010 (corporate KYC) ✅
4. If all OK → accept communication and record in kind 34003

#### Cross-enterprise delegation (NIP-26)

For tasks requiring access to another company's data, explicit delegation signed by an authorized human is required:

> *"I, the CEO of Company A, authorize DW `npub-X` of Partner B to access client Y's records for 48 hours."*

Signed by the CEO → verifiable by anyone → auditable → revocable at any time.

#### Illustrative use case

**Scenario:** A legal services firm with 3 active Digital Workers.

**09:00** — The CEO authenticates. The relay records the event.

**09:01** — DW Renewals activates. The relay checks: does it have a valid mandate 34002 signed by the CEO? Yes. The DW publishes a NIP-90 task: *"Review records and detect pending renewals."* The relay signs the event with the DW's nsec → the event exists on the network.

**09:02** — DW Integration (external system) receives the task. Verifies signature → valid → returns 12 clients with permits about to expire.

**09:03** — DW Renewals wants to send notifications to all 12 clients. The relay checks the mandate: may it send notifications? Yes, but **only to Green-category clients**. Checks each client → 10 Green, 2 Orange. Signs the 10. The 2 Orange → urgent notification sent to the responsible attorney for approval.

**09:15** — DW Case Status detects an unfavorable resolution. Wants to notify the client directly. The relay checks the mandate: *"denials → notify attorney, never the client directly."* **Rejects the event.** Generates urgent notification for the attorney. Kind 34003: immutable record.

**12:00** — CEO revokes DW Renewals (going on leave). Publishes a revoked kind 34002. The relay immediately stops signing this DW's events. **Revocation time: seconds.**

At no point could any DW bypass the rules. Not out of goodwill — because the `nsec` lives at the relay and the relay does not sign what the mandates do not permit.

#### The passport analogy

> *Each country issues its own passports (sovereign internal governance). All accept each other's passports (external interoperability) because there are common verifiable standards. No country imposes its law on others, but all can verify the identity of any citizen.*

The Nodus Protocol is the identity and passport infrastructure for digital workforces at global scale.

---

## 6. Reference Implementation

**Nodus OS** is the first certified implementation of the Nodus Protocol.

Nodus OS implements the four protocol layers over a real production architecture:

- **Governance layer:** private Nostr relay per company (strfry), kinds 34000–34010, NIP-26 implemented
- **Synchronous transport:** Google A2A (JSON-RPC/HTTP + SSE) via agent runtime
- **Persistent sessions:** ACP — the personal agent layer bridging users to the Nodus ecosystem
- **Tools:** MCP Gateway (TypeScript) with per-tenant credential management

**Reference stack:**
- Agent runtime: Google ADK + FastAPI
- Multi-step workflows: LangGraph
- Memory and knowledge: vector store (Qdrant) + graph (PostgreSQL)
- Private Nostr relay: strfry + writePolicy hooks
- Deployment: Kubernetes (GitOps via ArgoCD)
- Personal agent: OpenClaw ACP

Repository: [github.com/nodus-factory](https://github.com/nodus-factory) *(protocol repository coming soon)*

---

## 7. Conformance and Certification

An implementation is compatible with Nodus Protocol v0.1 if it meets:

### Minimum conformance checklist

**Identity:**
- [ ] Each DW has a unique, securely generated keypair (npub/nsec)
- [ ] Each DW has a kind 34000 profile published on the relay
- [ ] The `entity_type` field is present and correct
- [ ] DW nsec keys never leave the server (custodial)

**Mandates:**
- [ ] Every DW action has a reference to a valid, active kind 34002 mandate
- [ ] The relay rejects actions from DWs without a valid mandate
- [ ] The relay refuses DELETE and UPDATE on kinds 34002 and 34003

**Delegation:**
- [ ] DW actions carry NIP-26 delegation proof from the owner
- [ ] The relay verifies delegation proofs

**Audit:**
- [ ] Every significant action generates a kind 34003 event
- [ ] The audit log is append-only
- [ ] The relay refuses DELETE and UPDATE on kind 34003

**Human/DW Separation:**
- [ ] The relay rejects kind 34002, 34005 events emitted by `digital_worker` entities
- [ ] There is a HITL mechanism for actions marked `hitl_required`

**Revocation:**
- [ ] Kind 34005 (emergency-stop) stops all tenant DWs in <30 seconds
- [ ] Individual mandate revocation is effective in <10 seconds

### Certification process

*(To be defined in v0.2 — including corporate KYC via kind 34010, audit by Nodus operator, and registration on the public Nodus Relay)*

---

## 8. Appendix

### A. Glossary

| Term | Definition |
|------|-----------|
| **A2A** | Agent-to-Agent. Direct communication between Digital Workers. |
| **ACP** | Agent Communication Protocol. Persistent stateful sessions between agents. |
| **Audit log** | Immutable record of all significant DW actions. Kind 34003. |
| **Delegation** | NIP-26 mechanism by which a human authorizes a DW to act on their behalf. |
| **DW** | Digital Worker. AI agent with identity, mandate, and defined limits. |
| **Emergency-stop** | Kind 34005. Immediately stops all DWs in a tenant. |
| **HITL** | Human-in-the-Loop. Human intervention in an agent workflow. |
| **Corporate KYC** | Know Your Customer applied to companies. Link between legal entity and npub. Kind 34010. |
| **Mandate** | Signed document defining what a DW can and cannot do. Kind 34002. |
| **MCP** | Model Context Protocol. Standard for connecting agents to external tools. |
| **npub** | Nostr public key. Public identifier of an entity. |
| **nsec** | Nostr private key. Never shared. For DWs, lives custodially at the relay. |
| **NIP** | Nostr Implementation Possibility. Specification of Nostr protocol functionality. |
| **Nostr** | Decentralized protocol based on keypairs and relays. Foundation of the Nodus Protocol governance layer. |
| **Policy Relay** | Extended Nostr relay acting as a Signing Service with mandate enforcement. |
| **Relay** | Nostr server that distributes events. The Nodus Protocol uses private relays (per company) and a public relay (for the Nodus network). |
| **Tenant** | Company or organization operating DWs under the Nodus Protocol. |

### B. Referenced Nostr NIPs

| NIP | Title | URL |
|-----|-------|-----|
| NIP-01 | Basic protocol flow | https://github.com/nostr-protocol/nostr/blob/master/01.md |
| NIP-26 | Delegated event signing | https://github.com/nostr-protocol/nostr/blob/master/26.md |
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
| v0.1 | March 2026 | First public draft. 4 layers, kinds 34000–34010, graduated governance, conceptual Policy Relay. |

---

*Nodus Factory · © 2026 · CC BY 4.0*
