# UGP Governed-Output Exchange

## Open Protocol Specification — Working Draft

**Layer:** UGP Layer 2 (Governance), with constraint binding into Layer 3
**Status:** Draft for public review · Maintained by the UGP Steward (see GOVERNANCE.md)
**Licence:** Apache 2.0 (specification, reference implementations, conformance suite)

> **Scope note.** This document specifies how a governed output is *requested* and *returned*: the request and response messages a workload exchanges with a governing service, how a returned compliance token is bound to the request that produced it, and the checks a requester or relying party performs on the result. It builds on the token format and verification procedure in [`CORE.md`](CORE.md) and changes neither. It does not specify how the governing service evaluates, refines, or signs; those belong to the enforcement layer and are out of scope. It does not specify transport security or caller authorisation, which remain the job of TLS, OAuth, mTLS, or equivalent. The key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY are interpreted as in RFC 2119.

---

## 1. Purpose

[`CORE.md`](CORE.md) defines what a compliance token is and how any party verifies one. It does not define how a workload *obtains* a governed output: there is no message for "here is a candidate output (or a prompt), govern it against this constraint set and return the output with its token". Without that message, every deployment in which the governing step runs somewhere other than inside the workload's own process has to invent its own call, and integrators cannot tell whether two implementations interoperate.

This document fills that gap. It defines one request/response exchange, **Govern**, which makes the following a first-class protocol topology:

