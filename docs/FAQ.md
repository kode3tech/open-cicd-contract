# Frequently Asked Questions (FAQ)

## General Questions

### What is open-cicd-contract?

**open-cicd-contract** is an open specification that defines a standard, provider-agnostic CI/CD contract. It establishes a set of standardized tasks (`cicd:*`) that decouple pipeline logic from CI/CD platforms, making pipelines portable and reusable.

### Is this a CI/CD tool or platform?

No. This is a **specification**, not a tool or platform. It defines **what** a compliant pipeline should expose (tasks, inputs, outputs), not **how** to implement or execute it.

### Why do I need this?

Modern CI/CD pipelines are tightly coupled to specific providers (GitHub Actions, GitLab CI, etc.). This creates:
- Vendor lock-in
- Difficulty migrating between platforms
- Business logic scattered in provider YAML
- Hard-to-test pipelines

The open-cicd-contract solves this by establishing a contract that's portable across providers.

### How is this different from existing CI/CD platforms?

This specification **complements** existing platforms, it doesn't replace them. Your CI/CD provider still handles orchestration, scheduling, credentials, and artifacts. The contract just standardizes **what tasks exist** and **how they're invoked**.

---

## Adoption Questions

### Do I have to rewrite my entire pipeline?

No. You can adopt incrementally:
1. Start with one task (e.g., `cicd:build`)
2. Gradually migrate other tasks
3. Keep existing provider-specific logic until ready to migrate

See the [Migration Guide](MIGRATION-GUIDE.md) for step-by-step instructions.

### What if my pipeline doesn't fit this model?

The contract is intentionally flexible. Most pipelines can be mapped to the standard tasks:
- **Build** anything: code, containers, infrastructure
- **Quality** checks: linting, testing, security scans
- **Validate** readiness: integration tests, smoke tests
- **Publish** artifacts: packages, images, binaries

If you have unique requirements, you can:
- Extend tasks internally (keep them provider-agnostic)
- Propose an RFC to evolve the specification

### Can I use this with [my CI/CD provider]?

Yes. The specification is provider-agnostic by design. You can use it with:
- GitHub Actions
- GitLab CI
- Azure DevOps
- Jenkins
- CircleCI
- Bitbucket Pipelines
- Any other CI/CD platform

See [examples/providers/](../examples/providers/) for reference implementations.

### What happens if I switch CI/CD providers?

That's the point! You only need to update the **provider adapter** (the thin orchestration layer). Your **Control Plane** (the actual pipeline implementation) remains unchanged.

---

## Implementation Questions

### What is a Control Plane?

The **Control Plane** is the execution layer that implements the contract tasks. It's the actual code/scripts/tooling that runs when you invoke `cicd:build`, `cicd:quality`, etc.

Common Control Plane implementations:
- Make (Makefile)
- Task (Taskfile.yml)
- npm/yarn scripts (package.json)
- Shell scripts
- Language-specific build tools (Gradle, Maven, etc.)

### Which Control Plane tool should I use?

Choose based on your project:
- **Make**: ubiquitous, simple, works everywhere
- **Task**: modern, cross-platform, easier syntax than Make
- **npm scripts**: if you already use Node.js
- **Shell scripts**: maximum flexibility, but requires more maintenance
- **Language-specific tools**: if your ecosystem already has conventions (Gradle for Java, Poetry for Python, etc.)

There's no "right" answer—use what fits your team and project best.

### Can I have multiple Control Plane implementations?

Yes, but it's not recommended. Pick one approach per repository to avoid confusion and maintenance overhead.

However, you might use different Control Planes across different repositories based on language/ecosystem.

### Do I need to use Make?

No. Make is just one option. Use any task runner or build tool that can:
1. Execute named tasks (`cicd:init`, `cicd:build`, etc.)
2. Return proper exit codes
3. Be invoked consistently

### How do I test my Control Plane locally?

Set required environment variables and run tasks directly:

```bash
export CI_VERSION="1.0.0-local"
export CI_TEMP_PATH="./.opencicd/_temp"
export CI_DIST_PATH="./.opencicd/_dist"
export CI_REPORTS_PATH="./.opencicd/_reports"

make cicd:init
make cicd:build
make cicd:quality
```

