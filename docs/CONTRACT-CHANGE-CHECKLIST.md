# Contract change checklist

Use this checklist when modifying the normative contract.

The contract is the interface that providers/orchestrators and control planes depend on.

Canonical contract for v1:

- `contracts/v1/CONTRACT-open-cicd-contract.md`

---

## 1) Scope and intent

- Does this change keep the project provider/tool agnostic?
- Does it avoid defining implementation details ("what", not "how")?
- Does it avoid scope creep into governance/policy enforcement (provider-native concerns)?

If not, the change likely belongs in an RFC clarification or should be rejected.

---

## 2) Compatibility classification

Decide the SemVer impact:

- PATCH: editorial/clarification only
- MINOR: additive and backward-compatible
- MAJOR: breaking

Common breaking patterns:

- Renaming tasks
- Reordering tasks/stages
- Tightening MAY/SHOULD to MUST
- Changing meaning of inputs/outputs

---

## 3) Normative language

- Are you using MUST / MUST NOT / SHOULD / SHOULD NOT / MAY correctly?
- Is each MUST testable/observable at the interface boundary?

If a requirement cannot be validated (even conceptually), it is often too implementation-specific.

---

## 4) Task definition quality

If adding/modifying a task:

- Task name uses the `cicd:*` namespace
- Mandatory vs optional is explicit
- Purpose is clear and provider-neutral
- Expected behavior is described without tool assumptions
- Exit codes are specified (success/failure conventions)

---

## 5) Ordering constraints

- Does the change specify where the task fits in the lifecycle?
- Are ordering rules minimal but sufficient for interoperability?

---

## 6) Inputs and outputs

- Are inputs defined as environment variables or conventions (not provider features)?
- Are outputs described in terms of `CI_*_PATH` conventions and file artifacts?
- Are defaults and optional inputs clearly marked?

---

## 7) Examples (non-normative)

- Do examples demonstrate the contract without becoming requirements?
- Are examples clearly labeled non-normative?
- Do provider examples remain thin (orchestration only)?

---

## 8) Documentation updates

- `CHANGELOG.md` updated under `Unreleased`
- RFC updated/added if the change introduces a new concept
- Links are correct (README, docs index)

---

## 9) Reviewer notes (recommended)

Include in the PR description:

- Compatibility impact (PATCH/MINOR/MAJOR)
- What changed in the contract surface
- Why it is provider/tool agnostic
- Any open questions for maintainers
