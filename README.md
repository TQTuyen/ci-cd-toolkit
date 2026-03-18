# CI/CD Toolkit

Reusable GitHub Actions workflows and composite actions for Node.js projects. Supports **pnpm**, **npm**, and **Bun** package managers, **Nx monorepos**, and deployments to **AWS** and **GCP**.

## Quick Start

Reference workflows from your repo:

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main]
  pull_request:

jobs:
  lint:
    uses: org/ci-cd-toolkit/.github/workflows/ci-lint.yml@main
    with:
      package-manager: pnpm

  test:
    uses: org/ci-cd-toolkit/.github/workflows/ci-test.yml@main
    with:
      package-manager: pnpm
      coverage: true
    secrets:
      codecov-token: ${{ secrets.CODECOV_TOKEN }}

  build:
    uses: org/ci-cd-toolkit/.github/workflows/ci-build.yml@main
    with:
      package-manager: pnpm
      artifact-name: app-build
```

## Composite Actions

### `setup-node`

Sets up Node.js with the specified package manager and dependency caching.

| Input | Default | Description |
|---|---|---|
| `node-version` | `20` | Node.js version |
| `node-version-file` | | File containing Node.js version (`.nvmrc`, `.node-version`) |
| `package-manager` | `pnpm` | `pnpm`, `npm`, or `bun` |
| `package-manager-version` | `latest` | Package manager version |
| `install-dependencies` | `true` | Whether to install dependencies |
| `working-directory` | `.` | Working directory |

| Output | Description |
|---|---|
| `run-command` | Run command prefix for the selected package manager |
| `cache-hit` | Whether dependency cache was hit |

### `setup-docker`

Sets up Docker Buildx with optional registry login.

| Input | Default | Description |
|---|---|---|
| `registry` | | `ghcr`, `ecr`, `gar`, `dockerhub`, or empty |
| `ghcr-token` | | GitHub token for GHCR |
| `dockerhub-username` | | DockerHub username |
| `dockerhub-token` | | DockerHub token |
| `aws-region` | `us-east-1` | AWS region for ECR |
| `gcp-region` | `us-central1` | GCP region for GAR |

| Output | Description |
|---|---|
| `registry-url` | The registry URL |

### `setup-nx`

Sets up Nx SHAs for affected commands.

| Input | Default | Description |
|---|---|---|
| `main-branch` | `main` | Main branch name |
| `nx-cloud-access-token` | | Nx Cloud access token |

| Output | Description |
|---|---|
| `base` | Base SHA |
| `head` | Head SHA |

### `setup-aws`

Configures AWS credentials via OIDC (preferred) or static keys.

| Input | Default | Description |
|---|---|---|
| `role-to-assume` | | IAM role ARN for OIDC |
| `aws-access-key-id` | | Static access key ID |
| `aws-secret-access-key` | | Static secret access key |
| `aws-region` | `us-east-1` | AWS region |

| Output | Description |
|---|---|
| `account-id` | AWS account ID |

### `setup-gcp`

Configures GCP authentication via Workload Identity Federation (preferred) or service account key.

| Input | Default | Description |
|---|---|---|
| `workload-identity-provider` | | WIF provider resource name |
| `service-account` | | Service account email for WIF |
| `credentials-json` | | SA key JSON (fallback) |
| `project-id` | | GCP project ID |
| `token-format` | `access_token` | Token format |

| Output | Description |
|---|---|
| `project-id` | GCP project ID |
| `access-token` | GCP access token |

### `codecov`

Uploads coverage reports to Codecov.

| Input | Default | Description |
|---|---|---|
| `token` | *required* | Codecov upload token |
| `files` | | Coverage report files |
| `flags` | | Codecov flags |
| `working-directory` | | Directory containing coverage files |

## CI Workflows

All CI workflows share these common inputs:

| Input | Default | Description |
|---|---|---|
| `node-version` | `20` | Node.js version |
| `package-manager` | `pnpm` | Package manager |
| `working-directory` | `.` | Working directory |
| `nx-affected` | `false` | Use Nx affected commands |
| `concurrency-group` | | Custom concurrency group |

### `ci-lint.yml`

| Input | Default | Description |
|---|---|---|
| `lint-command` | `lint` | Script name to run |

### `ci-typecheck.yml`

| Input | Default | Description |
|---|---|---|
| `typecheck-command` | `typecheck` | Script name to run |

### `ci-test.yml`

| Input | Default | Description |
|---|---|---|
| `test-command` | `test` | Script name to run |
| `coverage` | `false` | Upload coverage to Codecov |
| `coverage-files` | | Coverage report files |
| `coverage-flags` | | Codecov flags |

| Secret | Description |
|---|---|
| `codecov-token` | Codecov upload token |

### `ci-build.yml`

| Input | Default | Description |
|---|---|---|
| `build-command` | `build` | Script name to run |
| `artifact-name` | | Artifact name (empty = skip upload) |
| `artifact-path` | `dist` | Build output path |

| Output | Description |
|---|---|
| `artifact-id` | Uploaded artifact ID |

## CD Workflows

### `cd-docker.yml`

Build and push Docker images to any supported registry.

| Input | Default | Description |
|---|---|---|
| `image-name` | *required* | Image name (without registry prefix) |
| `registry` | `ghcr` | `ghcr`, `ecr`, `gar`, `dockerhub` |
| `dockerfile` | `Dockerfile` | Dockerfile path |
| `context` | `.` | Build context |
| `build-args` | | Build args (newline-separated) |
| `platforms` | `linux/amd64` | Target platforms |
| `push` | `true` | Push the image |
| `tags` | | Additional tags |
| `aws-region` | `us-east-1` | AWS region (ECR) |
| `gcp-region` | `us-central1` | GCP region (GAR) |

| Secret | Description |
|---|---|
| `registry-token` | GHCR or DockerHub token |
| `aws-role-arn` | AWS role ARN (ECR) |
| `wif-provider` | GCP WIF provider (GAR) |
| `service-account` | GCP service account (GAR) |

### `cd-deploy-ecr.yml`

Build and push to AWS ECR.

| Input | Default | Description |
|---|---|---|
| `ecr-repository` | *required* | ECR repository name |
| `dockerfile` | `Dockerfile` | Dockerfile path |
| `context` | `.` | Build context |
| `build-args` | | Build args |
| `platforms` | `linux/amd64` | Target platforms |
| `aws-region` | `us-east-1` | AWS region |

| Secret | Description |
|---|---|
| `aws-role-arn` | *required* — AWS IAM role ARN |

| Output | Description |
|---|---|
| `image-uri` | Full ECR image URI |

### `cd-deploy-s3.yml`

Sync files to S3 with optional CloudFront invalidation.

| Input | Default | Description |
|---|---|---|
| `s3-bucket` | *required* | S3 bucket name |
| `source-path` | `dist` | Local source path |
| `cloudfront-distribution-id` | | CloudFront distribution ID |
| `aws-region` | `us-east-1` | AWS region |
| `artifact-name` | | Download artifact first |
| `delete-removed` | `true` | Delete files not in source |

| Secret | Description |
|---|---|
| `aws-role-arn` | AWS role ARN (OIDC) |
| `aws-access-key-id` | Static key (fallback) |
| `aws-secret-access-key` | Static secret (fallback) |

### `cd-deploy-gar.yml`

Build and push to Google Artifact Registry.

| Input | Default | Description |
|---|---|---|
| `repository` | *required* | GAR repo path (`project/repo/image`) |
| `gcp-region` | `us-central1` | GCP region |
| `dockerfile` | `Dockerfile` | Dockerfile path |
| `context` | `.` | Build context |
| `build-args` | | Build args |
| `platforms` | `linux/amd64` | Target platforms |
| `project-id` | | GCP project ID |

| Secret | Description |
|---|---|
| `wif-provider` | *required* — WIF provider |
| `service-account` | *required* — GCP SA email |

| Output | Description |
|---|---|
| `image-uri` | Full GAR image URI |
| `image-digest` | Image digest |

### `cd-deploy-gcs.yml`

Sync files to GCS with optional Cloud CDN invalidation.

| Input | Default | Description |
|---|---|---|
| `bucket` | *required* | GCS bucket (`gs://...`) |
| `source-path` | `dist` | Local source path |
| `artifact-name` | | Download artifact first |
| `delete-removed` | `true` | Delete files not in source |
| `cdn-invalidate` | `false` | Invalidate Cloud CDN |
| `cdn-url-map` | | CDN URL map name |
| `project-id` | | GCP project ID |

