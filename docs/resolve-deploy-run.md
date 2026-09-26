# Resolve Deploy Run

Workflow: [`.github/workflows/resolve-deploy-run.yaml`](../.github/workflows/resolve-deploy-run.yaml)

Finds the deploy run to promote and outputs its commit SHA. It checks that the run is a successful run of the deploy workflow on the expected branch, so a promote can't ship an arbitrary or failed commit. Promote rebuilds that commit instead of reusing the deploy run's artifact, so artifacts can expire quickly.

| Input | Default | Purpose |
|---|---|---|
| `run-id` | latest successful | Deploy run ID to promote |
| `workflow-file` | `deploy.yaml` | File name of the deploy workflow |
| `workflow-name` | `deploy` | The deploy workflow's `name:`, checked against the run |
| `branch` | `main` | Branch the run must have run on |

Outputs: `sha` (the run's commit) and `run-id`.

## Example: promote to production

`.github/workflows/promote.yaml` in the calling repo, rebuilding the resolved commit and deploying it to production:

```yaml
name: promote to production

on:
  workflow_dispatch:
    inputs:
      run_id:
        description: "deploy run ID to promote (empty = latest successful)"
        required: false
        type: string

permissions: {}

concurrency:
  group: ${{ github.workflow }}
  cancel-in-progress: false

jobs:
  resolve:
    permissions:
      actions: read # Look up deploy runs
    uses: meridth/workflows/.github/workflows/resolve-deploy-run.yaml@<sha> # v1.3.0
    with:
      run-id: ${{ inputs.run_id }}

  build:
    needs: resolve
    permissions:
      contents: read # Check out the promoted commit
    uses: meridth/workflows/.github/workflows/hugo-ci.yaml@<sha> # v1.3.0
    with:
      ref: ${{ needs.resolve.outputs.sha }}
      upload-artifact: site

  deploy:
    needs: [resolve, build]
    permissions:
      contents: read # Check out functions/ when deploying Pages Functions
    uses: meridth/workflows/.github/workflows/cloudflare-pages-deploy.yaml@<sha> # v1.3.0
    secrets: inherit # zizmor: ignore[secrets-inherit] environment secrets reach the deploy job only this way
    with:
      artifact: site
      environment: production
      environment-url: https://example.com
      cloudflare-project: example
      cloudflare-branch: main
      ref: ${{ needs.resolve.outputs.sha }}
```

Pin to a release commit SHA with the version in a comment, as shown. See [Releases](https://github.com/meridth/workflows/releases).
