# Nodus Protocol — Event Kinds Reference

> **Protocol version:** 1.0  
> **Status:** First Public Release  
> **License:** CC BY 4.0

This document is the canonical reference for all Nostr event kinds used in the Nodus Protocol.

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
| **Publisher** | Human |

**Description:** User message directed at a specific DW. The DW listens via `#p` subscription on the relay.

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
    ["request", "<request_uuid>"]
  ],
  "content": "Send an email to alice@example.com with the meeting summary",
  "sig": "<schnorr_sig_hex>"
}
```

**Required tags:** `p` (target DW), `session`, `request`

---

### kind:10002 — RESPONSE_DW

| Field | Value |
|-------|-------|
| **Name** | `RESPONSE_DW` |
| **Layer** | Session Layer |
| **Publisher** | DW |

**Description:** Final accumulated DW response. Always preceded by zero or more kind:10006 streaming chunks.

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
  "content": "I sent the email to alice@example.com.",
  "sig": "<schnorr_sig_hex>"
}
```

**Required tags:** `p`, `session`, `request`, `agent`  
**Optional tags:** `delegation` (when NIP-26 delegation is active)

---

### kind:10003 — HITL_REQUEST

| Field | Value |
|-------|-------|
| **Name** | `HITL_REQUEST` |
| **Layer** | Session Layer |
| **Publisher** | DW |

**Description:** Human approval request. Published when the DW determines an action requires human authorisation per its mandate. The DW pauses execution and waits for kind:10004.

```json
{
  "id": "<sha256>",
  "pubkey": "<dw_pubkey_hex>",
  "created_at": 1714000002,
  "kind": 10003,
  "tags": [
    ["p", "<user_pubkey_hex>"],
    ["session", "<session_uuid>"],
    ["request", "<request_uuid>"]
  ],
  "content": "{\"event_id\":\"hitl_a3b4c5d6e7f8\",\"action\":\"send_email\",\"description\":\"Send email to alice@example.com\",\"input_type\":\"text\",\"choices\":null,\"urgency\":\"normal\"}",
  "sig": "<schnorr_sig_hex>"
}
```

**Content JSON fields:**

| Field | Type | Description |
|-------|------|-------------|
| `event_id` | `string` | Internal HITL ID (`hitl_<12hex>`) |
| `action` | `string` | Action name requiring approval |
| `description` | `string` | Human-readable request text |
| `input_type` | `"text"\|"choice"` | Expected response type |
| `choices` | `string[]\|null` | Options if `input_type=choice` |
| `urgency` | `"normal"\|"high"` | Urgency level |

---

### kind:10004 — HITL_RESPONSE

| Field | Value |
|-------|-------|
| **Name** | `HITL_RESPONSE` |
| **Layer** | Session Layer |
| **Publisher** | Human |

**Description:** Human response to a kind:10003. Cryptographically signed by the human. Constitutes the **cryptographic proof** of the human decision.

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
- **NIP-07:** `window.nostr.signEvent(event)` — via browser extension (Alby, nos2x, etc.)
- **Custodial:** server-side signing with user's encrypted keypair

---

### kind:10005 — RESPONSE_AGENT

| Field | Value |
|-------|-------|
| **Name** | `RESPONSE_AGENT` |
| **Layer** | Session Layer |
| **Publisher** | Agent (internal A2A) |

**Description:** Response from one agent to another in synchronous A2A communication. Different from kinds 10010–10013 which is Nostr-native A2A.

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
| **Publisher** | DW |

**Description:** Real-time streaming chunk. Allows the client to display the DW response progressively before the final kind:10002 is published.

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

### kind:10010 — A2A_REQUEST

| Field | Value |
|-------|-------|
| **Name** | `A2A_REQUEST` |
| **Layer** | Session Layer — A2A sublayer |
| **Publisher** | Sender DW |

**Description:** Task execution request from one DW to another DW, 100% via Nostr relay, without an HTTP server intermediary.

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

### kind:10011 — A2A_RESPONSE

