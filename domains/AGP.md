# AI Governance Protocol (AGP)

## Open Protocol Specification — Working Draft

**Layer:** UGP Layer 2 (Governance — Execution Semantics), with constraint binding into Layer 3
**Status:** Draft for public review · Maintained by the UGP Steward (see GOVERNANCE.md)
**Licence:** Apache 2.0 (specification, reference implementations, conformance suite)
**Date:** June 2026

> **Scope note.** This document specifies the *open protocol layer* of the AI Governance Protocol — the wire formats, exchange sequences, and conformance requirements that allow independently built systems to interoperate. It defines what is transmitted and how it is verified. It does not specify the hardware enforcement mechanisms that generate trust-anchored signatures; those are provided by licensed implementations and are out of scope for this specification.

---

## 1. Purpose

AGP is the foundational interoperability protocol of the UGP family. It defines how governed AI systems exchange and verify *compliance tokens* — cryptographic attestations that an output was produced through a verified governance process — across system, organisational, and jurisdictional boundaries.

Existing protocols carry identity (TLS, OAuth, SAML), authorisation (OAuth, XACML), or platform state (remote attestation). None of them carries *governance process proof*: evidence that a specific output was evaluated against a specific constraint set and refined to satisfy it before release. AGP fills that gap. It is the protocol that lets one system prove its governance posture to another — to a regulator, a counterparty, a downstream consumer, an insurer, or a certification body.

AGP is transport-agnostic. Tokens and messages may be carried over HTTP headers, gRPC metadata, message-queue payloads, or any other transport.

---

## 2. Terminology

- **Compliance token** — a self-describing record attesting the governance process that produced an output. The canonical wire format is defined in §4.
- **Governance state** — the complete governance posture of a system: active constraint configuration, fallback tier, convergence parameters, and health metrics.
- **Governance-state identifier** — a hash identifying the exact constraint configuration in force.
- **Trust anchor** — the root of trust whose signature appears on a token. AGP treats trust anchors abstractly (see §8); it does not mandate any particular anchor type.
- **Verifier** — any party that validates a received token.
- **Authority** — a party in the governance trust hierarchy that can vouch for signing keys (see §5.3).

The key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY are to be interpreted as in RFC 2119.

---

## 3. Protocol Overview

A complete AGP interaction comprises four phases:

1. **Handshake and capability negotiation** (§7) — two systems discover a common interoperability baseline before exchanging any governed output.
2. **Constraint advertisement and negotiation** (§6) — systems advertise and, where required, reconcile the constraint configurations under which they operate.
3. **Token generation and exchange** (§4) — a governed output is accompanied by a compliance token.
4. **Token verification** (§5) — the receiving system validates the token.

Governance state may additionally be exchanged continuously and out of band (§9) for federated and monitored deployments.

---

## 4. Compliance Token Format

The compliance token is the fundamental unit of governance interoperability. A token MUST be self-describing and extensible: a verifier that does not recognise a domain extension MUST still be able to parse and verify the base fields.

### 4.1 Canonical Wire Format

```json
{
  "ugp_version": "51.0",
  "token_type": "universal_governance_proof",
  "domain": "agp",

  "governance_state": {
    "identifier": "sha256:...",
    "constraint_set_uri": "ugp://constraints/{domain}/{category}/v{n}",
    "version_counter": 51,
    "authority_signature": "ecdsa-p256:...",
    "domain_extensions": {}
  },

  "convergence_proof": {
    "iterations_executed": 5,
    "divergence_at_entry": 0.847,
    "divergence_at_exit": 0.003,
    "reconciliation_hash": "sha256:...",
    "wcet_bound_satisfied": true
  },

  "output_binding": {
    "output_hash": "sha256:...",
    "timestamp_us": 1735812000000000,
    "hardware_signature": "..."
  },

  "provenance_chain": {
    "previous_token": "sha256:...",
    "ledger_uri": "ugp://provenance/session/..."
  },

  "cross_domain_attestations": [],
  "domain_extensions": {}
}
```

### 4.2 Field Requirements