> A workload sends a governing service a candidate output or a prompt, naming the constraint set to apply. The service returns the governed output together with a compliance token. The workload, and any regulator, auditor, insurer, or counterparty it passes the output to, verifies the token under [`CORE.md` §5](CORE.md#5-token-verification) and §6 below. None of them has to trust the service's operator or software stack, only the trust anchor that signed the token.

The exchange is the same whether the governing service runs in the same process, on the same host, in the customer's own network, or is operated by a third party. Topology changes where the call travels. It does not change what is sent or what is verified.

---

## 2. Roles

- **Requester.** The workload that needs a governed output: an application, an agent, a pipeline stage, or a gateway acting on their behalf.
- **Governing service.** A generation-conformant implementation ([`CONFORMANCE.md`](CONFORMANCE.md#generation-conformant)) that accepts Govern requests, runs the governance step under its enforcement layer, and returns the governed output with a compliance token. The governing service may also operate the model that produces the output (§4.2).
- **Relying party.** Any party that consumes the governed output and verifies its token: the requester itself, a downstream system, a regulator, an auditor, an insurer. A relying party does not need to take part in the exchange; it only needs the output, the token, and the verification material of [`TRUST-MODEL.md`](TRUST-MODEL.md).

One party can hold more than one role. A requester that verifies the token before acting on the output is also a relying party.

---

## 3. Exchange Overview

```
 Requester                         Governing service                  Relying party
     │                                     │                                │
     │── handshake (CORE §6.1) ───────────▶│                                │
     │◀──────────── negotiated params ─────│                                │
     │                                     │                                │
     │── GovernRequest ───────────────────▶│                                │
     │   constraint_set_uri (+ identifier) │ resolve constraint set         │
     │   candidate | prompt (or ref)       │ evaluate, refine, sign         │
     │   request_id, issued_at_us          │ (enforcement layer, out of     │
     │                                     │  scope)                        │
     │◀── GovernResponse ──────────────────│                                │
     │   status, governed output, token    │                                │
     │                                     │                                │
     │  verify: CORE §5 + EXCHANGE §6      │                                │
     │                                     │                                │
     │── output + token ──────────────────────────────────────────────────▶│
     │                                     │      verify: CORE §5 (+ §6.2   │
     │                                     │      where request is shared)  │
```

1. **Handshake.** Before the first Govern request in a relationship, the parties MUST complete the handshake of [`CORE.md` §6.1](CORE.md#61-handshake-and-capability-negotiation), or rely on a pre-configured relationship that fixes the same parameters. The handshake settles the protocol version, the trust-anchor types the requester will accept, the constraint domains the service supports, and the freshness window (§7). It does not need repeating per request.
2. **Request.** The requester sends a `GovernRequest` (§4).
3. **Governance.** The governing service resolves the named constraint set, runs the governance step, and generates a compliance token for the output it releases. How it does this is out of scope.
4. **Response.** The service returns a `GovernResponse` (§5) carrying the output and its token, or a governance-failure or error response.
5. **Verification.** The requester verifies the token (§6) before acting on the output, and may pass output and token to any relying party, which verifies independently.

---

## 4. The Govern Request

### 4.1 Canonical Form

```json
{
  "ugp_version": "51.0",
  "message_type": "ugp.govern.request",
  "request_id": "rq_7f3c9a1e5b2d48c6a0e1f2d3c4b5a697",
  "issued_at_us": 1735812000000000,

  "domain": "agp",
  "constraint_set": {
    "uri": "ugp://constraints/agp/customer-comms/v12",
    "identifier": "sha256:..."
  },

  "input": {
    "mode": "candidate",
    "media_type": "text/plain; charset=utf-8",
    "content": "Draft reply to the customer ...",
    "content_ref": null,
    "content_hash": "sha256:..."
  },

  "generation": null,

  "provenance": {
    "previous_token": "sha256:...",
    "session_ref": "ugp://provenance/session/..."
  },

  "options": {
    "deadline_us": 1735812000250000,
    "release_on_failure": false
  },

  "extensions": {}
}
```

### 4.2 Field Requirements

| Field | Requirement | Meaning |
|---|---|---|
| `ugp_version` | MUST | Protocol version, as negotiated in the handshake. |
| `message_type` | MUST | `ugp.govern.request`. |
| `request_id` | MUST | Requester-chosen identifier with at least 128 bits of randomness, unique per request (§7). |
| `issued_at_us` | MUST | Time the request was created, Unix epoch microseconds. |
| `domain` | MUST | The sub-protocol under which the output is to be governed (`agp`, `ma-gp`, …). |
| `constraint_set.uri` | MUST | The constraint set and version to apply, in the form of [`FCSL.md` §3](FCSL.md#3-identity-and-versioning). |
| `constraint_set.identifier` | SHOULD | The content hash the requester expects that URI to resolve to (§4.3). |
| `input.mode` | MUST | `candidate` or `generate` (§4.4). |
| `input.media_type` | MUST | Media type of the input content. |
| `input.content` / `input.content_ref` | MUST (exactly one) | The input inline, or a URI from which the service retrieves it (§4.5). |
| `input.content_hash` | MUST | Hash of the input bytes (§4.5). |
| `generation` | MUST in `generate` mode; MUST be `null` in `candidate` mode | Generation parameters (§4.4). |
| `provenance.previous_token` | SHOULD | The prior token in the session, so the returned token chains to it. |
| `provenance.session_ref` | MAY | The provenance ledger or session the output belongs to. |
| `options.deadline_us` | MAY | Absolute time after which the requester will not use a response (§5.4). |
| `options.release_on_failure` | MAY (default `false`) | Whether the service may return a failed output for diagnosis (§5.3). |
| `extensions` | MAY | Self-describing, registered request extensions (§11). |

A requester MUST NOT put caller credentials in the request body. Authentication and authorisation of the requester are carried by the transport (§9).

### 4.3 Constraint-Set Selection and Pinning

The requester selects the constraint set. The governing service MUST NOT substitute another set, another version, or a locally modified variant.

- If `constraint_set.identifier` is present and the URI does not resolve to that identifier at the service, the service MUST reject the request with `EX_CONSTRAINT_SET_MISMATCH` and MUST NOT govern the input under any other set.
- If `constraint_set.identifier` is absent, the service governs under the set the URI currently resolves to, and the requester learns which one from the token's `governance_state.identifier`. A requester that has not pinned the set MUST still check the returned identifier against its own policy (§6.2).

Requests carry no per-request tolerance or constraint parameters. Anything that changes what counts as compliant MUST be expressed as a distinct constraint set with its own identity hash, so that the token always shows exactly which rules governed the output. A request that tries to loosen a declared tolerance is not a valid request.

### 4.4 Input Modes

**`candidate`: govern an output produced elsewhere.** The requester supplies an output candidate, typically produced by a model the requester operates or calls directly. The service evaluates it against the constraint set and refines it as the enforcement layer allows. The governed output it returns MAY differ from the candidate. The token's `convergence_proof.divergence_at_entry` describes the candidate as submitted. `generation` MUST be `null`.

**`generate`: generate and govern.** The requester supplies a prompt, or a reference to one, and the governing service produces the output with a model it operates or has been configured to call, then governs it. `generation` MUST be present:

```json
"generation": {
  "model_ref": "opaque-model-identifier",
  "parameters": {}
}
```

`model_ref` and `parameters` are opaque to this specification and are agreed between requester and service. They are covered by the request hash (§6.2), so the token commits to them. The token attests that the *released output* was governed against the named constraint set. It does not attest that a particular model produced it, unless a domain extension says so (for example an FA-GP cross-domain attestation).

### 4.5 Inline Content, References, and Hashing

- `input.content` carries the input inline. Text media types are carried as strings. Other media types are carried as base64url without padding, and `input.encoding` MUST then be set to `base64url`.
- `input.content_ref` is a URI the governing service dereferences to obtain the input, for large payloads or when the requester keeps content in its own store. The requester is responsible for making the reference resolvable by the service, and for its access control.
- `input.content_hash` MUST be computed over the exact input bytes (after base64url decoding, if applicable). The service MUST compute the hash of the bytes it actually received or retrieved and MUST reject the request with `EX_INPUT_HASH_MISMATCH` if they differ. This stops a reference changing between request and retrieval.

A requester using a third-party governing service SHOULD treat everything in `input` as disclosed to that service's operator. Privacy-preserving deployment, where the input must not be disclosed, needs a governing service inside the requester's own boundary, or enforcement-layer confidentiality guarantees, which are out of scope here.

### 4.6 Serialisation

Requests MUST be serialised under the canonical, byte-deterministic rules of [`CORE.md` §4.5](CORE.md#45-serialisation). The request hash in §6.2 is computed over that canonical form.

---

## 5. The Govern Response

### 5.1 Canonical Form

```json
{
  "ugp_version": "51.0",
  "message_type": "ugp.govern.response",
  "request_id": "rq_7f3c9a1e5b2d48c6a0e1f2d3c4b5a697",
  "status": "governed",

  "output": {
    "media_type": "text/plain; charset=utf-8",
    "content": "Reply to the customer ...",
    "content_ref": null
  },

  "token": { "...": "a complete compliance token in the canonical form of CORE §4.1" },

  "verification_hints": {
    "trust_anchor_refs": ["..."],
    "constraint_set_ref": "ugp://constraints/agp/customer-comms/v12"
  },

  "extensions": {}
}
```

### 5.2 Field Requirements

| Field | Requirement | Meaning |
|---|---|---|
| `ugp_version` | MUST | Protocol version. |
| `message_type` | MUST | `ugp.govern.response`. |
| `request_id` | MUST | The `request_id` of the request being answered. |
| `status` | MUST | `governed`, `governance_failure`, or `error` (§5.3). |
| `output` | MUST when `status` is `governed`; see §5.3 otherwise | The governed output, inline or by reference, under the same encoding rules as §4.5. |
| `token` | MUST when `status` is `governed` or `governance_failure` | The compliance token, carrying the exchange binding of §6.1. |
| `verification_hints` | MAY | Pointers to verification material. **Not trusted**: a verifier MUST NOT treat any hint as a trust anchor or as proof of anything (§6.3). |
| `error` | MUST when `status` is `error` | `{ "code": "EX_...", "message": "...", "retryable": true|false }` (§8). |
| `extensions` | MAY | Self-describing, registered response extensions. |

The response envelope is not covered by the token's signature, like the token envelope of [`CORE.md` §4.4](CORE.md#44-token-envelope). Everything a relying party depends on is in the token, or is checked against it.

### 5.3 Status Semantics

- **`governed`.** The governance step completed and `convergence_proof.divergence_at_exit` is within the constraint set's declared tolerance. `output` is the exact released output, and `token.output_binding.output_hash` MUST be the hash of its bytes.
- **`governance_failure`.** The governance step ran but could not bring the output within tolerance (or within the WCET bound, where one is declared). The service MUST NOT present the output as governed. By default `output` MUST be omitted. If and only if the request set `release_on_failure: true`, the service MAY include the failed output for diagnosis, and the response MUST still carry `status: governance_failure`. Where the enforcement layer can issue a token over the failed attempt, the service SHOULD include it, and it MUST be treated as a governance failure under [`CORE.md` §4.3](CORE.md#43-convergence-proof-semantics).
- **`error`.** The request was not governed: rejected, unresolvable, or not processed in time. `output` and `token` MUST be omitted. Errors are listed in §8.

A requester MUST NOT release, act on, or forward as governed any output whose response status is not `governed` and whose token has not passed verification (§6).

### 5.4 Deadlines

If `options.deadline_us` is present and the service cannot complete before it, the service SHOULD return `EX_DEADLINE_EXCEEDED` rather than a late response. A requester MUST discard any response it receives after its own deadline, and MUST NOT treat a timeout as permission to use an ungoverned output. Fallback behaviour on timeout (retry, degrade to a stricter constraint set, refuse) is requester policy and SHOULD be declared in the handshake, consistent with [`domains/AGP.md` §6.2](../domains/AGP.md#62-constraint-compatibility-negotiation).

---

## 6. Binding and Verification

### 6.1 The Exchange Binding

So that a token can be matched to the request that produced it, a governing service MUST include an `exchange` binding in the token's top-level `domain_extensions`, inside the canonical form covered by the token's signature:

```json
"domain_extensions": {
  "ugp.exchange": {
    "request_id": "rq_7f3c9a1e5b2d48c6a0e1f2d3c4b5a697",
    "request_hash": "sha256:...",
    "input_mode": "candidate"
  }
}
```

| Field | Requirement | Meaning |
|---|---|---|
| `request_id` | MUST | The request's `request_id`. |
| `request_hash` | MUST | SHA-256 over the canonical serialisation of the full `GovernRequest` (§4.6). |
| `input_mode` | MUST | `candidate` or `generate`, copied from the request. |

Because `request_hash` commits to the whole request (constraint-set URI and pinned identifier, input hash, generation parameters, provenance link, issue time), a verifier holding the request can confirm that this token answers *this* request and no other.

Placing the binding in `domain_extensions` keeps the base token unchanged: a verifier that does not recognise `ugp.exchange` still parses and verifies the base fields under [`CORE.md` §9](CORE.md#9-versioning-and-extensions). `ugp.exchange` is a cross-domain extension and is registered through the extension framework (§11).

### 6.2 Requester Verification

On receiving a `governed` response, the requester MUST, before acting on the output:

1. **Run the Core verification sequence**: all five steps of [`CORE.md` §5](CORE.md#5-token-verification) over `token`, with the output-binding step computed over the bytes of `output` as received.
2. **Check request correspondence**: confirm that `ugp.exchange.request_id` equals the request's `request_id`, and that `ugp.exchange.request_hash` equals the hash the requester computes over its own canonical request. A mismatch MUST be reported as `EX_BINDING_MISMATCH`.
3. **Check constraint-set correspondence**: confirm that `governance_state.constraint_set_uri` equals the requested URI, and, if the request pinned `constraint_set.identifier`, that `governance_state.identifier` equals it. If the request did not pin, confirm the returned identifier is acceptable under the requester's own policy. A mismatch MUST be reported as `EX_CONSTRAINT_SET_MISMATCH`.
4. **Check provenance correspondence**: if the request carried `provenance.previous_token`, confirm `token.provenance_chain.previous_token` equals it. A mismatch MUST be reported as `EX_CHAIN_MISMATCH`.

Steps 2–4 are additions for the exchange. They do not alter the Core sequence, and a failure in any of them MUST cause the response to be rejected even if the Core sequence passed.

### 6.3 Relying-Party Verification

A relying party that receives only the output and the token, and not the request, performs the Core sequence of [`CORE.md` §5](CORE.md#5-token-verification). That is sufficient to establish that this output was governed under the stated constraint set and signed by the stated trust anchor. If the relying party also receives the original request, it SHOULD perform §6.2 steps 2–4 as well, which additionally establishes that the output answers that request.

Partial verification under [`CORE.md` §5.1](CORE.md#51-partial-verification) applies unchanged and MUST be reported as partial.

### 6.4 What the Exchange Does and Does Not Establish

The exchange adds nothing to what a valid signature means. Following [`TRUST-MODEL.md` §6](TRUST-MODEL.md#6-what-the-open-layer-guarantees--and-what-it-doesnt):

**A verified exchange establishes** that the returned output is the one the token binds, that the token was signed by an anchor the verifier accepted, that it names the constraint set the requester asked for, and that it answers this specific request, without the verifier trusting the governing service's operator, network path, or software stack.

**It does not, by itself, establish** that the anchor is strong, that the governance step could not have been bypassed, or, in `generate` mode, which model produced the output. Those depend on the enforcement layer and on the trust the verifier places in the anchor. Treating `verification_hints` as anything more than pointers would reintroduce trust in the service, which is why §5.2 forbids it.

---

## 7. Freshness, Replay, and Retries

- The governing service MUST reject a request whose `issued_at_us` falls outside the freshness window settled in the handshake, with `EX_STALE_REQUEST`.
- The service MUST keep a record of `request_id` values for at least the freshness window. A request that reuses a `request_id` with a **different** `request_hash` MUST be rejected with `EX_REQUEST_ID_REUSED`.
- A request that repeats a `request_id` with an **identical** `request_hash` is a retry. The service MAY return the original response unchanged, and SHOULD NOT govern the input a second time. This makes retries over unreliable transports safe without issuing two tokens for one request.
- Relying parties continue to apply the token freshness checks of [`CORE.md` §5.2](CORE.md#52-freshness). The exchange binding gives them one more replay signal: a token whose `request_id` has already been seen with a different output is suspect.

---

## 8. Exchange Error Codes

Exchange errors are reported in `GovernResponse.error.code` (service side) or by the requester's verifier (requester side). They are distinct from, and additional to, the token-verification failure codes of [`CORE.md` §5.3](CORE.md#53-failure-reporting). The symbolic codes below are normative; the code registry and test vectors are maintained in the conformance suite.

| Code | Raised by | Meaning | Retryable |
|---|---|---|---|
| `EX_VERSION_UNSUPPORTED` | Service | `ugp_version` was not negotiated or is not supported. | No (re-handshake) |
| `EX_MALFORMED_REQUEST` | Service | The request is not well formed or breaks a MUST in §4. | No |
| `EX_DOMAIN_UNSUPPORTED` | Service | The service does not govern under `domain`. | No |
| `EX_CONSTRAINT_SET_UNKNOWN` | Service | `constraint_set.uri` does not resolve at the service. | No |
| `EX_CONSTRAINT_SET_MISMATCH` | Service / Requester | The URI resolves to a different identifier than pinned, or the token names a different set (§4.3, §6.2). | No |
| `EX_INPUT_UNAVAILABLE` | Service | `input.content_ref` could not be retrieved. | Yes |
| `EX_INPUT_HASH_MISMATCH` | Service | The input bytes do not match `input.content_hash`. | No |
| `EX_GENERATION_UNSUPPORTED` | Service | `generate` mode, or the named `model_ref`, is not offered. | No |
| `EX_STALE_REQUEST` | Service | `issued_at_us` is outside the freshness window. | No (re-issue) |
| `EX_REQUEST_ID_REUSED` | Service | `request_id` reused with a different request. | No (re-issue) |
| `EX_DEADLINE_EXCEEDED` | Service | Governance could not complete before `deadline_us`. | Yes |
| `EX_CAPACITY` | Service | The service cannot accept the request now. | Yes |
| `EX_BINDING_MISMATCH` | Requester | The token's `ugp.exchange` binding does not match the request (§6.2). | No |
| `EX_CHAIN_MISMATCH` | Requester | The token does not chain to the requested `previous_token` (§6.2). | No |

A governance failure is not an error: it is reported as `status: governance_failure` (§5.3), because the service did its job and the output did not pass.

Authentication and authorisation failures are reported by the transport (for example HTTP 401/403, gRPC `UNAUTHENTICATED`/`PERMISSION_DENIED`), not by an exchange code.

---

## 9. Transport Bindings (Informative)

The exchange is transport-agnostic, like the Core. The messages in §4 and §5 are normative; the bindings below are informative patterns. Resource paths, header names, and service names in this section are illustrative and become normative only when registered under §11.

**HTTP.** The requester `POST`s the canonical `GovernRequest` with `Content-Type: application/ugp+json` to a governing-service URI agreed out of band or advertised in the handshake. The service answers with the canonical `GovernResponse` in the same media type. A `governance_failure` is a successful exchange and SHOULD use status `200`. Exchange errors MAY be mapped to `4xx`/`5xx` with the `GovernResponse` (carrying `error`) as the body. Where the requester wants the governed output as a raw body, for example to stream it straight to a client, the service MAY return the output as the body and the token, base64url-encoded in canonical form, in a `UGP-Token` response header, with `UGP-Request-Id` and `UGP-Status` alongside. Transport security is TLS. Caller authorisation is OAuth 2.0, mTLS, or equivalent.

**gRPC.** A unary method carrying the two messages, for example `ugp.exchange.v1.GoverningService/Govern(GovernRequest) returns (GovernResponse)`, with the token optionally duplicated in `ugp-token-bin` trailing metadata for interceptors that verify without parsing the body.

**Message queues.** The request is published to the service's request topic with `request_id` as the correlation identifier and a reply-to address. The response is published to the reply-to address. Because delivery may repeat, the retry rule of §7 is what makes at-least-once queues safe.

**In-process.** A governing service linked into the requester's process exposes the same request and response as a library call. The messages, the binding, and the verification are unchanged. Only the transport disappears.

---

## 10. Deployment Topologies (Informative)

All of the following use the same messages and the same verification. They differ only in where the governing service runs and who operates it.

| Topology | Where the governing service runs | Note |
|---|---|---|
| **Co-located** | Inside the requester's process or host. | Lowest latency. The requester's operator also operates the enforcement layer. |
| **Customer-hosted service** | A separate service inside the requester's own network or tenancy. | Several workloads share one governing service; input stays within the customer boundary. |
| **Third-party hosted service** | A service operated by another party (a vendor, a managed-service provider, a cloud provider). | The requester and relying parties verify the token without trusting the operator (§6.4); the operator does see the input (§4.5). |
| **Inline gateway** | A proxy between the workload and a model provider, which governs in `candidate` mode on the provider's response before returning it. | The workload's existing provider calls are unchanged; the gateway is the requester. |

Delegated and offline verification ([`CORE.md` §5.3](CORE.md#53-failure-reporting), [`TRUST-MODEL.md` §3.1](TRUST-MODEL.md#31-trust-chain-resolution)) compose with every topology: a relying party may verify through a trusted intermediary, or against pre-distributed trust anchors with no connection to the governing service at all.

---

## 11. Extensions and Versioning

- The `ugp.exchange` token extension (§6.1), the request and response `extensions` objects, and the transport identifiers of §9 are registered through the extension framework of [`CORE.md` §9](CORE.md#9-versioning-and-extensions).
- Request and response messages follow the Core's major/minor rules. A governing service MUST ignore unknown request extensions it is not asked to honour. A requester MUST ignore unknown response extensions, but MUST NOT ignore anything that affects §6 verification.
- This document adds messages and an optional token extension. It changes no existing field or verification step, so existing tokens and verifiers are unaffected and the change is backward-compatible (minor version).

---

## 12. Conformance

Exchange conformance is declared alongside the existing classes in [`CONFORMANCE.md`](CONFORMANCE.md#exchange-conformance):

- An **exchange-conformant governing service** is generation-conformant and additionally accepts `GovernRequest`, returns `GovernResponse` per §5, enforces the constraint-set rules of §4.3, emits the `ugp.exchange` binding of §6.1, applies the freshness and retry rules of §7, and reports the errors of §8.
- An **exchange-conformant requester** constructs canonical requests per §4, performs the full verification of §6.2 before acting on any output, never acts on an output whose status is not `governed`, and discards late responses per §5.4. A requester needs to be verification-conformant, not generation-conformant.

---

## 13. Open Questions for Review

- **Signature coverage.** This document requires `ugp.exchange` to be inside the signed canonical form. [`CORE.md`](CORE.md) does not yet state the exact scope of `output_binding.hardware_signature`; that statement belongs in the Core and should be added there.
- **Streaming outputs.** A token is issued over a complete released output. Incremental release, where governed fragments stream to the requester, needs a chaining scheme and is deferred.
- **Asynchronous requests.** Long-running governance (accept now, deliver later via callback or polling) is not yet specified. The request and response messages are designed to carry over unchanged, with `request_id` as the correlation key.
- **Batching.** A batch request carrying several inputs, each governed and tokenised independently, is a natural extension and is deferred to keep this draft minimal.

---

*This is a working draft published for review. The error-code registry, the `ugp.exchange` extension registration, and the exchange conformance test vectors are maintained by the UGP Steward alongside the Core vectors.*
