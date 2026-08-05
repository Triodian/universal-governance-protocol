<!--
Thanks for contributing to the UGP. Please keep changes focused and read
CONTRIBUTING.md before opening. Security/verification-bypass issues must NOT
be filed here — see SECURITY.md.
-->

## What this changes

<!-- One or two sentences. Link the issue this addresses: "Closes #123". -->

## Type of change

- [ ] Editorial / clarification (no observable protocol change)
- [ ] Backward-compatible protocol change (minor version)
- [ ] Backward-incompatible protocol change (major version)
- [ ] Reference implementation / SDK / tooling
- [ ] Community-health or repo scaffolding

## Affected documents / areas

<!-- e.g. spec/CORE.md §4, domains/DLGP.md -->

## Backward compatibility

<!--
Minor versions must not break existing verifiers; older implementations must
gracefully handle unknown extensions (CORE.md §9). Describe the impact, or write
"none".
-->

## Conformance test vectors

<!--
For any change to observable behaviour, describe the vectors added or updated:
cases that must verify, and — for a rejection/bypass fix — cases that must now be
rejected with the correct error code. See spec/CONFORMANCE.md.
-->

## Checklist

- [ ] I have read [CONTRIBUTING.md](../blob/main/CONTRIBUTING.md).
- [ ] This change respects the open / enforcement-layer boundary (nothing here requires enforcement-layer internals to be *verified*).
- [ ] This change does not add adversary-sensitive operational detail that the public specs deliberately exclude.
- [ ] Internal Markdown links still resolve.
- [ ] For observable behaviour changes, conformance vectors are added or updated (or an issue is filed to track them).
- [ ] I agree my contribution is licensed under Apache 2.0.
