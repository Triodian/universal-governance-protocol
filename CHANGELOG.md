# Changelog

All notable changes to the UGP open protocol layer are recorded here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
The **protocol** is versioned independently of this repository via the `ugp_version`
field in the compliance token (see [`spec/CORE.md`](spec/CORE.md#9-versioning-and-extensions));
entries below note the protocol version they affect where relevant.

## [Unreleased]

### Added
- `spec/EXCHANGE.md`: the governed-output exchange. Defines the `GovernRequest`
  (candidate output or prompt, inline or by reference, plus `constraint_set_uri`
  and optional pinned identifier) and `GovernResponse` (governed output plus
  compliance token); the `ugp.exchange` token extension binding a token to its
  request; requester verification steps additional to CORE §5; freshness and
  retry rules; exchange error codes; informative HTTP, gRPC, queue and in-process
  bindings; and deployment topologies including third-party hosted governing
  services. Backward-compatible (minor): no existing field or verification step
  changes. Cross-references added to CORE, CONFORMANCE, README and GLOSSARY.
- Initial public documentation set for the open protocol layer:
  - Core specification, FCSL, trust-and-verification model, and conformance classes (`spec/`).
  - Domain sub-protocols: AGP, SGP, FA-GP, MA-GP, DLGP, CDP (`domains/`).
  - Public domain-profile catalogue (`domains/PROFILES.md`).
  - Governance and stewardship overview, glossary, and FAQ.
  - Community-health files: Code of Conduct, Security Policy, Contributing guide.
  - Repository scaffolding: issue/PR templates, CODEOWNERS, `.gitignore`, CI link check.

### Notes
- Documents are **working drafts published for review**. Conformance test vectors,
  the formal trust-hierarchy schema, and per-domain extension schemas are maintained
  by the UGP Steward and evolve alongside these documents.

<!--
Template for future releases:

## [X.Y.Z] - YYYY-MM-DD
### Added
### Changed
### Deprecated
### Removed
### Fixed
### Security
-->
