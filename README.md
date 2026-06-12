# github-actions-workflows

Org-wide reusable GitHub Actions workflows for `minhdqdev-org`.

## Available Workflows

### `ci.yml` — Docker CI

Builds and pushes a Docker image to Harbor, then sends a success notification to NotiHub.
Branch is detected at runtime to apply the correct tag strategy:

| Branch | Tag produced | Environment | ArgoCD strategy |
|--------|-------------|-------------|-----------------|
| `develop` | `dev-<short_sha>` | DEV | `latest`, `regexp:^dev-.*$` |
| `rc/*` | `<version>-rc.<run_number>` | UAT | `semver`, `regexp:^.*-rc\.[0-9]+$` |
| `main` / `master` | `<version>` + `latest` | PROD | `semver` |

Version is resolved from `package.json`, `pyproject.toml`, or `Cargo.toml`. Falls back to
the short commit SHA if none are present.

#### Usage

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [develop, main, "rc/**"]

jobs:
  ci:
    uses: minhdqdev-org/github-actions-workflows/.github/workflows/ci.yml@main
```

#### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `dockerfile` | `./Dockerfile` | Path to the Dockerfile |
| `dockerfile_target` | `""` | Build target for multi-stage Dockerfiles |

#### Custom Dockerfile path

```yaml
jobs:
  ci:
    uses: minhdqdev-org/github-actions-workflows/.github/workflows/ci.yml@main
    with:
      dockerfile: ./deploy/dockerfile
```

#### Multi-stage build target

```yaml
jobs:
  ci:
    uses: minhdqdev-org/github-actions-workflows/.github/workflows/ci.yml@main
    with:
      dockerfile_target: production
```

#### Test prerequisite before Docker build

```yaml
jobs:
  test:
    if: github.ref == 'refs/heads/main'
    runs-on: [self-hosted, minhdqdev-org, manual]
    steps:
      - uses: actions/checkout@v4
      - run: bun install --frozen-lockfile
      - run: bun test --coverage

  ci:
    needs: [test]
    if: always() && (needs.test.result == 'success' || needs.test.result == 'skipped')
    uses: minhdqdev-org/github-actions-workflows/.github/workflows/ci.yml@main
```

## Requirements

- **Runner**: `[self-hosted, minhdqdev-org, manual]` with Harbor credentials injected via
  `envFrom` (`HARBOR_USERNAME`, `HARBOR_PASSWORD`) on the runner pod.
- **Registry**: `harbor.internal.minhdq.dev/minhdqdev/<repo-name>`
- **Build cache**: stored at `harbor.internal.minhdq.dev/minhdqdev/<repo-name>:buildcache`

## Further reading

- [Shared CI Workflows standard](https://github.com/minhdqdev-org/engineering-docs/blob/main/docs/design-guidelines/shared-ci-workflows.md)
- [ArgoCD Image Updater standard](https://github.com/minhdqdev-org/engineering-docs/blob/main/docs/design-guidelines/argocd-image-updater.md)