This is one of the key benefits: you can test your pipeline logic **before** pushing to CI.

### What environment variables are required?

The core contract defines:
- `CI_CONTROLPLANE_CMD`: command to invoke Control Plane (e.g., `make`, `task`)
- `CI_VERSION`: version/build identifier
- `CI_TEMP_PATH`: temporary working directory
- `CI_DIST_PATH`: distributable artifacts directory
- `CI_REPORTS_PATH`: reports/logs directory

See [contracts/v1/CONTRACT-open-cicd-contract.md](../contracts/v1/CONTRACT-open-cicd-contract.md) for the complete reference.

### How do I handle secrets?

Secrets remain **provider-managed**. The CI/CD provider injects secrets as environment variables before invoking Control Plane tasks.

Example (GitHub Actions):
```yaml
- name: cicd:publish
  run: make cicd:publish
  env:
    DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
    DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
```

---

## Task-Specific Questions

### What belongs in cicd:init?

Initialization tasks:
- Environment verification
- Dependency installation
- Directory setup
- Metadata display (version, commit, etc.)

### What's the difference between cicd:quality and cicd:validate?

- **cicd:quality**: checks code quality (linting, unit tests, coverage, security scans)
- **cicd:validate**: validates the build output is correct (integration tests, smoke tests, artifact verification)

Think of `quality` as "is the code good?" and `validate` as "does the build work?"

### Is cicd:release mandatory?

No. `cicd:release` is **optional**. Use it if you need deployment/release capabilities.

If you only publish artifacts (not deploy), you can skip `cicd:release`.

### Can I skip tasks?

Providers MAY skip **optional** tasks (`cicd:release`), but MUST implement all **mandatory** tasks:
- `cicd:init`
- `cicd:build`
- `cicd:quality`
- `cicd:validate`
- `cicd:publish`

### Can tasks run in parallel?

The specification defines a **logical order**, but doesn't mandate serial execution. If your Control Plane can safely parallelize subtasks (e.g., linting and testing within `cicd:quality`), that's fine.

However, the **top-level task order** should be preserved:
```
init → build → quality → validate → publish → release
```

---

## Compliance Questions

### How do I know if I'm compliant?

A project is compliant if:
- All mandatory tasks are implemented
- Tasks respect the defined execution order
- Exit codes follow conventions (0 = success, non-zero = failure)
- Standard environment variables are supported

### Is there a validator?

Not yet, but it's on the roadmap. We welcome contributions!

A validation CLI could check:
- Presence of required tasks
- Proper exit codes
- Environment variable handling

### Can I add custom tasks?

Yes, but keep them **internal to your Control Plane**. The contract only defines `cicd:*` tasks. You can have additional tasks (e.g., `cicd:build:docker`, `cicd:quality:security`) as long as they don't conflict with the standard tasks.

Custom tasks should not be invoked by provider adapters—keep providers focused on standard tasks.

### What if I need to extend the specification?

If you have a use case not covered by the current contract:
1. Check if it can be implemented within existing tasks
2. If not, propose an RFC following the [RFC Process](RFC-PROCESS.md)
3. Engage with the community for feedback

---

## Compatibility Questions

### Does this work with monorepos?

Yes. You can:
- Implement contract tasks at the root that orchestrate across services
- Have per-service Control Planes with a root-level dispatcher

Example:
```makefile
cicd:build:
	cd services/api && make cicd:build
	cd services/worker && make cicd:build
```

### Can I use Docker?

Absolutely. Docker is a common use case:

```makefile
cicd:build:
	docker build -t myapp:$(CI_VERSION) .

cicd:publish:
	docker push myapp:$(CI_VERSION)
```

### Does this work with microservices?

Yes. Each service can have its own Control Plane, or you can have a centralized Control Plane that orchestrates across services.

### What about multi-language projects?

Use a polyglot Control Plane. Make and Task work well for this:

```makefile
cicd:build:
	cd backend && ./gradlew build
	cd frontend && npm run build
	cd scripts && python -m build
```

---

## Governance Questions

### Who maintains this specification?

The open-cicd-contract project is community-driven and maintained by its contributors. See [GOVERNANCE.md](../GOVERNANCE.md) for details.

### How do I contribute?

