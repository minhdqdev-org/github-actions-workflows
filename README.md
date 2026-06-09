# github-actions-workflows

This repository contains a collection of reusable GitHub Actions workflows.

## Available Workflows

### Django CI (`ci/django.yml`)

**Banking-grade** CI/CD pipeline for Django applications with comprehensive testing, quality gates, security scanning, and image vulnerability analysis. Inspired by enterprise banking pipelines but optimized for startup speed and flexibility.

**Features:**
- ✅ Unit testing with coverage reporting (HTML artifacts)
- ✅ Code quality checks with threshold-based gates (ruff)
- ✅ Django-specific validation (check --deploy, migrations)
- ✅ Multi-layer security scanning (Bandit, Safety, Trivy)
- ✅ Container image vulnerability scanning
- ✅ SBOM (Software Bill of Materials) generation
- ✅ Consolidated security gate with severity thresholds
- ✅ GitHub Security tab integration (SARIF reports)
- ✅ Automated ArgoCD deployment updates

**Pipeline Architecture:**
```
┌─────────┐
│  test   │────┐
└─────────┘    │
               │
┌──────────┐   │
│quality   │   │
│  gate    │───┤
└──────────┘   │
               ├──> ┌─────────┐ ──> ┌────────┐ ──> ┌──────────┐ ──> ┌─────────┐
┌──────────┐   │    │ django  │     │ build  │     │  scan    │     │security │
│ django   │   │    │ checks  │     │  &     │     │  image   │     │  gate   │
│ checks   │───┤    └─────────┘     │ push   │     └────────┘     └─────────┘
└──────────┘   │                    └────────┘                          │
               │                                                         ▼
┌──────────┐   │                                                    ┌─────────┐
│security  │   │                                                    │ argocd  │
│  scan    │───┘                                                    │ update  │
└──────────┘                                                        └─────────┘
```

**Usage:**

```yaml
jobs:
  ci:
    uses: minhdqdev/github-actions-workflows/ci/django.yml@main
    with:
      image_name: my-django-app
      
      # Testing
      python-version: "3.11"
      run_tests: true
      min_coverage: 70
      upload_coverage_report: true
      
      # Quality Gate (Startup Defaults)
      run_quality_gate: true
      quality_gate_strict: true
      quality_gate_max_errors: 5      # Banking: 0, Startup: 5
      quality_gate_max_warnings: 50   # Banking: 10, Startup: 50
      ruff_config: "ruff.toml"        # Optional custom config
      
      # Django Checks
      run_django_checks: true
      django_checks_strict: true      # Always enforce Django best practices
      
      # Security Scanning
      run_security_scan: true
      security_scan_strict: false
      generate_sbom: true             # Generate SBOM for compliance
      
      # Image Security
      scan_image_security: true
      image_security_severity: "CRITICAL,HIGH"
      image_security_strict: false
      
      # Security Gate (Consolidated)
      security_gate_max_critical: 0   # Zero tolerance
      security_gate_max_high: 2       # Banking: 0, Startup: 2
      security_gate_max_medium: 10    # Banking: 5, Startup: 10
      
      # Build & Deploy
      dockerfile: "./deploy/dockerfile"
    secrets:
      HARBOR_USERNAME: ${{ secrets.HARBOR_USERNAME }}
      HARBOR_PASSWORD: ${{ secrets.HARBOR_PASSWORD }}
```

**Security Scanning Layers:**
1. **Bandit**: Python code security (injection, hardcoded secrets, etc.)
2. **Safety**: Dependency CVE database checking
3. **Trivy Filesystem**: SCA (Software Composition Analysis) for dependencies
4. **Trivy Image**: Container image vulnerability scanning (OS + app layers)
5. **Security Gate**: Aggregates all findings with severity-based thresholds

**Artifacts Generated:**
- `coverage-report`: HTML coverage report (30 days)
- `quality-report`: Ruff JSON findings (30 days)
- `security-reports`: Bandit, Safety, Trivy FS reports (30 days)
- `trivy-image-report`: Container scan results (30 days)
- `sbom`: CycloneDX & SPDX Bill of Materials (90 days)
- `security-gate-report`: Consolidated security summary (90 days)