| Field | Requirement | Meaning |
|---|---|---|
| `ugp_version` | MUST | The protocol version the token conforms to (see §10). |
| `token_type` | MUST | Identifies the record as a governance proof. |
| `domain` | MUST | The sub-protocol that generated the token (`agp`, `sgp`, `fa-gp`, `ma-gp`, …). |
| `governance_state.identifier` | MUST | Hash of the constraint set governing the output. |
| `governance_state.constraint_set_uri` | MUST | Resolvable identifier of the constraint set and version. |
| `governance_state.authority_signature` | MUST | Binds the governance state to an authority identity. |
| `convergence_proof` | MUST | Evidence of the refinement process (see §4.3). |
| `output_binding.output_hash` | MUST | Binds the token to the exact output it governs. |
| `output_binding.timestamp_us` | MUST | Generation time, Unix epoch microseconds. |
| `output_binding.hardware_signature` | MUST | Non-repudiable signature from the trust anchor. |
| `provenance_chain.previous_token` | SHOULD | Links to the prior token in the session. |
| `cross_domain_attestations` | MAY | References to tokens from other domains (see §8 of the family). |
| `domain_extensions` | MAY | Self-describing domain-specific fields. |

### 4.3 Convergence Proof Semantics

`convergence_proof` records the refinement process by which the output was brought into compliance. `divergence_at_entry` and `divergence_at_exit` are normalised distances (0 = full compliance) before and after governance. `iterations_executed` records the number of refinement cycles. `wcet_bound_satisfied` is a boolean confirming the process completed within its declared worst-case time bound — a mandatory field for any verifier asserting suitability for real-time or safety-critical use.

A token whose `divergence_at_exit` exceeds the constraint set's declared tolerance MUST be treated as a governance failure, not a successful attestation.

### 4.4 Token Envelope

For transmission, a token MAY be wrapped in an envelope carrying delivery metadata: source system identifier, destination system identifier, session context, and routing information. The envelope is transport-level metadata and is not covered by the token's signature.

### 4.5 Serialisation

Implementations MUST produce canonical, byte-deterministic serialisation so that cryptographic verification yields identical results across platforms. Conforming implementations MUST agree on field ordering, number formatting, and whitespace per the canonical serialisation rules in the conformance suite. Tokens MAY be compressed, fragmented and reassembled, or batched for constrained or high-throughput transports, provided the canonical form is recoverable for verification.

---

## 5. Token Verification

A verifier MUST execute the following steps, in order, and MUST define a failure mode for each:

1. **Signature verification** — confirm `output_binding.hardware_signature` is valid for the asserted trust anchor.
2. **Governance-state validation** — confirm `governance_state.identifier` matches the referenced constraint set.
3. **Output binding confirmation** — confirm `output_binding.output_hash` matches the received output.
4. **Freshness check** — confirm the token is neither expired nor replayed (see §5.2).
5. **Trust chain resolution** — establish trust in the signing key (see §5.3).

Any step's failure MUST cause the token to be rejected with a standardised error code (§5.4).

### 5.1 Partial Verification

A verifier that lacks access to the full constraint set MAY perform *partial verification*: confirming that convergence occurred, that constraints were enforced, and that the trust anchor signed the output, without evaluating the constraint set itself. Partial verification supports privacy-preserving governance, where a system proves it was governed without disclosing its full configuration. A partial verification result MUST be reported as such and MUST NOT be represented as a full verification.

### 5.2 Freshness

Verifiers MUST guard against replay using one or more of: monotonic counters, timestamp bounds, and revocation-list consultation. The acceptable freshness window is a verifier policy and SHOULD be declared during the handshake.

### 5.3 Trust Chain Resolution

A verifier that does not directly recognise a signing key MUST be able to establish trust by traversing a hierarchical governance trust model anchored in well-known root authorities. The resolution path MUST itself be verifiable. Trust hierarchy formats and root-authority discovery are defined in the conformance suite.

### 5.4 Failure Reporting

Verification failures MUST be reported with standardised error codes indicating the failing step (signature, governance-state, binding, freshness, trust-chain) and SHOULD include remediation guidance. Real-time deployments MAY declare verification latency bounds; verifiers SHOULD support cached and offline verification using pre-distributed trust anchors and delegated verification through a trusted intermediary.

---

## 6. Constraint Binding (FCSL)

AGP carries constraints expressed in the Formal Constraint Specification Language (FCSL), the UGP Layer 3 constraint language. AGP does not define FCSL; it defines how FCSL constraints are encoded and reconciled within protocol messages.

### 6.1 Constraint Set Advertisement

A governed system SHOULD be able to advertise its active constraint configuration — type metadata, version, regulatory source references, and trust-anchor compatibility indicators — so that verifiers and auditors can discover it. Advertisement MAY use a privacy-preserving form that proves constraint satisfaction without disclosing the constraints.

### 6.2 Constraint Compatibility Negotiation

