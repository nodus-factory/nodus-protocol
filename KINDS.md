# Nodus Protocol — Event Kinds Reference

> **Protocol version:** v0.2  
> **Status:** Release Candidate  
> **Last updated:** April 2026  
> **Reference implementation:** Nodus OS ADK (private)  
> **License:** CC BY 4.0

This document is the canonical reference for all Nostr event kinds used in the Nodus Protocol. Every kind includes its complete structure extracted from the reference implementation.

---

## Layer Convention

| Range | Layer | NIP base |
|-------|-------|----------|
| `10001–10021` | **Session Layer** — operational messaging between agents and humans | NIP-16 (replaceable) |
| `34000–34010` | **Governance Layer** — identity, mandates, audit, emergency controls | NIP-33 (parameterized replaceable) |

---

## Session Layer (kinds 10001–10021)

### kind:10001 — MESSAGE_USER

| Field | Value |
|-------|-------|
| **Name** | `MESSAGE_USER` |
| **Layer** | Session Layer |
| **Milestone** | M0 (implemented) |
| **Publisher** | Human / Frontend (`nodus-llibreta-v2`) |
| **Status** | ✅ Implemented |

**Description:** User message directed at a specific DW. The DW listens via `#p` subscription on the relay and processes it with the ADK runner.

**DW subscription filter:**
```json
["REQ", "<sub_id>", {
  "kinds": [10001],
  "#p": ["<dw_pubkey_hex>"],
  "since": "<unix_ts - 30>"
}]
```

**Event structure:**
```json
{
  "id": "<sha256 of serialised>",
  "pubkey": "<user_pubkey_hex>",
  "created_at": 1714000000,
  "kind": 10001,
  "tags": [
    ["p", "<dw_pubkey_hex>"],
    ["session", "<session_uuid>"],
    ["request", "<request_uuid>"],
    ["token", "<jwt_optional>"]
  ],
  "content": "Send an email to albert@example.com with the meeting summary",
  "sig": "<schnorr_sig_hex>"
}
```

**Required tags:** `p` (target DW), `session`, `request`  
**Optional tags:** `token` (JWT for user authentication at the DW)

---

### kind:10002 — RESPONSE_DW

| Field | Value |
|-------|-------|
| **Name** | `RESPONSE_DW` |
| **Layer** | Session Layer |
| **Milestone** | M0 (implemented) |
| **Publisher** | DW (`nodus-adk-runtime`) |
| **Status** | ✅ Implemented |

**Description:** Final accumulated DW response, published at the end of `run_async()`. Always preceded by zero or more kind:10006 streaming chunks.

```json
{
  "id": "<sha256>",
  "pubkey": "<dw_pubkey_hex>",
  "created_at": 1714000001,
  "kind": 10002,
  "tags": [
    ["p", "<user_pubkey_hex>"],
    ["session", "<session_uuid>"],
    ["request", "<request_uuid>"],
    ["agent", "<agent_name>"],
    ["delegation", "<owner_pubkey>", "<conditions>", "<sig>"]
  ],
  "content": "I sent the email to albert@example.com. Subject: Meeting summary. Body: [...]",
  "sig": "<schnorr_sig_hex>"
}
```

**Required tags:** `p`, `session`, `request`, `agent`  
**Optional tags:** `delegation` (if `NODUS_PROTOCOL_DELEGATION_V1` active)

---

### kind:10003 — HITL_REQUEST

| Field | Value |
|-------|-------|
| **Name** | `HITL_REQUEST` |
| **Layer** | Session Layer |
| **Milestone** | M4 Constitutional HITL (implemented) |
| **Publisher** | DW (`nodus-adk-runtime`) |
| **Status** | ✅ Implemented |

**Description:** Human approval request. Published when the DW detects a pending `ToolConfirmation` in the ADK session. The `content` is a JSON payload describing the action to be approved.

