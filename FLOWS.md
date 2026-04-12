# Nodus Protocol — Protocol Flows

> **Protocol version:** v0.2  
> **Status:** Release Candidate  
> **Last updated:** April 2026  
> **Reference implementation:** Nodus OS ADK (private)  
> **License:** CC BY 4.0

This document describes the primary protocol flows extracted directly from the reference implementation. Each flow includes the sequence diagram, involved kinds, and references to specific implementation files.

---

## Flow 1: Normal Work Session (M0–M3)

**Components:** `nodus-llibreta-v2` (frontend), `nostr_adk_transport.py`, strfry relay  
**Feature flags:** `NOSTR_ADK_TRANSPORT_V1`, optionally `NODUS_PROTOCOL_MANDATES_V1`, `NODUS_PROTOCOL_AUDIT_V1`

```
Human (frontend)                    Relay (strfry)            DW (nodus-adk-runtime)
      │                                  │                           │
      │  kind:10001 MESSAGE_USER          │                           │
      │  tags: [p=DW, session, request]  │                           │
      ├─────────────────────────────────►│                           │
      │                                  │  EVENT → #p subscription  │
      │                                  ├──────────────────────────►│
      │                                  │                           │ _handle_message()
      │                                  │                           │
      │                                  │                           │ [if MANDATES_V1]
      │                                  │◄──────────────────────────┤ REQ kind:34002 #d=DW
      │                                  │  kind:34002 mandate       │
      │                                  ├──────────────────────────►│ MandateService.get_active_mandate()
      │                                  │                           │ → is_action_permitted()
      │                                  │                           │
      │                                  │                           │ runner.run_async() [streaming]
      │                                  │                           │
      │  kind:10006 STREAMING_CHUNK       │                           │ (per Part.text)
      │◄─────────────────────────────────┤◄──────────────────────────┤
      │  kind:10006 STREAMING_CHUNK       │                           │
      │◄─────────────────────────────────┤◄──────────────────────────┤
      │                                  │                           │
      │                                  │                           │ [if AUDIT_V1]
      │                                  │◄──────────────────────────┤ EVENT kind:34003 (audit)
      │                                  │                           │
      │  kind:10002 RESPONSE_DW           │                           │
      │◄─────────────────────────────────┤◄──────────────────────────┤ (final accumulated response)
```

**Key implementation:**
- `nostr_adk_transport.py` → `_listen_loop()` subscribes to kind:10001 with `#p` filter
- `_handle_message()` calls `_build_agent_for_user()` + `runner.run_async()`
- Per text chunk → `_publish(ws, KIND_STREAMING_CHUNK, text, base_tags)`
- Final → `_publish(ws, KIND_RESPONSE_DW, full_response, final_tags)`
- If `NODUS_PROTOCOL_DELEGATION_V1` active → all events carry `["delegation", owner_pubkey, conditions, sig]`

---

## Flow 2: Constitutional HITL (M4)

**Components:** `nostr_adk_transport.py`, `nostr-hitl-signer.ts`, `use-constitutional-hitl.ts`  
**Feature flags:** `NODUS_PROTOCOL_CONSTITUTIONAL_HITL_V1`

```
Human                     Relay                      DW                    HITLService
  │                         │                         │                        │
  │  kind:10001              │                         │                        │
  ├────────────────────────►│                         │                        │
  │                         ├────────────────────────►│                        │
  │                         │                         │ run_async() →          │
  │                         │                         │ ToolConfirmation       │
  │                         │                         │ detected               │
  │                         │                         │                        │
  │                         │                         │ create_event_async()   │
  │                         │                         ├───────────────────────►│
  │                         │                         │                        │ stores HITL
  │                         │                         │                        │ (event_id=hitl_xxx)
  │  kind:10003 HITL_REQUEST│                         │                        │
  │◄────────────────────────┤◄────────────────────────┤                        │
  │  content: {event_id,    │                         │                        │
  │    action, description, │  kind:10002 (placeholder)                        │
  │    input_type}          │◄────────────────────────┤                        │
  │◄────────────────────────┤                         │                        │
  │                         │                         │                        │
  │ [user sees HitlInlineCard]                         │                        │
  │ [NIP-07 available?]     │                         │                        │
  │    → window.nostr.signEvent(kind:10004)            │                        │
  │    or custodial → POST /api/nostr/hitl/respond     │                        │
  │                         │                         │                        │
  │  kind:10004 HITL_RESPONSE│                        │                        │
  │  SIGNED by human         │                        │                        │
  ├────────────────────────►│                         │                        │
  │                         │                         │                        │
  │                         │ [DW detects via HTTP SSE or polling]              │
  │                         │                         │ validates signature    │
  │                         │                         │                        │
  │                         │                         │ [if AUDIT_V1]          │
  │                         │◄────────────────────────┤ kind:34003             │
  │                         │                         │ hitl_decision:approved │
```

