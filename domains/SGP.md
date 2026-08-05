# Security Governance Protocol (SGP)

## Open Protocol Specification — Working Draft

**Layer:** UGP domain extension over AGP (Layer 2 governance semantics, security domain)
**Status:** Draft for public review · Maintained by the UGP Steward (see GOVERNANCE.md)
**Licence:** Apache 2.0 (specification, reference implementations, conformance suite)
**Date:** June 2026

> **Scope note.** This document specifies the *open protocol layer* of the Security Governance Protocol — the security compliance token format, the verification and attestation-exchange sequences, and the conformance requirements that let independently built systems interoperate. It does not specify the hardware enforcement, containment, or remediation mechanisms that generate trust-anchored security tokens; those are provided by licensed implementations and are out of scope. Operational detail whose disclosure would aid an attacker (specific detection signatures, containment-evasion countermeasures, baseline-profiling internals) is deliberately omitted.

---

## 1. Purpose

SGP applies the UGP governance model to **security posture**. Where AGP attests how an AI output was produced, SGP attests that a system, process, network, or supply chain was *continuously governed* against a defined security baseline — and produces a token that proves it.

The core idea is that a security-relevant deviation (a process departing from its baseline behaviour) is the security analogue of an AI output diverging from a governance constraint. SGP therefore reuses the UGP divergence-detection, containment, remediation, and attestation pattern, specialised for security.

The distinction SGP draws — and the gap it fills — is between *configuration-state attestation* (proving a system was set up correctly) and *process attestation* (proving the system was continuously monitored and governed through a verified security process). Endpoint compliance checks, vulnerability scans, and software bills of materials provide the former. SGP provides the latter.

SGP tokens are AGP-compatible: any AGP verifier can validate their base fields; SGP-aware verifiers additionally interpret the security extension.

---

## 2. Terminology

- **Security compliance token** — an attestation of continuous security governance posture (§4).
- **Security baseline** — the established profile of expected behaviour against which divergence is measured.
- **Behavioural-divergence metric** — a normalised measure of how far current behaviour departs from baseline.
- **Containment event** — a governed isolation action taken in response to divergence.
- **Remediation convergence** — the governed process of returning a contained entity to within security bounds.
- **Security zone** — a bounded domain (e.g. an IEC 62443 zone) within which a security posture applies.

RFC 2119 keywords apply.

---

## 3. Protocol Overview

SGP defines four protocol-layer capabilities:

1. **Security compliance token format and continuous attestation** (§4).
2. **Security token verification** (§5).
3. **Cross-domain and supply-chain security attestation** (§6).
4. **Security governance state, monitoring, and audit exchange** (§7).

Containment and remediation are governance behaviours performed by licensed enforcement implementations; this specification covers only how their *outcomes* are attested and exchanged.

---

## 4. Security Compliance Token

### 4.1 Format

An SGP token is an AGP compliance token (`domain: "sgp"`) carrying a security domain extension:

```json
{
  "domain": "sgp",
  "governance_state": {
    "identifier": "sha256:...",
    "constraint_set_uri": "ugp://constraints/sgp/{policy-set}/v{n}"
  },
  "domain_extensions": {
    "security": {
      "security_state_identifier": "sha256:...",
      "behavioural_divergence_at_attestation": 0.012,
      "containment_event_summary": {
        "events_since_last_token": 0,
        "highest_tier_reached": "monitor"
      },
      "remediation_status": "none_required",
      "security_level": "iec62443-sl3",
      "attestation_interval_us": 1000000
    }
  }
}
```

### 4.2 Field Semantics

- **`security_state_identifier`** — hash of the security policy set governing the system.
- **`behavioural_divergence_at_attestation`** — current aggregate divergence from baseline at the time the token was generated.
- **`containment_event_summary`** — a summary (not a detailed log) of containment activity since the previous token: count and the highest containment tier reached (`monitor`, `restrict`, `isolate`, `terminate`). Detailed forensic records are exchanged separately under §7, subject to access control.
- **`remediation_status`** — one of `none_required`, `in_progress`, `converged`, `non_converged`.
- **`security_level`** — declared assurance level (e.g. an IEC 62443 Security Level) where applicable.
- **`attestation_interval_us`** — the interval at which continuous attestation tokens are generated.

### 4.3 Continuous Attestation and Chaining

SGP tokens are generated continuously at a declared interval and on demand. Each token reflects the governance process executed since the previous token. Tokens MUST be chainable (via `provenance_chain.previous_token`) so that a verifier can confirm there are no gaps in attestation coverage. A verifier SHOULD be able to detect declining posture by analysing the trend in `behavioural_divergence_at_attestation` across a token chain.

### 4.4 Privacy and Aggregation