```json
{
  "id": "<sha256>",
  "pubkey": "<dw_pubkey_hex>",
  "created_at": 1714000002,
  "kind": 10003,
  "tags": [
    ["p", "<user_pubkey_hex>"],
    ["session", "<session_uuid>"],
    ["request", "<request_uuid>"],
    ["delegation", "<owner_pubkey>", "<conditions>", "<sig>"]
  ],
  "content": "{\"event_id\":\"hitl_a3b4c5d6e7f8\",\"action\":\"send_email\",\"description\":\"Send email to albert@example.com\",\"agent_name\":\"nodus_standard_assistant\",\"input_type\":\"text\",\"choices\":null,\"urgency\":\"normal\"}",
  "sig": "<schnorr_sig_hex>"
}
```

**Content JSON fields:**

| Field | Type | Description |
|-------|------|-------------|
| `event_id` | `string` | Internal HITL ID (`hitl_<12hex>`) |
| `action` | `string` | Tool/function name requiring approval |
| `description` | `string` | Human-readable request text |
| `agent_name` | `string` | ADK agent name |
| `input_type` | `"text"\|"choice"` | Expected response type |
| `choices` | `string[]\|null` | Options if `input_type=choice` |
| `urgency` | `"normal"\|"high"` | Urgency level |

---

### kind:10004 — HITL_RESPONSE

| Field | Value |
|-------|-------|
| **Name** | `HITL_RESPONSE` |
| **Layer** | Session Layer |
| **Milestone** | M4 Constitutional HITL (implemented) |
| **Publisher** | Human (via NIP-07 or custodial) |
| **Status** | ✅ Implemented |

**Description:** Human response to a kind:10003. Cryptographically signed by the human (NIP-07 via browser extension, or custodial via server). Constitutes the **cryptographic proof** of the human decision.

```json
{
  "id": "<sha256>",
  "pubkey": "<human_pubkey_hex>",
  "created_at": 1714000010,
  "kind": 10004,
  "tags": [
    ["request", "<kind:10003_event_id>"],
    ["session", "<session_uuid>"],
    ["approved", "true"]
  ],
  "content": "approved",
  "sig": "<schnorr_sig_hex_of_human>"
}
```

**Required tags:** `request` (reference to kind:10003), `approved` (`"true"` or `"false"`)  
**`content`:** `"approved"` or `"rejected"`

**Signing methods:**
- **NIP-07:** `window.nostr.signEvent(event)` — via Alby, nos2x, etc.
- **Custodial:** `POST /api/nostr/hitl/respond` — server signs with user's AES-256-GCM encrypted keypair

---

### kind:10005 — RESPONSE_AGENT

| Field | Value |
|-------|-------|
| **Name** | `RESPONSE_AGENT` |
| **Layer** | Session Layer |
| **Milestone** | M0 (implemented) |
| **Publisher** | Internal agent |
| **Status** | ✅ Implemented |

**Description:** Response from one internal agent to another in A2A communication within the system (HTTP v1 transport). Different from kind:10010–10013 which is A2A Nostr-native (v0.2).

```json
{
  "kind": 10005,
  "pubkey": "<agent_pubkey_hex>",
  "tags": [
    ["p", "<target_agent_pubkey_hex>"],
    ["session", "<session_uuid>"]
  ],
  "content": "<agent result>"
}
```

---

### kind:10006 — STREAMING_CHUNK

| Field | Value |
|-------|-------|
| **Name** | `STREAMING_CHUNK` |
| **Layer** | Session Layer |
| **Milestone** | M0 (implemented) |
| **Publisher** | DW (`nodus-adk-runtime`) |
| **Status** | ✅ Implemented |

**Description:** Real-time streaming chunk. One event per `Part.text` returned by `runner.run_async()`. Allows the frontend to display the response progressively.

```json
{
  "id": "<sha256>",
  "pubkey": "<dw_pubkey_hex>",
  "created_at": 1714000001,
  "kind": 10006,
  "tags": [
    ["p", "<user_pubkey_hex>"],
    ["session", "<session_uuid>"],
    ["request", "<request_uuid>"]
  ],
  "content": "I sent the email",
  "sig": "<schnorr_sig_hex>"
}
```

