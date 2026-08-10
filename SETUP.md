# Repository setup

This file is a one-time checklist for publishing the repository. It is **not** part of
the protocol documentation and can be deleted after setup.

## 1. Configured values

The repository is set up for the following, so no placeholder editing is needed before the first push:

| Setting | Value | Where |
|---|---|---|
| Maintainer / reviewer | `@damohickey1966` | `.github/CODEOWNERS` |
| Repository slug | `triodian/universal-governance-protocol` | `.github/ISSUE_TEMPLATE/config.yml` |
| Contact address | `info@triodian.com` | `SECURITY.md`, `CODE_OF_CONDUCT.md`, `GOVERNANCE.md` |

Optional: `LICENSE` references the Apache 2.0 text by URL. Paste the full Apache 2.0 body into it if your convention requires the complete text.

> Stewardship is currently **Orthogramic Pty Ltd (interim)**, with a documented intent to transfer to the **Casuarina Foundation** once it is established (see `GOVERNANCE.md`). You do not need to create the Foundation to publish the repository; the governance docs already describe the interim arrangement and the planned hand-over.

Sanity check before pushing (should return nothing):

```bash
grep -rn "@ORG/\|ORG/REPO" . --include='*.md' --include='*.yml' --include='CODEOWNERS'
```

## 2. Initialise and push

```bash
cd ugp-docs

git init
git add .
git commit -m "Initial public documentation set for the UGP open protocol layer"
git branch -M main

# Create the repo first (via the GitHub UI or `gh repo create`), then:
git remote add origin git@github.com:orthogramic/universal-governance-protocol.git
git push -u origin main
```

Using the GitHub CLI end to end:

```bash
gh repo create orthogramic/universal-governance-protocol --public --source=. --remote=origin \
  --description "Universal Governance Protocol — open standard for verifiable AI governance" \
  --push
```

## 3. Configure the repository (GitHub UI)

- **Settings → General**: set the description and, if used, a docs site URL.
- **Settings → Security → Code security**: enable **Private vulnerability reporting** (this is what `SECURITY.md` and the issue-template contact link point to).
- **Settings → Discussions**: enable Discussions (the issue-template config routes questions there).
- **Branch protection** on `main`: require a pull request, require the **Docs checks** workflow to pass, and require review from **CODEOWNERS**.
- **Topics** (suggested): `ai-governance`, `open-standard`, `protocol`, `attestation`, `compliance`, `specification`.
- Add a short repo **About** blurb and pin `README.md`.

## 4. Optional: publish a docs site

The Markdown renders fine directly on GitHub. If you later want a browsable site,
GitHub Pages can serve these files without duplicating them — add a Pages workflow
or point Pages at the repository root / `docs/` and choose a theme. No content changes
are needed to start; the internal relative links already resolve.

## 5. Optional: split implementations later

This repo is documentation-first. If you add reference implementations, SDKs, or the
conformance suite, you can either colocate them (uncomment the relevant blocks in
`.gitignore`) or — cleaner at scale — put each in its own repo under the same GitHub org,
with this repo remaining the canonical source of truth for the specification.
