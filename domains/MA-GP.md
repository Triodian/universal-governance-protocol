# Multi-Agent Governance Protocol (MA-GP)

## Open Protocol Specification — Working Draft

**Layer:** UGP domain extension over AGP (multi-agent coordination governance)
**Status:** Draft for public review · Maintained by the UGP Steward (see GOVERNANCE.md)
**Licence:** Apache 2.0 (specification, reference implementations, conformance suite)
**Date:** June 2026

> **Scope note.** This document specifies the *open protocol layer* of the Multi-Agent Governance Protocol — the formats and exchanges for propagating governance across agent boundaries, governing tool use, and attesting collective action. It does not specify the hardware enforcement that binds action boundaries or signs coordination tokens; those are provided by licensed implementations and are out of scope.

---

## 1. Purpose

When an AI agent's output becomes another agent's input, governance can be lost at the boundary. As agents autonomously invoke tools and other agents, chains of AI-to-AI interaction form in which no single system is accountable for the final outcome and constraints are diluted at each handoff. MA-GP calls this the **accountability horizon problem** and addresses it at the protocol layer.

MA-GP defines how governance constraints and provenance *propagate* across agent boundaries, how *handoffs* preserve governance state, how *tool invocations* are governed, and how a *collective* multi-agent operation is attested as a whole — not merely as a set of individually governed agents. Its tokens are AGP-compatible and add a multi-agent domain extension.

---

## 2. Terminology

- **Agent** — an autonomous system that produces outputs, invokes tools, or invokes other agents.
- **Propagation** — transfer of a constraint set across an agent boundary such that the receiver inherits it.
- **Handoff** — transfer of control and governance state from one agent to another.
- **Collective action** — a coordinated operation across multiple agents, governed as a unit.
- **AIBOM** — AI Bill of Materials: the attested inventory of components constituting an AI system.

RFC 2119 keywords apply.

---

## 3. Protocol Overview

MA-GP defines protocol artefacts for:

1. **Cross-agent constraint propagation** (§4).
2. **Agent handoff verification** (§5).
3. **Provenance and AI Bill of Materials** (§6).
4. **Action boundaries and tool-use governance** (§7).
5. **Federated learning attestation** (§8).
6. **Coordination and collective attestation** (§9).

Hardware enforcement of action boundaries is performed by licensed implementations; the protocol covers how permitted-action constraints and governed outcomes are *declared, exchanged, and attested*.

---

## 4. Cross-Agent Constraint Propagation

When Agent A's output is consumed by Agent B, B MUST be able to inherit and respect the constraints that governed A's output.

The protocol defines a serialisable constraint-set representation for inter-agent transfer with integrity binding. On receipt, B's governance pipeline MUST validate the propagated constraints against its own constraint store *before* processing A's output. A propagation token attests that constraint inheritance was *verified*, not merely asserted, and links A's and B's governance-state identifiers for end-to-end lineage.

Where multiple agents propagate constraints to one receiver, the protocol defines constraint merging and priority resolution across heterogeneous sets. Propagation MUST support a timeout; on propagation failure, the receiver MUST isolate the received output rather than process it ungoverned.

---

## 5. Agent Handoff Verification

A handoff is the moment control passes between agents. MA-GP defines a handoff sequence:

1. The initiating agent captures an attested snapshot of its governance state (constraint set, governance-state identifier, convergence history, token chain).
2. The snapshot is transferred to the receiving agent.
3. The receiver verifies the snapshot within an isolated governance context before acknowledging completion.
4. A handoff-completion token attests governance continuity across the boundary.

If verification fails, the handoff MUST enter an isolation state and the received output MUST NOT be processed, preventing governance-state loss. The protocol supports partial (multi-stage) handoff, rollback to the pre-handoff state on failure, and parallel handoff for fan-out patterns.

---

## 6. Provenance and AI Bill of Materials

### 6.1 Supply-Chain Provenance

MA-GP defines a multi-hop provenance chain binding model identity (origin), output identity (production), and participant identity (each supply-chain node), with each hop attested rather than self-reported. The chain supports branching (fine-tuning, adaptation), merging (ensembles), and revocation (recalled models), and MAY gate access based on supply-chain status.

### 6.2 AIBOM Attestation

An AI Bill of Materials enumerates every model, dataset, library, and configuration in a deployed system. MA-GP defines an *attested* AIBOM in which each component entry is bound to an attested identity and chained into a composite record, so the inventory cannot be falsified without compromising the attesting pipeline. The protocol defines an AIBOM verification exchange for third-party auditors, delta attestation for incremental updates, dynamic AIBOM for runtime component changes, and optional AIBOM-gated deployment (no valid AIBOM, no production release).