**Key implementation:**
- `_detect_and_handle_hitl()` in `nostr_adk_transport.py` — inspects `session.events[-5:]` for `ToolConfirmation`
- `hitl_service.create_event_async()` — registers with existing HITLService (HTTP v1)
- `_publish(ws, KIND_HITL_REQUEST, json.dumps({...}), base_tags)` — publishes kind:10003
- Frontend: `use-constitutional-hitl.ts` → `respond(requestEventId, approved, sessionId)`
  - NIP-07: `window.nostr.signEvent(event)` → `POST /api/nostr/hitl/respond-signed`
  - Custodial: `POST /api/nostr/hitl/respond` → `nostr-hitl-signer.ts:signAndPublishHitlResponse()`

**Custodial signing detail (`nostr-hitl-signer.ts`):**
```typescript
const eventTemplate = {
  kind: 10004,
  created_at: Math.floor(Date.now() / 1000),
  tags: [
    ['request', requestEventId],
    ['session', sessionId],
    ['approved', approved ? 'true' : 'false'],
  ],
  content: approved ? 'approved' : 'rejected',
};
const signed = finalizeEvent(eventTemplate, secretKeyBytes);
```

---

## Flow 3: Emergency Stop (M5)

**Components:** `nostr-emergency-service.ts`, `emergency-routes.ts`, `nostr_adk_transport.py`  
**Feature flags:** `NODUS_PROTOCOL_EMERGENCY_V1`

```
Admin (Backoffice)        Relay                      DW (polling every 30s)
       │                    │                              │
       │  POST /api/emergency/stop                        │
       │  { reason: "Suspicious activity" }               │
       │                    │                              │
       │  publishEmergencyStop()                          │
       │  kind:34005         │                              │
       │  tags: [d=tenant,  │                              │
       │    tenant=tenant_id]│                             │
       ├───────────────────►│                              │
       │                    │                              │
       │  { event_id, ts }  │                              │
       │◄───────────────────┤                              │
       │                    │  (asyncio polling task)      │
       │                    │                              │ _emergency_polling_loop()
       │                    │                              │ → asyncio.sleep(30)
       │                    │                              │ → _query_emergency_stop(tenant_id)
       │                    │◄─────────────────────────────┤
       │                    │  REQ {kinds:[34005,34006],   │
       │                    │       #tenant:[tenant_id]}   │
       │                    ├──────────────────────────────►
       │                    │  EVENT kind:34005            │
       │                    ├──────────────────────────────►
       │                    │  EOSE                        │
       │                    ├──────────────────────────────►
       │                    │                              │ self._emergency_active = True
       │                    │                              │
       │                    │     [new kind:10001 arrives] │
       │                    ├─────────────────────────────►│ _handle_message()
       │                    │                              │ → if self._emergency_active: return
       │                    │                              │   (message discarded)
       │                    │                              │
  [Admin resumes]           │                              │
       │  POST /api/emergency/resume                       │
       │  kind:34006         │                              │
       ├───────────────────►│                              │
       │                    │  (next polling cycle)        │
       │                    │                              │ latest_resume_at > latest_stop_at
       │                    │                              │ self._emergency_active = False
```

