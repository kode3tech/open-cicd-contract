# Getting Started with open-cicd-contract

## Overview

This guide helps you adopt the **open-cicd-contract** specification in your project, step by step.

By the end, your pipeline will:
- Be provider-agnostic (portable between GitHub Actions, GitLab CI, Azure DevOps, etc.)
- Follow a standardized contract (`cicd:*` tasks)
- Keep business logic in your repository, not in provider YAML
- Be testable locally before running in CI

---

## Prerequisites

Before starting:
- Familiarity with CI/CD concepts (build, test, publish)
- A project repository with existing code
- Basic knowledge of task runners (Make, npm scripts, Task, etc.) or shell scripting

---

## Step 1: Understand the Contract

Read the core concepts:

1. **[README.md](../README.md)**: high-level overview
2. **[contracts/v1/CONTRACT-open-cicd-contract.md](../contracts/v1/CONTRACT-open-cicd-contract.md)**: normative task definitions
3. **[rfcs/RFC-0001-open-cicd-contract.md](../rfcs/RFC-0001-open-cicd-contract.md)**: design rationale

Key takeaway: the specification defines **what tasks exist** (`cicd:init`, `cicd:build`, etc.), not **how you implement them**.

---

## Step 2: Choose Your Control Plane

The **Control Plane** is the tool that executes your contract tasks.

Common options:
- **Make** (ubiquitous, simple, POSIX-friendly)
- **Task** (modern, cross-platform, YAML-based)
- **npm scripts** (Node.js projects)
- **Gradle/Maven tasks** (Java projects)
- **Shell scripts** (maximum flexibility)

Choose based on your project's existing tooling. If unsure, start with **Make** or **Task**.

See [examples/control-plane/](../examples/control-plane/) for reference implementations.

---

## Step 3: Implement Mandatory Tasks

Create implementations for the 5 mandatory tasks:

### cicd:init
Initializes the pipeline environment.

**Example responsibilities:**
- Verify required environment variables
- Set up temporary directories
- Display pipeline metadata (version, commit, timestamp)

### cicd:build
Builds your project artifacts.

**Example responsibilities:**
- Compile code
- Bundle assets
- Generate distribution files

### cicd:quality
Runs quality checks.

**Example responsibilities:**
- Run linters
- Run unit tests
- Check code coverage
- Run security scanners

### cicd:validate
Validates pipeline readiness.

**Example responsibilities:**
- Run integration tests
- Verify artifact integrity
- Check for required files

### cicd:publish
Publishes artifacts to storage.

**Example responsibilities:**
- Upload to artifact repository
- Push container images
- Publish packages (npm, PyPI, Maven, etc.)

---

## Step 4: Example Makefile Control Plane

Here's a minimal Makefile implementing the contract:

```makefile
.PHONY: cicd:init cicd:build cicd:quality cicd:validate cicd:publish

cicd:init:
	@echo "==> cicd:init"
	@echo "Project: my-app"
	@echo "Version: $(CI_VERSION)"
	@mkdir -p $(CI_TEMP_PATH) $(CI_DIST_PATH) $(CI_REPORTS_PATH)

cicd:build:
	@echo "==> cicd:build"
	@npm ci
	@npm run build
	@cp -r dist/* $(CI_DIST_PATH)/

cicd:quality:
	@echo "==> cicd:quality"
	@npm run lint
	@npm run test:unit
	@npm run test:coverage

cicd:validate:
	@echo "==> cicd:validate"
	@npm run test:integration
	@test -f $(CI_DIST_PATH)/index.html

cicd:publish:
	@echo "==> cicd:publish"
	@echo "Publishing artifacts to $(CI_ARTIFACT_REGISTRY)"
	@# Add your publish logic here
```

Save this as `Makefile` in your repository root.

---

## Step 5: Test Locally

Run tasks locally to verify they work:

```bash
export CI_VERSION="1.0.0-local"
export CI_TEMP_PATH="./.opencicd/_temp"
export CI_DIST_PATH="./.opencicd/_dist"
export CI_REPORTS_PATH="./.opencicd/_reports"
export CI_ARTIFACT_REGISTRY="local"

make cicd:init
make cicd:build
make cicd:quality
make cicd:validate
make cicd:publish
```

