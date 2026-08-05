# UGP Trust and Verification Model

## Open Specification — Working Draft

**Status:** Draft for public review · Maintained by the UGP Steward (see GOVERNANCE.md)
**Licence:** Apache 2.0

> **Scope note.** This document explains how trust is established over UGP tokens at the open protocol layer, and — just as importantly — what the open layer does and does not guarantee. The mechanisms that make a signature non-forgeable and the governance step non-bypassable belong to the enforcement layer and are out of scope.

---

## 1. Two kinds of attestation

It is worth being precise about what a UGP token attests, because it differs from attestation mechanisms already in wide use.

**Configuration-state attestation** (for example, a platform attestation quote) tells you a system *was in a known-good configuration* — that approved components were active. That is a statement about the *facility*.

**Process attestation** — what a UGP compliance token provides — tells you that *a specific output was produced through a verified governance process*: evaluated against a specific constraint set, refined until it satisfied that set within declared tolerance, and only then released. That is a statement about the *product*.

The analogy is the difference between certifying that a factory holds the right equipment and certifying that *this specific item* was manufactured through verified procedures. Safety and quality regimes generally require the second.

---

## 2. Trust anchors, treated abstractly

The open layer treats the signer of a token as an abstract **trust anchor**. A token's signature is non-repudiable *relative to the asserted anchor*; the protocol's job is to let a verifier check that relationship and to resolve trust in the anchor's key.

The open layer deliberately does **not** mandate a particular anchor type. Different deployments will use different anchors, and a token generated under one anchor type can be verified by a system configured for another (§4). What the open layer standardises is the *verification interface* over heterogeneous anchors, so verifiers validate governance-process proof regardless of the specific mechanism that produced the signature.

The *strength* of an anchor — how hard it is to forge a signature or bypass the governance step that precedes it — is a property of the enforcement layer, not of this specification. The open layer can tell you a token verifies; it cannot, by itself, tell you the anchor is strong. That is why the enforcement layer exists, and why the two conformance classes are distinct (see [`CONFORMANCE.md`](CONFORMANCE.md)).

---

## 3. The verification procedure

The normative verification sequence is defined in [`CORE.md`](CORE.md#5-token-verification): signature verification, governance-state validation, output-binding confirmation, freshness check, and trust-chain resolution, each with a defined failure mode. Two aspects deserve emphasis here.

### 3.1 Trust-chain resolution

A verifier that does not directly recognise a signing key MUST be able to establish trust by traversing a **hierarchical governance trust model** anchored in well-known root authorities. The resolution path MUST itself be verifiable. Trust-hierarchy formats and root-authority discovery are defined in the conformance suite and maintained by the UGP Steward.

An **authority** in this hierarchy can vouch for signing keys. Verification MAY be **federated** across multiple attestation authorities and MAY be **cached**, including offline verification against pre-distributed trust anchors and delegated verification through a trusted intermediary — important for real-time and disconnected deployments.

### 3.2 Freshness and revocation

Verifiers MUST guard against replay (monotonic counters, timestamp bounds, revocation-list consultation). Any attestation MUST be **revocable**, with revocation discoverable by verifiers. Where a token participates in a chain (§5), revocation of one token MUST be propagatable so downstream verifiers can re-evaluate trust.

---

## 4. Cross-anchor and cross-organisational verification

A token generated under one trust-anchor type MUST be verifiable by a system configured for another, through the common verification interface. Across organisational boundaries, **mutual-recognition agreements** declare which governance configurations each party accepts from the other; a **posture-equivalence mapping** lets a verifier answer, cryptographically, "does this counterparty maintain governance equivalent to my minimum requirement?" without replicating the counterparty's full configuration.

Systems using non-UGP governance may participate through a **governance bridge** that translates their attestations into UGP tokens; bridged tokens MUST be marked as bridged so verifiers apply appropriate trust policy.

---

## 5. Chains of evidence

Tokens chain through `provenance_chain.previous_token`, and compose across domains through `cross_domain_attestations`. A verifier can therefore reconstruct an **unbroken history** of a session, and detect a **gap** in attestation coverage (a missing link) as readily as an invalid signature. For continuous-attestation domains (such as security posture), gap detection is a first-class check: a break in the chain is itself a finding.

---

## 6. What the open layer guarantees — and what it doesn't

**The open layer guarantees:**

- A well-formed token can be parsed and its base fields verified by any conformant verifier.
- A token binds a specific output to a specific, versioned constraint set and a specific trust anchor.
- Tampering with a signed token, or with the output it binds, is detectable.
- Chains and cross-domain references are verifiable, and gaps are detectable.
- Verification can be performed independently by any relying party — the format is open.

**The open layer does not, by itself, guarantee:**

- That the trust anchor is strong (non-forgeable) — that is an enforcement-layer property.
- That the governance step could not be bypassed before the token was generated — that is an enforcement-layer property.
- That a *partial* verification implies a *full* one — partial results MUST be reported as partial.
- That a valid signature over a non-compliant result is acceptable — a token whose exit divergence exceeds tolerance is a governance *failure* regardless of signature validity.

This honest boundary is the point of the layer separation. A verification-conformant open-source implementation is genuinely useful — it can check everything the format expresses — but the assurance that a regulator or insurer ultimately relies on comes from a generation-conformant enforcement layer producing anchors they accept.

---

*This is a working draft published for review. The formal trust-hierarchy schema and root-authority discovery procedure are maintained separately by the UGP Steward.*
