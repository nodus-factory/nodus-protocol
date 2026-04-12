# Nodus Protocol — Implementation Guide

> **Protocol version:** 1.0  
> **Status:** First Public Release  
> **License:** CC BY 4.0

This guide describes how to build a Nodus Protocol-conformant system. It is addressed to implementors who want to create a Digital Worker (DW) runtime, a governance relay, or a client application.

For the protocol specification, see [SPEC.md](SPEC.md). For complete kind reference, see [KINDS.md](KINDS.md). For protocol flows, see [FLOWS.md](FLOWS.md).

---

## 1. What You Need to Build

A minimal Nodus Protocol implementation consists of four components:

1. **DW runtime** — generates keypairs, publishes kind:34000, receives kind:10001, publishes kind:10002/10003/10006
2. **Governance relay** — a Nostr relay with write policies enforcing mandate immutability and DW/human separation
3. **Owner client** — creates and signs kind:34001, kind:34002; publishes kind:34005/34006; signs kind:10004 HITL responses
4. **Audit subscriber** — subscribes to kind:34003 for compliance and audit

Optional extensions:
- **Policy Relay** — holds DW private keys, enforces mandates at signing time (recommended for production)
- **A2A bridge** — implements kinds 10010–10013 for DW-to-DW Nostr-native communication
- **Federation bridge** — publishes to and subscribes from external relays via `relay_hint` discovery

---

## 2. DW Keypair Generation

Each DW MUST have a secp256k1 keypair. The `nsec` (private key) MUST be generated with a cryptographically secure random number generator.

**TypeScript (nostr-tools):**
```typescript
import { generatePrivateKey, getPublicKey } from 'nostr-tools'
const nsec = generatePrivateKey()          // 32-byte hex
const npub = getPublicKey(nsec)            // 32-byte hex
```

**Python (cryptography library):**
```python
from cryptography.hazmat.primitives.asymmetric.ec import generate_private_key, SECP256K1
from cryptography.hazmat.backends import default_backend
key = generate_private_key(SECP256K1(), default_backend())
```

> **Private key storage:** DW `nsec` MUST be stored securely. In production, the recommended architecture is the Policy Relay (section 12), where the DW process never holds its own key.

---

## 3. Publishing a DW Profile (kind:34000)

On startup, a DW MUST publish its profile to the relay:

```json
{
  "kind": 34000,
  "pubkey": "<dw_pubkey_hex>",
  "tags": [
    ["d", "<dw_pubkey_hex>"],
    ["p", "<owner_pubkey_hex>"]
  ],
  "content": "{
    \"name\": \"<dw_name>\",
    \"entity_type\": \"digital_worker\",
    \"capabilities\": [\"<capability_1>\", \"<capability_2>\"],
    \"limits\": [],
    \"transports\": [{
      \"type\": \"nostr-session\",
      \"relay\": \"<relay_wss_url>\",
      \"kinds\": [10001]
    }],
    \"nodus_version\": \"1.0\"
  }"
}
```

Sign the event with BIP-340 Schnorr (secp256k1). Publish via WebSocket `["EVENT", <event>]` message to the relay.

---

## 4. Subscribing to Messages (kind:10001)

The DW MUST subscribe using a `#p` filter to receive only messages addressed to it, with a `since` filter to avoid processing stale messages:

```json
["REQ", "<sub_id>", {
  "kinds": [10001],
  "#p": ["<dw_pubkey_hex>"],
  "since": <current_unix_timestamp_minus_30>
}]
```

---

## 5. Publishing a Response (kind:10002)

After processing a kind:10001 message, publish the final response:

```json
{
  "kind": 10002,
  "pubkey": "<dw_pubkey_hex>",
  "tags": [
    ["p", "<user_pubkey_hex>"],
    ["session", "<session_uuid>"],
    ["request", "<request_uuid>"],
    ["agent", "<agent_name>"],
    ["delegation", "<owner_pubkey>", "<conditions>", "<sig>"]
  ],
  "content": "<response_text>"
}
```

The `delegation` tag is REQUIRED when NIP-26 delegation is active (see section 7). The `session` and `request` tags MUST match the values from the incoming kind:10001.

---

## 6. Mandate Lookup (kind:34002)

Before acting, the DW MUST fetch and validate its mandate:

```json
["REQ", "<sub_id>", {
  "kinds": [34002],
  "#d": ["<dw_pubkey_hex>"],
  "limit": 1
}]
```

Parse the `content` JSON. Validate:

1. `valid_from` ≤ current timestamp
2. `valid_until` is null OR > current timestamp
3. Requested capability is present in `capabilities` array
4. Requested capability is not in `limits` array
5. If the action is in `hitl_required`, initiate HITL flow (section 8) before proceeding
6. If the action is in `auto_approve`, proceed without HITL regardless of `hitl_required`

