# Constraint Declaration Protocol (CDP)

## Open Protocol Specification — Working Draft

**Layer:** UGP domain extension over the Core (application-layer-to-governance-layer constraint declaration)
**Status:** Draft for public review · Maintained by the UGP Steward (see GOVERNANCE.md)
**Licence:** Apache 2.0 (specification, reference implementations, conformance suite)

> **Scope note.** This document specifies the *open protocol layer* of CDP — the declaration format, the declaration-and-acceptance exchange, the tier and scope validation rules, and the conformance requirements that let a third-party application-layer vendor and a governance layer interoperate. Where AGP attests how a governed output was produced and DLGP attests how data was governed, CDP governs the seam *upstream* of enforcement: how constraints authored by one party are declared to, validated by, and accepted into the enforcement pipeline of another. It does not specify the enforcement that anchors a vendor signature, makes the acceptance pipeline non-bypassable, or enforces the accepted constraints; those are provided by licensed implementations and are out of scope.

---

## 1. Purpose

CDP applies the UGP governance model to the trust boundary between an **application-layer vendor** and a **governance layer**. When a third-party vendor brings its own domain expertise and constraint library to a governed deployment — a refrigeration-management product, a process optimiser, a clinical-decision package, an agent framework — two failure modes arise if there is no declaration protocol:

- **Duplication failure.** The vendor and the governance layer separately canonicalise the same constraints. Two semantic models drift apart, audit evidence splits across two attestation chains, and integration cost doubles for no governance gain.
- **Blind-trust failure.** The governance layer accepts the vendor's declared constraints without attestation, scope-binding, or cross-checking against the operator's regulatory floor. A compromised or buggy vendor declaration silently becomes a compromised or buggy constraint set, defeating the trust boundary the governance layer exists to maintain.

CDP closes this seam. It defines a **tiered constraint-authority model** and a wire-level **declaration-and-acceptance exchange** so a vendor can declare constraints it genuinely owns, the operator can retain non-negotiable requirements the vendor cannot override, and every accepted constraint is attested and reconciled rather than duplicated or blindly trusted.

CDP tokens and records are Core-compatible (`domain: "cdp"`).

---

## 2. Terminology

- **Declaring party** — an application-layer vendor that authors and declares constraints.
- **Accepting party** — the operator's governance layer that validates and accepts (or rejects) declared constraints.
- **Declaration** — a signed record of a vendor's proposed constraints, their scope, and their authority tier.
- **Authority tier** — the level at which a constraint may act, in a tiered model where higher tiers cannot be overridden by lower ones.
- **Scope** — the bounded region of the deployment within which a declared constraint may apply.
- **Acceptance record** — a signed record of the accepting party's disposition of a declaration.

RFC 2119 keywords apply.

---

## 3. The Tiered Authority Model

CDP rests on a simple principle: **a vendor may declare only constraints it legitimately owns, and never constraints that would loosen the operator's own non-negotiable floor.**

Constraints are organised into authority tiers. A representative arrangement:

- **Operator-mandatory (highest).** The operator's regulatory and safety floor. A vendor declaration MUST NOT relax these; a declaration that attempts to is rejected.
- **Vendor-authoritative.** Constraints within the vendor's genuine domain of expertise, which the operator accepts on the vendor's attested authority within a bound scope.
- **Advisory (lowest).** Constraints the vendor recommends but which the operator may adopt, modify, or decline.

Reconciliation is deterministic: where tiers conflict, the **higher tier prevails**, and the election is recorded in the acceptance record so an auditor can see exactly which party's constraint governed and why.

---

## 4. Declaration Format

A declaration is a signed record carrying:

```json
{
  "domain": "cdp",
  "domain_extensions": {
    "declaration": {
      "declaring_party": "...",
      "declaring_party_signature": "...",
      "constraint_set_uri": "ugp://constraints/cdp/{vendor}/{set}/v{n}",
      "constraint_set_binding": "sha256:...",
      "authority_tier": "vendor_authoritative | advisory",
      "scope": "ugp://scope/...",
      "provenance": "ugp://constraints/source/..."
    }
  }
}
```

The declaration binds the vendor's identity, the exact constraint set (by content hash), the tier at which it is claimed, and the scope within which it may act. Constraints themselves are expressed in **FCSL** (see [`../spec/FCSL.md`](../spec/FCSL.md)); CDP defines how they are declared, not how they are written.

---

## 5. The Declaration-and-Acceptance Exchange

1. **Declare.** The declaring party submits a signed declaration.
2. **Validate.** The accepting party validates the declaration against three checks:
   - **Signature and authority** — the declaring party is who it claims and is entitled to declare at the claimed tier.
   - **Scope** — the declared scope is within the region the vendor is permitted to govern.
   - **Floor reconciliation** — the declaration does not relax any operator-mandatory constraint.
3. **Dispose.** The accepting party emits a signed **acceptance record** with a disposition of `accepted`, `rejected`, or `accepted_with_modification`, and — on acceptance — binds the accepted constraint set into its own governance state.
4. **Attest.** The accepted constraints thereafter appear in the `governance_state` of tokens the pipeline produces, with provenance tracing back through the declaration to the vendor.

The exchange MUST support a timeout and a declared fallback behaviour. On validation failure, the accepting party MUST NOT enforce the declared constraints, and MUST record the rejection.

---

## 6. Verification and Audit

A verifier confirms, over the Core base fields plus the CDP extension, that:

1. The declaration was signed by the declaring party.
2. The acceptance record was signed by the accepting party.
3. The accepted tier and scope are consistent with the declaration.
4. No operator-mandatory constraint was relaxed.

Because declaration and acceptance are both signed and chained, an auditor can reconstruct **who declared what, at which tier, within what scope, and how the operator disposed of it** — the evidence that distinguishes governed vendor integration from blind trust. Anomalies in declarations MAY be federated across peer deployments so that a divergent or suspicious declaration is visible beyond a single site.

---

## 7. Conformance

An implementation is **CDP-conformant** if it:

1. Produces and parses declarations and acceptance records per §4–§5 as valid Core artefacts with the CDP extension.
2. Enforces the tier and scope validation rules of §3 and §5, including floor reconciliation.
3. Emits signed acceptance records and binds accepted constraints into governance state.
4. Executes the verification and audit checks of §6.
5. Passes the published CDP conformance test vectors.

Enforcement of accepted constraints, and the non-bypassability of the acceptance pipeline, require a licensed enforcement layer and are out of scope for protocol conformance.

---

## 8. Relationship to Other UGP Protocols

CDP sits upstream of enforcement and feeds it: constraints accepted through CDP become the `governance_state` that AGP, DLGP, and the domain profiles then attest against. It composes with the other domains through Core cross-domain attestation, so a governed output can be traced not only to the constraint set that governed it but to the *declaration and acceptance* by which that constraint set entered the pipeline.

---

*This is a working draft published for review. The formal tier schema and conformance test vectors are maintained separately by the UGP Steward.*
