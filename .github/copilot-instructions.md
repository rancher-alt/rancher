# Rancher Workspace Guidelines

## Build And Test
- Prefer Make/script entrypoints over ad-hoc commands because they run through the repository's dapper workflow.
- Common commands:
  - `make ci` for full validation/build/package/test/chart checks.
  - `./scripts/validate` for lint + module consistency checks.
  - `./scripts/build` to build server and agent binaries.
  - `./scripts/test` for unit + integration flows used by CI.
  - `go generate` after API/build config changes (see conventions below).
- Fast local image iteration:
  - `REPO="localhost:5000/my-test-repo/image" TAG="dev" make quick`
  - Optional cross-arch: add `ARCH="amd64"`.
- Prefer targeted checks while iterating:
  - `CGO_ENABLED=0 go test -tags=test ./pkg/<area>/...`

## Architecture
- `main.go` is the CLI entrypoint and bootstraps server startup.
- `pkg/rancher/` wires major subsystems and HTTP/API setup.
- `pkg/controllers/management/` contains most management controller registration and lifecycle logic.
- `pkg/apis/` defines API types; generated clients/controllers live in `pkg/generated/`.
- `cmd/agent/` is the Rancher agent binary entrypoint.
- `chart/` contains the Helm chart and chart tests.
- `package/` contains Docker packaging assets and image build plumbing.

## Conventions That Matter
- Treat `build.yaml` as the single source of truth for build-time versions and config values.
- After changing `build.yaml` or API types, run `go generate` and include generated outputs in the same change.
- Keep module versions aligned between root `go.mod` and `pkg/apis/go.mod`; validation checks will fail if they drift.
- Prefer existing controller patterns in `pkg/controllers/management/`:
  - Register via `Register(...)` entrypoints.
  - Follow established ordering (notably early/late auth registration patterns).
- Minimize manual edits in generated areas (`pkg/generated/`, generated constants) unless explicitly regenerating.

## Testing Expectations
- Default to focused package tests first, then broader test runs.
- Integration/e2e flows are heavier and environment-dependent; do not run long full-suite integration tests unless requested.
- If a command is expensive or requires external services/cluster setup, call that out before running it.

## Documentation Map (Link, Do Not Duplicate)
- Build configuration and adding build values: `docs/build.md`
- Local image build and Helm deployment for dev: `docs/development.md`
- Project overview and release/install pointers: `README.md`
- Contribution process pointer: `CONTRIBUTING.md`
- Helm chart usage and values context: `chart/README.md`
- Integration test setup details: `tests/integration/README.md`

## Change Hygiene
- Keep edits narrowly scoped; avoid opportunistic refactors in unrelated packages.
- When modifying scripts/build logic, preserve existing environment variable names and wiring unless the task requires changes.
- When unsure about project-specific behavior, prefer reading the nearest docs or analogous implementation in the same package before introducing a new pattern.
