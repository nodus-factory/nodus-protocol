# Nodus Protocol — Protocol Flows

> **Protocol version:** 1.0
> **Status:** First Public Release
> **License:** CC BY 4.0

This document describes the primary protocol flows. Each flow is defined in terms of abstract roles — **Initiator**, **Relay**, **Worker**, **Owner** — to remain independent of any specific implementation.

For kind definitions and event structures, see [KINDS.md](KINDS.md).
For implementation guidance, see [IMPLEMENTATION-GUIDE.md](IMPLEMENTATION-GUIDE.md).

---

## Roles

| Role | Description |
|------|-------------|
| **Initiator** | Any entity (human or DW) that sends a task request |
| **Worker** | A Digital Worker receiving and processing requests |
| **Relay** | A Nostr relay enforcing write policies |
| **Owner** | The human or organisation that authorises the Worker and signs governance events |

---

## Flow 1 — Normal Work Session

**Kinds involved:** 10001, 10002, 10006, 34002 (optional), 34003 (optional)

A human sends a task to a Worker. The Worker validates its mandate (if active), processes the task, streams partial responses, and publishes a final response. All significant actions are recorded in the audit log.

```
Initiator                          Relay                           Worker
    │                                │                                │
    │  EVENT kind:10001               │                                │
    │  tags: [p=Worker, session,      │                                │
    │         request, token?]        │                                │
    ├───────────────────────────────►│                                │
    │                                │  EVENT → #p subscription       │
    │                                ├───────────────────────────────►│
    │                                │                                │ receives MESSAGE_USER
    │                                │                                │
    │                                │                                │ [mandate active?]
    │                                │◄───────────────────────────────┤ REQ kinds:[34002]
    │                                │  kind:34002 mandate            │
    │                                ├───────────────────────────────►│ validate: permitted?
    │                                │                                │
    │                                │                                │ process task
    │                                │                                │   (streaming)
    │  EVENT kind:10006               │                                │
    │  STREAMING_CHUNK                │◄───────────────────────────────┤ (per chunk)
    │◄───────────────────────────────┤                                │
    │  EVENT kind:10006               │                                │
    │◄───────────────────────────────┤◄───────────────────────────────┤
    │                                │                                │
    │                                │                                │ [audit active?]
    │                                │◄───────────────────────────────┤ EVENT kind:34003
    │                                │                                │
    │  EVENT kind:10002               │                                │
    │  RESPONSE_DW (final)            │◄───────────────────────────────┤
    │◄───────────────────────────────┤                                │
```

**Mandate validation (if active):**
1. Worker fetches kind:34002 with `#d = Worker pubkey`
2. Checks `valid_from ≤ now` and `valid_until = null OR > now`
3. Checks capability is in `capabilities` and not in `limits`
4. If capability is in `hitl_required` → proceeds to Flow 2 (HITL) before acting
5. If no valid mandate found → Worker MUST refuse the action

**Delegation (if active):**
All events published by the Worker carry a `["delegation", owner_pubkey, conditions, sig]` tag — a NIP-26 proof that the Owner authorised this Worker to act.

---

## Flow 2 — Constitutional HITL

**Kinds involved:** 10001, 10002, 10003, 10004, 34003

The Worker encounters an action in `hitl_required`. It publishes a HITL request, suspends execution, and waits for a cryptographically signed human approval before proceeding.

```
Initiator              Relay                Worker              Owner client
    │                    │                    │                      │
    │  kind:10001         │                    │                      │
    ├───────────────────►│                    │                      │
    │                    ├───────────────────►│                      │
    │                    │                    │ detects hitl_required │
    │                    │                    │                      │
    │                    │                    │ suspend execution     │
    │                    │◄───────────────────┤ EVENT kind:10003      │
    │  kind:10003         │                    │ HITL_REQUEST          │
    │  HITL_REQUEST       │                    │                      │
    │◄───────────────────┤                    │                      │
    │                    │                    │                      │
    │ [shows approval UI]│                    │                      │
    │                    │                    │                      │
    │                    │                    │ REQ kinds:[10004]     │
    │                    │                    │ #request=<10003.id>   │
    │                    │◄───────────────────┤                      │
    │                    │                    │                      │
    │                    │                    │ waiting...            │
    │                    │                    │                      │
    │ Owner signs         │                    │                      │
    │ kind:10004 via      │                    │                      │
    │ NIP-07 or custodial │                    │                      │
    │                    │◄──────────────────────────────────────────┤
    │                    │  EVENT kind:10004  │                      │
    │                    │  HITL_RESPONSE     │                      │
    │                    │  signed by Owner   │                      │
    │                    ├───────────────────►│                      │
    │                    │                    │ verify signature      │
    │                    │                    │ check approved=true   │
    │                    │                    │                      │
    │                    │                    │ [audit: hitl_decision]│
    │                    │◄───────────────────┤ EVENT kind:34003      │
    │                    │                    │                      │
    │                    │                    │ resume execution      │
    │                    │◄───────────────────┤ EVENT kind:10002      │
    │  kind:10002         │                    │ RESPONSE_DW           │
    │◄───────────────────┤                    │                      │
```

