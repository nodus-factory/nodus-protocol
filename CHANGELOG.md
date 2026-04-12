# Changelog

All notable changes to the Nodus Protocol specification are documented here.

---

## [v1.0.1] — April 2026 — Spec accuracy fixes

### Changes

**SPEC.md §5.6 Policy Relay — two additions:**
- `relay_proof` tag: cryptographic proof that a DW event passed through the Policy Relay. Format: `["relay_proof", sha256(event_id + policy_relay_pubkey_hex)]`. REQUIRED on all events signed in Policy Relay mode. Relays MAY enforce rejection of DW events missing a valid `relay_proof` in enforcement mode.
- Emergency stop relay enforcement: formal description of the relay daemon architecture (subscribe to kind:34005/34006, maintain durable per-tenant cache, write-policy plugin reads cache per event).

**KINDS.md:**
- kind:10002 `relay_proof` tag documented as a conditional tag (REQUIRED in Policy Relay mode)
- kind:34003 `d` tag formula corrected: `SHA-256(worker_pubkey_hex + ":" + session_id + ":" + timestamp_ms)` — components joined with `:` separators (aligned with reference implementation)

**FLOWS.md §Flow 2 — Constitutional HITL:**
- Clarified placeholder kind:10002 behaviour: implementations SHOULD publish an immediate kind:10002 placeholder after kind:10003 to unblock bridges and frontends. The final kind:10002 with the actual result is published after the Owner approves and execution is resumed.

---

## [v1.0] — April 2026 — First Public Release

### Summary

v1.0 is the first release intended for external implementors. The specification has been cleaned and fully decoupled from the Nodus OS reference implementation. All implementation-specific details have been moved to `IMPLEMENTATION-GUIDE.md`.

### Changes from v0.2

**Specification**
- All references to internal components (`nodus-adk-runtime`, `nodus-backoffice`, `nodus-llibreta-v2`, `strfry`, Python file names) removed from SPEC.md, KINDS.md, and FLOWS.md
- SPEC.md section 6 rewritten: feature flags and activation order removed; replaced by formal conformance checklist with MUST/SHOULD/MAY levels per RFC 2119
- "Reference Implementation" section replaced by "Certified Implementations" table (open to any conformant implementation)
- Version references updated from `"0.2"` to `"1.0"` in all kind content JSON examples

**KINDS.md**
- All "Publisher: X (`component-name`)" annotations changed to abstract role names (Worker, Owner, Initiator)
- Feature flag annotations removed (flags belong to implementation guides, not specifications)
- JSON examples use abstract placeholders throughout (`<worker_pubkey_hex>`, `<initiator_pubkey_hex>`, etc.)

**FLOWS.md**
- All flows rewritten using abstract roles: Initiator, Worker, Relay, Owner
- All references to implementation files and component names removed
- Sequence diagrams use role names only
- Summary table added

**IMPLEMENTATION-GUIDE.md** (moved from SPEC.md section 6)
- All activation flags, step-by-step implementation guidance, Python/TypeScript code samples, and component references consolidated here
- This document is addressed to implementors, not specification readers

**CONTRIBUTING.md**
- Updated to reflect v1.0 scope

---

## [v0.2] — April 2026 — Release Candidate

### New in v0.2

**M9 — A2A Nostr-Native**
- Kinds 10010–10013 for direct DW-to-DW delegation without HTTP intermediary
- Streaming A2A support (kind:10012 with `done` flag)

**M10 — Multi-Relay Federation**
- `relay_hint` tag on kind:34001 for cross-tenant relay discovery
- `federation_scope` values: `read-only | delegate | full`

**M11 — Public DW Marketplace**
- kind:34000 published to public relay for discoverability
- kind:34010 KYC Corp Claim for legal entity verification

**M12 — Cross-Enterprise HITL**
- Cross-tenant HITL bridge for external relay approval
- Validation via kind:34001 org-relation

**M13 — Verifiable Employment Contracts**
- `contract_hash = sha256(mandate_id + ":" + org_relation_id + ":" + kyc_claim_id)`
- Public contract verification from relay data

### Completed from v0.1

All milestones M0–M8 implemented in the reference implementation:

- **M0** — DW & Human Identities (kind:34000 + kind:34001)
- **M1** — Mandates (kind:34002, immutable, signed by owner)
- **M2** — Audit Log (kind:34003, append-only)
- **M3** — NIP-26 Delegation
- **M4** — Constitutional HITL (kind:10003/10004)
- **M5** — Emergency Stop/Resume (kind:34005/34006, <30 second halt)
- **M6** — Room UX + Async HITL Inbox (kind:10020/10021)
- **M7** — MCP Governance (kind:34004)
- **M8** — Policy Relay (DW nsec never leaves the relay)

### New spec documents

- `KINDS.md` — formal event kinds reference
- `FLOWS.md` — formal protocol flows with sequence diagrams

---

## [v0.1] — March 2026 — First Internal Draft

### Added

- Abstract and motivation
- Design principles P1–P6
- Core concepts: Digital Worker, cryptographic identity, owner, mandate, delegation, A2A, HITL, audit log
- Technical specification: 4 layers (Nostr, A2A, ACP, MCP)
- Nostr NIPs reference (NIP-01/26/42/44/46/59/89/90)
- Nodus Protocol kinds: 34000–34010
- Synchronous A2A transport
- Persistent ACP sessions
- MCP governance (kind 34004)
- Policy Relay (conceptual)
- Enterprise control: 4 layers + cryptographic hierarchy
- Emergency controls and revocation (kinds 34005/34006)
- Constitutional separation Human/DW: 4 safeguards
- Graduated governance model: 4 levels
- Federation and cross-enterprise communication
- Minimum conformance checklist
- Glossary, NIP references, external protocols