| Secret | Description |
|---|---|
| `wif-provider` | WIF provider |
| `service-account` | GCP SA email |
| `credentials-json` | SA key JSON (fallback) |

### `cd-deploy-cloud-run.yml`

Deploy a container to Cloud Run.

| Input | Default | Description |
|---|---|---|
| `service` | *required* | Cloud Run service name |
| `image` | *required* | Container image URI |
| `region` | `us-central1` | GCP region |
| `project-id` | | GCP project ID |
| `env-vars` | | Env vars (newline-separated `KEY=VALUE`) |
| `flags` | | Extra `gcloud run deploy` flags |

| Secret | Description |
|---|---|
| `wif-provider` | *required* — WIF provider |
| `service-account` | *required* — GCP SA email |

| Output | Description |
|---|---|
| `url` | Cloud Run service URL |

### `cd-npm-publish.yml`

Publish a package to npm with provenance.

| Input | Default | Description |
|---|---|---|
| `node-version` | `20` | Node.js version |
| `package-manager` | `pnpm` | Package manager |
| `working-directory` | `.` | Working directory |
| `registry-url` | `https://registry.npmjs.org` | npm registry |
| `access` | `public` | `public` or `restricted` |
| `provenance` | `true` | Publish with provenance |
| `dry-run` | `false` | Dry run mode |

