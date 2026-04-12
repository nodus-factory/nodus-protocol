# Changelog

## April 2026 — First Public Release

Initial public release of the Nodus Protocol.

Defines the governance layer for Digital Workers (DWs) in enterprise and civic environments:

**Identity layer:**
- kind:34000 — DW and human profiles (self-sovereign, relay-published)
- kind:34001 — org-relation (owner→DW cryptographic hierarchy)

**Mandate layer:**
- kind:34002 — immutable owner-signed policy (relay enforces no DELETE/UPDATE)

**Audit layer:**
- kind:34003 — append-only audit log (relay enforces no DELETE/UPDATE)

**Tool governance:**
- kind:34004 — MCP Server profile (DW verifies gateway before tool calls)

**Emergency controls:**
- kind:34005 — emergency stop (halts all tenant DWs within 30 seconds)
- kind:34006 — emergency resume

**Legal identity:**
- kind:34010 — KYC Corp Claim (verifiable link between legal entity and cryptographic identity)

**Session layer:**
- kinds 10001–10006 — DW↔Human messaging, HITL, streaming
- kinds 10010–10013 — A2A Nostr-native (DW↔DW without HTTP intermediary)
- kinds 10020–10021 — Async HITL inbox

**Governance mechanisms:**
- NIP-26 delegation (verifiable authority proof per event)
- Policy Relay (NIP-46 variant — DW nsec never leaves the relay)
- Constitutional HITL (human approvals are cryptographic signatures)
- Cross-enterprise HITL (approval from human at external organisation)
- Multi-relay federation (relay_hint discovery for cross-tenant DW collaboration)
- Verifiable employment contracts (contract_hash = sha256(mandate + org_relation + kyc))

**Reference implementation:** Nodus OS (Nodus Factory) — first certified production implementation.

---

*Nodus Protocol Working Group · nodus.social*
