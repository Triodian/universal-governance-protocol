# Governance and Stewardship

The Universal Governance Protocol is an open standard maintained by its **Steward**. This document explains who the Steward is today, the planned transfer of stewardship to an independent foundation, the open/enforcement layer separation, and how contribution and decision-making work in the interim.

---

## 1. Who stewards the UGP

**Today (interim stewardship).** The UGP is managed by **Orthogramic Pty Ltd**, which authored the protocol and currently maintains the specifications, the reference materials, the conformance suites, and this repository. In this phase the project is small and maintainer-led: the current maintainer is **Damian Hickey**.

**Planned (independent stewardship).** Orthogramic intends to transfer stewardship of the open protocol layer to an independent, public-interest body — the **Casuarina Foundation** — once that Foundation is established. On transfer, the Casuarina Foundation becomes "the Steward" wherever this repository uses that term, and assumes maintenance of the specification, the conformance and certification programmes, and the public-benefit commitments below.

The word **"Steward"** is used throughout the specification documents deliberately, so that the protocol text does not have to change when stewardship transfers. Where a spec says "maintained by the UGP Steward," that means Orthogramic Pty Ltd today and the Casuarina Foundation after hand-over.

---

## 2. A public good with a custodian

The UGP rests on a deliberate separation:

- an **open, royalty-free protocol layer** that anyone can implement, and
- an **enforcement layer** that carries the cryptographic weight of compliance.

The open layer drives adoption; the enforcement layer anchors trust at the point where regulatory acceptance demands it. The **compliance token** is the bridge between them. This is the same structural pattern used by Arm (open architecture, licensed cores), Bluetooth (open protocol, certified radios), and HDMI (open connector, licensed content protection). In each case the open specification is genuinely open, and the licensed component sits precisely where trust and value concentrate.

The project is committed to four principles, which the interim Steward holds now and which are intended to be entrenched in the Casuarina Foundation's constitution on hand-over:

- **Open by construction.** The protocol specifications, FCSL, the constraint taxonomy, the compliance-token format, the reference implementations, the conformance suites, and the SDKs are published under an open licence (Apache 2.0) and are intended to remain free to access, free to implement, and free to teach with, in perpetuity, under a **Public Access Guarantee** to be made irrevocable by the Foundation.
- **Enforcement where it matters.** Trust-anchored compliance tokens — the evidence that regulators, insurers, and procurement processes accept — are issued only by licensed implementations of the enforcement layer.
- **Independent stewardship.** Once established, the Foundation is intended to operate the protocol, the conformance suites, and the certification programme independently, and to sell no products itself.
- **Ecosystem, not gatekeeping.** Membership, training-partner, and constraint-library programmes are intended to let the people who build with the UGP benefit commercially from doing so — within bounds set by the Foundation's governance provisions.

---

## 3. What the Steward does

- **Publishes and maintains the open protocol layer.** The specifications in this repository, FCSL, the constraint taxonomy, and the compliance-token format, updated through a transparent process.
- **Operates conformance and certification.** The conformance test suites, the certification programme for implementations, and the implementer directory.
- **Stewards the enforcement layer's availability.** The enforcement layer is intended to be made available to qualifying implementers on **fair, reasonable, and non-discriminatory (FRAND)** terms after hand-over. In the interim, licensing of the enforcement layer is handled by Orthogramic Pty Ltd.
- **Advises** governments and standards bodies on structurally enforceable AI governance.
- **Funds public-benefit programmes** from licensing income once the Foundation is established (for example a developing-nation programme), through a published, priority-ordered allocation.

## 4. What the Steward does not do

A clean line between mission stewardship and commercial activity is what makes the structure durable. The Steward does **not**:

- Sell hardware or software products to end customers — commercial implementations are the responsibility of licensees.
- Charge for specification access — the Public Access Guarantee is intended to be irrevocable.
- Let the certification body also profit from paid training — certification is kept vendor-neutral and, once the Foundation runs it, structurally separated from training delivery (training is delivered by independent partners).

> **Note on the interim phase.** While Orthogramic Pty Ltd both authors the protocol and holds the enforcement-layer commercial interest, the separation above is a stated commitment rather than a structurally independent fact. Making it structurally independent is precisely the purpose of transferring stewardship to the Casuarina Foundation. This note will be removed on hand-over.

---

## 5. Decision-making (interim)

While the project is maintainer-led:

- **Proposals** are made through GitHub issues and pull requests (see [`CONTRIBUTING.md`](CONTRIBUTING.md)).
- **Decisions** on the specification are made by the maintainer, in the open, with rationale recorded in the issue or pull request.
- **Backward compatibility** is treated as a hard constraint on minor versions (see [`spec/CORE.md`](spec/CORE.md#9-versioning-and-extensions)).
- **Conformance test vectors** are the arbiter of observable behaviour, not prose alone (see [`spec/CONFORMANCE.md`](spec/CONFORMANCE.md)).

As the project grows, review responsibilities will be delegated to named reviewers and, after hand-over, to the Foundation's technical committees. The [`CODEOWNERS`](.github/CODEOWNERS) file records current review routing.

---

## 6. The transfer to the Casuarina Foundation

On establishment of the Casuarina Foundation, Orthogramic Pty Ltd intends to:

- assign or exclusively license the open protocol layer to the Foundation under terms that preserve the Public Access Guarantee;
- transfer operation of the conformance and certification programmes to the Foundation;
- place the enforcement-layer patent rights under the Foundation's stewardship, licensed back to Orthogramic (and to others on FRAND terms) so that the Foundation, not the operating company, holds the governance root;
- seat a board that represents the public interest, technical excellence, civil society, and the developing world, rather than licensees or model developers.

Until that transfer completes, this repository and its specifications are maintained by Orthogramic Pty Ltd under the commitments in this document.

---

## 7. The open/enforcement boundary, precisely

For contributors and implementers, the boundary is worth stating plainly:

**In this repository (open, Apache 2.0):** the Core specification, FCSL, the trust and verification model, the domain sub-protocols and profiles, conformance requirements and test vectors, reference implementations, and SDKs. A **verification-conformant** implementation — one that receives and checks tokens — can be built entirely from these materials.

**Not in this repository (licensed enforcement layer):** the mechanisms that generate trust-anchored signatures and make the governance step non-bypassable. A **generation-conformant** implementation — one that produces trust-anchored tokens a regulator will accept — requires the enforcement layer.

You can build, test, verify, teach, and extend on the open layer without any licence. A licence is required only to *issue trust-anchored tokens*.

---

## 8. Contact

- **Project / specification:** open an issue or discussion in this repository.
- **Security and verification issues:** see [`SECURITY.md`](SECURITY.md) — please report privately, not in a public issue.
- **Stewardship, licensing, and the Foundation transfer:** Orthogramic Pty Ltd — info@orthogramic.com.

*This is a plain-language overview of an evolving governance arrangement. The Casuarina Foundation's constitution and irrevocable provisions will govern once established; until then, the commitments here are those of Orthogramic Pty Ltd as interim Steward.*
