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

### 4. NPM Package Publish (`npm-publish.yml`)

Builds, tests and publishes an NPM package to [npmjs.com](https://www.npmjs.com) when a semver tag (`vX.Y.Z`) is pushed.

**Features:**
- Triggered exclusively by strict semver tags (`v1.2.3` — no pre-release suffixes)
- Automatically syncs `package.json` version to the pushed tag
- Runs `npm ci` → `npm run test` → `npm run build` → `npm publish`
- Each step is individually skippable via inputs
- Uses an org-level `NPM_TOKEN` secret (passed via `secrets: inherit`)

**Usage:**
```yaml
jobs:
  publish:
    uses: Gravitate-Health/reusable-workflows/.github/workflows/npm-publish.yml@main
    secrets: inherit
    with:
      node_version: "20"
      test_script: "test"    # set to '' to skip
      build_script: "build"  # set to '' for pure-JS packages
      access: "public"
```

**Configuration:**
| Input | Default | Description |
|---|---|---|
| `node_version` | `"20"` | Node.js version |
| `test_script` | `"test"` | `npm run` script for tests; `''` to skip |
| `build_script` | `"build"` | `npm run` script for building; `''` to skip |
| `access` | `"public"` | `"public"` or `"restricted"` (scoped private packages) |
| `sync_version` | `true` | Sync `package.json` version to the git tag |
| `registry_url` | `https://registry.npmjs.org` | Target NPM registry |

**Required secret:** `NPM_TOKEN` — an npmjs automation token. See setup instructions below.

Copy `examples/npm-publish.yml` to `.github/workflows/` in your repository.

---

## Setting up the NPM Token (npmjs.com + GitHub Org secret)

### Step 1 — Create an NPM Automation Token

1. Log in to [npmjs.com](https://www.npmjs.com) with the publish account (ideally a machine/bot account for your org).
2. Go to **Avatar → Access Tokens → Generate New Token → Classic Token**.
3. Choose type **Automation** (bypasses 2FA—required for CI; Publish tokens require 2FA on every publish).
4. Copy the token (`npm_…`).

> **Tip:** Use a dedicated npmjs machine account scoped only to the packages you need to publish rather than a personal account.

### Step 2 — Add it as a GitHub Organisation Secret

**Option A — GitHub web UI**

1. Go to your GitHub organisation → **Settings → Secrets and variables → Actions**.
2. Click **New organisation secret**.
3. Name: `NPM_TOKEN` | Value: the token from Step 1.
4. Under *Repository access*, choose **Selected repositories** (recommended) and add each repo that should publish, or choose **All repositories** for org-wide access.
5. Click **Add secret**.

**Option B — GitHub CLI (`gh`)**

```bash
# Set for the whole org, visible to all repos (adjust --visibility as needed)
gh secret set NPM_TOKEN \
  --org <YOUR_ORG> \
  --visibility all \
  --body "npm_xxxxxxxxxxxxxxxxxxxx"

# Or restrict to selected repos
gh secret set NPM_TOKEN \
  --org <YOUR_ORG> \
  --repos "repo1,repo2" \
  --body "npm_xxxxxxxxxxxxxxxxxxxx"
```

> Requires `gh` ≥ 2.x and org admin rights.

### Step 3 — Wire it up in your project

Copy `examples/npm-publish.yml` to `.github/workflows/npm-publish.yml` in the target repository. Because the example uses `secrets: inherit`, the org secret `NPM_TOKEN` is automatically forwarded to the reusable workflow — no extra configuration needed.

### Step 4 — Publish a release

```bash
# Standard flow: bump version locally, push the tag
npm version patch   # or minor / major / 1.2.3
git push --follow-tags
```

The pushed `vX.Y.Z` tag triggers the workflow which will test, build and publish to npmjs.

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
