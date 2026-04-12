# Security Policy

## Scope

This repository contains a **protocol specification** — not executable code. However, we take security seriously: vulnerabilities in the protocol design itself (e.g. cryptographic weaknesses, privilege escalation vectors, delegation bypass attacks) are in scope.

**In scope:**
- Cryptographic design weaknesses (NIP-26 delegation, BIP-340 signing, relay_proof verification)
- Privilege escalation: DW entities gaining human-level authority
- Mandate bypass: a conformant implementation that could bypass hitl_required
- Emergency stop bypass: a way to circumvent kind:34005 enforcement

**Out of scope:**
- Vulnerabilities in specific implementations (report those to the respective vendor)
- Social engineering or phishing attacks

## Reporting

Please report security issues by email to: **security@nodus.social**

Do **not** open a public GitHub issue for security vulnerabilities.

We will acknowledge receipt within 72 hours and aim to publish a protocol advisory within 30 days.

## Disclosure Policy

We follow responsible disclosure. We ask that you give us a reasonable time to address the issue before public disclosure.
