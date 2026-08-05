# Universal Governance Protocol (UGP)

**An open standard for verifiable AI governance.**

Status: Working drafts for public review · Licence: Apache 2.0 · Steward: Orthogramic Pty Ltd (interim; transferring to the Casuarina Foundation — see GOVERNANCE.md)

---

The Universal Governance Protocol is an open, royalty-free family of protocols for **proving how an AI output was produced** — not that a policy existed on paper, but that a specific output was checked against a defined set of constraints, refined until it satisfied them, and released only after that check passed.

Every governed output under the UGP carries a **compliance token**: a self-describing, cryptographically signed record attesting the governance process that produced it. Any party — a regulator, an auditor, an insurer, a downstream system, a counterparty — can verify that token independently, without trusting the software stack that produced it.

This repository contains the **open protocol layer**: the wire formats, exchange sequences, constraint language, conformance requirements, reference implementations, and SDKs. It is published under the Apache 2.0 licence and is free to access, free to implement, and free to teach with, in perpetuity.

## What's here

| Document | Purpose |
|---|---|
| [`spec/CORE.md`](spec/CORE.md) | The UGP Core Specification — layer model, the compliance token wire format, verification, versioning, and interoperability rules. Start here. |
| [`spec/FCSL.md`](spec/FCSL.md) | The Formal Constraint Specification Language — how you express the rules a governed system must satisfy. |
| [`spec/TRUST-MODEL.md`](spec/TRUST-MODEL.md) | The trust and verification model: trust anchors, trust-chain resolution, partial verification, and what the open layer does and does not guarantee. |
| [`spec/CONFORMANCE.md`](spec/CONFORMANCE.md) | Conformance classes and how implementations are tested. |
| [`domains/`](domains/) | The domain sub-protocols: [AGP](domains/AGP.md), [SGP](domains/SGP.md), [FA-GP](domains/FA-GP.md), [MA-GP](domains/MA-GP.md), [DLGP](domains/DLGP.md), [CDP](domains/CDP.md), and the [domain profile catalogue](domains/PROFILES.md). |
| [`GOVERNANCE.md`](GOVERNANCE.md) | How the protocol is stewarded, how membership works, and the open/enforcement layer separation. |
| [`GLOSSARY.md`](GLOSSARY.md) | Definitions of the core terms. |
| [`FAQ.md`](FAQ.md) | Frequently asked questions. |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | How to propose changes and extensions. |

## The one-paragraph version

Most AI governance today is implemented in software, and anything implemented purely in software can, in principle, be bypassed, disabled, or rewritten by a sufficiently privileged process — and its records altered after the fact. For routine applications that is often acceptable. For medical decision support, industrial control, financial decisioning, critical infrastructure, and frontier AI, regulators and auditors increasingly want evidence that does not depend on trusting the software stack that produced it. The UGP is designed to provide that evidence. It defines an **open protocol** anyone can implement, and it separates that open protocol from the **enforcement layer** that anchors trust — the same pattern used by Arm (open architecture, licensed cores), Bluetooth (open protocol, certified radios), and HDMI (open connector, licensed content protection). The open specification drives adoption; the enforcement layer anchors trust where regulatory acceptance demands it.

## The layer model

The UGP is structured in three layers. This repository is **Layer 2 and Layer 3** — the open, implementable layers.

```
┌──────────────────────────────────────────────────────────────┐
│  Layer 3 — Constraint Layer                                    │
│  FCSL: the language for expressing what a system must satisfy  │
│  (open, Apache 2.0)                                            │
├──────────────────────────────────────────────────────────────┤
│  Layer 2 — Governance Layer                                    │
│  Core + domain sub-protocols: token formats, state machines,   │
│  verification, exchange sequences, conformance                 │
│  (open, Apache 2.0 — this repository)                          │
├══════════════════════════════════════════════════════════════┤
│  ── Compliance Token: the bridge between the layers ──         │
├══════════════════════════════════════════════════════════════┤
│  Layer 1 — Enforcement Layer                                   │
│  The trust anchor that generates valid, trust-anchored         │
│  signatures. Provided by licensed implementations.             │
│  Out of scope for this specification.                          │
└──────────────────────────────────────────────────────────────┘
```

A **software-only implementation** can implement the full protocol — parse and produce tokens, evaluate constraints, run the state machines, verify received tokens — under Apache 2.0, for free. What a software-only implementation cannot do is generate the *trust-anchored signature* that a regulator or insurer will accept as non-forgeable evidence. That signature comes from the enforcement layer, and it is what turns a well-formed token into an accepted one. The two conformance classes — **verification-conformant** and **generation-conformant** — are defined in [`spec/CONFORMANCE.md`](spec/CONFORMANCE.md).

## Design principles

- **Governance is a process, not a configuration.** Platform attestation tells you a system *was in a known-good state*. A UGP compliance token tells you *this specific output was produced through a verified governance process*. The difference is the difference between certifying that a factory holds the right equipment and certifying that a particular item was made through verified procedures.
- **One format, many domains.** Every sub-protocol produces tokens in the same base format, so governance evidence composes: a training-time token, a deployment token, a multi-agent token, and a security token can form one continuous, verifiable chain.
- **Open by construction.** The specifications, the constraint language, the reference implementations, the conformance suites, and the SDKs are open and remain open. See [`GOVERNANCE.md`](GOVERNANCE.md).
- **Verifiable by anyone.** Because the token format and verification procedure are open, evidence can be checked by the parties who rely on it, not just the party who produced it.

## Status

These are **working drafts published for review**. Conformance test vectors, the formal trust-hierarchy schema, and the per-domain extension schemas are maintained by the UGP Steward and evolve alongside these documents. Version and compatibility rules are in [`spec/CORE.md`](spec/CORE.md#versioning).

## Licence

Specifications, reference implementations, and conformance suites in this repository are licensed under **Apache 2.0**. See [`LICENSE`](LICENSE).

The enforcement layer that generates trust-anchored signatures is a separate, licensed component and is out of scope for this repository. See [`GOVERNANCE.md`](GOVERNANCE.md) for how the two relate.
