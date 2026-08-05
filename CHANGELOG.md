# Changelog

All notable changes to the UGP open protocol layer are recorded here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
The **protocol** is versioned independently of this repository via the `ugp_version`
field in the compliance token (see [`spec/CORE.md`](spec/CORE.md#9-versioning-and-extensions));
entries below note the protocol version they affect where relevant.

## [Unreleased]

### Added
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
