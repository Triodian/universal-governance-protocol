# Formal Constraint Specification Language (FCSL)

## Open Specification — Working Draft

**Layer:** UGP Layer 3 (Constraint)
**Status:** Draft for public review · Maintained by the UGP Steward (see GOVERNANCE.md)
**Licence:** Apache 2.0 (specification, compiler, reference implementations, conformance suite)

> **Scope note.** This document describes the open constraint layer of the UGP: a domain-agnostic language for expressing the constraints a governed system must satisfy, and how those constraints are identified, versioned, and bound into protocol messages. It does not specify how constraints are enforced below the software trust boundary; enforcement is provided by licensed implementations and is out of scope.

---

## 1. Why a constraint language

A governance protocol is only as useful as the rules it can carry. FCSL is the UGP's answer to a single question: *how does an organisation express its own regulatory, contractual, safety, and policy requirements in a form a governance pipeline can enforce, attest, and version?*

FCSL is **domain-agnostic**. The same language expresses a medical-device safety limit, a financial-decisioning policy, an industrial-control envelope, a content constraint, or a frontier-alignment principle. Each domain sub-protocol reuses FCSL; none redefines it. FCSL has a **publicly available compiler**, so constraints can be authored, checked, and compiled by anyone, independent of any enforcement vendor.

FCSL expresses *what must be true of an output*. It does not express *how the model produces the output*. This is deliberate: it is what makes the whole architecture **model-agnostic**. Constraints are evaluated against outputs, not against model internals, so any model that produces text or structured data can be governed regardless of provider or architecture.

---

## 2. What an FCSL constraint set contains

An FCSL constraint set is a versioned, identifiable artefact. At minimum it carries:

- **Constraint definitions** — the individual conditions an output must satisfy, expressed against a defined output representation.
- **Tolerances** — for each constraint, the bound within which an output is considered compliant. Tolerances are first-class: a token's `convergence_proof.divergence_at_exit` is meaningful only relative to a declared tolerance (see [`CORE.md`](CORE.md#43-convergence-proof-semantics)).
- **Composition** — how constraints combine. FCSL provides a composition algebra so that constraint sets can be built from smaller sets, layered, and prioritised.
- **Provenance references** — links from each constraint back to its authority: the regulation, contract, standard, or policy it derives from. This is what lets a verifier confirm the *derivation lineage* of the constraints that governed an output, end to end.
- **Version and identity** — a version counter and a hash by which the exact configuration is identified. This hash is what appears in a token's `governance_state.identifier`.

FCSL supports **parameterised types** (a constraint template instantiated with deployment-specific values), **subtype polymorphism** (a specialised constraint standing in for a general one), and a **composition algebra** for combining and prioritising constraint sets.

---

## 3. Identity and versioning

Every constraint set is addressable by a URI of the form:

```
ugp://constraints/{domain}/{category}/v{n}
```

and is bound to a content hash. The pairing matters: the URI names *which* constraint set and version; the hash proves the *exact bytes* that were in force. A token binds both, so an auditor can always establish precisely which rules governed a given output, and can detect any later modification.

Constraint sets are **versioned over time**, so an organisation can demonstrate which rules governed which outputs across a period — including through recalibrations and re-authoring. Constraint-set updates between systems with overlapping configurations MAY be transmitted as **differentials** rather than full sets.

---

## 4. Constraint provenance

FCSL treats provenance as a required property, not an optional annotation. A constraint carries a verifiable chain from its **source** (a regulation, a contract clause, a safety standard, an organisational policy, an ethical framework) through its **FCSL specification** to its **enforcement**. A verifier SHOULD be able to confirm this chain end to end, so that "this output was governed" can be traced back to "this is the regulation that required it."

Where constraints derive from multiple stakeholders — a regulator, an organisation, and an ethical framework each contributing rules — FCSL composition binds the resolved set to a single composite identifier, with conflicts resolved deterministically by **selecting the strictest applicable constraint**.

---

## 5. Binding into protocol messages

FCSL is the constraint layer; the sub-protocols define how FCSL constraints are *carried and reconciled* in protocol messages. Three capabilities recur across the sub-protocols and are worth stating once here:

- **Advertisement.** A governed system SHOULD be able to advertise its active constraint configuration — type metadata, version, regulatory-source references, and trust-anchor compatibility — so verifiers and auditors can discover it. Advertisement MAY use a **privacy-preserving form** that proves constraint satisfaction without disclosing the constraints themselves.
- **Compatibility negotiation.** Before two systems exchange governed outputs, they SHOULD determine whether their constraint sets are mutually compatible, so governance failures surface at negotiation time rather than at output-exchange time.
- **Propagation.** When one system's output becomes another's input, the receiving system inherits and validates the propagated constraints against its own store *before* processing the output (see [`../domains/MA-GP.md`](../domains/MA-GP.md)).

---

## 6. Authoring quality

A constraint is only as good as its ability to separate compliant outputs from violating ones. The UGP treats **per-constraint empirical validation** as good practice: a constraint whose reference does not demonstrably distinguish compliant from non-compliant outputs should be re-authored rather than deployed. Several domain profiles formalise this as a *validation-gated loading* discipline, in which a constraint set is admitted only once it carries evidence that it validates. The open layer defines the *evidence and interchange formats* for this discipline; the enforcement of loading gates is a Layer 1 concern.

Authoring methodology, calibration procedures, and validation harnesses are described in the Steward's authoring guides, which accompany but are separate from this specification.

---

## 7. Conformance

An FCSL processor is **conformant** if it parses constraint sets in the published grammar, computes canonical constraint-set identifiers that match the reference compiler, resolves composition and priority deterministically, and passes the published FCSL conformance test vectors. The reference compiler is the normative implementation for identity-hash computation.

---

*This is a working draft published for review. The formal grammar, the composition-algebra semantics, and the conformance test vectors are maintained separately by the UGP Steward.*