| Field | Value |
|-------|-------|
| **Name** | `A2A_RESPONSE` |
| **Layer** | Session Layer — A2A sublayer |
| **Publisher** | Receiver DW |

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

### kind:10012 — A2A_STREAM

| Field | Value |
|-------|-------|
| **Name** | `A2A_STREAM` |
| **Layer** | Session Layer — A2A sublayer |
| **Publisher** | Receiver DW |

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

### kind:10013 — A2A_ERROR

| Field | Value |
|-------|-------|
| **Name** | `A2A_ERROR` |
| **Layer** | Session Layer — A2A sublayer |
| **Publisher** | Receiver DW |

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
| **Publisher** | DW |

**Description:** Asynchronous approval request. Used for actions initiated outside of an active conversation (scheduled jobs, long-running processes). Accumulates in an inbox for the owner.

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
| **Publisher** | Human |

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

---

### kind:34000 — nodus:dw-profile / nodus:human-profile

| Field | Value |
|-------|-------|
| **Name** | `nodus:dw-profile` / `nodus:human-profile` |
| **Layer** | Governance Layer — Identity |
| **Publisher** | DW (for DW profiles) / Human client (for human profiles) |

**Description:** Full actor profile (DW or human). Contains capabilities, available transports, and metadata.

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
  "content": "{\"name\":\"Athena\",\"description\":\"Root orchestrator agent\",\"owner\":\"<owner_hex>\",\"entity_type\":\"digital_worker\",\"capabilities\":[\"orchestrate\",\"email\"],\"limits\":[],\"transports\":[{\"type\":\"nostr-session\",\"relay\":\"ws://relay.example\",\"kinds\":[10001]}],\"nodus_version\":\"1.0\"}",
  "sig": "<schnorr_sig_hex>"
}
```

**Content JSON fields:**

| Field | Type | Description |
|-------|------|-------------|
| `name` | `string` | DW display name |
| `description` | `string` | Human-readable description |
| `owner` | `string\|null` | Owner pubkey hex |
| `entity_type` | `"digital_worker"\|"human"` | Entity type (REQUIRED) |
| `capabilities` | `string[]` | Declared capabilities |
| `limits` | `string[]` | Declared limitations |
| `transports` | `Transport[]` | Available communication channels |
| `nodus_version` | `string` | Protocol version (`"1.0"`) |

**Public discovery profile (opt-in):**
```json
{
  "kind": 34000,
  "tags": [
    ["d", "<dw_pubkey_hex>"],
    ["p", "<dw_pubkey_hex>"],
    ["t", "nodus-dw"]
  ],
  "content": "{\"name\":\"Athena\",\"about\":\"Root orchestrator\",\"nodus_version\":\"1.0\",\"capabilities\":[\"orchestrate\",\"email\"],\"limits\":[]}"
}
```

Discovery query (public relay): `{"kinds": [34000], "#t": ["nodus-dw"]}`

---

### kind:34001 — nodus:org-relation

| Field | Value |
|-------|-------|
| **Name** | `nodus:org-relation` |
| **Layer** | Governance Layer — Hierarchy |
| **Publisher** | Owner |

**Description:** Declares the `OwnerOf` hierarchical relationship between an Owner and a DW. Signed by the owner. Used for federation discovery via `relay_hint` tag.

```json
{
  "id": "<sha256>",
  "pubkey": "<owner_pubkey_hex>",
  "created_at": 1714000000,
  "kind": 34001,
  "tags": [
    ["d", "<owner_pubkey_hex>-<dw_pubkey_hex>"],
    ["p", "<dw_pubkey_hex>"]
  ],
  "content": "{\"relation\":\"OwnerOf\",\"subject\":\"<owner_pubkey_hex>\",\"object\":\"<dw_pubkey_hex>\",\"nodus_version\":\"1.0\"}",
  "sig": "<schnorr_sig_hex_owner>"
}
```

**Cross-tenant federation variant:**
```json
{
  "kind": 34001,
  "tags": [
    ["d", "<owner_hex>-<external_dw_hex>"],
    ["p", "<external_dw_pubkey_hex>"],
    ["relay_hint", "wss://relay.org-b.example"],
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
| **Publisher** | Owner |

**Description:** Mandate defining exactly what the DW can and cannot do. **Immutable**: the relay MUST reject any DELETE or UPDATE. Signed by the owner.

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
  "content": "{\"dw\":\"<dw_pubkey_hex>\",\"capabilities\":[\"send_email\",\"read_calendar\",\"orchestrate\"],\"limits\":[\"no_delete_without_confirmation\"],\"hitl_required\":[\"send_email\",\"delete_*\",\"financial_*\"],\"auto_approve\":[\"read_calendar\"],\"max_auto_cost_eur\":0,\"valid_from\":1714000000,\"valid_until\":null,\"nodus_version\":\"1.0\"}",
  "sig": "<schnorr_sig_hex_owner>"
}
```

**Content JSON fields:**

| Field | Type | Description |
|-------|------|-------------|
| `dw` | `string` | DW pubkey hex |
| `capabilities` | `string[]` | Permitted actions (wildcards: `delete_*`) |
| `limits` | `string[]` | Explicit restrictions |
| `hitl_required` | `string[]` | Actions requiring human approval |
| `auto_approve` | `string[]` | Always auto-approved (overrides `hitl_required`) |
| `max_auto_cost_eur` | `number` | Maximum auto-approved cost in EUR |
| `valid_from` | `number` | Unix timestamp — validity start |
| `valid_until` | `number\|null` | Unix timestamp — validity end (null = indefinite) |
| `nodus_version` | `string` | Protocol version |

> **Critical relay rule:** Relays MUST reject any DELETE or UPDATE on kind:34002.

**Mandate query by DW:**
```json
{"kinds": [34002], "#d": ["<dw_pubkey_hex>"], "limit": 1}
```

---

### kind:34003 — nodus:audit-event

| Field | Value |
|-------|-------|
| **Name** | `nodus:audit-event` |
| **Layer** | Governance Layer — Audit |
| **Publisher** | DW |

**Description:** Immutable, append-only audit record. The `d` tag is `sha256(dw_pubkey + session_id + timestamp_ms)` to guarantee uniqueness. The relay MUST block overwrites.

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
    ["action", "agent_response"]
  ],
  "content": "{\"action\":\"agent_response\",\"result_hash\":\"<sha256_of_result>\",\"timestamp\":1714000010,\"dw\":\"<dw_pubkey_hex>\",\"session_id\":\"<uuid>\",\"mandate_ref\":\"<event_id_or_null>\"}",
  "sig": "<schnorr_sig_hex>"
}
```

**Action variants:**
- `agent_response` — final agent response
- `hitl_request:<action>` — HITL request published
- `hitl_decision:approved` / `hitl_decision:rejected` — human decision received

> **Critical relay rule:** Relays MUST reject any DELETE or UPDATE on kind:34003.

---

### kind:34004 — nodus:mcp-server-profile

| Field | Value |
|-------|-------|
| **Name** | `nodus:mcp-server-profile` |
| **Layer** | Governance Layer — MCP |
| **Publisher** | MCP Gateway |

**Description:** MCP Gateway profile. DWs MUST verify this profile before calling any tool via the gateway.

```json
{
  "kind": 34004,
  "pubkey": "<mcp_gateway_pubkey_hex>",
  "tags": [["d", "<gateway_identifier>"]],
  "content": "{\"name\":\"Example MCP Gateway\",\"url\":\"https://mcp.example.org\",\"tools\":[\"calendar\",\"email\",\"drive\"],\"owner\":\"<owner_pubkey_hex>\",\"authorized_dws\":[\"<dw_pubkey_hex>\"],\"valid_until\":null}"
}
```

---

### kind:34005 — nodus:emergency-stop

| Field | Value |
|-------|-------|
| **Name** | `nodus:emergency-stop` |
| **Layer** | Governance Layer — Emergency |
| **Publisher** | Owner |

**Description:** Emergency halt order. DWs MUST poll every 30 seconds; if a kind:34005 more recent than the last kind:34006 is detected, the DW MUST discard all new messages.

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
  "content": "{\"tenant\":\"<tenant_id>\",\"reason\":\"Suspicious activity detected\",\"authorized_by\":\"<owner_npub>\"}",
  "sig": "<schnorr_sig_hex_owner>"
}
```

**Emergency polling query:**
```json
{"kinds": [34005, 34006], "#tenant": ["<tenant_id>"], "limit": 10}
```

**Active condition:** `latest_stop_at > 0 AND latest_stop_at > latest_resume_at`

---

### kind:34006 — nodus:emergency-resume

| Field | Value |
|-------|-------|
| **Name** | `nodus:emergency-resume` |
| **Layer** | Governance Layer — Emergency |
| **Publisher** | Owner |

**Description:** Resumes DW activity. MUST be more recent than the last kind:34005 to deactivate the emergency.

```json
{
  "kind": 34006,
  "pubkey": "<owner_pubkey_hex>",
  "tags": [["d", "<tenant_id>"], ["tenant", "<tenant_id>"]],
  "content": "{\"tenant\":\"<tenant_id>\",\"reason\":\"Situation resolved\",\"authorized_by\":\"<owner_npub>\"}",
  "sig": "<schnorr_sig_hex_owner>"
}
```

---

### kind:34010 — nodus:kyc-corp-claim

| Field | Value |
|-------|-------|
| **Name** | `nodus:kyc-corp-claim` |
| **Layer** | Governance Layer — Legal Identity |
| **Publisher** | Owner |

**Description:** Link between a legal entity (company) and its cryptographic Nostr identity. Published to the public relay to be verifiable by third parties without contacting any central authority.

```json
{
  "id": "<sha256>",
  "pubkey": "<owner_pubkey_hex>",
  "created_at": 1714000200,
  "kind": 34010,
  "tags": [
    ["d", "kyc-<tenant_id>"],
    ["t", "nodus-kyc"],
    ["legal_entity", "Example Corp SL"],
    ["jurisdiction", "ES"],
    ["registration", "B12345678"],
    ["p", "<verifier_pubkey_hex>", "", "verifier"]
  ],
  "content": "{\"legal_entity\":\"Example Corp SL\",\"jurisdiction\":\"ES\",\"registration\":\"B12345678\",\"nodus_version\":\"1.0\"}",
  "sig": "<schnorr_sig_hex_owner>"
}
```

---

## Visual Summary

```
Session Layer (NIP-16 replaceable)
  10001  MESSAGE_USER        Human → DW
  10002  RESPONSE_DW         DW → Human     (final accumulated)
  10003  HITL_REQUEST        DW → Human     (pending approval)
  10004  HITL_RESPONSE       Human → DW     (CRYPTOGRAPHICALLY SIGNED)
  10005  RESPONSE_AGENT      Agent → Agent  (synchronous A2A)
  10006  STREAMING_CHUNK     DW → Human     (real-time chunk)
  10010  A2A_REQUEST         DW A → DW B    (Nostr-native)
  10011  A2A_RESPONSE        DW B → DW A
  10012  A2A_STREAM          DW B → DW A    (chunk)
  10013  A2A_ERROR           DW B → DW A    (error)
  10020  INBOX_ITEM          DW → Human     (async HITL)
  10021  INBOX_RESOLVED      Human → DW     (resolution)

Governance Layer (NIP-33 parameterized replaceable)
  34000  nodus:dw-profile       DW/Human identity at the relay
  34001  nodus:org-relation     Owner→DW relationship (hierarchy + federation)
  34002  nodus:policy           Owner-signed mandate (IMMUTABLE)
  34003  nodus:audit-event      Immutable append-only audit
  34004  nodus:mcp-profile      MCP Gateway profile
  34005  nodus:emergency-stop   Halt all tenant DWs (<30 seconds)
  34006  nodus:emergency-resume Resume DWs
  34010  nodus:kyc-corp-claim   Verifiable legal identity
```

---

*Nodus Protocol Working Group · nodus.social · CC BY 4.0*