See [CONTRIBUTING.md](../CONTRIBUTING.md) for contribution guidelines.

You can:
- Report issues
- Propose RFCs for spec changes
- Submit examples
- Improve documentation

### How is the specification versioned?

The specification follows semantic versioning:
- **MAJOR**: breaking changes
- **MINOR**: backward-compatible additions
- **PATCH**: clarifications, typos, non-normative changes

See [VERSIONING.md](../VERSIONING.md) for details.

### Can this specification change?

Yes, but changes must:
- Go through the RFC process
- Preserve backward compatibility when possible
- Be clearly documented in the changelog

Breaking changes require a major version bump.

---

## Comparison Questions

### How does this compare to GitHub Actions reusable workflows?

GitHub Actions reusable workflows are **provider-specific**. They work only on GitHub.

open-cicd-contract is **provider-agnostic**. Your Control Plane works with any CI/CD platform.

### What about Jenkins Shared Libraries?

Similar concept, but tied to Jenkins. open-cicd-contract works with Jenkins **and** all other providers.

### How is this different from Dagger, Earthly, or Bazel?

Tools like Dagger and Earthly are **execution engines**—they run your pipeline.

open-cicd-contract is a **specification**—it defines what tasks exist, not how to execute them.

You can use Dagger/Earthly/Bazel **as your Control Plane** while still following the open-cicd-contract specification.

### Is this like Tekton?

Tekton is a **Kubernetes-native CI/CD platform**.

open-cicd-contract is a **specification** that works with any platform, including Tekton.

---

## Troubleshooting Questions

### Tasks work locally but fail in CI

Common causes:
- Missing environment variables
- Different tool versions
- Absolute paths instead of relative paths
- Provider-specific implicit behavior

**Solution:** Ensure consistent environment setup and use portable paths.

### Exit codes aren't being respected

Ensure your Control Plane tool respects exit codes:
- In Make: use `.PHONY` targets and `set -e` in shell scripts
- In shell scripts: always use `exit 1` on errors
- Avoid swallowing errors with `|| true`

### Can't install Control Plane tool in CI

Most CI/CD platforms support installing tools. Examples:
- **Make**: usually pre-installed
- **Task**: `sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d`
- **npm**: `npm install -g npm`

See provider documentation for tool installation.

---

## Advanced Questions

### Can I use this for infrastructure as code (IaC)?

Yes! You can build/validate/publish infrastructure:

```makefile
cicd:build:
	terraform plan -out=tfplan

cicd:validate:
	terraform validate

cicd:publish:
	terraform apply tfplan
```

### How do I handle approvals?

Approvals are **provider-specific** and remain in provider configuration.

Example (GitHub Actions with environments):
```yaml
deploy:
  environment: production
  runs-on: ubuntu-latest
  steps:
    - name: cicd:release
      run: make cicd:release
```

### Can I use this with GitOps?

Yes. Use `cicd:publish` to push manifests/charts, and let GitOps tools (ArgoCD, Flux) handle deployment:

```makefile
cicd:publish:
	helm package charts/myapp
	helm push myapp-$(CI_VERSION).tgz oci://registry.example.com/charts
```

### What about matrix builds?

Matrix builds are **provider-specific orchestration**. The provider handles spawning multiple jobs with different parameters, and each job invokes the same Control Plane tasks.

---

## Getting Help

### Where can I ask questions?

- Open an issue on [GitHub](https://github.com/kode3tech/open-cicd-contract/issues)
- Start a discussion on [GitHub Discussions](https://github.com/kode3tech/open-cicd-contract/discussions)
- Check the [examples/](../examples/) directory

### How do I report a bug in the specification?

Open an issue with:
- What's unclear or incorrect
- Expected behavior
- Suggested fix

### Can I request a new feature?

Yes! Propose an RFC following the [RFC Process](RFC-PROCESS.md).

---

## Resources

- [Getting Started Guide](GETTING-STARTED.md)
- [Migration Guide](MIGRATION-GUIDE.md)
- [Contract Reference](../contracts/v1/CONTRACT-open-cicd-contract.md)
- [RFC-0001: Motivation and Design](../rfcs/RFC-0001-open-cicd-contract.md)
- [Examples](../examples/)
- [Contributing Guidelines](../CONTRIBUTING.md)
