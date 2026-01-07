# Example: Control Plane implemented with a task runner

This is a **non-normative** example that uses a task runner file to implement the Control Plane.

The key idea is that the CI/CD provider invokes a single entry point (the Control Plane), and the Control Plane exposes the standardized contract task names.

In other words:

- The **Provider** orchestrates and invokes tasks.
- The **Control Plane** implements the tasks and contains the pipeline logic.

Providers typically invoke tasks via a single command entry point (represented by `CI_CONTROLPLANE_CMD`):

```text
${CI_CONTROLPLANE_CMD} cicd:init
```

## Control Plane mapping

Example mapping (illustrative):

- `cicd:init` -> `./cicd/scripts/init.sh`
- `cicd:build` -> `./cicd/scripts/build.sh`
- `cicd:quality` -> `./cicd/scripts/quality.sh`
- `cicd:validate` -> `./cicd/scripts/validate.sh`
- `cicd:publish` -> `./cicd/scripts/publish.sh`
- `cicd:release` -> `./cicd/scripts/release.sh` (optional)

## Example Taskfile (illustrative)

```yaml
version: '3'

tasks:
  cicd:init:
    cmds:
      - ./cicd/scripts/init.sh

  cicd:build:
    cmds:
      - ./cicd/scripts/build.sh

  cicd:quality:
    cmds:
      - ./cicd/scripts/quality.sh

  cicd:validate:
    cmds:
      - ./cicd/scripts/validate.sh

  cicd:publish:
    cmds:
      - ./cicd/scripts/publish.sh

  cicd:release:
    cmds:
      - ./cicd/scripts/release.sh
```

## Notes

- The contract only requires the tasks and their behaviors.
- The tool used by the Control Plane is an implementation detail.
- Providers should call the Control Plane, not re-implement logic in YAML.
