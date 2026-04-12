# Changelog

All notable changes to the Nodus Protocol specification will be documented here.

## [v0.2] — April 2026 (Release Candidate)

### New in v0.2

#### M9 — A2A Nostr-Native
- kinds 10010–10013 for direct DW-to-DW delegation without HTTP intermediary
- `A2ANostrV2Client` and `A2ANostrV2Listener` components
- Streaming A2A support (kind:10012 chunks with `done` flag)
- Comparison vs A2A HTTP v1 transport

#### M10 — Multi-Relay Federation
- `relay_hint` tag on kind:34001 for cross-tenant relay discovery
- `federation_scope` tag values: `read-only | delegate | full`
- `RelayFederation` component for federated subscriptions and publications
- Feature flag: `NODUS_FEDERATION_V2`

#### M11 — Public DW Marketplace
- kind:34000 published to public relay for discoverability
- kind:34010 KYC Corp Claim for legal entity verification (`nodus:kyc-corp-claim`)
- `nostr_marketplace_opt_in` per-user preference
- Feature flag: `NODUS_MARKETPLACE_V2`

#### M12 — Cross-Enterprise HITL
- `cross_tenant_hitl` bridge for external relay HITL approval
- `CrossTenantHitlCard` UI component with indigo branding
- Validation via kind:34001 org-relation (responder must appear in cross-tenant 34001)
- Feature flag: `NODUS_CROSS_TENANT_HITL_V2`

#### M13 — Verifiable Employment Contracts
- `contract_hash = sha256(mandate_id + ":" + org_relation_id + ":" + kyc_claim_id)`
- Human-readable contract statement generation
- Public contract verification endpoint: `GET /api/contracts/verify/:hash`
- Feature flag: `NODUS_VERIFIABLE_CONTRACTS_V2`

### Completed from v0.1

All milestones M0–M8 are implemented in the Reference Implementation:

- **M0** — DW & Human Identities (kind:34000 + kind:34001)
- **M1** — Mandates (kind:34002, immutable, signed by owner)
- **M2** — Audit Log (kind:34003, append-only)
- **M3** — NIP-26 Delegation (cryptographic authority proof on every event)
- **M4** — Constitutional HITL (kind:10003/10004, NIP-07 + custodial)
- **M5** — Emergency Stop/Resume (kind:34005/34006, <30 second halt)
- **M6** — Room UX + Async HITL Inbox (kind:10020/10021)
- **M7** — MCP Governance (kind:34004, DW verifies MCP gateway profile)
- **M8** — Policy Relay (NIP-46 variant, DW nsec never leaves the relay)

### New spec documents

- `KINDS.md` — formal event kinds reference (all 16 kinds with JSON examples)
- `FLOWS.md` — formal protocol flows (8 flows with sequence diagrams)

### Spec changes

- Section 5.2 — updated kinds table with all 16 implemented kinds
- Section 5.6 — Policy Relay expanded with real WS protocol
- Sections 5.12–5.16 — new v0.2 features (A2A Nostr, federation, marketplace, cross-HITL, contracts)
- Section 6.1 — Reference Implementation Status table (M0–M13)
- Section 6.2 — Activation Flags with recommended activation order

---

## [v0.1] — March 2026 (Internal Draft)

### Added
- Abstract and motivation
- Design principles P1–P6
- Core concepts: Digital Worker, cryptographic identity, owner, mandate, delegation, A2A, HITL, audit log
- Technical specification: 4 layers (Nostr, A2A, ACP, MCP)
- Nostr NIPs reference table (NIP-01/26/42/44/46/59/89/90)
- Nodus Protocol kinds: 34000–34010
- Synchronous A2A transport (Google A2A compatibility)
- Persistent ACP sessions
- MCP governance (kind 34004)
- Nodus Policy Relay (conceptual)
- Enterprise control: 4 control layers + cryptographic hierarchy
- Panic button and revocation (kinds 34005/34006)
- Constitutional separation Human/DW: 4 safeguards
- Graduated governance model: 4 levels
- Federation and cross-enterprise communication
- Illustrative use case (legal services firm)
- Reference implementation (Nodus OS)
- Minimum conformance checklist
- Glossary, NIP references, external protocols
