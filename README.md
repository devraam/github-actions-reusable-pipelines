# Reusable GitHub Actions CI Pipelines for Go Microservices

Centralized, reusable CI pipeline templates for Go microservices at scale. This pattern is used in production across dozens of microservices in a 600+ repository platform, where a single set of workflow templates enforces consistent quality, security scanning, and build standards for every service.

## Why centralized pipelines

When dozens of teams own hundreds of repositories, per-repo pipeline definitions drift fast: different linter versions, inconsistent security scanning, unpredictable build behavior. Reusable workflows (`workflow_call`) solve this:

- **One source of truth** — pipeline logic lives here; service repos contain a ~15-line caller.
- **Progressive enforcement** — quality gates run in *informative mode* first (`QUALITY_GATE_ENABLED: false`), letting teams see findings without blocking, then flip to enforcing per repo when ready.
- **Consolidated reporting** — a final `pipeline-report` job aggregates every stage into a single job summary, so developers read one report instead of five logs.
- **Deterministic checkouts** — a single `CHECKOUT_REF` is resolved once and propagated to every job, eliminating ref-drift between jobs on busy branches.
- **Automatic module detection** — the `APPDIR` pattern locates the Go module inside monorepo-style layouts without per-repo configuration.

## Pipeline stages

| Stage | Tool | Purpose |
|---|---|---|
| Lint | golangci-lint | Style + static analysis |
| Unit tests | go test + coverage | Correctness, coverage report |
| SAST | gosec | Static application security testing |
| Dependency scan | govulncheck | Known-vulnerability detection in deps |
| Build & image scan | docker build + Trivy | Container build (no push) + CVE scan |
| Report | consolidated summary | Single aggregated pipeline report |

## Usage

In a service repository, add `.github/workflows/ci.yml`:

```yaml
name: CI
on:
  pull_request:
  push:
    branches: [main]

jobs:
  ci:
    uses: devraam/github-actions-reusable-pipelines/.github/workflows/go-ci.yml@main
    with:
      app-dir: "."
      go-version: "1.22"
      quality-gate-enabled: false   # informative mode; set true to enforce
```

See [`examples/caller.yml`](examples/caller.yml) for a complete example.

## Design notes

- Workflows are sanitized reference implementations of patterns I run in production (multicloud Azure + AWS platform, Go microservices, ACR/ECR registries). Client-specific values, registries, and secrets have been removed.
- Secrets are consumed exclusively through `secrets: inherit` or explicit inputs — never hardcoded, never echoed.
- Image builds run with `--no-push` in CI; publishing happens in a separate, gated release workflow.

## Author

Alexander Abril — Senior DevOps Engineer / Platform Engineer
[linkedin.com/in/abrilalexander](https://www.linkedin.com/in/abrilalexander) · [github.com/devraam](https://github.com/devraam)
