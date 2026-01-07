# Governance

This document describes how the **open-cicd-contract** project is maintained.

## Scope

- This project is a **specification**.
- It defines a provider-agnostic CI/CD contract.
- It does not ship a CI/CD engine.

## Roles

- **Maintainers**: review and merge changes, curate releases, and enforce project scope.
- **Contributors**: propose changes via issues and pull requests.

## Decision making

- Changes SHOULD be discussed in issues before implementation.
- Changes that introduce new concepts, pillars, or scope boundaries SHOULD be captured in an RFC.
- Changes to the contract interface MUST be made in the contract document.

## Backward compatibility

- Backward compatibility is a first-class constraint.
- Breaking changes require a major version bump.

## Releases

Releases are tied to the contract version. See `VERSIONING.md`.

## Moderation

Project maintainers enforce the Code of Conduct. See `CODE_OF_CONDUCT.md`.
