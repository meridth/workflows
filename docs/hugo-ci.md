# Hugo CI

Workflow: [`.github/workflows/hugo-ci.yaml`](../.github/workflows/hugo-ci.yaml)

Builds a Hugo site. Its job reports as the `build-site` check; called from a job named `build`, it appears as `build / build-site`. Set `upload-artifact` to hand the built `public/` to a later job in the same run, such as [a11y](a11y.md).

| Input | Default | Purpose |
|---|---|---|
| `hugo-version` | `0.166.0` | Hugo extended version |
| `git-info` | `false` | Full history (blobs on demand) for sites using `enableGitInfo` |
| `base-url` | site config | Override Hugo's `baseURL`, e.g. `http://localhost:4173/` for a11y |
| `upload-artifact` | none | Artifact name for `public/`; empty skips the upload |

## Example: build and scan a Hugo site

`.github/workflows/ci.yaml` in the calling repo, building once and scanning that build with [a11y](a11y.md):

```yaml
name: ci

on:
  pull_request:
    branches: [main]

permissions: {}

concurrency:
  group: ${{ github.workflow }}-${{ github.head_ref || github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

jobs:
  build:
    permissions:
      contents: read # Clone the repository
    uses: meridth/workflows/.github/workflows/hugo-ci.yaml@<sha> # v1.2.0
    with:
      base-url: http://localhost:4173/
      upload-artifact: site

  a11y:
    needs: build
    permissions:
      contents: read # Clone the repository for package.json and the pa11y config
    uses: meridth/workflows/.github/workflows/a11y.yaml@<sha> # v1.2.0
    with:
      artifact: site
```

Required checks: `build / build-site` and `a11y / pa11y`.

Pin to a release commit SHA with the version in a comment, as shown. See [Releases](https://github.com/meridth/workflows/releases).