**Example Configurations:**

<details>
<summary>🏦 Banking-Grade - Maximum Security</summary>

```yaml
jobs:
  ci:
    uses: minhdqdev/github-actions-workflows/ci/django.yml@main
    with:
      image_name: my-banking-app
      python-version: "3.11"
      min_coverage: 90
      quality_gate_strict: true
      quality_gate_max_errors: 0
      quality_gate_max_warnings: 10
      django_checks_strict: true
      scan_image_security: true
      image_security_strict: true
      security_gate_max_critical: 0
      security_gate_max_high: 0
      security_gate_max_medium: 5
      generate_sbom: true
    secrets:
      HARBOR_USERNAME: ${{ secrets.HARBOR_USERNAME }}
      HARBOR_PASSWORD: ${{ secrets.HARBOR_PASSWORD }}
```
</details>

<details>
<summary>🚀 Startup - Balanced (Recommended)</summary>

```yaml
jobs:
  ci:
    uses: minhdqdev/github-actions-workflows/ci/django.yml@main
    with:
      image_name: my-startup-app
      min_coverage: 70
      quality_gate_max_errors: 5
      quality_gate_max_warnings: 50
      scan_image_security: true
      security_gate_max_critical: 0
      security_gate_max_high: 2
      security_gate_max_medium: 10
    secrets:
      HARBOR_USERNAME: ${{ secrets.HARBOR_USERNAME }}
      HARBOR_PASSWORD: ${{ secrets.HARBOR_PASSWORD }}
```
</details>

<details>
<summary>🧪 Development - Fast Iteration</summary>

```yaml
jobs:
  ci:
    uses: minhdqdev/github-actions-workflows/ci/django.yml@main
    with:
      image_name: my-app-dev
      min_coverage: 60
      quality_gate_strict: false
      security_scan_strict: false
      image_security_strict: false
      generate_sbom: false
    secrets:
      HARBOR_USERNAME: ${{ secrets.HARBOR_USERNAME }}
      HARBOR_PASSWORD: ${{ secrets.HARBOR_PASSWORD }}
```
</details>

<details>
<summary>⚡ Emergency Hotfix - Minimal Checks</summary>

```yaml
jobs:
  ci:
    uses: minhdqdev/github-actions-workflows/ci/django.yml@main
    with:
      image_name: my-app-hotfix
      run_quality_gate: false
      run_security_scan: false
      scan_image_security: false
    secrets:
      HARBOR_USERNAME: ${{ secrets.HARBOR_USERNAME }}
      HARBOR_PASSWORD: ${{ secrets.HARBOR_PASSWORD }}
```
</details>

### Next.js CI (`ci/nextjs.yml`)

Installs dependencies, lints, builds a Next.js application, extracts version from `package.json`, and pushes a Docker image to Harbor.

**Usage:**

```yaml
jobs:
  ci:
    uses: minhdqdev/github-actions-workflows/ci/nextjs.yml@main
    with:
      image_name: my-nextjs-app
      # Optional inputs:
      # node-version: "20"
      # dockerfile: "./Dockerfile"
    secrets:
      HARBOR_USERNAME: ${{ secrets.HARBOR_USERNAME }}
      HARBOR_PASSWORD: ${{ secrets.HARBOR_PASSWORD }}
```

### Generic Docker CI (`ci/docker.yml`)

A generic workflow for building and pushing Docker images for any project type.

**Usage:**

```yaml
jobs:
  ci:
    uses: minhdqdev/github-actions-workflows/ci/docker.yml@main
    with:
      image_name: my-app
      # Optional inputs:
      # dockerfile: "./Dockerfile"
      # version: "1.0.0"
    secrets:
      HARBOR_USERNAME: ${{ secrets.HARBOR_USERNAME }}
      HARBOR_PASSWORD: ${{ secrets.HARBOR_PASSWORD }}
```
