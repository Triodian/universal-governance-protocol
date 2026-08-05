# Frontier AI Alignment Protocol (FA-GP)

## Open Protocol Specification — Working Draft

**Layer:** UGP domain extension over AGP (training- and alignment-time governance)
**Status:** Draft for public review · Maintained by the UGP Steward (see GOVERNANCE.md)
**Licence:** Apache 2.0 (specification, reference implementations, conformance suite)
**Date:** June 2026

> **Scope note.** This document specifies the *open protocol layer* of the Frontier AI Alignment Protocol — the alignment token format, the alignment-verification exchange, and conformance requirements for interoperability. It does not specify the hardware enforcement that binds principles, bounds training, or signs alignment tokens; those are provided by licensed implementations and are out of scope.

---

## 1. Purpose

AGP governs AI *outputs* at inference time. FA-GP extends governance *upstream* into the processes that produce and modify models — training, alignment, distillation, and synthetic-data generation — and defines how the resulting attestations are exchanged and verified.

The motivating gap is that alignment work today is asserted in documents. There is no standard, verifiable way for a model owner, deployer, auditor, or regulator to confirm that a deployed model was actually shaped by a governed process: that constitutional principles were enforced at runtime, that reward optimisation stayed within bounds, that human feedback was authentic, that a distilled model preserved its teacher's alignment, or that synthetic training data met its constraints.

FA-GP provides the protocol artefacts that make those facts attestable and that link them into a continuous chain from training to deployment. An FA-GP **alignment token** is AGP-compatible and adds an alignment domain extension; it is the unit that connects training-time governance to the inference-time tokens AGP already defines.

---

## 2. Terminology

- **Alignment token** — an attestation of a governed training-, alignment-, or data-generation process (§4).
- **Principle set** — the constitutional principles a model is governed against.
- **Reward boundary** — a bound on reward-function optimisation intended to prevent reward hacking.
- **Distillation lineage** — the attested relationship binding a student model to its teacher's governance state.
- **Alignment lineage** — the chain of alignment tokens across a model's versions.

RFC 2119 keywords apply.

---

## 3. Protocol Overview

FA-GP defines protocol artefacts across five governed processes, plus a verification protocol:

1. **Constitutional principle attestation** (§4.2).
2. **Reward-boundary and feedback-provenance attestation** (§4.3).
3. **Distillation governance and authorisation** (§4.4).
4. **Synthetic-data compliance and provenance** (§4.5).
5. **Training-time governance attestation** (§4.6).
6. **Alignment Verification Protocol** (§5) — the cross-platform exchange for requesting and verifying alignment status.

The enforcement that makes each process governed is performed by licensed implementations; this specification covers the *attestation and exchange* of governed outcomes.

---

## 4. Alignment Token

### 4.1 Base Format

An FA-GP token is an AGP compliance token (`domain: "fa-gp"`) with an alignment extension. The extension is a union of the attestation types below; a token carries whichever apply to the process it attests.

```json
{
  "domain": "fa-gp",
  "domain_extensions": {
    "alignment": {
      "attestation_type": "training | principle | reward | feedback | distillation | synthetic_data",
      "model_version": "...",
      "principle_set_id": "sha256:...",
      "training_governance_id": "sha256:...",
      "lineage": { "previous_model_token": "sha256:...", "teacher_token": "sha256:..." }
    }
  }
}
```

### 4.2 Constitutional Principle Attestation

Attests that a model's outputs were governed against a defined principle set, and that conflicts between principles were resolved deterministically (by selecting the strictest applicable principle). A composite principle-set identifier binds the resolved set to the token, supporting multi-stakeholder deployments where regulators, organisations, and ethical frameworks each contribute principles. Per-principle adherence MAY be attested within a single token.

### 4.3 Reward Boundary and Feedback Provenance

- **Reward boundary attestation** records that reward-function optimisation stayed within declared bounds, treating reward hacking as a measurable divergence between the reward signal and intended behaviour. The token binds the optimisation result to its boundary constraints.
- **Feedback provenance attestation** binds human feedback used in alignment to authenticated source identities, enforces temporal ordering, and records a lineage chain from individual feedback events through reward-model updates to parameter changes — yielding an audit-grade trail. Adversarial-feedback-detection internals are out of scope; the protocol carries only the attested result and the provenance chain.

