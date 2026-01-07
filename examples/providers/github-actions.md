# Example: GitHub Actions adapter (Non-Normative)

This is a **non-normative** provider adapter example for **GitHub Actions**.

It demonstrates how a CI/CD provider can act as an orchestrator that:

- Invokes a repository-owned Control Plane entry point
- Calls contract tasks (`cicd:*`) by name

This file is intentionally minimal and contains no business logic.

## Example workflow

```yaml
name: open-cicd-contract

on:
  push:
  pull_request:

env:
  CI: "true"
  CI_PROVIDER: "github-actions"
  PROFILE: "ci"

  # Control Plane entry point (repository-owned)
  CI_CONTROLPLANE_CMD: "task"

jobs:
  INIT:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: OPENCICD INIT TASK
        run: $CI_CONTROLPLANE_CMD cicd:init

  BUILD:
    runs-on: ubuntu-latest
    needs: [INIT]
    steps:
      - uses: actions/checkout@v4

      - name: OPENCICD BUILD TASK
        run: $CI_CONTROLPLANE_CMD cicd:build
```

## Notes

- Replace `task` with your repository's Control Plane entry point (or set `CI_CONTROLPLANE_CMD` accordingly).
- This example is intentionally minimal and shows orchestration only.
- Real pipelines will typically include the full required order and additional stages.
- Providers may include platform-native concerns (approvals, notifications, credential injection, artifact storage, environment/tooling setup), but the `cicd:*` task implementation belongs in the Control Plane.
- The adapter YAML should remain thin: call tasks and nothing else.