**Cryptographic proof of authority:**
The kind:10004 event is signed with the Owner's private key. Any third party can verify — without consulting any server — that a human with authorised keys approved this specific action, at this specific time, referencing this specific HITL request.

**If rejected (approved=false):**
The Worker publishes a kind:10002 explaining the refusal and records `hitl_decision:rejected` in the audit log.

---

## Flow 3 — A2A Request (Worker-to-Worker)

**Kinds involved:** 10010, 10011, 10012, 10013

One Worker delegates a subtask to another Worker, fully via the relay. No HTTP intermediary is involved.

```
Worker A (Sender)              Relay                  Worker B (Receiver)
      │                          │                           │
      │  EVENT kind:10010         │                           │
      │  A2A_REQUEST              │                           │
      │  tags: [p=WorkerB,        │                           │
      │   request_id, action,     │                           │
      │   mandate?]               │                           │
      ├─────────────────────────►│                           │
      │                          │  EVENT → #p subscription  │
      │                          ├──────────────────────────►│
      │                          │                           │ processes subtask
      │                          │                           │   (streaming)
      │  kind:10012 A2A_STREAM    │                           │
      │◄─────────────────────────┤◄──────────────────────────┤ (per chunk, done=false)
      │  kind:10012 A2A_STREAM    │                           │
      │◄─────────────────────────┤◄──────────────────────────┤ (final chunk, done=true)
      │                          │                           │
      │  kind:10011 A2A_RESPONSE  │                           │
      │◄─────────────────────────┤◄──────────────────────────┤ (final result)
```

**Error path:**
If Worker B cannot process the request, it publishes kind:10013 (A2A_ERROR) with an `error` field in the content JSON.

**Mandate reference:**
The `["mandate", "<kind:34002_event_id>"]` tag on kind:10010 is OPTIONAL but RECOMMENDED when Worker A is acting under a known mandate scope. Worker B MAY require it.

---

## Flow 4 — Async HITL (Inbox)

**Kinds involved:** 10020, 10021

An asynchronous approval request — originating from a cron job, graph trigger, or long-running process — that does not occur within an active conversation thread. The Owner resolves it at their own pace via an inbox interface.

```
Worker / Cron              Relay                   Owner (inbox)
      │                      │                          │
      │  EVENT kind:10020     │                          │
      │  INBOX_ITEM           │                          │
      │  tags: [p=Owner,      │                          │
      │   session, action]    │                          │
      ├─────────────────────►│                          │
      │                      │  persisted on relay       │
      │                      │                          │
      │                      │  [Owner polls/subscribes] │
      │                      ├─────────────────────────►│
      │                      │  kind:10020              │
      │                      │                          │ Owner reviews in inbox
      │                      │                          │ signs kind:10021
      │                      │◄─────────────────────────┤
      │                      │  EVENT kind:10021         │
      │                      │  INBOX_RESOLVED           │
      │                      │  tags: [request=10020.id, │
      │                      │   approved=true]          │
      │ [Worker polls/        │                          │
      │  subscribes]          │                          │
      │◄─────────────────────┤                          │
      │  kind:10021           │                          │
      │ validates + proceeds  │                          │
```

---

## Flow 5 — Emergency Stop

**Kinds involved:** 34005, 34006

The Owner publishes a halt signal. All Workers belonging to the tenant detect it within 30 seconds and discard all incoming requests until a resume signal is received.

```
Owner                    Relay                   Worker(s)
  │                        │                        │
  │  EVENT kind:34005       │                        │ [polling every 30s]
  │  EMERGENCY_STOP         │                        │ REQ kinds:[34005,34006]
  │  tags: [d=tenant_id,    │                        │ #tenant=[tenant_id]
  │   tenant=tenant_id]     │                        │
  ├──────────────────────►│                        │
  │                        ├───────────────────────►│
  │                        │  kind:34005             │ latest_stop_at > latest_resume_at
  │                        │                        │ → EMERGENCY ACTIVE
  │                        │                        │ → discard all kind:10001
  │                        │                        │
  │ [... time passes ...]  │                        │
  │                        │                        │
  │  EVENT kind:34006       │                        │
  │  EMERGENCY_RESUME       │                        │
  ├──────────────────────►│                        │
  │                        ├───────────────────────►│
  │                        │  kind:34006             │ latest_resume_at > latest_stop_at
  │                        │                        │ → EMERGENCY CLEARED
  │                        │                        │ → resume normal operation
```

**Timing requirement:** Workers MUST detect kind:34005 and halt within **30 seconds** of publication.

**Only the Owner can publish kind:34005 and kind:34006.** The relay MUST reject these events from Workers (verified via kind:34000 `entity_type` field).

---

## Flow 6 — Mandate Creation

**Kinds involved:** 34000, 34001, 34002

The Owner establishes a Worker's governance context: publishes the Worker's identity relationship and signs its mandate.

