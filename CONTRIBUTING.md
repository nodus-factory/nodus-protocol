# Contributing to the Nodus Protocol

Thank you for your interest in the Nodus Protocol.

The specification is at **v1.0.1**. Feedback, questions, and proposals are welcome.

## How to Contribute

### Issues

Use GitHub Issues to:
- Ask questions about the specification
- Report ambiguities or inconsistencies
- Propose new kinds, extensions, or governance mechanisms

### Pull Requests

For corrections or additions to the specification:

1. Fork the repository
2. Create a branch: `spec/your-topic`
3. Make your changes
4. Open a PR with a clear description of what you are changing and why

Keep PRs focused. One topic per PR.

### Discussions

For broader architectural discussions — new protocol layers, governance models, federation patterns — use GitHub Discussions.

## Scope

The documents in this repository are **protocol specifications**, not implementation guides. When contributing:

- Write in terms of abstract roles (Initiator, Worker, Owner, Relay) — not specific systems or libraries
- Use MUST / SHOULD / MAY per [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
- Keep JSON examples using abstract placeholders (`<worker_pubkey_hex>`, `<tenant_id>`, etc.)
- Implementation details belong in `IMPLEMENTATION-GUIDE.md`, not in SPEC.md, KINDS.md, or FLOWS.md

## Design Principles

All contributions MUST respect the [Design Principles](SPEC.md#3-design-principles), especially:

- **P1** — Clear and verifiable identity
- **P4** — Immutable auditability
- **P6** — Ontological separation: Human / Digital Worker

## Registering a Certified Implementation

To register your implementation as conformant with the Nodus Protocol:

1. Verify your implementation against the [conformance checklist](SPEC.md#minimum-conformance-checklist)
2. Prepare a public conformance test report (automated or documented manual verification)
3. Open a pull request adding your implementation to the Certified Implementations table in SPEC.md

## License

By contributing, you agree that your contributions will be licensed under [CC BY 4.0](LICENSE).