---

### kind:10010 — A2A_REQUEST (v0.2)

| Field | Value |
|-------|-------|
| **Name** | `A2A_REQUEST` |
| **Layer** | Session Layer — A2A sublayer |
| **Milestone** | M9 A2A Nostr-Native (v0.2) |
| **Publisher** | Sender DW (`nodus-adk-runtime`) |
| **Feature flag** | `NODUS_A2A_NOSTR_V2` |
| **Status** | ✅ Implemented, flag OFF by default |

**Description:** Task execution request from one DW to another DW, 100% via Nostr, without an HTTP intermediary server.

```json
{
  "id": "<sha256>",
  "pubkey": "<dw_a_pubkey_hex>",
  "created_at": 1714000020,
  "kind": 10010,
  "tags": [
    ["p", "<dw_b_pubkey_hex>"],
    ["session", "<session_uuid>"],
    ["request_id", "<8char_uuid>"],
    ["action", "summarize_document"],
    ["mandate", "<kind:34002_event_id_optional>"]
  ],
  "content": "{\"action\":\"summarize_document\",\"params\":{\"doc_id\":\"abc123\",\"lang\":\"en\"}}",
  "sig": "<schnorr_sig_hex>"
}
```

---

### kind:10011 — A2A_RESPONSE (v0.2)

| Field | Value |
|-------|-------|
| **Name** | `A2A_RESPONSE` |
| **Layer** | Session Layer — A2A sublayer |
| **Milestone** | M9 (v0.2) |
| **Publisher** | Receiver DW |
| **Status** | ✅ Implemented, flag OFF |

```json
{
  "kind": 10011,
  "pubkey": "<dw_b_pubkey_hex>",
  "created_at": 1714000025,
  "tags": [
    ["p", "<dw_a_pubkey_hex>"],
    ["request_id", "<8char_uuid>"],
    ["session", "<session_uuid>"]
  ],
  "content": "{\"result\": \"The document is about...\"}",
  "sig": "<schnorr_sig_hex>"
}
```

---

### kind:10012 — A2A_STREAM (v0.2)

| Field | Value |
|-------|-------|
| **Name** | `A2A_STREAM` |
| **Layer** | Session Layer — A2A sublayer |
| **Milestone** | M9 (v0.2) |
| **Publisher** | Receiver DW |
| **Status** | ✅ Implemented, flag OFF |

**Description:** Streaming chunk in an A2A response. `content.done = true` signals the final chunk.

```json
{
  "kind": 10012,
  "pubkey": "<dw_b_pubkey_hex>",
  "tags": [["p", "<dw_a_pubkey_hex>"], ["request_id", "<id>"], ["session", "<id>"]],
  "content": "{\"chunk\": \"The document is about\", \"done\": false}"
}
```

---

### kind:10013 — A2A_ERROR (v0.2)

| Field | Value |
|-------|-------|
| **Name** | `A2A_ERROR` |
| **Layer** | Session Layer — A2A sublayer |
| **Milestone** | M9 (v0.2) |
| **Publisher** | Receiver DW |
| **Status** | ✅ Implemented, flag OFF |

```json
{
  "kind": 10013,
  "pubkey": "<dw_b_pubkey_hex>",
  "tags": [["p", "<dw_a_pubkey_hex>"], ["request_id", "<id>"], ["session", "<id>"]],
  "content": "{\"error\": \"Document not found: abc123\"}"
}
```

---

### kind:10020 — INBOX_ITEM

| Field | Value |
|-------|-------|
| **Name** | `INBOX_ITEM` |
| **Layer** | Session Layer — Async HITL sublayer |
| **Milestone** | M6 Room UX (implemented) |
| **Publisher** | DW / Cron / Graph |
| **Status** | ✅ Implemented |

**Description:** Asynchronous approval request (for cron or long-running process actions, not within an active conversation thread). Accumulates in the frontend InboxPanel.

