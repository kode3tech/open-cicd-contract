# Versioning and Releases

This repository is a specification.

## Semantic versioning

The contract follows semantic versioning:

- **MAJOR**: breaking changes to the contract interface
- **MINOR**: backward-compatible additions
- **PATCH**: clarifications, editorial fixes, and typos

## What is versioned

- The normative contract under `contracts/` is the source of truth for versioning.
- RFCs may evolve over time; they are rationale documents and are not versioned as an API.
- Examples are non-normative and may change without affecting contract compliance.

## Release process (lightweight)

A typical release includes:

1. Update the contract document version field
2. Update `CHANGELOG.md`
3. Tag a release (e.g., `v1.1.0`)

## Compatibility policy

- Compatibility is guaranteed within a major version.
- Consumers SHOULD pin to a major version.