```
Owner client                  Relay                    Worker
    │                           │                         │
    │  [Worker publishes        │                         │
    │   its own profile]        │                         │
    │                           │◄────────────────────────┤ EVENT kind:34000
    │                           │  DW_PROFILE              │ entity_type: digital_worker
    │                           │                         │
    │  EVENT kind:34001          │                         │
    │  ORG_RELATION              │                         │
    │  signed by Owner           │                         │
    ├──────────────────────────►│                         │
    │                           │                         │
    │  EVENT kind:34002          │                         │
    │  MANDATE                   │                         │
    │  signed by Owner           │                         │
    ├──────────────────────────►│                         │
    │                           │                         │
    │                           │  [Worker fetches        │
    │                           │   kind:34002 on startup]│
    │                           ├────────────────────────►│
    │                           │  kind:34002              │ mandate loaded ✓
```

**Immutability:** The relay MUST reject any DELETE or UPDATE on kind:34002 and kind:34003. Once signed and published, a mandate cannot be altered — only superseded by a new version with a different `d` tag value.

---

## Flow 7 — Cross-Tenant HITL (Federation)

**Kinds involved:** 10003, 10004, 34001 (cross-tenant)

A Worker belonging to Tenant A requires approval from a human at Tenant B. The approval is cryptographically tied to the approver's keypair — no shared infrastructure needed.

```
Worker (Tenant A)     Relay A            Relay B           Owner (Tenant B)
      │                  │                  │                     │
      │  kind:10003       │                  │                     │
      │  HITL_REQUEST     │                  │                     │
      │  (cross-tenant)   │                  │                     │
      ├─────────────────►│                  │                     │
      │                  │  [federation     │                     │
      │                  │   bridge relays  │                     │
      │                  │   to Relay B]    │                     │
      │                  ├─────────────────►│                     │
      │                  │                  ├────────────────────►│
      │                  │                  │  kind:10003          │ [approval UI shown]
      │                  │                  │                     │
      │                  │                  │◄────────────────────┤ kind:10004
      │                  │                  │  HITL_RESPONSE       │ signed by Owner B
      │                  │◄─────────────────┤                     │
      │◄─────────────────┤                  │                     │
      │ validate:         │                  │                     │
      │ - sig valid?      │                  │                     │
      │ - pubkey in       │                  │                     │
      │   cross-tenant    │                  │                     │
      │   kind:34001?     │                  │                     │
      │ proceed ✓         │                  │                     │
```

**Authorisation prerequisite:** Tenant A's relay MUST have a kind:34001 (`org-relation`) event identifying the Tenant B approver's pubkey as authorised for cross-tenant HITL. Without it, the Worker MUST reject the kind:10004 even if the signature is valid.

---

## Flow 8 — Public Marketplace Discovery

**Kinds involved:** 34000, 34001, 34010

An organisation discovers available Workers on the public Nodus relay and verifies their identity before delegating work.

```
Organisation (buyer)          Public Relay              Worker operator
      │                           │                           │
      │  REQ kinds:[34000]         │                           │
      │  #t=["nodus-dw"]           │                           │
      ├──────────────────────────►│                           │
      │                           │  [Worker profiles]        │
      │◄──────────────────────────┤  kind:34000 (marketplace) │
      │                           │  per Worker               │
      │  [inspect capabilities,   │                           │
      │   limits, nodus_version]   │                           │
      │                           │                           │
      │  REQ kinds:[34010]         │                           │
      │  #t=["nodus-kyc"]          │                           │
      ├──────────────────────────►│                           │
      │◄──────────────────────────┤  kind:34010 KYC claim     │
      │                           │                           │
      │  [verify legal entity,     │                           │
      │   jurisdiction,            │                           │
      │   registration number]     │                           │
      │                           │                           │
      │  [compute contract hash]   │                           │
      │  sha256(mandate_id +       │                           │
      │   org_relation_id +        │                           │
      │   kyc_claim_id)            │                           │
      │  → verifiable offline ✓    │                           │
```

**Verifiable contract:** Any third party (auditor, regulator, counterparty) can compute the contract hash offline from public relay data, without access to any private system.

---

## Summary Table

| Flow | Trigger | Key kinds | HITL? | Audit? |
|------|---------|-----------|-------|--------|
| 1 — Normal session | Human sends task | 10001, 10002, 10006, 34002, 34003 | No | Yes (optional) |
| 2 — Constitutional HITL | Action in `hitl_required` | 10003, 10004, 34003 | Yes | Yes |
| 3 — A2A request | Worker delegates subtask | 10010, 10011, 10012, 10013 | No | Optional |
| 4 — Async inbox HITL | Cron / background trigger | 10020, 10021 | Yes (async) | Optional |
| 5 — Emergency stop | Owner halts tenant | 34005, 34006 | No | Implicit |
| 6 — Mandate creation | Owner provisions Worker | 34000, 34001, 34002 | No | No |
| 7 — Cross-tenant HITL | External org approval | 10003, 10004, 34001 | Yes (cross-org) | Yes |
| 8 — Marketplace discovery | Buyer finds Workers | 34000, 34010 | No | No |

---

*Nodus Protocol Working Group · CC BY 4.0*
