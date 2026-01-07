# Provider adapter examples (Non-Normative)

This directory contains **non-normative** provider-specific examples.

Rules for these examples:

- They are **adapter examples**, not the recommended way.
- Provider configuration MUST remain a thin orchestrator that calls the repository-owned Control Plane.
- Core pipeline implementation logic MUST NOT live in provider YAML.

In open-cicd-contract terms:

- **Provider**: a CI/CD engine (implementation-defined) that orchestrates execution and invokes tasks. It MAY include platform-native concerns (approvals, notifications, credential injection, artifact storage, environment/tooling setup).
- **Control Plane**: the project-owned implementation that contains the pipeline logic and implements `cicd:*` tasks.

In these examples, the provider configuration is intentionally limited to:

- Checkout code
- Set required environment variables
- Call `cicd:*` tasks via the Control Plane entry point (`CI_CONTROLPLANE_CMD`)

## Available provider examples

- [GitHub Actions](github-actions.md)

## Normative reference

The normative contract is defined in:

- [contracts/v1/CONTRACT-open-cicd-contract.md](../../contracts/v1/CONTRACT-open-cicd-contract.md)