Before two systems exchange governed outputs, they SHOULD determine whether their constraint sets are mutually compatible, so that governance failures are surfaced at negotiation time rather than at output exchange. Negotiation MUST support a timeout and a declared fallback behaviour for time-sensitive applications.

### 6.3 Constraint Provenance Verification

A verifier SHOULD be able to confirm, end to end, the derivation lineage of the constraints that governed a received output — from regulatory source through FCSL specification to enforcement. Constraint set updates between systems with overlapping configurations MAY be transmitted as differentials.

---

## 7. Handshake and Capability Negotiation

Every AGP interaction between two systems begins with a handshake whose purpose is governance interoperability — distinct from transport security (TLS) or authorisation (OAuth).

### 7.1 Capability Advertisement

The initiating system advertises: supported protocol versions, available trust anchor types, active constraint domains, supported token extensions, and current governance tier state.

### 7.2 Negotiation Rules

The parties select the highest mutually supported protocol version, the strongest mutually trusted trust anchor type, and the intersection of supported constraint domains. They then exchange governance trust anchor references sufficient to verify each other's signatures.

### 7.3 Failure and Optimisation

Handshake failure MUST be reported with a code identifying the specific incompatibility (version mismatch, no common trust anchor, constraint domain incompatibility) and recommended remediation. Implementations MAY support handshake resumption, pipelining, proxying through a governance intermediary, and a zero-round-trip mode for pre-configured relationships. Negotiation outcomes SHOULD be recorded in a tamper-evident audit log.

---

## 8. Cross-Platform Interoperability

AGP is explicitly designed so that a token generated under one trust anchor type can be verified by a system configured for another. The protocol presents a common verification interface over heterogeneous anchors, so verifiers validate governance process proof regardless of the specific mechanism that produced the signature.

A *governance bridge* MAY allow systems using non-AGP governance mechanisms to participate by translating their attestations into AGP compliance tokens. Bridged tokens MUST be marked as bridged so that verifiers can apply appropriate trust policy.

Cross-organisational federation is supported through mutual recognition agreements that declare which governance configurations each party accepts from the other (see also §9).

---

## 9. Governance State Exchange

Beyond per-output tokens, AGP supports continuous exchange of governance *state* for federated and monitored deployments.

A governance state message encodes a system's posture: active constraint set identifier, fallback tier state, convergence parameters, governance health metrics (such as divergence-event rates and token generation rates), and governance policy version. Monitoring parties (regulators, auditors, parent organisations) MAY subscribe to state updates with declared frequency, filtering, and access control.

Federated deployments SHOULD support atomic constraint updates — all federated systems apply a change, or none do — and MUST define precedence for conflicting policy updates (regulatory overrides organisational; organisational overrides system-local). State messages MAY be transmitted as differentials, MAY use selective disclosure, and MUST include replay protection.

---

## 10. Versioning and Extensions

AGP uses major/minor version semantics: minor versions MUST preserve backward compatibility; major versions MAY break it. A newer implementation MUST be able to process messages from an older one, and an older implementation MUST gracefully handle unknown extensions.

New token extension types, constraint domain bindings, and verification procedures are introduced through a registered extension framework. Registration SHOULD pass a defined validation process (compatibility and security review) so that the protocol does not fragment into incompatible variants. Deprecated versions and extensions MUST be sunset with a declared transition period.

---

## 11. Conformance

An implementation is **AGP-conformant** if it:

1. Produces and parses tokens in the canonical format of §4, including canonical serialisation.
2. Executes the full verification sequence of §5 with standardised failure reporting.
3. Supports constraint advertisement and compatibility negotiation per §6.
4. Completes the handshake and capability negotiation of §7.
5. Correctly handles unknown domain extensions (parse-and-verify base fields).
6. Passes the published conformance test vectors for its declared protocol version.

A **generation-conformant** implementation additionally produces trust-anchored signatures via a licensed enforcement layer. A **verification-conformant** implementation need not generate tokens but MUST verify them per §5.

---

## 12. Relationship to Other UGP Protocols

AGP is the horizontal protocol that the domain-specific sub-protocols extend. Security governance (SGP), frontier-AI alignment (FA-GP), multi-agent governance (MA-GP), and the other UGP sub-protocols all produce tokens in the AGP-compatible format and reuse AGP's verification, handshake, and state-exchange machinery, adding their own domain extension fields. A token generated under any sub-protocol is verifiable by any AGP verifier at the base-field level, which is the foundation of UGP cross-domain interoperability.

---

*This is a working draft published for review. Conformance test vectors and the formal trust-hierarchy schema are maintained separately by the UGP Steward.*