**Key implementation:**
- `_emergency_polling_loop()` → asyncio task launched at `start()` if `NODUS_PROTOCOL_EMERGENCY_V1`
- `_query_emergency_stop()` → REQ/EOSE pattern, returns `True` if `latest_stop_at > latest_resume_at`
- `_handle_message()` → first line: `if self._emergency_active: return`

---

## Flow 4: Policy Relay — Delegated Signing (M8)

**Components:** `policy_relay_server.py`, `nostr_adk_transport.py`  
**Feature flags:** `NODUS_POLICY_RELAY_V1` + `NODUS_POLICY_RELAY_URL`

```
DW (NostrAdkTransport)       Policy Relay (WebSocket)         Relay (strfry)
        │                           │                               │
        │ [NODUS_POLICY_RELAY_V1]   │                               │
        │ _publish() called         │                               │
        │                           │                               │
        │  WS: {"id":"...",          │                               │
        │  "method":"sign_event",   │                               │
        │  "params":{               │                               │
        │    "event": {unsigned},   │                               │
        │    "dw_pubkey":"<hex>"    │                               │
        │  }}                       │                               │
        ├──────────────────────────►│                               │
        │                           │ get_dw_nsec(dw_pubkey)        │
        │                           │ (from NODUS_DW_NSEC_MAP)      │
        │                           │                               │
        │                           │ _check_mandate_log_only()     │
        │                           │ (M8.1: observational)         │
        │                           │                               │
        │                           │ _sign_event(unsigned, nsec)   │
        │                           │ → _NostrSigner.create_signed_event()
        │                           │                               │
        │  {"id":"...",             │                               │
        │   "result": {signed_event}}                               │
        │◄──────────────────────────┤                               │
        │                           │                               │
        │  WS: ["EVENT", signed]    │                               │
        ├───────────────────────────┼──────────────────────────────►│
        │                           │  ["OK", event_id, true]       │
        │◄──────────────────────────┼───────────────────────────────┤
```

**Policy Relay WS protocol (request):**
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