### 4.4 Distillation Governance and Authorisation

- **Distillation governance attestation** binds a student model to its teacher's governance state and records that alignment was verified continuously during distillation (not merely evaluated afterward), via divergence comparison between teacher and student outputs.
- **Distillation authorisation attestation** records that distillation proceeded under a valid authorisation token issued by the teacher owner, within declared parameter boundaries (compression ratio, layer selection, capability scope). This lets contractual distillation terms be expressed as verifiable authorisation rather than relying on assertion alone. Authorisation tokens MAY be time-limited, revocable, and delegable.

### 4.5 Synthetic-Data Compliance and Provenance

- **Synthetic-data compliance attestation** records that generated training data satisfied FCSL-expressed constraints *at generation time*, per sample or per batch, rather than being checked after training.
- **Synthetic-data provenance attestation** binds each sample to its generation parameters, timestamp, and the governance state active at generation, and tracks incorporation into training so that model capabilities can be attributed to specific data subsets.

### 4.6 Training-Time Governance Attestation

Attests that the training process itself ran within governed boundaries: that proposed parameter updates were evaluated against the constraint set, that per-checkpoint compliance tokens were generated, and that the run's governance integrity can be reconstructed checkpoint by checkpoint. Supports multi-objective, distributed, and rollback-capable training. This is the artefact that turns "we trained responsibly" into a verifiable, checkpointed record.

---

## 5. Alignment Verification Protocol

The Alignment Verification Protocol is the cross-platform exchange that lets any party request and verify a model's alignment status, regardless of where it was trained or deployed. It is an FA-GP-specific profile of AGP verification (§5 of AGP).

### 5.1 Request / Response

A verifier issues an alignment verification query identifying the model (and optionally a version or property of interest). The response is an alignment token, or token chain, sufficient to verify the requested property. The request/response format MUST allow a third-party auditor to verify alignment *without access to the model itself*.

### 5.2 Comparison and Degradation Detection

The protocol MUST support comparison across model versions and across platforms through standardised attestation comparison, so that:

- alignment can be benchmarked across heterogeneous models, and
- **alignment degradation** across versions can be detected by longitudinal analysis of a model's alignment lineage.

### 5.3 Federation and Revocation

Verification MAY be federated across multiple attestation authorities and MAY be cached. An alignment attestation MUST be revocable if a model fails a subsequent assessment, with revocation discoverable by verifiers.

---

## 6. Constraint Binding

FA-GP expresses principle sets, reward boundaries, and synthetic-data constraints in FCSL, and carries them using AGP constraint binding (§6 of AGP). Principle composition reuses FCSL composition with alignment-specific conflict resolution (strictest-principle selection). Constraint provenance — from regulatory or organisational source through to enforcement — is verifiable end to end.

---

## 7. Conformance

An implementation is **FA-GP-conformant** if it:

1. Produces and parses alignment tokens per §4 as valid AGP tokens with the alignment extension.
2. Correctly represents whichever attestation types apply to the governed process.
3. Implements the Alignment Verification Protocol of §5, including verification without model access, version comparison, degradation detection, and revocation.
4. Maintains alignment lineage via token chaining.
5. Passes the published FA-GP conformance test vectors.

Generation of trust-anchored alignment tokens, and the enforcement of principle, reward, distillation, data, and training governance, require a licensed enforcement layer and are out of scope for protocol conformance.

---

## 8. Relationship to Other UGP Protocols

FA-GP closes the upstream end of the UGP evidence chain. Because alignment tokens are AGP-compatible, a token generated when a model was trained can be referenced by the inference-time AGP tokens that govern its outputs, which can in turn be referenced during multi-agent orchestration (MA-GP) and security monitoring (SGP). The result is a single, continuous, cross-domain chain of governance evidence spanning *how a model was built, how it behaves, and how it is operated* — the property regulators most often find missing when training-time and deployment-time governance are treated separately.

---

*This is a working draft published for review. Conformance test vectors and the alignment-attestation schema are maintained separately by the UGP Steward.*
