# Contributing to the UGP

The UGP is developed in the open and maintained by the UGP Steward — Triodian today, transferring to the Casuarina Foundation once established (see [`GOVERNANCE.md`](GOVERNANCE.md)). Contributions — corrections, clarifications, reference-implementation improvements, new test vectors, and proposals for new extensions or domains — are welcome.

## Ground rules

- **Backward compatibility is a feature.** Minor versions must not break existing verifiers. A newer implementation must process older messages; an older implementation must gracefully handle unknown extensions (parse-and-verify the base fields). Proposals that break compatibility are major-version changes and are held to a higher bar. See [`spec/CORE.md`](spec/CORE.md#9-versioning-and-extensions).
- **The open/enforcement boundary is fixed.** This repository is the open layer. Contributions must not require the enforcement layer to be understood or present in order to be *verified*. If a proposal only makes sense with enforcement-layer internals, it belongs in a licensed implementation, not here.
- **Don't fragment the protocol.** New token-extension types, constraint-domain bindings, and verification procedures go through the registered extension framework and a compatibility-and-security review, so the ecosystem doesn't split into incompatible variants.
- **Adversary-sensitive detail stays out.** For security-relevant domains, operational detail whose disclosure would aid an attacker (detection signatures, baseline-profiling internals, evasion countermeasures) is deliberately excluded from public specifications. Contributions must respect that line.

## How to propose a change

1. **Open an issue** describing the problem or gap. For anything beyond a typo or clarification, describe the motivating case and the interoperability impact.
2. **For specification changes**, include: the affected document and section, the proposed wording, backward-compatibility analysis, and — where relevant — new or updated conformance test vectors.
3. **For a new domain sub-protocol or profile**, follow the shape of the existing domain documents: a scope note that names what is governed and what is attested, the token extension fields, the verification additions, the conformance requirements, and the relationship to other UGP protocols. State clearly what is in scope for the open layer and what is left to enforcement.
4. **For reference-implementation or SDK changes**, ensure the change is validated against the published conformance vectors.

## Conformance is the arbiter

Prose is not normative on its own; **conformance test vectors** are. A change to observable behaviour should come with vectors that pin it down: valid cases that must verify, malformed and tampered cases that must be rejected with the right error code, and chain and extension cases as applicable. See [`spec/CONFORMANCE.md`](spec/CONFORMANCE.md).

## Reviews and stewardship

The project is currently maintainer-led: proposals are reviewed by the maintainer in the open, with rationale recorded in the issue or pull request (see [`GOVERNANCE.md`](GOVERNANCE.md)). As the project grows, and after hand-over to the Casuarina Foundation, review will be delegated to named reviewers and technical committees. The specifications remain free to read, implement, and propose against.

## Licence of contributions

By contributing, you agree your contributions are licensed under **Apache 2.0**, consistent with the rest of the open layer.
