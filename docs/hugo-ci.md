# Hugo CI

Workflow: [`.github/workflows/hugo-ci.yaml`](../.github/workflows/hugo-ci.yaml)

Builds a Hugo site. Its job reports as the `build-site` check; called from a job named `build`, it appears as `build / build-site`. Set `upload-artifact` to hand the built `public/` to a later job in the same run, such as [a11y](a11y.md).

| Input | Default | Purpose |
|---|---|---|
| `hugo-version` | `0.166.0` | Hugo extended version |
| `git-info` | `false` | Full history (blobs on demand) for sites using `enableGitInfo` |
| `ref` | triggering commit | Commit, branch, or tag to build; promote passes the resolved deploy commit |
| `base-url` | site config | Override Hugo's `baseURL`, e.g. `http://localhost:4173/` for a11y |
| `upload-artifact` | none | Artifact name for `public/`; empty skips the upload |
| `prepare-command` | none | Shell command run before Hugo, e.g. to write `data/` files; empty skips it |
| `node-version` | `24` | Node.js version available to `prepare-command` |

| Secret | Purpose |
|---|---|
| `prepare-secret` | Optional. Available to `prepare-command` as `$PREPARE_SECRET`, and to no other step |

`prepare-command` runs right after checkout, with Node installed and nothing else: there is no `npm ci`, so no third-party package code runs in the step that can see `prepare-secret`. A script that needs only Node's built-in modules fits this best.

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

## Example: generate data before building

Pass a secret only to the prepare step, under the name your script expects:

```yaml
jobs:
  build:
    permissions:
      contents: read # Clone the repository
    uses: meridth/workflows/.github/workflows/hugo-ci.yaml@<sha> # v1.4.0
    with:
      upload-artifact: site
      prepare-command: FEED_URL="$PREPARE_SECRET" node scripts/fetch-data.mjs
    secrets:
      prepare-secret: ${{ secrets.FEED_URL }}
```

Pin to a release commit SHA with the version in a comment, as shown. See [Releases](https://github.com/meridth/workflows/releases).
