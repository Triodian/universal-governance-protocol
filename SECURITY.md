# Security Policy

The Universal Governance Protocol is a governance and assurance protocol, so a security issue here can mean two different things:

1. **A vulnerability in a reference implementation, SDK, or conformance tool** in this repository (a parsing flaw, a verification bypass, an unsafe default).
2. **A weakness in the protocol design itself** — an interoperability or verification gap that would let a token be forged, mis-verified, replayed, or accepted when it should be rejected, at the *open-layer* level.

Both are in scope for this policy. Please report either privately, not in a public issue.

## What is *not* in scope here

The mechanisms that generate trust-anchored signatures and make the governance step non-bypassable belong to the **enforcement layer**, which is a separate, licensed component and is out of scope for this repository (see [`GOVERNANCE.md`](GOVERNANCE.md)). Reports about enforcement-layer internals should go through the licensed-implementation channel, not here.

For security-sensitive domains (for example, the security-governance profile), some operational detail is **deliberately excluded** from the public specifications precisely because its disclosure would aid an attacker. Please do not open issues asking for that detail, and do not include it in reports.

## How to report

- **Do not** open a public issue, discussion, or pull request for a suspected vulnerability.
- Use **GitHub's private vulnerability reporting** ("Report a vulnerability" under the Security tab) if it is enabled on this repository, **or**
- Email the maintainer at info@orthogramic.com (Orthogramic Pty Ltd, interim steward).

Please include, where you can:

- the affected document, file, or protocol behaviour;
- a clear description of the issue and why it matters;
- a minimal reproduction — for implementation bugs, a failing input; for protocol-design issues, a concrete token or exchange sequence that is accepted when it should be rejected (or vice versa);
- any suggested remediation.

## What to expect

- **Acknowledgement** of your report within a few business days.
- An initial assessment of scope and severity, and whether it is an implementation issue or a protocol-design issue.
- Coordinated handling: we will agree a disclosure timeline with you. Protocol-design issues may require a specification change and new conformance test vectors, which take longer than a code fix.
- **Credit** for your report in the changelog or advisory, unless you prefer to remain anonymous.

## Coordinated disclosure

We ask that you give us a reasonable opportunity to remediate before any public disclosure. Because a verification-affecting protocol issue can impact every conforming implementation, the maintainer will coordinate advisories (through the Casuarina Foundation once it is established) so that implementers can update together.

## Conformance is part of the fix

For issues affecting observable protocol behaviour, a fix is not complete until it is pinned by **conformance test vectors** (see [`spec/CONFORMANCE.md`](spec/CONFORMANCE.md)): a case that now verifies correctly, and — for a bypass — a case that is now correctly rejected with the right error code.