| Secret | Description |
|---|---|
| `npm-token` | *required* — npm auth token |

### `cd-github-release.yml`

Create a GitHub release with optional artifacts.

| Input | Default | Description |
|---|---|---|
| `tag-name` | `github.ref_name` | Release tag |
| `generate-notes` | `true` | Auto-generate release notes |
| `draft` | `false` | Create as draft |
| `prerelease` | `false` | Mark as pre-release |
| `artifact-name` | | Attach artifact to release |

| Output | Description |
|---|---|
| `release-id` | Release ID |
| `release-url` | Release URL |

## Quality & Automation Workflows

### `quality-codeql.yml`

| Input | Default | Description |
|---|---|---|
| `languages` | `javascript-typescript` | Languages to analyze |
| `build-mode` | `none` | CodeQL build mode |
| `queries` | `security-extended` | Query suite |

### `auto-dependabot-merge.yml`

| Input | Default | Description |
|---|---|---|
| `update-types` | `patch,minor` | Update types to auto-merge |

## Examples

See the [`examples/`](./examples/) directory for complete caller workflow files:

- **[`nestjs-api-aws.yml`](./examples/nestjs-api-aws.yml)** — NestJS API with ECR deploy
- **[`nestjs-api-gcp.yml`](./examples/nestjs-api-gcp.yml)** — NestJS API with GAR + Cloud Run deploy
- **[`react-spa-aws.yml`](./examples/react-spa-aws.yml)** — React SPA with S3/CloudFront deploy
- **[`react-spa-gcp.yml`](./examples/react-spa-gcp.yml)** — React SPA with GCS/CDN deploy
- **[`nx-monorepo.yml`](./examples/nx-monorepo.yml)** — Nx monorepo with affected commands
- **[`npm-library.yml`](./examples/npm-library.yml)** — npm publish on release

## Secrets Reference

| Secret | Used by | Description |
|---|---|---|
| `AWS_ROLE_ARN` | ECR, S3 workflows | AWS IAM role for OIDC auth |
| `AWS_ACCESS_KEY_ID` | S3 (fallback) | Static AWS key |
| `AWS_SECRET_ACCESS_KEY` | S3 (fallback) | Static AWS secret |
| `GCP_WIF_PROVIDER` | GAR, GCS, Cloud Run | GCP Workload Identity Federation provider |
| `GCP_SERVICE_ACCOUNT` | GAR, GCS, Cloud Run | GCP service account email |
| `CODECOV_TOKEN` | ci-test | Codecov upload token |
| `NPM_TOKEN` | cd-npm-publish | npm authentication token |

## Permissions Reference

| Workflow | Permissions needed |
|---|---|
| CI workflows | `contents: read` |
| `cd-docker` | `contents: read`, `packages: write`, `id-token: write` |
| `cd-deploy-ecr` | `contents: read`, `id-token: write` |
| `cd-deploy-s3` | `contents: read`, `id-token: write` |
| `cd-deploy-gar` | `contents: read`, `id-token: write` |
| `cd-deploy-gcs` | `contents: read`, `id-token: write` |
| `cd-deploy-cloud-run` | `contents: read`, `id-token: write` |
| `cd-npm-publish` | `contents: read`, `id-token: write` |
| `cd-github-release` | `contents: write` |
| `quality-codeql` | `security-events: write`, `contents: read` |
| `auto-dependabot-merge` | `contents: write`, `pull-requests: write` |

## License

[MIT](./LICENSE)