All tasks should complete without errors.

---

## Step 6: Create a Provider Adapter

Choose your CI/CD provider and create a thin orchestration layer.

### GitHub Actions Example

Create `.github/workflows/cicd.yml`:

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:

env:
  CI_CONTROLPLANE_CMD: make

jobs:
  pipeline:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Set environment
        run: |
          echo "CI_VERSION=${{ github.sha }}" >> $GITHUB_ENV
          echo "CI_TEMP_PATH=./.opencicd/_temp" >> $GITHUB_ENV
          echo "CI_DIST_PATH=./.opencicd/_dist" >> $GITHUB_ENV
          echo "CI_REPORTS_PATH=./.opencicd/_reports" >> $GITHUB_ENV
      
      - name: cicd:init
        run: ${{ env.CI_CONTROLPLANE_CMD }} cicd:init
      
      - name: cicd:build
        run: ${{ env.CI_CONTROLPLANE_CMD }} cicd:build
      
      - name: cicd:quality
        run: ${{ env.CI_CONTROLPLANE_CMD }} cicd:quality
      
      - name: cicd:validate
        run: ${{ env.CI_CONTROLPLANE_CMD }} cicd:validate
      
      - name: cicd:publish
        run: ${{ env.CI_CONTROLPLANE_CMD }} cicd:publish
        if: github.ref == 'refs/heads/main'
```

See [examples/providers/](../examples/providers/) for more provider examples.

---

## Step 7: Add Optional Tasks (if needed)

If you need deployment/release capabilities, implement `cicd:release`:

```makefile
cicd:release:
	@echo "==> cicd:release"
	@echo "Deploying to production..."
	@# Add deployment logic here
```

---

## Step 8: Document Your Implementation

Create a `CICD.md` or add to your `README.md`:

```markdown
## CI/CD Pipeline

This project follows the [open-cicd-contract](https://github.com/kode3tech/open-cicd-contract) specification.

### Local Execution

```bash
make cicd:init
make cicd:build
make cicd:quality
make cicd:validate
```

### Control Plane

We use **Make** as our Control Plane. See `Makefile` for task implementations.

### Provider

We use GitHub Actions. See `.github/workflows/cicd.yml`.
```

---

## Common Use Cases

### Case 1: Simple Node.js Application

**Tasks:**
- `cicd:init`: verify Node version, install dependencies
- `cicd:build`: run `npm run build`
- `cicd:quality`: run ESLint + Jest
- `cicd:validate`: smoke test build output
- `cicd:publish`: publish to npm registry

### Case 2: Docker-based Service

**Tasks:**
- `cicd:init`: verify Docker daemon
- `cicd:build`: `docker build -t myapp:${CI_VERSION} .`
- `cicd:quality`: run Trivy security scan
- `cicd:validate`: run integration tests in container
- `cicd:publish`: `docker push` to registry

### Case 3: Python Package

**Tasks:**
- `cicd:init`: setup virtual environment
- `cicd:build`: `python -m build`
- `cicd:quality`: run pytest + mypy + black
- `cicd:validate`: install wheel and run smoke tests
- `cicd:publish`: upload to PyPI

---

## Next Steps

Once your pipeline is working:

1. **Optimize**: add caching, parallelize quality checks
2. **Extend**: add `cicd:release` for deployment
3. **Migrate**: use [MIGRATION-GUIDE.md](MIGRATION-GUIDE.md) to convert existing pipelines
4. **Share**: contribute your patterns back to the community

---

## Troubleshooting

### Tasks fail locally but work in CI
- Check environment variable differences
- Verify file paths are relative, not absolute
- Ensure dependencies are installed the same way

### Exit codes not respected
- Always use `set -e` in shell scripts
- Return non-zero exit codes on failures
- Check Make `.PHONY` declarations

### Provider-specific issues
- Keep provider YAML minimal (orchestration only)
- Move logic to Control Plane tasks
- Test Control Plane tasks independently

---

## Resources

- [Contract Reference](../contracts/v1/CONTRACT-open-cicd-contract.md)
- [Provider Examples](../examples/providers/)
- [Control Plane Examples](../examples/control-plane/)
- [Migration Guide](MIGRATION-GUIDE.md)
- [FAQ](FAQ.md)
