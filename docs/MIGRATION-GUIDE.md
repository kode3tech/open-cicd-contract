# Migration Guide: Adopting open-cicd-contract

## Purpose

This guide helps you migrate existing CI/CD pipelines to the **open-cicd-contract** specification.

You'll learn how to:
- Extract business logic from provider YAML
- Map existing pipeline steps to contract tasks
- Preserve existing functionality while gaining portability

---

## Migration Philosophy

The goal is **NOT** to rewrite your pipeline from scratch.

Instead:
1. **Identify** existing pipeline logic
2. **Map** that logic to contract tasks
3. **Extract** logic into a Control Plane
4. **Simplify** provider configuration to orchestration only

---

## General Migration Strategy

### Step 1: Inventory Your Current Pipeline

Document what your pipeline does:

```
- Install dependencies
- Run linters
- Run unit tests
- Build Docker image
- Push to registry
- Deploy to staging (on main branch)
```

### Step 2: Map to Contract Tasks

Match your steps to contract tasks:

| Your Step | Contract Task |
|-----------|---------------|
| Install dependencies | `cicd:init` |
| Build Docker image | `cicd:build` |
| Run linters + tests | `cicd:quality` |
| Integration tests | `cicd:validate` |
| Push to registry | `cicd:publish` |
| Deploy to staging | `cicd:release` |

### Step 3: Extract Logic to Control Plane

Move implementation details out of provider YAML into your repository (Makefile, Task, scripts, etc.).

### Step 4: Simplify Provider Configuration

Reduce provider YAML to thin orchestration that calls Control Plane tasks.

---

## Provider-Specific Migration Examples

### From GitHub Actions

#### Before (GitHub Actions specific)

`.github/workflows/ci.yml`:

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Test
        run: npm test
      
      - name: Build
        run: npm run build
      
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Push to registry
        if: github.ref == 'refs/heads/main'
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push myapp:${{ github.sha }}
```

**Problems:**
- Business logic in YAML
- Hard to test locally
- Tied to GitHub Actions
- Hard to reuse across repos

#### After (Contract-based)

**Makefile** (Control Plane):

```makefile
.PHONY: cicd:init cicd:build cicd:quality cicd:validate cicd:publish

cicd:init:
	@echo "==> cicd:init"
	@npm ci
	@mkdir -p $(CI_DIST_PATH)

cicd:build:
	@echo "==> cicd:build"
	@npm run build
	@docker build -t myapp:$(CI_VERSION) .

cicd:quality:
	@echo "==> cicd:quality"
	@npm run lint
	@npm test

cicd:validate:
	@echo "==> cicd:validate"
	@docker run --rm myapp:$(CI_VERSION) npm run test:smoke

cicd:publish:
	@echo "==> cicd:publish"
	@docker push myapp:$(CI_VERSION)
```

`.github/workflows/cicd.yml` (Provider adapter):

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
      
      - name: Setup
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Set environment
        run: |
          echo "CI_VERSION=${{ github.sha }}" >> $GITHUB_ENV
          echo "CI_DIST_PATH=./.opencicd/_dist" >> $GITHUB_ENV
      
      - name: cicd:init
        run: ${{ env.CI_CONTROLPLANE_CMD }} cicd:init
      
      - name: cicd:build
        run: ${{ env.CI_CONTROLPLANE_CMD }} cicd:build
      
      - name: cicd:quality
        run: ${{ env.CI_CONTROLPLANE_CMD }} cicd:quality
      
      - name: cicd:validate
        run: ${{ env.CI_CONTROLPLANE_CMD }} cicd:validate
      
      - name: Docker login
        if: github.ref == 'refs/heads/main'
        run: echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
      
      - name: cicd:publish
        if: github.ref == 'refs/heads/main'
        run: ${{ env.CI_CONTROLPLANE_CMD }} cicd:publish
```

**Benefits:**
- Business logic in Makefile (testable locally with `make cicd:build`)
- Provider YAML reduced to orchestration
- Can switch to GitLab CI, Azure DevOps, etc. by changing only the provider adapter
- Logic reusable across repositories

---

### From GitLab CI

#### Before (GitLab CI specific)

`.gitlab-ci.yml`:

```yaml
stages:
  - build
  - test
  - publish

variables:
  IMAGE_TAG: $CI_COMMIT_SHA

build:
  stage: build
  script:
    - npm ci
    - npm run build
    - docker build -t myapp:$IMAGE_TAG .
  artifacts:
    paths:
      - dist/

test:
  stage: test
  script:
    - npm run lint
    - npm run test

publish:
  stage: publish
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker push myapp:$IMAGE_TAG
  only:
    - main
```

#### After (Contract-based)

**Taskfile.yml** (Control Plane using Task):

```yaml
version: '3'

tasks:
  cicd:init:
    desc: Initialize pipeline
    cmds:
      - npm ci
      - mkdir -p {{.CI_DIST_PATH}}

  cicd:build:
    desc: Build artifacts
    cmds:
      - npm run build
      - docker build -t myapp:{{.CI_VERSION}} .

  cicd:quality:
    desc: Quality checks
    cmds:
      - npm run lint
      - npm run test

  cicd:validate:
    desc: Validate build
    cmds:
      - docker run --rm myapp:{{.CI_VERSION}} npm run smoke-test

  cicd:publish:
    desc: Publish artifacts
    cmds:
      - docker push myapp:{{.CI_VERSION}}
```

`.gitlab-ci.yml` (Provider adapter):

```yaml
stages:
  - init
  - build
  - quality
  - validate
  - publish

variables:
  CI_CONTROLPLANE_CMD: task
  CI_VERSION: $CI_COMMIT_SHA
  CI_DIST_PATH: ./.opencicd/_dist

init:
  stage: init
  script:
    - $CI_CONTROLPLANE_CMD cicd:init

build:
  stage: build
  script:
    - $CI_CONTROLPLANE_CMD cicd:build

quality:
  stage: quality
  script:
    - $CI_CONTROLPLANE_CMD cicd:quality

validate:
  stage: validate
  script:
    - $CI_CONTROLPLANE_CMD cicd:validate

publish:
  stage: publish
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - $CI_CONTROLPLANE_CMD cicd:publish
  only:
    - main
```

---

### From Azure DevOps

#### Before (Azure Pipelines specific)

`azure-pipelines.yml`:

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '20.x'
  
  - script: npm ci
    displayName: 'Install dependencies'
  
  - script: npm run lint
    displayName: 'Lint'
  
  - script: npm test
    displayName: 'Test'
  
  - script: npm run build
    displayName: 'Build'
  
  - script: docker build -t myapp:$(Build.BuildId) .
    displayName: 'Build Docker image'
  
  - script: |
      echo $(DOCKER_PASSWORD) | docker login -u $(DOCKER_USERNAME) --password-stdin
      docker push myapp:$(Build.BuildId)
    displayName: 'Push to registry'
    condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
```

#### After (Contract-based)

**Makefile** (Control Plane):

```makefile
.PHONY: cicd:init cicd:build cicd:quality cicd:validate cicd:publish

cicd:init:
	@echo "==> cicd:init"
	@npm ci

cicd:build:
	@echo "==> cicd:build"
	@npm run build
	@docker build -t myapp:$(CI_VERSION) .

cicd:quality:
	@echo "==> cicd:quality"
	@npm run lint
	@npm test

cicd:validate:
	@echo "==> cicd:validate"
	@test -f dist/index.html

cicd:publish:
	@echo "==> cicd:publish"
	@docker push myapp:$(CI_VERSION)
```

`azure-pipelines.yml` (Provider adapter):

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  CI_CONTROLPLANE_CMD: make
  CI_VERSION: $(Build.BuildId)

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '20.x'
  
  - script: $(CI_CONTROLPLANE_CMD) cicd:init
    displayName: 'cicd:init'
  
  - script: $(CI_CONTROLPLANE_CMD) cicd:build
    displayName: 'cicd:build'
  
  - script: $(CI_CONTROLPLANE_CMD) cicd:quality
    displayName: 'cicd:quality'
  
  - script: $(CI_CONTROLPLANE_CMD) cicd:validate
    displayName: 'cicd:validate'
  
  - script: |
      echo $(DOCKER_PASSWORD) | docker login -u $(DOCKER_USERNAME) --password-stdin
      $(CI_CONTROLPLANE_CMD) cicd:publish
    displayName: 'cicd:publish'
    condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
```

