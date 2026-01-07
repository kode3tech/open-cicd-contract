# RFC process

RFCs are used to capture **design decisions** and preserve project intent over time.

This repository is contract-first:

- **RFCs** explain motivation, trade-offs, and scope.
- The **Contract** (`contracts/`) defines compliance and is the normative interface.

---

## When you SHOULD write an RFC

Write an RFC when a change:

- Introduces a new concept or pillar
- Adds a new stage/task to the lifecycle
- Changes scope boundaries (what is in/out)
- Adds new compliance rules that affect portability
- Has meaningful trade-offs that future contributors must understand

Pure editorial changes usually do not require an RFC.

---

## RFC lifecycle

Recommended status progression:

- **Draft**: proposal under discussion; not yet a project decision.
- **Accepted**: maintainers agree with the direction; contract work can proceed.
- **Rejected**: maintainers decided not to pursue.
- **Superseded**: replaced by a newer RFC.

---

## RFC naming and location

- Location: `rfcs/`
- Filename: `RFC-XXXX-short-title.md` (example: `RFC-0002-contract-extension-points.md`)

Choose a short title that describes the "what".

---

## RFC template (recommended sections)

### 1. Title
A concise title and RFC number.

### 2. Status
One of: Draft / Accepted / Rejected / Superseded.

### 3. Summary
One-paragraph description of what the RFC proposes.

### 4. Motivation
The problem being solved. Include:

- What hurts today
- Who is impacted
- Why existing patterns are insufficient

### 5. Goals
Explicit list of goals.

### 6. Non-goals
Explicit list of what this RFC will not solve.

### 7. Proposal
Describe the proposed change. Focus on:

- Concepts and boundaries
- The new interface surface (at a high level)
- How it preserves provider/tool neutrality

### 8. Compatibility
State whether this is expected to be PATCH/MINOR/MAJOR and why.

### 9. Alternatives
List the options considered, including "do nothing".

### 10. Open questions
What needs feedback to decide.

### 11. Appendix (optional)
Mermaid diagrams, illustrative examples, prior art.

---

## What RFCs MUST NOT do

- RFCs MUST NOT define provider-specific YAML as the primary solution.
- RFCs MUST NOT introduce a pipeline engine.
- RFCs MUST NOT encode business logic into provider adapters.

---

## How RFCs relate to the contract

After an RFC is **Accepted**:

1. Translate the proposal into normative changes in `contracts/`.
2. Add/adjust non-normative examples under `examples/`.
3. Update `CHANGELOG.md` under `Unreleased`.

An RFC being accepted does not automatically mean the contract has been updated.
