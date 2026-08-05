# UGP Domain Sub-Protocols

Every UGP sub-protocol extends the [Core](../spec/CORE.md): it produces tokens in the same base format, reuses Core verification, handshake, and state-exchange machinery, and adds its own domain-extension fields. A token generated under any sub-protocol is verifiable by any UGP verifier at the base-field level. This is the foundation of cross-domain interoperability.

## The core sub-protocols

| Protocol | Governs | Attests |
|---|---|---|
| [**AGP** — AI Governance Protocol](AGP.md) | AI outputs at inference time | That an output was governed against its constraint set before it was acted upon. The horizontal protocol the others extend. |
| [**SGP** — Security Governance Protocol](SGP.md) | Security posture of a system, network, or supply chain | That an entity was *continuously* governed against a security baseline — process attestation, not configuration snapshots. |
| [**FA-GP** — Frontier AI Alignment Protocol](FA-GP.md) | Training-, alignment-, and data-generation-time processes | That a deployed model was shaped by a governed process — linking training-time evidence to deployment-time tokens. |
| [**MA-GP** — Multi-Agent Governance Protocol](MA-GP.md) | Agent-to-agent handoffs, tool use, and collective action | That governance and provenance are preserved across agent boundaries, and that a coordinated operation as a whole was governed. |
| [**DLGP** — Data-Lifecycle Governance Protocol](DLGP.md) | Data at rest, in transit, and in use | That every operation on a governed data object satisfied its constraint set, linked into one lifecycle chain. |
| [**CDP** — Constraint Declaration Protocol](CDP.md) | The seam between a third-party vendor and a governance layer | That vendor-supplied constraints were declared, scope-bound, reconciled against the operator's floor, and accepted — not duplicated or blindly trusted. |

## Domain profiles

Beyond the core sub-protocols, the same architecture has been specialised into a large library of **domain profiles** for specific regulated and safety-critical settings — cloud-native infrastructure, industrial control, medical devices, autonomous vehicles, aviation, post-quantum readiness, and many more. Each profile describes *what is governed and what is attested* in its domain, and produces tokens in the universal format.

The public catalogue of profiles is in [`PROFILES.md`](PROFILES.md).

## How they compose

Because every token shares the base format, evidence forms a continuous chain across domains:

```
FA-GP            AGP              MA-GP            SGP
(training)  →   (inference)  →   (orchestration) → (monitoring)
   │               │                 │                │
   └──────── one verifiable cross-domain evidence chain ───────┘
```

A regulator asking "show me how this system was built, how it behaves, and how it is operated" can be answered with a single chain of tokens rather than four disconnected reports.
