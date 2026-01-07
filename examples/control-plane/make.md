# Example: Control Plane implemented with Make

This is a **non-normative** example that uses `make` targets to implement the Control Plane.

In open-cicd-contract terms, this Makefile is the **Control Plane** (implementation). A CI/CD **Provider** orchestrates execution and invokes `cicd:*` tasks via the Control Plane entry point.

Then invoke tasks:

```sh
make cicd:init
make cicd:build
```

## Example Makefile targets (illustrative)

```make
.PHONY: cicd:init cicd:build cicd:quality cicd:validate cicd:publish cicd:release

cicd:init:
	./cicd/scripts/init.sh

cicd:build:
	./cicd/scripts/build.sh

cicd:quality:
	./cicd/scripts/quality.sh

cicd:validate:
	./cicd/scripts/validate.sh

cicd:publish:
	./cicd/scripts/publish.sh

cicd:release:
	./cicd/scripts/release.sh
```

## Notes

- The Makefile is the Control Plane entry point in this example.
- The scripts are illustrative; they represent the project-owned pipeline logic.
- A provider adapter can invoke tasks consistently (for example: `make cicd:build`).