Tokens MAY use partial disclosure to prove security compliance without revealing specific policy detail (see §5.1 of AGP). Fleet-level posture MAY be reported through aggregate tokens. Compromised systems' tokens MUST be revocable, and revocation MUST be propagatable (§6.4).

---

## 5. Verification

SGP verification extends AGP verification (§5 of the AGP specification) with security-specific checks:

1. Perform the full AGP verification sequence over the base fields.
2. Confirm `security_state_identifier` matches the referenced policy set.
3. Confirm the token chain is unbroken back to the verifier's last-seen token (no attestation gap).
4. Evaluate `behavioural_divergence_at_attestation` and `remediation_status` against the verifier's acceptance policy.

A token reporting `remediation_status: non_converged`, or a divergence metric above the verifier's threshold, MUST NOT be treated as a clean attestation even if its signature is valid.

---

## 6. Cross-Domain and Supply-Chain Attestation

### 6.1 Cross-Organisational Verification

An SGP token generated in one security domain MUST be verifiable by a system in another domain or organisation without the verifier replicating the originator's full policy set, using AGP trust-chain resolution to a shared authority.

### 6.2 Posture Equivalence Mapping

Because organisations express security governance differently, SGP defines a posture-equivalence mapping that translates between configurations and maps them to common assurance levels. This lets a verifier answer, cryptographically, "does this counterparty maintain posture equivalent to my minimum requirement?"

### 6.3 Supply-Chain Attestation Chains

Each entity in a supply chain (component manufacturer, software builder, integrator, deployer, operator) MAY present an SGP token attesting the governance applied during its phase. These tokens are cryptographically chained into an end-to-end record that any downstream party can verify. The chain attests *continuous governance process*, not merely static component provenance, and MAY be bound to a software bill of materials. A single summary token MAY represent the posture of an entire chain.

### 6.4 Revocation Propagation

Compromise of any entity MUST be propagatable through the attestation chain with delivery confirmation, so that downstream verifiers can re-evaluate trust.

---

## 7. Governance State, Monitoring, and Audit

SGP reuses AGP governance-state exchange (§9 of AGP) for security posture, and adds an audit exchange:

- **Telemetry exchange.** Security governance events (divergence events, containment events, remediation outcomes, token generation/verification events, policy changes) MAY be exchanged as telemetry carrying tamper-evident provenance, so a consumer can confirm the telemetry has not been altered.
- **Audit ledger.** Governance events SHOULD be recorded in a cryptographically chained, tamper-evident ledger.
- **Compliance reporting.** Telemetry SHOULD be mappable to regulatory requirements (NIS2, IEC 62443, ISO 27001, SOC 2) to produce certification-ready evidence.
- **Federated audit.** A governance authority MAY conduct supply-chain-wide audit from a single point through federated telemetry exchange.

Detailed forensic state (captured process memory, connection state, etc.) is access-controlled and exchanged only between authorised parties; it is never part of the routine token.

---

## 8. Integration Profiles

SGP defines optional integration profiles so that security governance interoperates with adjacent environments. Each profile is a constraint on how SGP tokens and state are exchanged in that environment; none changes the base token format.

- **Cloud-native profile.** Per-container and per-invocation security tokens; multi-cloud posture continuity using AGP cross-platform verification.
- **Industrial (OT) profile.** Security attestation that coexists with safety-instrumented operation under a strict precedence rule — *safety takes precedence over security* — and that respects deterministic-timing constraints; tokens carry an IEC 62443 Security Level field.
- **Zero-trust profile.** Continuous, posture-aware access decisions driven by a live token stream, where access is a continuously evaluated function of current posture rather than a one-time grant, including immediate revocation on a containment event.

---

## 9. Conformance

An implementation is **SGP-conformant** if it:

1. Produces and parses security tokens per §4 as valid AGP tokens with the security extension.
2. Generates continuous, chainable attestation with gap detection.
3. Executes the verification sequence of §5.
4. Supports cross-domain verification and supply-chain chaining per §6.
5. Exchanges governance state and audit telemetry with tamper-evident provenance per §7.
6. Passes the published SGP conformance test vectors.

Generation of trust-anchored security tokens, and the containment/remediation behaviours they attest, require a licensed enforcement layer and are out of scope for protocol conformance.

---

## 10. Relationship to Other UGP Protocols

SGP is an AGP domain extension. Its tokens compose with tokens from other domains through AGP `cross_domain_attestations`: a security token can sit alongside an AI-output token and an alignment token in a single verifiable chain, so that an auditor can trace one continuous line of evidence across an AI system's security posture, its outputs, and its provenance.

---

*This is a working draft published for review. Detection-signature formats, baseline-profiling procedures, and containment internals are intentionally excluded from the public specification.*