```json
{
  "kind": 10020,
  "pubkey": "<dw_pubkey_hex>",
  "tags": [
    ["p", "<owner_pubkey_hex>"],
    ["session", "<uuid>"],
    ["action", "<action_name>"]
  ],
  "content": "<description of pending action>"
}
```

---

### kind:10021 — INBOX_RESOLVED

| Field | Value |
|-------|-------|
| **Name** | `INBOX_RESOLVED` |
| **Layer** | Session Layer — Async HITL sublayer |
| **Milestone** | M6 (implemented) |
| **Publisher** | Human |
| **Status** | ✅ Implemented |

```json
{
  "id": "<sha256>",
  "pubkey": "<human_pubkey_hex>",
  "created_at": 1714000050,
  "kind": 10021,
  "tags": [
    ["request", "<kind:10020_event_id>"],
    ["approved", "true"],
    ["p", "<dw_pubkey_hex_optional>"]
  ],
  "content": "approved",
  "sig": "<schnorr_sig_hex>"
}
```

---

## Governance Layer (kinds 34000–34010)

All Governance Layer kinds are **parameterized replaceable events** (NIP-33). The `["d", "<identifier>"]` tag defines the unique identifier within a pubkey+kind pair.

### kind:34000 — nodus:dw-profile / nodus:human-profile

| Field | Value |
|-------|-------|
| **Name** | `nodus:dw-profile` / `nodus:human-profile` |
| **Layer** | Governance Layer — Identity |
| **Milestone** | M0 Identities (implemented) |
| **Publisher** | DW (for DW profiles) / System (`nodus-llibreta-v2`) for human profiles |
| **Feature flag** | `NODUS_PROTOCOL_IDENTITY_V1` |
| **Status** | ✅ Implemented, flag OFF by default |

**Description:** Full actor profile (DW or human) at the relay. Contains capabilities, available transports, and metadata.

**DW Profile:**
```json
{
  "id": "<sha256>",
  "pubkey": "<dw_pubkey_hex>",
  "created_at": 1714000000,
  "kind": 34000,
  "tags": [
    ["d", "<dw_pubkey_hex>"],
    ["p", "<owner_pubkey_hex_optional>"]
  ],
  "content": "{\"name\":\"Athena\",\"description\":\"Root orchestrator agent\",\"owner\":\"<owner_hex>\",\"tenant\":\"default\",\"entity_type\":\"digital_worker\",\"capabilities\":[\"orchestrate\",\"email\"],\"limits\":[],\"transports\":[{\"type\":\"nostr-session\",\"relay\":\"ws://nostr-relay:7777\",\"kinds\":[10001]}],\"nodus_version\":\"0.2\"}",
  "sig": "<schnorr_sig_hex>"
}
```

**Content JSON fields:**

| Field | Type | Description |
|-------|------|-------------|
| `name` | `string` | DW name (env `NODUS_DW_NAME`) |
| `description` | `string` | Description (env `NODUS_DW_DESCRIPTION`) |
| `owner` | `string\|null` | Owner pubkey hex |
| `tenant` | `string\|null` | Tenant identifier |
| `entity_type` | `"digital_worker"\|"human"` | Entity type |
| `capabilities` | `string[]` | Declared capabilities |
| `limits` | `string[]` | Declared limitations |
| `transports` | `Transport[]` | Available communication channels |
| `nodus_version` | `"0.1"\|"0.2"` | Protocol version |

**Marketplace Profile (v0.2) — published to public relay:**
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

---

### kind:34001 — nodus:org-relation

| Field | Value |
|-------|-------|
| **Name** | `nodus:org-relation` |
| **Layer** | Governance Layer — Hierarchy |
| **Milestone** | M0.3 Identities (implemented) |
| **Publisher** | Owner (tenant admin) |
| **Feature flag** | `NODUS_PROTOCOL_IDENTITY_V1` |
| **Status** | ✅ Implemented, flag OFF by default |

