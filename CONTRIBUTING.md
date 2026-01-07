# Contributing

Thanks for your interest in contributing to **open-cicd-contract**.

This repository defines an **open specification** (a contract), not an execution engine.

## What to contribute

High-value contributions typically include:

- Clarifying ambiguous contract behavior
- Improving portability (provider/tool neutrality)
- Adding or refining RFC rationale and scope boundaries
- Adding minimal, **non-normative** examples under `examples/`
- Fixing typos and improving diagrams

## What not to contribute

To keep the project focused, please avoid:

- Adding a pipeline engine or orchestration framework
- Introducing provider-specific assumptions into the contract
- Putting business logic into provider adapters

## How changes are proposed

This project uses an RFC-driven workflow:

- **RFCs** live under `rfcs/` and capture motivation, trade-offs, and scope.
- The **normative contract** lives under `contracts/` and defines compliance requirements.

If your change modifies the contract interface (tasks, ordering, inputs/outputs, conventions), it MUST be proposed as a change to the contract document.
If your change introduces a new concept or rule, it SHOULD be justified in an RFC.

For a detailed step-by-step workflow (RFC → contract → examples → release), see:

- `docs/README.md`

## Pull request guidelines

- Keep changes small and reviewable
- Use clear normative language (MUST/SHOULD/MAY) only in the contract
- Prefer Mermaid diagrams over long prose when it improves clarity
- Maintain backward compatibility within a major version

## Development setup

This repository is documentation-first. No build is required.

## License

By contributing, you agree that your contributions will be licensed under the MIT License, as described in `LICENSE`.