If no valid mandate exists, the DW MUST refuse the action and publish a kind:10002 explaining the refusal.

---

## 7. NIP-26 Delegation

The owner generates a delegation token pre-authorising the DW to act on their behalf. The DW includes it in all events as a `delegation` tag.

**Delegation tag format:**
```
["delegation", "<delegator_pubkey_hex>", "<conditions_string>", "<delegation_sig_hex>"]
```

**Conditions string format:**
```
kind=10001&kind=10002&kind=10003&kind=10006&created_at><valid_from_unix_ts>&created_at<<valid_until_unix_ts>
```

The delegation token MUST be pre-computed by the owner and securely provided to the DW at initialisation time.

**Verification (any third party):**
1. Parse the `delegation` tag: `[_, delegator_pubkey, conditions, sig]`
2. Compute: `sha256("nostr:delegation:" + event.pubkey + ":" + conditions)`
3. Verify Schnorr signature `sig` over that hash using `delegator_pubkey`
4. Parse `conditions` to confirm kind and timestamp constraints are satisfied

---

## 8. HITL Flow (kinds 10003 / 10004)

When an action is in `hitl_required` and not in `auto_approve`:

**Step 1 — DW publishes HITL request (kind:10003):**
```json
{
  "kind": 10003,
  "pubkey": "<dw_pubkey_hex>",
  "tags": [
    ["p", "<user_pubkey_hex>"],
    ["session", "<session_uuid>"],
    ["request", "<request_uuid>"]
  ],
  "content": "{
    \"event_id\": \"hitl_<12hex>\",
    \"action\": \"<action_name>\",
    \"description\": \"<human_readable_request>\",
    \"input_type\": \"text\",
    \"urgency\": \"normal\"
  }"
}
```

**Step 2 — DW subscribes and waits for kind:10004:**
```json
["REQ", "<sub_id>", {
  "kinds": [10004],
  "#request": ["<kind:10003_event_id>"],
  "limit": 1
}]
```

**Step 3 — DW validates the response:**
- Verify `event.pubkey` is the owner's pubkey (or an authorised human from a cross-tenant kind:34001)
- Check `approved` tag: `"true"` to proceed, `"false"` to abort
- Verify the event signature is valid

**Step 4 — DW records the HITL decision in audit (kind:34003):**
```
action: "hitl_decision:approved" or "hitl_decision:rejected"
```

---

## 9. Audit Logging (kind:34003)

After each significant action, the DW MUST publish an audit event:

```json
{
  "kind": 34003,
  "pubkey": "<dw_pubkey_hex>",
  "tags": [
    ["d", "<sha256(dw_pubkey_hex + session_id + timestamp_ms)>"],
    ["p", "<user_pubkey_hex>"],
    ["mandate", "<kind:34002_event_id>"],
    ["session", "<session_uuid>"],
    ["action", "<action_name>"]
  ],
  "content": "{
    \"action\": \"<action_name>\",
    \"result_hash\": \"<sha256_of_result>\",
    \"timestamp\": <unix_ts>,
    \"dw\": \"<dw_pubkey_hex>\",
    \"session_id\": \"<uuid>\",
    \"mandate_ref\": \"<event_id_or_null>\"
  }"
}
```

**`d` tag uniqueness:** `sha256(dw_pubkey_hex + session_id + timestamp_ms)` ensures no two audit events for the same DW+session can share a `d` value, enforcing append-only semantics at the relay level.

**Action variants:**
- `agent_response` — final agent response
- `hitl_request:<action>` — HITL request published
- `hitl_decision:approved` / `hitl_decision:rejected` — human decision received
- `mandate_check:permitted` / `mandate_check:denied` — mandate validation result

---

## 10. Emergency Stop Polling (kinds 34005 / 34006)

The DW MUST poll for emergency signals at least every 30 seconds:

```json
["REQ", "<sub_id>", {
  "kinds": [34005, 34006],
  "#tenant": ["<tenant_id>"],
  "limit": 10
}]
```

**Emergency active condition:**
```
latest_stop_at > 0 AND latest_stop_at > latest_resume_at
```

Where `latest_stop_at` is the `created_at` of the most recent kind:34005, and `latest_resume_at` is the `created_at` of the most recent kind:34006.

When active, the DW MUST:
1. Discard all incoming kind:10001 messages without processing
2. Continue polling for kind:34006 (resume signal)
3. Log the emergency state

---

## 11. Governance Relay Write Policy

A conformant relay MUST enforce the following rules at write time:

| Rule | Condition | Action |
|------|-----------|--------|
| R1 | Event kind is 34002 AND method is DELETE | REJECT |
| R2 | Event kind is 34003 AND method is DELETE | REJECT |
| R3 | Event kind is 34002 or 34005 AND author has `entity_type: "digital_worker"` | REJECT |
| R4 | DW event AND `REQUIRE_DELEGATION=true` AND no valid `delegation` tag | REJECT |
| R5 | kind:34005 active for tenant AND event is from a DW (not owner) | REJECT |

**Checking R3:** The relay must fetch the author's kind:34000 profile and inspect `entity_type` in the `content` JSON. This lookup SHOULD be cached (the profile is replaceable but rarely changes).

**Checking R5:** The relay must maintain a per-tenant emergency state, updated whenever a kind:34005 or kind:34006 event is accepted.

---

## 12. Policy Relay (Recommended for Production)

The Policy Relay holds DW private keys and enforces mandates at signing time. The DW process never holds its own `nsec`.

**Architecture:**

```
DW process (no nsec)
    │
    │  WebSocket connection to Policy Relay
    │  {"method": "sign_event", "params": {"event": <unsigned_event>, "dw_pubkey": "<hex>"}}
    ▼
Policy Relay (isolated process)
    ├── Holds DW nsec (loaded from secure storage at startup)
    ├── Fetches DW mandate (kind:34002) from relay
    ├── Validates: is the action permitted by the mandate?
    ├── If YES: signs event with DW nsec → returns {"result": <signed_event>}
    └── If NO: returns {"error": "mandate_violation: <reason>"}
```

**WebSocket protocol:** See SPEC.md section 5.6 for the full message format.

**Security properties:**
- DW process compromise does not expose the private key
- Mandate enforcement is cryptographic (the DW cannot publish a non-compliant event)
- All sign requests SHOULD be logged for audit

**Key storage options (in order of security):**
1. Hardware Security Module (HSM)
2. Secrets management service (Vault, AWS Secrets Manager, etc.)
3. Encrypted file with key derived from an environment variable
4. Environment variable (minimum viable, not recommended for sensitive deployments)

---

## 13. Human Owner Client

The owner client creates and signs governance events. It requires access to the owner's `nsec`.

**Publishing a mandate (kind:34002):**

```typescript
const mandate = {
  dw: dwPubkeyHex,
  capabilities: ["send_email", "read_calendar"],
  limits: ["no_financial_without_hitl"],
  hitl_required: ["send_email", "delete_*"],
  auto_approve: ["read_calendar"],
  max_auto_cost_eur: 0,
  valid_from: Math.floor(Date.now() / 1000),
  valid_until: null,
  nodus_version: "1.0"
}

const event = {
  kind: 34002,
  pubkey: ownerPubkeyHex,
  created_at: Math.floor(Date.now() / 1000),
  tags: [["d", dwPubkeyHex], ["p", dwPubkeyHex]],
  content: JSON.stringify(mandate)
}
// Sign with owner nsec (BIP-340 Schnorr) and publish to relay
```

**Signing a HITL response (kind:10004):**

The owner's client SHOULD use NIP-07 (browser extension: Alby, nos2x) for key management, so the `nsec` never leaves the browser.

```typescript
// NIP-07 signing
const event = {
  kind: 10004,
  pubkey: humanPubkeyHex,
  created_at: Math.floor(Date.now() / 1000),
  tags: [
    ["request", hitlRequestEventId],
    ["approved", "true"]
  ],
  content: "approved"
}
const signed = await window.nostr.signEvent(event)
// Publish signed event to relay
```

---

## 14. Testing Conformance

To verify your implementation against the protocol:

**Identity:**
```
REQ {kinds:[34000], "#d":["<dw_pubkey_hex>"]}
→ MUST return event with entity_type: "digital_worker"
```

**Mandate immutability:**
```
Send: ["EVENT", {kind:1, id:..., ...}]  -- first
Send: ["EVENT", {kind:34002, id:..., ...}] -- DELETE attempt
→ Relay MUST respond: ["OK", id, false, "blocked: mandate immutable"]
```

**NIP-26 delegation:**
```
Inspect kind:10002 events from DW
→ MUST contain ["delegation", owner_pubkey, conditions, sig] tag
→ sig MUST verify against sha256("nostr:delegation:" + dw_pubkey + ":" + conditions)
```

**HITL flow:**
```
Send kind:10001 with an action in hitl_required
→ DW MUST publish kind:10003 within 5 seconds
→ DW MUST NOT publish kind:10002 until kind:10004 received
```

**Audit:**
```
After each agent response, query:
REQ {kinds:[34003], authors:["<dw_pubkey_hex>"], "#session":["<session_id>"]}
→ MUST return at least one audit event
```

**Emergency stop:**
```
Publish kind:34005 for tenant
Wait 35 seconds
Send kind:10001 to DW
→ DW MUST NOT respond (must discard)
```

---

*Nodus Protocol Working Group · nodus.social · CC BY 4.0*
