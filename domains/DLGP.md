# Data-Lifecycle Governance Protocol (DLGP)

## Open Protocol Specification — Working Draft

**Layer:** UGP domain extension over the Core (governance of data across its lifecycle: at rest, in transit, in use)
**Status:** Draft for public review · Maintained by the UGP Steward (see GOVERNANCE.md)
**Licence:** Apache 2.0 (specification, reference implementations, conformance suite)

> **Scope note.** This document specifies the *open protocol layer* of DLGP — the data-access token format, the lifecycle-continuity exchange, and the conformance requirements that let independently built systems interoperate. Where AGP attests how an AI output was produced, DLGP attests that a data object was governed at every stage of its lifecycle: read from a store, moved across a network, and consumed by a computation. It does not specify the enforcement mechanisms that generate trust-anchored data tokens; those are provided by licensed implementations and are out of scope. Operational detail whose disclosure would aid an adversary (canary placement, exfiltration-detection internals, anti-coercion ceremony specifics) is deliberately omitted.

---

## 1. Purpose

DLGP applies the UGP governance model to **data itself**, across the three stages of the data lifecycle: **at rest** (in a store), **in transit** (on a network), and **in use** (inside a computation). Where AGP attests how an AI output was produced, DLGP attests that every operation on a governed data object — reading it, copying it, transmitting it, aggregating it, consuming it — satisfied a defined constraint set before it was permitted, and produced a token proving it.

The distinction DLGP draws — and the gap it fills — is between **credential-based access control** (proving a requester held sufficient software-defined privilege) and **governed access** (proving that a specific access operation on a specific data object satisfied a specific constraint set). Encryption at rest, database access controls, data-loss prevention, and confidential-computing attestation each provide a fragment of the former; none carries a continuous, verifiable proof that a data object was governed at every stage from store to consumption. DLGP provides that proof and links the stages into one chain.

DLGP tokens are Core-compatible (`domain: "dlgp"`): any UGP verifier can validate their base fields; DLGP-aware verifiers additionally interpret the data-lifecycle extension.

---

## 2. Terminology

- **Data-access token** — an attestation that a specific operation on a specific governed data object satisfied its constraint set (§4).
- **Data object** — the governed unit (a record, file, field, stream, or dataset), identified by a stable identifier and content binding.
- **Lifecycle stage** — one of `at_rest`, `in_transit`, `in_use`.
- **Access operation** — read, copy, transmit, aggregate, or consume.
- **Boundary** — a defined perimeter (a store, a network segment, a computation domain) that a data object crosses.

RFC 2119 keywords apply.

---

## 3. Protocol Overview

DLGP defines four protocol-layer capabilities:

1. **Data-access token format and per-operation attestation** (§4).
2. **Data-access token verification** (§5).
3. **Lifecycle-continuity chaining across stages** (§6).
4. **Governance state, monitoring, and audit exchange** (§7).

The enforcement that gates an operation — the access-gated constraint store, the release gate, line-rate transit enforcement, the in-use governance domain — is performed by licensed implementations; this specification covers how the *outcomes* of those operations are attested and exchanged.

---

## 4. Data-Access Token

### 4.1 Format

A DLGP token is a Core compliance token carrying a data-lifecycle extension:

```json
{
  "domain": "dlgp",
  "governance_state": {
    "identifier": "sha256:...",
    "constraint_set_uri": "ugp://constraints/dlgp/{policy-set}/v{n}"
  },
  "domain_extensions": {
    "data_lifecycle": {
      "data_object_id": "...",
      "data_object_binding": "sha256:...",
      "lifecycle_stage": "at_rest | in_transit | in_use",
      "access_operation": "read | copy | transmit | aggregate | consume",
      "boundary_crossed": "ugp://boundary/...",
      "authorised_principal": "...",
      "disposition": "permitted | denied | permitted_with_conditions"
    }
  }
}
```

### 4.2 Field Semantics

