# Copilot Instructions (open-cicd-contract)

## Purpose

This repository defines an open specification for a provider-agnostic CI/CD contract.

Your job (as Copilot / AI agent) is to help contributors evolve this project while preserving:
- clarity
- portability
- backward compatibility
- contract-first thinking (convention over implementation)

This repository is a specification, not an execution engine.

---

## Golden Rules

1. **Contract over implementation**
   - Prefer improving `CONTRACT-*.md` and `RFC-*.md` over adding code.
   - If code is added, it must be **reference-only**, minimal, and clearly separated.

2. **Provider-agnostic by default**
   - Never assume GitHub Actions, GitLab CI, Azure DevOps, etc.
   - Any provider content must be framed as an **adapter example**, not “the way”.

3. **Backwards compatibility is a first-class constraint**
   - Avoid breaking changes.
   - If a breaking change is necessary, it must be:
     - explicitly labeled as breaking
     - gated behind a major spec version bump

4. **Keep it simple**
   - No unnecessary complexity.
   - No “framework” behavior.
   - No vendor-specific abstractions.

5. **Docs in English**
   - All documentation must be written in clear, direct English.

6. **Mermaid diagrams are encouraged**
   - Prefer diagrams over long text when explaining concepts, flows, or interfaces.

---

## Repository Structure Expectations

The project should remain centered on these artifacts:

- `README.md`  
  Repository overview and high-level architecture.

- `RFC-*.md`  
  Motivation, principles, scope, and design decisions.

- `CONTRACT-*.md`  
  Normative contract definitions:
  tasks, ordering, inputs, outputs, conventions.

Optional (only if needed):
- `examples/`  
  Minimal reference examples (not production frameworks).

- `providers/`  
  Thin adapters that call contract tasks (no business logic).

---

## How to Propose Changes

### 1) Prefer RFC changes for design decisions
If you introduce a new concept, stage, or rule:
- update the relevant `RFC-*.md`
- justify the change (why it exists, what it solves, what it does NOT solve)

### 2) Prefer CONTRACT changes for normative rules
If you change the contract interface:
- update `CONTRACT-*.md`
- clearly mark:
  - MUST / SHOULD / MAY
  - mandatory vs optional tasks
  - ordering constraints
  - new inputs/outputs

### 3) When adding examples
Examples must:
- be minimal
- demonstrate the contract, not replace it
- avoid provider-specific logic beyond “invoke task X”

---

## Language and Writing Style

- Use short paragraphs.
- Use explicit statements (avoid vague marketing language).
- Define terms before using them.
- Use normative terms consistently:
  - MUST, MUST NOT, SHOULD, SHOULD NOT, MAY

---

## Versioning Rules

- Follow semantic versioning for the specification:
  - MAJOR: breaking changes
  - MINOR: backward-compatible additions
  - PATCH: clarifications and typos

If editing versioned docs:
- update the `Specification Version` field accordingly
- document changes clearly

---

## Safety Rails (Common Mistakes to Avoid)

Do NOT:
- add a “pipeline engine” or orchestration features
- embed provider YAML logic as the main solution
- move business logic into provider adapters
- create provider-specific assumptions in the contract
- introduce mandatory tools (Task, Make, etc.) into the spec
- expand scope into secrets, approvals, or environment promotion

---

## PR Checklist (Required)

When generating or reviewing a change, ensure:

- [ ] The change is provider-agnostic by default
- [ ] Contract changes are in `CONTRACT-*.md`
- [ ] Design rationale is captured in `RFC-*.md` when needed
- [ ] Backward compatibility impact is clearly stated
- [ ] Mermaid diagrams are used where helpful
- [ ] Normative language (MUST/SHOULD/MAY) is correct
- [ ] No business logic is added to provider adapters
- [ ] The directories referenced by `CI_TEMP_PATH`, `CI_DIST_PATH`, and `CI_REPORTS_PATH` remain non-versioned in examples

---

## Contribution Goals

The best contributions typically:
- clarify the contract interface
- reduce ambiguity
- improve portability
- add minimal reference examples
- define validation rules that can be automated later

Keep the project focused: **a CI/CD contract, not a CI/CD product.**