---

## 7. Action Boundaries and Tool-Use Governance

### 7.1 Action Boundaries

MA-GP defines how an agent's permitted action space is declared as a constraint set and how each proposed action (tool invocation, API call, file operation, code execution) is checked against it before execution is permitted. The protocol carries action-type classification, permission tokens, rate-limit declarations, and boundary-violation outcomes. Boundaries MAY be context-dependent (tighter in production than development), MUST be inheritable by spawned sub-agents, and SHOULD be governable across multi-step action sequences.

### 7.2 Tool-Use Governance

The tool-use protocol defines: a **tool capability declaration** (functions, parameter constraints, governance requirements); a **permission request/grant** exchange yielding a permission token; an **invocation attestation** binding parameters to the permission grant and governance state; **result validation** against expected constraints; and a **tool-use compliance token** proving the invocation was governed. It supports tool capability discovery, permission delegation for multi-agent tool sharing, invocation chaining with cumulative governance, and population-level rate governance.

This protocol is designed to interoperate with existing agent-tooling ecosystems as a governance layer over their invocation mechanisms.

---

## 8. Federated Learning Attestation

For distributed training where data sovereignty prevents centralised governance, MA-GP defines:

- **Per-node constraint compliance attestation** — each node produces an attested compliance proof for its local update without revealing its private data, and the aggregator verifies all contributions comply without accessing node data.
- **Aggregation governance attestation** — the aggregation step itself is governed: aggregation parameters are bounded, non-compliant or anomalous updates are detected by divergence analysis of update vectors and isolated, and the aggregated model carries a token proving the aggregation was governed (distinct from proving each update was compliant). Supports weighted, hierarchical, incremental, and rollback-capable aggregation.

Node-specific detection internals are out of scope; the protocol carries attested outcomes and proofs.

---

## 9. Coordination and Collective Attestation

### 9.1 Coordination Protocol

MA-GP defines a coordination exchange in which agents advertise capabilities and governance constraints, a coordination request specifies the collective action requirements and governance boundaries, and participating agents synchronise governance state across the operation.

### 9.2 Collective Compliance Token

The distinguishing artefact is the **collective compliance token**: an attestation that the *coordinated operation as a whole* was governed, capturing constraints that no individual agent's governance would capture on its own. Collective-action boundaries MUST prevent any participating agent from exceeding the collective constraint. The protocol supports dynamic membership (join/leave mid-operation), hierarchical coordination for multi-tier topologies, and conflict resolution for incompatible agent constraints.

### 9.3 Oracle Determinism Profile

An optional profile governs AI systems that supply data to consensus systems requiring determinism. It attests that an oracle's output was produced through a governed, determinism-verified process (consistency across evaluations, bounded within consensus-compatible ranges), yielding a token that an external validator can verify. This reconciles AI's inherent non-determinism with consensus systems' need for verifiable, deterministic inputs.

---

## 10. Token Format

MA-GP tokens are AGP compliance tokens (`domain: "ma-gp"`) carrying a multi-agent extension whose shape depends on the artefact (propagation, handoff, provenance, AIBOM, action, tool-use, federated, coordination). All MA-GP tokens MUST be valid AGP tokens at the base-field level and MUST chain via `provenance_chain.previous_token` so that an agent chain yields an unbroken governance lineage.

---

## 11. Conformance

An implementation is **MA-GP-conformant** if it:

1. Produces and parses MA-GP tokens per §10 as valid AGP tokens with the multi-agent extension.
2. Verifies constraint propagation before processing received outputs, and isolates on failure (§4).
3. Executes the handoff sequence with isolation-on-failure (§5).
4. Supports attested provenance and AIBOM verification (§6).
5. Declares and enforces action boundaries and the tool-use exchange (§7).
6. Produces collective compliance tokens for coordinated operations (§9).
7. Passes the published MA-GP conformance test vectors.

Generation of trust-anchored tokens and hardware enforcement of action boundaries require a licensed enforcement layer and are out of scope for protocol conformance.

---

## 12. Relationship to Other UGP Protocols

MA-GP is the protocol that keeps governance continuous when systems compose. Its tokens reference AGP output tokens, FA-GP alignment tokens, and SGP security tokens through AGP cross-domain attestation, so a multi-agent operation can present one verifiable chain showing that each participating model was aligned, each output was governed, each handoff preserved constraints, and the collective operation respected its collective bounds.

---

*This is a working draft published for review. Conformance test vectors and the per-artefact extension schemas are maintained separately by the UGP Steward.*
