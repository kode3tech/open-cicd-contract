# Release process

This repository is documentation-first.

Releases are tied to the contract version, and follow semantic versioning.

See:

- ../VERSIONING.md
- ../CHANGELOG.md

---

## Goals

A release should:

- Provide a stable, immutable reference for consumers (via tags)
- Communicate what changed (changelog + release notes)
- Preserve compatibility expectations (SemVer)

---

## Step-by-step

### 1) Decide the next version

- PATCH: clarifications/typos
- MINOR: backward-compatible additions
- MAJOR: breaking changes (new contract major path required)

### 2) Prepare the documentation

- Update the normative contract version field (if present).
- Ensure any RFC status changes are reflected (Draft → Accepted, etc.).
- Update `CHANGELOG.md`:
  - Ensure `Unreleased` contains the current changes.
  - Create or update the target version section.
  - Add a date.

### 3) Ensure the repository is in a releasable state

- README links are correct
- Examples are consistent with the contract
- No provider/tool-specific assumptions were accidentally introduced

### 4) Create a release commit

Use Conventional Commits.

Recommended patterns:

- `docs(changelog): prepare v1.1.0`
- `chore(release): v1.1.0`

Keep the release commit focused on versioning + changelog.

### 5) Tag the release

Tags should be:

- `vX.Y.Z` (example: `v1.1.0`)
- Annotated tags are recommended.

### 6) Publish the GitHub Release

- Title: `vX.Y.Z`
- Notes: short summary + highlights
- Link to the contract path and any key RFCs

---

## Immutability notes

Tags are immutable in practice only if the repository prevents tag rewriting.

Recommended repository settings:

- Protect `main` (no force pushes, require PRs)
- Protect tags matching `v*` (prevent deletion/force-update)
- Optionally sign tags (GPG/Sigstore) for authenticity
