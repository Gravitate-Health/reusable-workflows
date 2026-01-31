# Reusable Workflows

This repository contains reusable workflow files that perform useful jobs for Gravitate-Health projects.

## Available Workflows

### 1. Docker Build and Push (`main.yml`)

Builds and pushes Docker images to GitHub Container Registry with automatic versioning.

**Features:**
- Automatic semantic versioning for production
- Branch-based tagging (dev/main)
- Multi-architecture builds (amd64/arm64)
- Security scanning with Trivy
- Supports both dev and production environments

**Usage:**
```yaml
jobs:
  build:
    uses: Gravitate-Health/reusable-workflows/.github/workflows/main.yml@main
    permissions:
      contents: read
      packages: write
```

Copy `examples/docker-build-push.yml` to `.github/workflows/` in your repository.

---

### 2. Helm Chart Publishing (`helm-publish.yml`)

Publishes Helm charts to GitHub Container Registry as OCI artifacts.

**Features:**
- Lints and tests charts before publishing
- Version from tags (e.g., `v1.2.3`)
- Dev versions for non-release branches
- Automatic 'latest' tag for main branch

**Usage:**
```yaml
jobs:
  publish-helm:
    uses: Gravitate-Health/reusable-workflows/.github/workflows/helm-publish.yml@main
    permissions:
      contents: read
      packages: write
    with:
      chart_path: charts/my-service
      registry_namespace: charts  # optional
```

**Configuration:**
- `chart_path`: Path to your Chart.yaml directory (required)
- `registry_namespace`: GHCR namespace (defaults to 'charts')

Copy `examples/helm-publish.yml` to `.github/workflows/` and update the `chart_path` parameter.

---

### 3. Lens Upload (`lens-upload.yml`)

Tests and uploads lenses to FHIR servers (dev and prod).

**Features:**
- Runs `batch-test` before any uploads
- Uploads to dev server on every push
- Uploads to prod server only when a tag is provided
- Prevents errors by skipping prod when no tag exists

**Usage:**
```yaml
jobs:
  upload-lenses:
    uses: Gravitate-Health/reusable-workflows/.github/workflows/lenses-upload.yaml@main
    with:
      tag: ${{ github.ref_name }}  # optional, only for prod upload
```

**Behavior:**
- **No tag provided**: Tests and uploads to dev server only
- **Tag provided**: Tests, uploads to dev, and uploads to prod with the specified tag

Copy `examples/lens-upload.yml` to `.github/workflows/` in your repository.

---

## Quick Start

1. Choose the workflow(s) you need from the examples above
2. Copy the example file to `.github/workflows/` in your target repository
3. Customize the parameters as needed (chart paths, trigger conditions, etc.)
4. Commit and push to trigger the workflow

## Notes

- All workflows reference `@main` branch of the reusable workflows repository
- You can pin to a specific version/tag instead (e.g., `@v1.0.0`) for stability
- Ensure required permissions are set in each workflow
- Some workflows require repository secrets to be configured
