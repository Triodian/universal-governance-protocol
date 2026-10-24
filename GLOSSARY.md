# Glossary

**AGP — AI Governance Protocol.** The horizontal UGP sub-protocol for attesting AI outputs at inference time; the base that other domains extend. See [`domains/AGP.md`](domains/AGP.md).

**Authority.** A party in the governance trust hierarchy that can vouch for signing keys.

**Compliance token.** A self-describing, signed record attesting the governance process that produced an output — the fundamental unit of UGP interoperability. See [`spec/CORE.md`](spec/CORE.md#4-the-compliance-token).

**Configuration-state attestation.** Attestation that a system *was in a known-good configuration* — a statement about the facility. Contrast with **process attestation**.

**Conformance class.** The level an implementation targets: **verification-conformant** (can check tokens, buildable from the open layer) or **generation-conformant** (can additionally issue trust-anchored tokens, requires the enforcement layer). See [`spec/CONFORMANCE.md`](spec/CONFORMANCE.md).

**Constraint set.** A versioned, identifiable collection of FCSL constraints and their tolerances, addressed by a `ugp://constraints/...` URI and a content hash.

**Convergence proof.** The token field recording the refinement process — entry and exit divergence, iterations, and whether the worst-case time bound was met.

**Cross-domain attestation.** A reference within one token to a token from another domain, enabling a single continuous evidence chain across the AI lifecycle.

**Divergence.** A normalised distance measuring how far an output sits from its constraint-defined envelope; 0 means full compliance. Used as a governance trigger.

**Domain / sub-protocol.** A specialisation of the Core for a class of governance (AGP, SGP, FA-GP, MA-GP, DLGP, CDP, and the domain profiles).

**Domain profile.** A specialisation of the UGP for a specific regulated or safety-critical setting (e.g. industrial control, radiation oncology). See [`domains/PROFILES.md`](domains/PROFILES.md).

**Enforcement layer (Layer 1).** The licensed component that generates trust-anchored signatures and makes the governance step non-bypassable. Out of scope for the open specification.

**FCSL — Formal Constraint Specification Language.** The domain-agnostic UGP Layer 3 language for expressing the constraints a governed system must satisfy. See [`spec/FCSL.md`](spec/FCSL.md).

**Generation-conformant.** See **conformance class**.

**Govern exchange.** The request/response by which a requester obtains a governed output and its compliance token from a governing service. See [`spec/EXCHANGE.md`](spec/EXCHANGE.md).

**Governing service.** A generation-conformant implementation that accepts Govern requests and returns governed outputs with compliance tokens. It may be co-located with the requester, hosted by the customer, or hosted by a third party.

**Governance bridge.** A translator that lets a non-UGP governance mechanism participate by emitting UGP tokens; bridged tokens are marked as such.

**Governance state.** A system's complete governance posture: active constraint configuration, fallback tier, convergence parameters, and health metrics.

**Governance-state identifier.** A hash identifying the exact constraint configuration in force; appears in `governance_state.identifier`.

**Layer model.** Layer 1 (enforcement, licensed), Layer 2 (governance — Core and sub-protocols, open), Layer 3 (constraint — FCSL, open). See [`spec/CORE.md`](spec/CORE.md#2-the-layer-model).

**Partial verification.** Confirming a token was governed and trust-anchor-signed without evaluating the full constraint set; must be reported as partial, never as full.

**Process attestation.** Attestation that *a specific output was produced through a verified governance process* — a statement about the product. The property a UGP compliance token provides.

**Provenance chain.** The linkage of each token to its predecessor, allowing an unbroken session history to be reconstructed and gaps to be detected.

**Requester.** The workload, agent, or gateway that sends a Govern request. See [`spec/EXCHANGE.md`](spec/EXCHANGE.md).

**Trust anchor.** The root of trust whose signature appears on a token. Treated abstractly by the open layer. See [`spec/TRUST-MODEL.md`](spec/TRUST-MODEL.md).

**Verification-conformant.** See **conformance class**.

**WCET — worst-case execution time.** The bounded time within which the governance step completes; `wcet_bound_satisfied` records confirmation for real-time and safety-critical use.
