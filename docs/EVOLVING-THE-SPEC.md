# Evolving the open-cicd-contract specification

This project is a **specification**, not a CI/CD engine.

The goal of this guide is to provide a detailed, repeatable workflow for proposing and shipping changes while preserving:

- Clarity
- Portability (provider/tool neutrality)
- Backward compatibility
- Contract-first thinking ("what", not "how")

If you are new, start with:

- ../README.md (project overview)
- ../contracts/v1/CONTRACT-open-cicd-contract.md (normative contract)
- ../rfcs/RFC-0001-open-cicd-contract.md (architecture & scope)

---

## 1) Decide what kind of change you are making

### A. Editorial clarification (PATCH)

Use this when you are improving wording, adding diagrams, fixing typos, or clarifying ambiguous text **without changing**:

- Task names
- Required/optional behavior
- Inputs/outputs conventions

Examples:

- Reword a paragraph in the contract to remove ambiguity.
- Add a Mermaid diagram explaining ordering.

### B. Backward-compatible feature (MINOR)

Use this when you are **adding** something in a way that does not break existing compliant implementations.

Typical patterns:

- Add a new **optional** task.
- Add a new **optional** input/output.
- Add a new rule written as **SHOULD** or **MAY** (not MUST), unless it is a purely editorial MUST (rare).

### C. Breaking change (MAJOR)

Use this when existing implementations may become non-compliant without changes.

Common breaking triggers:

- Renaming a task (e.g., `cicd:build` → `cicd:compile`)
- Changing ordering constraints
- Tightening a rule from MAY/SHOULD to MUST
- Changing the meaning of an input/output

If you believe a breaking change is required:

- It MUST be explicitly labeled as breaking.
- It MUST land under a new major version contract path (example: `contracts/v2/`).

See ../VERSIONING.md.

---

## 2) Start with an issue (recommended)

Open an issue to align on:

- The problem being solved (not the solution)
- Alternatives considered
- Compatibility impact (patch/minor/major)
- Any open questions for maintainers

This reduces wasted work and helps converge early.

---

## 3) Use an RFC for design decisions

If you are introducing a new concept, new pillar, new rule, or meaningful scope boundary, write an RFC.

- RFCs live under `rfcs/`.
- The RFC explains *why* and *what*, not *how to implement*.

See [RFC process](RFC-PROCESS.md).

---

## 4) Update the normative contract (the interface)

The contract is the stable interface that providers and control planes depend on.

Rules:

- Contract changes MUST go to `contracts/`.
- Use normative language consistently: MUST / MUST NOT / SHOULD / SHOULD NOT / MAY.
- Prefer the simplest rule that achieves interoperability.
- Avoid provider-specific assumptions (GitHub Actions, GitLab CI, Azure DevOps, etc.).

When adding a new feature, make it as small and composable as possible:

- Define its name (e.g., task name `cicd:scan`)
- Define whether it is mandatory or optional
- Define ordering constraints (where it fits in the lifecycle)
- Define its inputs/outputs at the contract boundary
- Define exit code expectations

Use the [contract checklist](CONTRACT-CHANGE-CHECKLIST.md).

---

## 5) Update examples (non-normative)

Examples exist to help adoption.

Rules:

- Examples MUST be non-normative.
- Examples SHOULD demonstrate how a provider calls the control plane via `CI_CONTROLPLANE_CMD`.
- Examples MUST NOT add business logic into provider adapters.

If a feature adds a new task, add a minimal example showing:

- How a provider might invoke the task
- How a control plane might expose it

---

## 6) Update the changelog

While a change is in progress:

- Add an entry under `## Unreleased` in ../CHANGELOG.md.

When cutting a release:

- Move the entry to the target version section.
- Add a date.

---

## 7) Pull request expectations

A good PR for spec changes usually contains:

- RFC change (if needed)
- Contract change (if the interface changes)
- Example change (if it improves adoption)
- Changelog entry (for notable changes)

PRs SHOULD include:

- A clear problem statement
- Compatibility notes
- "Out of scope" clarifications if the change risks scope creep

See ../CONTRIBUTING.md.

---

## 8) Release

Releases are tied to the contract version.

A typical release includes:

- Contract version field updated
- Changelog updated
- Tag created (e.g., `v1.1.0`)
- GitHub Release created

See [Release process](RELEASE-PROCESS.md).