**Policy Relay WS protocol (response):**
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
    "sig": "<schnorr_sig>"
  }
}
```

Or on rejection:
```json
{
  "id": "<req_uuid>",
  "error": "mandate_violation: action not permitted"
}
```

**Key implementation:**
- `policy_relay_server.py` listens on WebSocket at port `POLICY_RELAY_PORT` (default 8080)
- `nsec_map` loaded from `NODUS_DW_NSEC_MAP` (JSON) or `POLICY_RELAY_NSEC_<PUBKEY>` (individual)
- `nostr_adk_transport.py`: `if self._policy_relay_client:` → `await self._policy_relay_client.sign_event(unsigned)`
- If Policy Relay rejects → `PolicyRelayError` → message discarded (no fallback to direct signing)

**Security guarantee (v0.2):** The DW's `nsec` never leaves the Policy Relay. It is physically impossible for the DW to publish events not authorised by the relay.

---

## Flow 5: A2A Nostr-Native (M9, v0.2)

**Components:** `a2a_nostr_v2.py`  
**Feature flags:** `NODUS_A2A_NOSTR_V2`

```
DW A (client)                    Relay                    DW B (listener)
     │                             │                            │
     │  A2ANostrV2Client.send_task()│                           │
     │  request_id = uuid8         │                           │
     │                             │                           │  A2ANostrV2Listener.listen()
     │                             │                           │  REQ {kinds:[10010], #p=[DW_B]}
     │                             │◄──────────────────────────┤
     │                             │                           │
     │  EVENT kind:10010           │                           │
     │  tags: [p=DW_B, session,    │                           │
     │    request_id, action]      │                           │
     │  content: {action, params}  │                           │
     ├────────────────────────────►│                           │
     │                             │  EVENT kind:10010         │
     │                             ├──────────────────────────►│
     │  REQ {kinds:[10011,10013],  │                           │ task_handler(action, params, session)
     │    #p=[DW_A], since=now-5}  │                           │
     ├────────────────────────────►│                           │
     │                             │                           │
     │                             │                           │ EVENT kind:10011
     │                             │                           │ tags: [p=DW_A, request_id, session]
     │                             │                           │ content: {result: ...}
     │                             │◄──────────────────────────┤
     │  EVENT kind:10011           │                           │
     │◄────────────────────────────┤                           │
     │  json.loads(content) =      │                           │
     │  {"result": ...}            │                           │
```

**Streaming A2A (kind:10012):**
```
DW B → kind:10012 (chunk, done=false)
DW B → kind:10012 (chunk, done=false)
DW B → kind:10011 (final) or kind:10012 (done=true)
DW A → receives all chunks via AsyncIterator
```

**Key implementation (`a2a_nostr_v2.py`):**
```python
# Client: publish + await response
await ws.send(json.dumps(["EVENT", signed_event]))
await ws.send(json.dumps(["REQ", sub_id, {
    "kinds": [KIND_A2A_RESPONSE, KIND_A2A_ERROR],
    "#p": [self.sender_pubkey_hex],
    "since": int(time.time()) - 5,
}]))

# Server: listen + execute
sub_id = f"a2a-listen-{self.dw_pubkey_hex[:8]}"
await ws.send(json.dumps(["REQ", sub_id, {
    "kinds": [KIND_A2A_REQUEST],
    "#p": [self.dw_pubkey_hex],
    "since": int(time.time()),
}]))
```

**Comparison: A2A HTTP v1 vs A2A Nostr-Native (v0.2)**

| Aspect | A2A HTTP v1 | A2A Nostr-Native v0.2 |
|--------|-------------|----------------------|
| Transport | HTTP POST → server → DW B | kind:10010 at relay |
| Dependency | nodus-adk-runtime HTTP server | None (relay only) |
| Audit | Manual kind:34003 | Automatic (events are immutable) |
| Cross-tenant | Via federation HTTP | Via relay_hint + kind:34001 |
| Signing | DW nsec or Policy Relay | Policy Relay (M8) |

---

## Flow 6: Cross-Tenant Federation (M10 + M12, v0.2)

**Components:** `relay_federation.py`, `cross_tenant_hitl.py`  
**Feature flags:** `NODUS_FEDERATION_V2`, `NODUS_CROSS_TENANT_HITL_V2`

### 6a: Federated Relay Discovery

```
DW Tenant A                     Local Relay A
     │                                │
     │  RelayFederation.discover_federation_relays()
     │                                │
     │  REQ {kinds:[34001], limit:50} │
     ├───────────────────────────────►│
     │                                │
     │  EVENT kind:34001 (with relay_hint + tenant)
     │◄───────────────────────────────┤
     │  EVENT kind:34001 ...          │
     │◄───────────────────────────────┤
     │  EOSE                          │
     │◄───────────────────────────────┤
     │                                │
     │  self._known_relays =          │
     │  {"tenant-b": "wss://relay.b.example"}
```

**Relay discovery from kind:34001:**  
Any org-relation event with a `relay_hint` tag teaches the DW about a new federated relay. The `federation_scope` tag (`read-only | delegate | full`) defines the allowed interaction level.

### 6b: Cross-Tenant HITL (M12)

```
DW Nodus (Tenant A)        Relay A         Relay B (Tenant B)      Human Tenant B
        │                    │                    │                       │
        │  kind:10003 HITL   │                    │                       │
        ├───────────────────►│                    │                       │
        │                    │                    │                       │
        │  CrossTenantHitlBridge.bridge_hitl_request()                    │
        │  federation.publish_to_federation(event, "tenant-b")            │
        │                    │  EVENT kind:10003  │                       │
        ├────────────────────┼───────────────────►│                       │
        │                    │                    │  Human sees kind:10003│
        │                    │                    │◄──────────────────────┤
        │  federation.subscribe_from_federation() │                       │
        │  (kinds:[10004], #p=[DW_A])             │                       │
        ├────────────────────┼───────────────────►│                       │
        │                    │                    │                       │
        │                    │                    │  Human signs kind:10004
        │                    │                    │◄──────────────────────┤
        │                    │                    │                       │
        │  _handle_external_response()            │                       │
        │◄───────────────────┼────────────────────┤                       │
        │                    │                    │                       │
        │  _is_authorized_human(responder_pubkey) │                       │
        │  REQ {kinds:[34001], #p=[responder]}    │                       │
        ├───────────────────►│                    │                       │
        │  (verifies responder appears in 34001 cross-tenant)             │
        │◄───────────────────┤                    │                       │
        │                    │                    │                       │
        │  on_response(response_event) → DW continues
```

**Authorisation validation (`cross_tenant_hitl.py`):**
```python
async def _is_authorized_human(self, pubkey_hex: str) -> bool:
    # REQ {kinds:[34001], "#p":[pubkey_hex], "limit":5}
    # Returns True if any 34001 event mentions this pubkey
    # The responder must appear in a cross-tenant org-relation
```

**Key design:** the human from company B uses **their own app and keypair** — no Nodus account required. Validation is purely cryptographic via the relay.

---

## Flow 7: Verifiable Contract (M13, v0.2)

**Components:** `nostr-contract-service.ts`

```
Admin (Backoffice)              Relay                      Third Party (auditor)
       │                           │                              │
       │  1. kind:34002 exists (mandate)                         │
       │  2. kind:34001 exists (org_relation)                    │
       │  3. Publishes kind:34010 (KYC claim)                    │
       │  POST /api/kyc-claims/publish                           │
       ├──────────────────────────►│                              │
       │                           │                              │
       │  POST /api/contracts/generate                           │
       │  { dw_id, mandate_event_id,                             │
       │    org_relation_event_id,                               │
       │    kyc_claim_event_id }                                  │
       │                           │                              │
       │  contract_hash = sha256(  │                              │
       │    mandate_id + ":" +     │                              │
       │    org_relation_id + ":"+ │                              │
       │    kyc_claim_id           │                              │
       │  )                        │                              │
       │                           │                              │
       │  EVENT kind:34002         │                              │
       │  (contract, d=hash,       │                              │
       │   "e" refs to mandate,    │                              │
       │   org-relation, kyc-claim)│                              │
       ├──────────────────────────►│                              │
       │                           │                              │
       │  { contract_hash,         │                              │
       │    nostr_uri: "nostr:event:<hash>" }                     │
       │◄──────────────────────────┤                              │
       │                           │                              │
       │                           │  GET /api/contracts/verify/:hash
       │                           │◄─────────────────────────────┤
       │                           │  REQ {kinds:[34002],         │
       │                           │       "#d":[contract_hash]}  │
       │                           │                              │
       │                           │  EVENT → verifies:           │
       │                           │  content.contract_hash == hash?
       │                           │  valid_until not expired?    │
       │                           │──────────────────────────────►
       │                           │  { valid: true, details: ... }
```

**Contract hash computation (`nostr-contract-service.ts`):**
```typescript
const contractHashInput = [mandateEventId, orgRelationEventId, kycClaimEventId ?? ""].join(":");
const contractHash = crypto.createHash("sha256").update(contractHashInput).digest("hex");
```

**Contract event tags:**
```typescript
const tags = [
  ["d", contractHash],
  ["p", dwPubkeyHex],
  ["e", mandateEventId, "", "mandate"],
  ["e", orgRelationEventId, "", "org-relation"],
  ["e", kycClaimEventId, "", "kyc-claim"],   // optional
  ["t", "nodus-contract"],
  ["expiration", String(validUntil)],        // optional
];
```

**What any third party can verify (without trusting Nodus):**
1. Fetch kind:34002 `#d=<contract_hash>` from the relay
2. Verify the three `e` tags point to real events
3. Recompute `sha256(mandate_id + ":" + org_relation_id + ":" + kyc_claim_id)` — must match
4. Verify owner signature on each event (BIP-340 Schnorr)
5. Check `expiration` tag if present

**Human-readable statement example:**
> *"Nodus Factory SL (reg. B12345678, ES) has authorised Digital Worker `npub1abc...` to perform [send_email, read_calendar, orchestrate] subject to HITL for financial actions, effective from 2026-04-01 with no expiry."*

---

## Flow 8: NIP-26 Delegation (M3)

**Components:** `nip26.py`, `nostr_adk_transport.py`  
**Feature flags:** `NODUS_PROTOCOL_DELEGATION_V1` + `NOSTR_OWNER_NSEC`

```
[On NostrAdkTransport initialisation]

Owner (NOSTR_OWNER_NSEC)          DW (NOSTR_AGENT_NSEC)
        │                               │
        │  create_delegation_token(     │
        │    owner_nsec,                │
        │    delegatee=DW_pubkey,       │
        │    allowed_kinds=[10001,10002,│
        │      10003,10004,10006],      │
        │    valid_seconds=86400*30     │
        │  )                            │
        │                               │
        │  delegation_string =          │
        │  "nostr:delegation:<DW_pubkey>:<conditions>"
        │  conditions = "kind=10001&kind=10002&...&created_at>T0&created_at<T1"
        │                               │
        │  sig = schnorr_sign(sha256(delegation_string), owner_sk)
        │                               │
        │  self._delegation_tag =       │
        │  ["delegation", owner_pubkey_hex, conditions, sig_hex]
        │──────────────────────────────►│
        │                               │
        │  [For each published event]   │
        │                               │
        │  tags = base_tags + [self._delegation_tag]
        │  event signed by DW_nsec with delegation tag
        │                               │
        │  [Third-party verification]   │
        │                               │
        │  verify_delegation_token(     │
        │    delegation_tag,            │
        │    delegatee=DW_pubkey,       │
        │    event_kind=10002,          │
        │    event_created_at=T         │
        │  )                            │
        │  → verifies: kind in allowed? │
        │  → verifies: T0 < T < T1?    │
        │  → verifies: schnorr_verify(sha256(delegation_string), owner_pubkey, sig)
```

**Delegation tag format (NIP-26 standard):**
```json
["delegation",
 "<owner_pubkey_hex>",
 "kind=10001&kind=10002&kind=10003&kind=10004&kind=10006&created_at>1714000000&created_at<1716592000",
 "<sig_hex>"
]
```

**Key implementation (`nip26.py`):**
```python
delegation_string = f"nostr:delegation:{delegatee_pubkey_hex}:{conditions}"
msg_bytes = hashlib.sha256(delegation_string.encode()).digest()
sig_bytes = _schnorr_sign(msg_bytes, owner_sk_bytes, aux_rand)
delegation_tag = ["delegation", owner_pubkey_hex, conditions, sig_hex]
```

---

## Flow Summary

| Flow | Kinds involved | Feature flags | Status |
|------|----------------|---------------|--------|
| 1. Normal session | 10001, 10002, 10006 | `NOSTR_ADK_TRANSPORT_V1` | ✅ Prod |
| 2. Constitutional HITL | 10001, 10002, 10003, 10004 | `NODUS_PROTOCOL_CONSTITUTIONAL_HITL_V1` | ✅ Impl |
| 3. Emergency Stop | 34005, 34006 | `NODUS_PROTOCOL_EMERGENCY_V1` | ✅ Impl |
| 4. Policy Relay | all kinds | `NODUS_POLICY_RELAY_V1` | ✅ Impl |
| 5. A2A Nostr-Native | 10010, 10011, 10012, 10013 | `NODUS_A2A_NOSTR_V2` | ✅ Impl |
| 6. Cross-Tenant Federation | 34001, 10003, 10004 | `NODUS_FEDERATION_V2`, `NODUS_CROSS_TENANT_HITL_V2` | ✅ Impl |
| 7. Verifiable Contract | 34002, 34010 | M13 implicit | ✅ Impl |
| 8. NIP-26 Delegation | all | `NODUS_PROTOCOL_DELEGATION_V1` | ✅ Impl |

---

*Nodus Factory · © 2026 · Reference Implementation of the Nodus Protocol (CC BY 4.0)*