- **`data_object_id`** — stable identifier of the governed data object.
- **`data_object_binding`** — content binding, so a token attests governance of *this* data, not merely a name.
- **`lifecycle_stage`** and **`access_operation`** — which stage and which operation the token attests.
- **`boundary_crossed`** — the perimeter the operation moved the object across, where applicable.
- **`authorised_principal`** — the identity on whose authority the operation proceeded.
- **`disposition`** — the governed outcome. `permitted_with_conditions` records that the operation was allowed subject to constraints (for example, redaction or aggregation limits) recorded in the constraint set.

### 4.3 Per-Operation Attestation

A token is generated **per governed operation**. The core insight DLGP encodes is architectural: a data-access operation departing from its governed constraint is the data-lifecycle analogue of an AI output diverging from a governance constraint, and is gated the same way — the operation is permitted only when a valid token authorising it can be generated.

### 4.4 Privacy

Tokens MAY use **partial disclosure** to prove an operation was governed without revealing the constraint detail or the data content (see [`../spec/CORE.md`](../spec/CORE.md#51-partial-verification)). This lets a data custodian prove governance to an auditor without exposing the governed data.

---

## 5. Verification

DLGP verification extends Core verification with data-specific checks:

1. Perform the full Core verification sequence over the base fields.
2. Confirm `data_object_binding` matches the data object presented (or its committed hash).
3. Confirm `access_operation` and `lifecycle_stage` are consistent with the operation being verified.
4. Evaluate `disposition` against the verifier's acceptance policy.
5. Confirm lifecycle continuity where the verifier requires it (§6).

A token reporting `disposition: denied`, or one whose object binding does not match, MUST NOT be treated as authorising the operation even if its signature is valid.

---

## 6. Lifecycle Continuity

The distinguishing property of DLGP is that the three stages **chain into one record**. A token attesting that a data object was governed *at rest* when it was read chains (via `provenance_chain.previous_token`) to a token attesting it was governed *in transit* when it was moved, which chains to a token attesting it was governed *in use* when it was consumed.

A verifier SHOULD be able to reconstruct the full lifecycle of a governed data object from its token chain, and MUST be able to detect a **gap** — a stage with no governing token — as a first-class finding. A gap means the object was, at some stage, handled outside governance, and MUST NOT be treated as continuous governance even if the surrounding tokens are individually valid.

---

## 7. Governance State, Monitoring, and Audit

DLGP reuses Core governance-state exchange for data-governance posture and adds an audit exchange:

- **Telemetry.** Data-governance events (access grants and denials, boundary crossings, aggregation events, policy changes) MAY be exchanged as telemetry carrying tamper-evident provenance.
- **Audit ledger.** Governance events SHOULD be recorded in a cryptographically chained, tamper-evident ledger.
- **Compliance reporting.** Telemetry SHOULD be mappable to data-protection and residency obligations to produce certification-ready evidence.

Detailed forensic state is access-controlled and exchanged only between authorised parties; it is never part of the routine token.

---

## 8. Conformance

An implementation is **DLGP-conformant** if it:

1. Produces and parses data-access tokens per §4 as valid Core tokens with the data-lifecycle extension.
2. Executes the verification sequence of §5.
3. Chains stages and detects lifecycle gaps per §6.
4. Exchanges governance state and audit telemetry with tamper-evident provenance per §7.
5. Passes the published DLGP conformance test vectors.

Generation of trust-anchored data tokens, and the enforcement of at-rest, in-transit, and in-use governance, require a licensed enforcement layer and are out of scope for protocol conformance.

---

## 9. Relationship to Other UGP Protocols

DLGP composes with the other domains through Core `cross_domain_attestations`: a data-access token can sit alongside the AGP output token of a model that consumed the data and the SGP posture token of the system that held it, so an auditor can trace one continuous line of evidence from *how the data was governed* to *how the output it fed was governed*.

---

*This is a working draft published for review. Detection internals, canary placement, and enforcement mechanisms are intentionally excluded from the public specification.*
