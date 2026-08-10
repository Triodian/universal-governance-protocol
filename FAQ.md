# Frequently Asked Questions

**Is the protocol actually open?**
Yes. The Core specification, FCSL, the trust and verification model, the domain sub-protocols and profiles, the conformance suites, the reference implementations, and the SDKs are published under Apache 2.0 and maintained by the UGP Steward (Orthogramic Pty Ltd today, transferring to the Casuarina Foundation — see GOVERNANCE.md). The licensed component is the enforcement layer that issues trust-anchored signatures. See [`GOVERNANCE.md`](GOVERNANCE.md).

**What can I build without a licence?**
Anything on the open layer. You can parse and produce tokens, evaluate FCSL constraints, run the state machines, verify received tokens, build tooling and auditors' verifiers, teach with the materials, and extend the protocol. A **verification-conformant** implementation is fully buildable from this repository. A licence is required only to *issue trust-anchored tokens* a regulator or insurer will accept — that is, to be **generation-conformant**.

**Why is there an enforcement layer at all? Isn't software governance enough?**
For many applications, software controls are appropriate. The enforcement layer exists for the cases where the evidence must not depend on trusting the surrounding software — where a regulator, insurer, or counterparty needs assurance that cannot be quietly bypassed or edited after the fact. The open layer is honest about this boundary: it can tell you a token verifies, but the *strength* of the trust anchor is an enforcement-layer property. See [`spec/TRUST-MODEL.md`](spec/TRUST-MODEL.md#6-what-the-open-layer-guarantees--and-what-it-doesnt).

**How is a compliance token different from a platform attestation quote?**
A platform quote attests *configuration state* — that a system was in a known-good setup. A UGP compliance token attests *process* — that a specific output was evaluated against a specific constraint set, refined to satisfy it, and only then released. It is the difference between certifying a factory has the right equipment and certifying a particular item was made through verified procedures.

**Is it tied to a particular model or provider?**
No. The architecture is model-agnostic. Constraints are evaluated against *outputs*, not model internals, so any model that produces text or structured data can be governed regardless of provider or architecture. You keep your existing providers and credentials.

**Does governance add unbounded latency?**
The governance step is designed to complete within a declared worst-case time bound, and the token records confirmation (`wcet_bound_satisfied`) that the bound was met for each output — which is a precondition for real-time and safety-critical use.

**Can I verify tokens myself?**
Yes. The token format and verification procedure are open, so you and your counterparties can verify tokens independently rather than relying on the issuer's assurances. Verification can be cached, offline, federated, or delegated.

**How does this relate to regulations we already report against?**
A compliance token is designed to be the durable, verifiable evidence artefact that maps onto obligations across frameworks — technical documentation, transparency, human-oversight configuration, quality-management process evidence, safety-lifecycle evidence, model-validation provenance, and continuous security posture. The token doesn't discharge an obligation on its own; it supplies evidence that documentation alone cannot. Your compliance team continues to own the obligations.

**What is FCSL and do I have to use it?**
FCSL is the open language for expressing the constraints a governed system must satisfy. It is domain-agnostic, versioned, and has a public compiler. It is how you express *your* rules — regulatory, contractual, safety, policy — rather than generic ones. See [`spec/FCSL.md`](spec/FCSL.md).

**What are "domain profiles"?**
Specialisations of the UGP for particular settings — cloud-native infrastructure, industrial control, medical devices, autonomous vehicles, aviation, on-device agents, and many more. Each describes what is governed and attested in its domain and produces tokens in the universal format so evidence composes across domains. See [`domains/PROFILES.md`](domains/PROFILES.md).

**Can systems that don't speak UGP participate?**
Yes, through a **governance bridge** that translates their attestations into UGP tokens. Bridged tokens are marked as bridged so verifiers can apply appropriate trust policy.

**How do I propose a change or a new domain?**
Through the extension and contribution process in [`CONTRIBUTING.md`](CONTRIBUTING.md). New extensions pass a compatibility and security review so the protocol does not fragment into incompatible variants.

**Who governs the protocol?**
The UGP is stewarded by Triodian today, which intends to transfer stewardship to an independent public-interest body, the Casuarina Foundation, once it is established. The commitments — specification kept free under a Public Access Guarantee, enforcement layer licensed on FRAND terms, certification kept separate from training, and no products sold by the steward — are described in [`GOVERNANCE.md`](GOVERNANCE.md).