**Description:** Declares the `OwnerOf` hierarchical relationship between an Owner and a DW. Signed by the owner. Used for federation discovery (via `relay_hint` tag in v0.2).

```json
{
  "id": "<sha256>",
  "pubkey": "<owner_pubkey_hex>",
  "created_at": 1714000000,
  "kind": 34001,
  "tags": [
    ["d", "<owner_pubkey_hex>-<dw_pubkey_hex>"],
    ["p", "<dw_pubkey_hex>"],
    ["tenant", "default"]
  ],
  "content": "{\"relation\":\"OwnerOf\",\"subject\":\"<owner_pubkey_hex>\",\"object\":\"<dw_pubkey_hex>\",\"tenant\":\"default\",\"nodus_version\":\"0.1\"}",
  "sig": "<schnorr_sig_hex_owner>"
}
```

**Cross-tenant federation (v0.2):**
```json
{
  "kind": 34001,
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

**`federation_scope` values:** `read-only` | `delegate` | `full`

---

### kind:34002 — nodus:policy (mandate)

| Field | Value |
|-------|-------|
| **Name** | `nodus:policy` |
| **Layer** | Governance Layer — Mandate |
| **Milestone** | M1 Mandates (implemented) |
| **Publisher** | Owner (tenant admin) via `nodus-backoffice` |
| **Feature flag** | `NODUS_PROTOCOL_MANDATES_V1` |
| **Status** | ✅ Implemented, flag OFF by default |

**Description:** Mandate defining exactly what the DW can and cannot do. **Immutable**: the relay must reject any DELETE or UPDATE. Signed by the owner.

```json
{
  "id": "<sha256>",
  "pubkey": "<owner_pubkey_hex>",
  "created_at": 1714000000,
  "kind": 34002,
  "tags": [
    ["d", "<dw_pubkey_hex>"],
    ["p", "<dw_pubkey_hex>"]
  ],
  "content": "{\"dw\":\"<dw_pubkey_hex>\",\"tenant\":\"my-tenant\",\"capabilities\":[\"send_email\",\"read_calendar\",\"orchestrate\"],\"limits\":[\"no_delete_without_confirmation\"],\"hitl_required\":[\"send_email\",\"delete_*\",\"financial_*\"],\"auto_approve\":[\"read_calendar\",\"list_memory\"],\"max_auto_cost_eur\":0,\"valid_from\":1714000000,\"valid_until\":null,\"nodus_version\":\"0.1\"}",
  "sig": "<schnorr_sig_hex_owner>"
}
```

**Content JSON fields:**

| Field | Type | Description |
|-------|------|-------------|
| `dw` | `string` | DW pubkey hex |
| `tenant` | `string\|null` | Tenant identifier |
| `capabilities` | `string[]` | Permitted actions (wildcards: `delete_*`) |
| `limits` | `string[]` | Explicit restrictions |
| `hitl_required` | `string[]` | Actions requiring human approval |
| `auto_approve` | `string[]` | Always auto-approved (overrides `hitl_required`) |
| `max_auto_cost_eur` | `number` | Maximum auto-approved cost in EUR |
| `valid_from` | `number` | Unix timestamp — validity start |
| `valid_until` | `number\|null` | Unix timestamp — validity end (null = indefinite) |
| `nodus_version` | `"0.1"` | Protocol version |

> **Critical relay rule:** Relays MUST reject any DELETE or UPDATE on kind:34002. Without this enforcement, the mandate has no legal or compliance value.

**Mandate query by DW:**
```python
{"kinds": [34002], "#d": ["<dw_pubkey_hex>"], "limit": 1}
```

---

### kind:34003 — nodus:audit-event

| Field | Value |
|-------|-------|
| **Name** | `nodus:audit-event` |
| **Layer** | Governance Layer — Audit |
| **Milestone** | M2 Audit Log (implemented) |
| **Publisher** | DW (`nodus-adk-runtime`) |
| **Feature flag** | `NODUS_PROTOCOL_AUDIT_V1` |
| **Status** | ✅ Implemented, flag OFF by default |

**Description:** Immutable, append-only audit record. The `d` tag is `sha256(dw_pubkey + session_id + timestamp_ms)` to guarantee uniqueness. The relay must block overwrites.

```json
{
  "id": "<sha256>",
  "pubkey": "<dw_pubkey_hex>",
  "created_at": 1714000010,
  "kind": 34003,
  "tags": [
    ["d", "<sha256_of_dw+session+ts_ms>"],
    ["p", "<user_npub_hex_or_empty>"],
    ["mandate", "<kind:34002_event_id_or_none>"],
    ["session", "<session_uuid>"],
    ["action", "agent_response"],
    ["tenant", "<tenant_id>"]
  ],
  "content": "{\"action\":\"agent_response\",\"result_hash\":\"<sha256_of_result>\",\"timestamp\":1714000010,\"dw\":\"<dw_pubkey_hex>\",\"user\":\"<user_hex>\",\"tenant\":\"default\",\"session_id\":\"<uuid>\",\"mandate_ref\":\"<event_id_or_null>\"}",
  "sig": "<schnorr_sig_hex>"
}
```

**Action variants:**
- `agent_response` — final agent response
- `hitl_request:<action>` — HITL request for a specific action
- `hitl_decision:approved` / `hitl_decision:rejected` — human decision

> **Critical relay rule:** Relays MUST reject any DELETE or UPDATE on kind:34003. The audit log is append-only.

---

### kind:34004 — nodus:mcp-server-profile

| Field | Value |
|-------|-------|
| **Name** | `nodus:mcp-server-profile` |
| **Layer** | Governance Layer — MCP |
| **Milestone** | M7 MCP Governance (implemented) |
| **Publisher** | MCP Gateway (`nodus-mcp-gateway`) |
| **Feature flag** | `NODUS_PROTOCOL_MCP_PROFILE_V1` |
| **Status** | ✅ Implemented, flag OFF by default |

**Description:** MCP Gateway profile at the relay. The DW verifies this before calling any tool.

```json
{
  "kind": 34004,
  "pubkey": "<mcp_gateway_pubkey_hex>",
  "tags": [["d", "nodus-mcp-gateway"]],
  "content": "{\"name\":\"Nodus MCP Gateway\",\"url\":\"https://mcp.nodus.local\",\"tools\":[\"calendar\",\"email\",\"drive\",\"crm\"],\"owner\":\"<owner_pubkey_hex>\",\"authorized_dws\":[\"<athena_pubkey_hex>\",\"<ofimatic_pubkey_hex>\"],\"valid_until\":null}"
}
```

---

### kind:34005 — nodus:emergency-stop

| Field | Value |
|-------|-------|
| **Name** | `nodus:emergency-stop` |
| **Layer** | Governance Layer — Emergency |
| **Milestone** | M5 Emergency Controls (implemented) |
| **Publisher** | Owner (tenant admin) via Backoffice |
| **Feature flag** | `NODUS_PROTOCOL_EMERGENCY_V1` |
| **Status** | ✅ Implemented, flag OFF by default |

**Description:** Emergency halt order. The DW polls every 30 s; if it detects a kind:34005 more recent than the last kind:34006, it discards all new messages.

```json
{
  "id": "<sha256>",
  "pubkey": "<owner_pubkey_hex>",
  "created_at": 1714000100,
  "kind": 34005,
  "tags": [
    ["d", "<tenant_id>"],
    ["tenant", "<tenant_id>"]
  ],
  "content": "{\"tenant\":\"1\",\"reason\":\"Suspicious activity detected\",\"authorized_by\":\"<owner_npub>\"}",
  "sig": "<schnorr_sig_hex_owner>"
}
```

**Emergency polling query:**
```python
{"kinds": [34005, 34006], "#tenant": ["<tenant_id>"], "limit": 10}
# Active if: latest_stop_at > 0 and latest_stop_at > latest_resume_at
```

---

### kind:34006 — nodus:emergency-resume

| Field | Value |
|-------|-------|
| **Name** | `nodus:emergency-resume` |
| **Layer** | Governance Layer — Emergency |
| **Milestone** | M5 Emergency Controls (implemented) |
| **Publisher** | Owner (tenant admin) via Backoffice |
| **Status** | ✅ Implemented, flag OFF by default |

**Description:** Resumes DW activity for the tenant. Must be more recent than the last kind:34005 to deactivate the emergency.

```json
{
  "kind": 34006,
  "pubkey": "<owner_pubkey_hex>",
  "tags": [["d", "<tenant_id>"], ["tenant", "<tenant_id>"]],
  "content": "{\"tenant\":\"1\",\"reason\":\"Situation resolved\",\"authorized_by\":\"<owner_npub>\"}",
  "sig": "<schnorr_sig_hex_owner>"
}
```

---

### kind:34010 — nodus:kyc-corp-claim (v0.2)

| Field | Value |
|-------|-------|
| **Name** | `nodus:kyc-corp-claim` |
| **Layer** | Governance Layer — Legal Identity |
| **Milestone** | M13 Verifiable Contracts (v0.2) |
| **Publisher** | Owner (tenant admin) via Backoffice |
| **Status** | ✅ Implemented (v0.2) |

**Description:** Link between a legal entity (company) and its cryptographic Nostr identity. Published to the public relay to be verifiable by third parties.

```json
{
  "id": "<sha256>",
  "pubkey": "<owner_pubkey_hex>",
  "created_at": 1714000200,
  "kind": 34010,
  "tags": [
    ["d", "kyc-<tenant_id>"],
    ["t", "nodus-kyc"],
    ["legal_entity", "Nodus Factory SL"],
    ["jurisdiction", "ES"],
    ["registration", "B12345678"],
    ["p", "<verifier_pubkey_hex>", "", "verifier"]
  ],
  "content": "{\"legal_entity\":\"Nodus Factory SL\",\"jurisdiction\":\"ES\",\"registration\":\"B12345678\",\"tenant_id\":\"1\",\"nodus_version\":\"0.2\"}",
  "sig": "<schnorr_sig_hex_owner>"
}
```

---

## Visual Summary

```
Session Layer (NIP-16 replaceable)
  10001  MESSAGE_USER        Human → DW
  10002  RESPONSE_DW         DW → Human   (final accumulated)
  10003  HITL_REQUEST        DW → Human   (pending approval)
  10004  HITL_RESPONSE       Human → DW   (CRYPTOGRAPHICALLY SIGNED)
  10005  RESPONSE_AGENT      Agent → Agent (internal A2A HTTP)
  10006  STREAMING_CHUNK     DW → Human   (real-time chunk)
  10010  A2A_REQUEST [v0.2]  DW A → DW B
  10011  A2A_RESPONSE [v0.2] DW B → DW A
  10012  A2A_STREAM [v0.2]   DW B → DW A  (chunk)
  10013  A2A_ERROR [v0.2]    DW B → DW A  (error)
  10020  INBOX_ITEM          DW/Cron → Human (async HITL)
  10021  INBOX_RESOLVED      Human → (all) (resolution)

Governance Layer (NIP-33 parameterized replaceable)
  34000  nodus:dw-profile       DW/Human identity at the relay
  34001  nodus:org-relation     Owner→DW relationship (hierarchy)
  34002  nodus:policy           Owner-signed mandate (IMMUTABLE)
  34003  nodus:audit-event      Immutable append-only audit
  34004  nodus:mcp-profile      MCP Gateway profile
  34005  nodus:emergency-stop   Halt all tenant DWs
  34006  nodus:emergency-resume Resume DWs
  34010  nodus:kyc-corp-claim [v0.2] Verifiable legal identity
```

---

*Nodus Factory · © 2026 · Reference Implementation of the Nodus Protocol (CC BY 4.0)*
