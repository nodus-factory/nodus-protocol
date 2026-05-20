# Response Bubbles — Session kinds 10007–10009

> **Status:** Normative (Session Layer extension)  
> **Issues:** [nodus-protocol#5](https://github.com/nodus-factory/nodus-protocol/issues/5), [nodus-protocol#6](https://github.com/nodus-factory/nodus-protocol/issues/6)  
> **Tags:** [SESSION_TAGS.md](./SESSION_TAGS.md)

These kinds enable **async, multi-delivery** responses over Nostr (Telegram-style activity inside a notebook page). They do not replace kind:10002 or kind:10006; they complement them for long-running work.

---

## kind:10007 — BUBBLE_EVENT

| Field | Value |
|-------|-------|
| **Layer** | Session |
| **Publisher** | Worker or Cron |
| **Direction** | Worker → Initiator (and Llibreta projector) |

**Description:** Structured activity inside a bubble: tool calls, partial messages, artifacts, non-final errors.

**Required tags:** `p`, `session`, `request`, `bubble` (see [SESSION_TAGS.md](./SESSION_TAGS.md))

**Recommended tags:** `phase`, `event`, `worker`, `seq`

```json
{
  "kind": 10007,
  "pubkey": "<worker_pubkey_hex>",
  "created_at": 1714000002,
  "tags": [
    ["p", "<initiator_pubkey_hex>"],
    ["session", "<session_id>"],
    ["request", "<request_uuid>"],
    ["bubble", "<bubble_uuid_v4>"],
    ["phase", "partial"],
    ["event", "message"]
  ],
  "content": "{\"type\":\"message\",\"text\":\"He trobat 8 emails\",\"metadata\":{}}",
  "sig": "<schnorr_sig_hex>"
}
```

### Content JSON

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | Yes | Logical event type (see table below) |
| `text` | string | No | Human-readable payload |
| `metadata` | object | No | Tool names, URLs, artifact refs, etc. |

**`type` values (normative):**

| `type` | Projector role | Typical UI |
|--------|----------------|------------|
| `message` | assistant | Text |
| `progress` | system | Status line |
| `thinking` | assistant | Hidden / trace |
| `tool_call` | tool | Icon + name |
| `tool_result` | tool | Summary |
| `artifact` | system | Card / link |
| `error` | system | Non-final error |

The optional tag `event` **MAY** duplicate `content.type` for filters that do not parse JSON.

---

## kind:10008 — BUBBLE_STATUS

| Field | Value |
|-------|-------|
| **Layer** | Session |
| **Publisher** | Worker |
| **Direction** | Worker → Initiator |

**Description:** Bubble lifecycle state change (`running`, `waiting`, `completed`, `failed`).

**Required tags:** `p`, `session`, `request`, `bubble`

```json
{
  "kind": 10008,
  "pubkey": "<worker_pubkey_hex>",
  "created_at": 1714000003,
  "tags": [
    ["p", "<initiator_pubkey_hex>"],
    ["session", "<session_id>"],
    ["request", "<request_uuid>"],
    ["bubble", "<bubble_uuid_v4>"]
  ],
  "content": "{\"status\":\"waiting\",\"reason\":\"hitl\"}",
  "sig": "<schnorr_sig_hex>"
}
```

### Content JSON

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `status` | string | Yes | `running` \| `waiting` \| `completed` \| `failed` |
| `reason` | string | No | e.g. `hitl`, `error`, `cancelled` |

**Note:** kind:10009 is the canonical **close** signal. kind:10008 with `status=completed` is informational; projectors **SHOULD** treat kind:10009 as the authoritative completion timestamp.

---

## kind:10009 — BUBBLE_FINAL

| Field | Value |
|-------|-------|
| **Layer** | Session |
| **Publisher** | Worker |
| **Direction** | Worker → Initiator |

**Description:** Explicit end of a response bubble. Projectors **MUST** set bubble `status=completed` and `completed_at` on this event.

**Required tags:** `p`, `session`, `request`, `bubble`

**Content:** Plain text **or** JSON:

```json
{ "summary": "He processat 8 emails i n'he arxivat 3." }
```

```json
{
  "kind": 10009,
  "pubkey": "<worker_pubkey_hex>",
  "created_at": 1714020000,
  "tags": [
    ["p", "<initiator_pubkey_hex>"],
    ["session", "<session_id>"],
    ["request", "<request_uuid>"],
    ["bubble", "<bubble_uuid_v4>"],
    ["phase", "final"]
  ],
  "content": "{\"summary\":\"He processat 8 emails i n'he arxivat 3.\"}",
  "sig": "<schnorr_sig_hex>"
}
```

Workers **SHOULD** prefer kind:10007 for intermediate updates and **MUST** publish kind:10009 when the bubble is done (instead of relying on HTTP or an open SSE connection).

---

## Legacy kinds (compatibility)

| Kind | Projector behavior |
|------|-------------------|
| **10006** | Append `type=message` text; does **not** close the bubble |
| **10002** | Append full `type=message` text; closes bubble **only** if tag `bubble_done=true` **or** kind:10009 already exists |
| **10003** | `type=hitl`; bubble → `waiting` |

Implementations that do not yet emit 10007–10009 **MAY** continue using 10002/10006 with `bubble` for correlation.

---

## Subscription examples

Indexer / UI (all events for one bubble):

```json
{
  "kinds": [10002, 10003, 10006, 10007, 10008, 10009],
  "#bubble": ["<bubble_uuid_v4>"]
}
```

DW inbox (unchanged, plus bubble on publish):

```json
{"kinds": [10001], "#p": ["<worker_pubkey_hex>"], "since": <now - 30>}
```
