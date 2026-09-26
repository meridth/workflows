# Cloudflare Pages Deploy

Workflow: [`.github/workflows/cloudflare-pages-deploy.yaml`](../.github/workflows/cloudflare-pages-deploy.yaml)

Deploys a built static site from an artifact to Cloudflare Pages, inside a GitHub environment. It doesn't build anything, so it works for any site: pair it with a build job in the same run, such as [Hugo CI](hugo-ci.md) with `upload-artifact`.

| Input | Default | Purpose |
|---|---|---|
| `artifact` | required | Artifact holding the built site |
| `directory` | `.` | Directory inside the artifact to deploy |
| `environment` | required | GitHub environment, e.g. `staging` or `production` |
| `environment-url` | required | URL shown on the deployment in GitHub |
| `cloudflare-project` | required | Cloudflare Pages project name |
| `cloudflare-branch` | required | Pages branch; the project's production branch deploys to production |
| `functions` | `false` | Also deploy Pages Functions from `functions/` |
| `ref` | triggering commit | Commit to take `functions/` from |

Each environment needs `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID` secrets. The deploy job declares the environment and reads them there. A reusable workflow only sees secrets its caller passes, and the caller job has no environment to read them from, so callers use `secrets: inherit`. zizmor flags that (`secrets-inherit`); ignore it inline with the reason.

Restrict each environment's deployment branches to `main`. Otherwise anyone who can push a branch can run a modified workflow in the environment and read its secrets.

## Example: deploy a Hugo site to staging on merge

`.github/workflows/deploy.yaml` in the calling repo:

```yaml
name: deploy

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions: {}

concurrency:
  group: ${{ github.workflow }}
  cancel-in-progress: false

jobs:
  build:
    permissions:
      contents: read # Clone the repository
    uses: meridth/workflows/.github/workflows/hugo-ci.yaml@<sha> # v1.3.0
    with:
      upload-artifact: site

  deploy:
    needs: build
    permissions:
      contents: read # Check out functions/ when deploying Pages Functions
    uses: meridth/workflows/.github/workflows/cloudflare-pages-deploy.yaml@<sha> # v1.3.0
    secrets: inherit # zizmor: ignore[secrets-inherit] environment secrets reach the deploy job only this way
    with:
      artifact: site
      environment: staging
      environment-url: https://staging.example.pages.dev
      cloudflare-project: example
      cloudflare-branch: staging
```

For promoting a staging deploy to production, see [Resolve Deploy Run](resolve-deploy-run.md#example-promote-to-production).

Pin to a release commit SHA with the version in a comment, as shown. See [Releases](https://github.com/meridth/workflows/releases).
