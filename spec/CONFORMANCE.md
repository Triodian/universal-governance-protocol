# UGP Conformance

## Open Specification — Working Draft

**Status:** Draft for public review · Maintained by the UGP Steward (see GOVERNANCE.md)
**Licence:** Apache 2.0

> **Scope note.** This document defines what it means to conform to the UGP, the conformance classes, and how conformance is tested. Certification of conformance is operated by the UGP Steward and is kept structurally independent of any training or commercial-implementation interest (see [`../GOVERNANCE.md`](../GOVERNANCE.md)).

---

## 1. Conformance classes

The UGP defines conformance so that the open protocol and the licensed enforcement layer can be tested independently. There are two primary classes.

### Verification-conformant

A **verification-conformant** implementation can receive, parse, and verify UGP tokens. It:

1. Parses tokens in the canonical wire format ([`CORE.md`](CORE.md#4-the-compliance-token)), including canonical serialisation.
2. Executes the full verification sequence ([`CORE.md`](CORE.md#5-token-verification)) with standardised failure reporting.
3. Resolves trust chains and freshness/revocation per the [trust model](TRUST-MODEL.md).
4. Correctly handles unknown domain extensions (parse-and-verify base fields).
5. Reports partial verification as partial, never as full.
6. Passes the published verification conformance test vectors for its declared protocol version.

A verification-conformant implementation **does not need to generate tokens**. This is the class most open-source implementations, auditors' tools, and relying-party systems will target. It can be built entirely from the open specification.

### Generation-conformant

A **generation-conformant** implementation additionally *produces* tokens carrying **trust-anchored signatures** via a licensed enforcement layer. It:

1. Meets all verification-conformant requirements.
2. Produces tokens whose signatures verify against a recognised trust anchor.
3. Populates `convergence_proof` from a real refinement process, including a truthful `wcet_bound_satisfied`.
4. Chains tokens correctly and, where applicable, emits the domain extensions for its declared domains.
5. Passes the published generation conformance test vectors.

Generation of trust-anchored signatures requires the enforcement layer and is therefore **out of scope for the open specification**. A software-only implementation can produce structurally valid tokens for testing and development, but their signatures will not verify against a recognised trust anchor and MUST NOT be presented as trust-anchored.

---

## 2. Domain conformance

Each domain sub-protocol defines its own conformance requirements *on top of* the Core (see each document under [`../domains/`](../domains/)). A domain-conformant implementation produces and parses that domain's tokens as valid Core tokens carrying the domain extension, implements the domain's required exchanges, and passes the domain's conformance test vectors. Domain conformance is layered: an SGP-conformant verifier is also Core-conformant.

---

## 3. Test vectors

Conformance is established against **published test vectors** maintained by the UGP Steward, not against prose alone. The vectors cover:

- Canonical serialisation (byte-for-byte determinism across platforms).
- Valid tokens that MUST verify.
- Malformed and tampered tokens that MUST be rejected, each with the expected error code.
- Chains with and without gaps.
- Unknown-extension handling.
- Freshness and revocation cases.
- Per-domain extension cases.

The reference implementations in this repository are validated against the same vectors and are a good starting point for a new implementation.

---

## 4. Declaring conformance

An implementation declares the class it targets, the protocol version, and the domains it supports. The UGP Steward operates the certification programme and the implementer directory; listing is available to implementations that pass the relevant vectors. Certification is **vendor-neutral** and is deliberately separated from training delivery so that the body that certifies does not also profit from preparing candidates for certification.

---

*This is a working draft published for review. Test vectors evolve with the specification and are versioned alongside it.*
