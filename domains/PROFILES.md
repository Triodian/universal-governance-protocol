# UGP Domain Profiles

A **domain profile** specialises the UGP for a particular regulated or safety-critical setting. Each profile describes *what is governed* and *what is attested* in its domain, expresses its rules in [FCSL](../spec/FCSL.md), and produces tokens in the [universal format](../spec/CORE.md#4-the-compliance-token) so its evidence composes with every other domain.

This catalogue is a **public, conceptual overview** of the profile library. It states the problem each profile addresses and the governance property it adds. Enforcement mechanisms, and any operational detail whose disclosure would aid an adversary, are out of scope here and are provided by licensed implementations.

Profiles are grouped by theme. Within each group, profiles build on the same Core and often on one another.

---

## Infrastructure and deployment topology

**Cloud-native governance.** Extends governance into virtualised, containerised, and serverless environments where a physical governance anchor is abstracted away. Governance is expressed at the container-runtime, orchestration, and serverless layers, using cloud-native trust anchors (secure enclaves, virtual trust modules, hypervisor attestation) so the non-bypassability property holds even when the trust anchor lives inside a virtual machine on shared infrastructure.

**Self-hosted / on-device actuation.** Governs a model running **locally on a personal device**, where there is no API boundary and no model provider in the loop, and the party being protected is the device owner. The threat model is co-resident, high-privilege software on the same machine. The profile adds default-deny gating of the modern local-agent actuation surface — shell and process execution, filesystem access, network egress, and local tool calls initiated by an on-device model — keyed to verification of a propagated constraint record. It responds to a live class of exploit in which local agent tooling is used to run commands the user never authorised.

**Governed interconnection (cross-boundary agent actions).** Governs an action that crosses a trust, network, account, tenancy, or organisational boundary — a representative case being an agent-initiated tool call over an agent-interconnection protocol, a cloud-to-cloud write, or a delegated action that a first party authorises a second party to take on a third resource. The profile provides a **release interlock** for the credential that performs the action: the action credential is available only against a **chained, multi-hop proof** that every hop executed in an attested environment *and* that the scope and parameters of the original delegation were preserved end to end. The distinguishing property is that per-hop code identity is not enough — the profile attests that the *semantic scope* of what the user originally authorised survived every handoff.

**Hardware-abstraction / design-around.** A profile that keeps the governance guarantees intact across different underlying trust-anchor implementations, so a deployment is not locked to one hardware substrate.

---

## AI-output governance and its guarantees

**Convergence quality and bounded discovery.** Strengthens the convergence decision at the heart of the governed loop. Rather than a bare threshold comparison — a score against a fixed constant that separates acceptable from unacceptable only by assertion — this profile carries a **quantified error bound** on how often a genuinely acceptable output is wrongly rejected or an unacceptable one wrongly accepted, and records that basis in the attestation.

**Conformal-governed convergence.** Replaces the threshold comparison with a **distribution-free, finite-sample calibrated test** as the convergence criterion, and binds the per-iteration statistical evidence into the attestation so that the basis of every termination decision is externally verifiable. It also formalises *validation-gated loading*: a constraint set is admitted to the enforcement path only once it carries evidence that it validates. The profile is careful about scope — the guarantee holds under stated calibration assumptions and does not claim to bound an adversarial miss rate.

**Aggregate-divergence governance.** Addresses a failure mode invisible to any per-output check: a **stream** of outputs in which every individual output is compliant, while the *aggregate* drifts toward a concentration, correlation, or margin-erosion state that no single output reveals. The profile governs an aggregate state built from windowed set-statistics over the stream, and distinguishes two very different kinds of aggregate movement — **drift toward correlated risk** (to be gated) versus **legitimate novelty within appetite** (to be reviewed, not blocked). It applies equally to frontier-model alignment monitoring, multi-agent populations, content and decision streams, and financial or actuarial books.

**Introspective governed-output drift detection.** A white-box complement for the case where an operator's *own* governing model is walked out of bounds under prompt-injection or jailbreak pressure. Where output-only detection sees only what the model finally emitted, this profile examines the model's own internal state against an authored reference, and adds a runtime causal-sensitivity check that distinguishes a genuine detection from a spurious correlation. It applies only to an operator's own model, not to API-only deployments.

**Hardware-constrained decoding.** Relocates grammar- and policy-constrained token generation out of the software control plane and into a non-bypassable interlock at the point where a model emits tokens. A per-token validity mask is fused into the generation datapath so that a token the policy forbids is *physically* prevented from being emitted, and the interlock is enabled only under attestation that the loaded policy is unmodified. The decision is single-pass and deterministic, and the emitted sequence's conformance is recorded as verifiable evidence.

**Rule-legibility pre-conditioning.** Sits upstream of a deterministic policy checker. A checker can only render a verdict on output it can *parse*, yet a model often emits content that is substantively fine but structurally unparseable (verbose, hedged, malformed, or over-long). This profile inserts a **bounded loop** that asks the model for a shorter, simpler restatement until the checker can parse it — while keeping the model strictly a *proposer*: the deterministic checker's verdict remains the sole enforcement act, the loop can never emit a non-accepted candidate, and it can never relax the checker. Every iteration's parseability evidence is attested.

**Multi-model deliberation.** Governs a decision produced by more than one model deliberating together, attesting that the deliberation ran within constraints rather than being a single ungoverned generation.

**Disposition-class regression gating.** Governs the **library-update lifecycle**. When a governed constraint library is recalibrated or extended and a new version is promoted, every past decision is implicitly re-decided — a case that used to abstain might now answer, a blocked case might now pass. In a clinical setting a silent flip like that is a safety event. This profile treats the **governance verdict itself** as the regression invariant: it replays a corpus of saved cases against the candidate library, and holds the new version non-promotable until a human validator has cleared every disposition change, executing the replay through the byte-identical enforcement path as production so the comparison is evidentiary rather than an estimate.

---

## Adversarial robustness

**AI adversary defence.** A family of profiles hardening the divergence-detection core against evasion. It adds active trajectory probing (closing a *temporal* evasion surface), a parallel multi-metric ensemble with arbitrated disagreement (a *geometric* surface), and a cross-representation ensemble with an embedding-independent symbolic cross-check (a *representational* surface). Together these reduce reliance on any single distance measure in any single representation.

**Reasoning-fabric governance.** Governs structured, multi-step analytical reasoning used in enterprise decision-making, attesting that a conclusion was produced through a complete, governed reasoning process — including a governed retrieval-and-reasoning loop — rather than ad-hoc generation.

---

## Frontier and general-capability AI

**AGI governance.** A profile for the case where the governed system may itself be a highly capable adversary with full knowledge of the constraint values, thresholds, and evaluation procedures. Its premise is that governance meant to be robust against such a system cannot live anywhere the system can influence — corrigibility, anti-persuasion, canary-constraint evaluation, and shutdown must sit below any component the system can reach.

**Frontier AI alignment.** The training- and alignment-time profile realised as the [FA-GP sub-protocol](FA-GP.md): constitutional-principle adherence, reward-boundary and feedback provenance, distillation lineage, synthetic-data compliance, and checkpointed training-time governance.

---

## Data and privacy

**Data-at-rest governance.** Gates access to and movement of stored data so that an attested authorisation is required before data crosses a defined boundary, with concurrent, tamper-evident audit records of access. Realised in the [DLGP sub-protocol](DLGP.md) as the *at-rest* stage.

**SaaS privacy assurance.** Governs a multi-tenant service's handling of customer data across jurisdictions, producing attested, regulator-format evidence that privacy constraints were enforced per tenant and per region.

---

## Regulated and safety-critical sectors

**Industrial control (SCADA/ICS).** Governs AI acting in operational-technology environments under a strict precedence rule — **safety takes precedence over security** — and within deterministic-timing constraints, carrying industrial security-level fields.

**Radiation oncology.** Governs AI in radiotherapy planning and delivery, encoding clinical tolerances, an approval-gate step, and a provenance ledger suitable for radiation-safety-committee sign-off.

**Autonomous-vehicle motion control.** Governs motion-control decisions for autonomous and assisted driving within a defined operational envelope.

**Crewed flight-control envelope.** Governs AI assistance within a crewed-aircraft flight-control envelope, keeping decisions inside certified bounds.

**UAS traffic management.** Governs uncrewed-aircraft-system traffic-management decisions in shared airspace.

**BCI / neural-interface governance.** Governs AI mediating brain-computer and neural interfaces, an especially high-trust setting for the party being interfaced with.

**Quantum-computing governance.** Addresses post-quantum cryptographic readiness for the governance architecture itself and for the systems it governs.

**Governed ideation.** Governs creative and discovery processes, attesting that generated concepts were produced within defined constraints and evaluated for novelty against a defined landscape.

---

## Graded evidence and multi-jurisdiction operation

Several newer profiles extend the token from a pass/withhold artefact toward **graded, jurisdiction-aware evidence**. They are described here at the conceptual level.

**Graded release token.** Carries, as signed fields in the release token, *how far* an output sat inside its envelope and *under what certified error rate*, so a relying party can verify not merely that an output passed but by how much margin — and can apply its **own** acceptance threshold to a single attested measurement, so different consumers can hold different bars against the same evidence.

**Measured-margin rating.** Translates a stream of attested governance margins into a priced-exposure figure (for insurance, capital, or SLA terms), conditioned only on **attested** evidence rather than self-reported logs, and binds the pricing basis into the provenance chain so it is itself verifiable.

**Jurisdiction-sharpening envelope.** Represents a vague governing norm ("safe", "compliant", "within mandate") as a **shared anchor** with a set of per-jurisdiction *sharpenings* over it, and provides a machine-checkable **mutual-recognition** relation: a verdict issued under one jurisdiction's sharpening can be verified as acceptable under another's, at the artefact level rather than by institutional assertion.

**Likeness attestation.** Governs, at the emission boundary, whether generated content reproduces a protected likeness or identity, recording the disposition as verifiable evidence.

---

*This catalogue is a public overview and evolves as new profiles are published. Each profile's normative extension schema and conformance vectors, where published, are maintained by the UGP Steward. Profiles describe what is governed and attested; enforcement mechanisms and adversary-sensitive operational detail are out of scope for the open specification.*