---

## Common Migration Patterns

### Pattern 1: Dependency Installation

**Before:** Scattered across multiple jobs/steps

**After:** Consolidate in `cicd:init`

```makefile
cicd:init:
	npm ci
	pip install -r requirements.txt
	go mod download
```

### Pattern 2: Multi-stage Builds

**Before:** Complex provider-specific caching

**After:** Standard build process in `cicd:build`

```makefile
cicd:build:
	docker build --target builder -t myapp:builder .
	docker build --target runtime -t myapp:$(CI_VERSION) .
```

### Pattern 3: Quality Gates

**Before:** Separate jobs with complex conditions

**After:** Single `cicd:quality` task with clear failures

```makefile
cicd:quality:
	npm run lint || exit 1
	npm run test:unit || exit 1
	npm run test:coverage -- --threshold=80 || exit 1
```

### Pattern 4: Conditional Publishing

**Before:** Provider-specific conditions in YAML

**After:** Logic in Control Plane, conditions in provider adapter

Control Plane:
```makefile
cicd:publish:
	@if [ "$(CI_BRANCH)" = "main" ]; then \
		docker push myapp:$(CI_VERSION); \
	else \
		echo "Skipping publish (not main branch)"; \
	fi
```

Or keep condition in provider YAML (recommended):
```yaml
- name: cicd:publish
  run: make cicd:publish
  if: github.ref == 'refs/heads/main'
```

---

## Migration Checklist

- [ ] Document current pipeline behavior
- [ ] Map existing steps to contract tasks
- [ ] Choose Control Plane tool (Make, Task, npm scripts, etc.)
- [ ] Create Control Plane task implementations
- [ ] Test tasks locally
- [ ] Simplify provider YAML to orchestration only
- [ ] Verify CI/CD runs successfully
- [ ] Remove old pipeline configuration
- [ ] Document new structure in README

---

## Incremental Migration Strategy

You don't have to migrate everything at once. Approach it incrementally:

### Phase 1: Extract Build Logic
Start with `cicd:build` only. Keep other steps in provider YAML.

### Phase 2: Add Quality Checks
Migrate linting and testing to `cicd:quality`.

### Phase 3: Complete Core Tasks
Add `cicd:init`, `cicd:validate`, `cicd:publish`.

### Phase 4: Optimize
Refine, add caching, parallelize.

---

## Troubleshooting

### Environment variables not available
- Export them in provider YAML before calling tasks
- Document required variables in Control Plane

### Secrets management
- Keep secrets in provider configuration
- Pass as environment variables to Control Plane tasks

### Different behavior in CI vs local
- Use same environment variables locally
- Ensure consistent tool versions
- Check for provider-specific implicit behavior

---

## Real-World Examples

### Example 1: Monorepo Migration

**Challenge:** Multiple services, shared dependencies

**Solution:**
```makefile
cicd:build:
	cd services/api && make build
	cd services/worker && make build
	cd services/frontend && make build
```

### Example 2: Multi-platform Docker Builds

**Challenge:** Build for multiple architectures

**Solution:**
```makefile
cicd:build:
	docker buildx build --platform linux/amd64,linux/arm64 \
		-t myapp:$(CI_VERSION) .
```

### Example 3: Gradual Rollout

**Challenge:** Deploy to staging, then production with approval

**Solution:**
```makefile
cicd:release:
	@if [ "$(CI_ENVIRONMENT)" = "staging" ]; then \
		kubectl apply -f k8s/staging/; \
	elif [ "$(CI_ENVIRONMENT)" = "production" ]; then \
		kubectl apply -f k8s/production/; \
	fi
```

Provider handles approval gates for production.

---

## Next Steps

After successful migration:

1. **Document your approach** in your repository
2. **Share lessons learned** with the community
3. **Reuse Control Plane** across similar projects
4. **Contribute examples** back to open-cicd-contract

---

## Resources

- [Getting Started Guide](GETTING-STARTED.md)
- [Contract Reference](../contracts/v1/CONTRACT-open-cicd-contract.md)
- [Provider Examples](../examples/providers/)
- [FAQ](FAQ.md)
