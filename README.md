# workflows

Reusable GitHub Actions workflows shared across Meridth, LLC repositories.

Pin callers to a release commit SHA with the version in a comment. Dependabot's `github-actions` ecosystem bumps these pins.

## mark-ready

Marks a draft PR ready for review once its checks pass, if it carries the opt-in label. The action removes the label after flipping the PR. Fork PRs are skipped, since their token is read-only.

`.github/workflows/mark-ready-when-ready.yaml` in the calling repo:

```yaml
name: Mark PR Ready When Ready

on:
  pull_request:
    types: [opened, edited, labeled, unlabeled, synchronize]

permissions: {}

concurrency:
  group: ${{ github.workflow }}-${{ github.head_ref || github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

jobs:
  mark-ready:
    permissions:
      checks: read # Poll check runs to know when they pass
      contents: write # Required by mark-ready-when-ready to flip the PR out of draft
      pull-requests: write # Mark the PR ready for review
      statuses: read # Poll commit statuses alongside check runs
    uses: meridth/workflows/.github/workflows/mark-ready.yaml@<sha> # v1.0.0
```

The calling repo needs a `Mark Ready When Ready` label, or pass another name with `with: { label: ... }`.

The caller must grant the permissions above: a reusable workflow can't exceed its caller's permissions.

## hugo-ci

Builds a Hugo site. Its job reports as the `build-site` check; called from a job named `build`, it appears as `build / build-site`. Set `upload-artifact` to hand the built `public/` to a later job in the same run, such as [a11y](#a11y).

| Input | Default | Purpose |
|---|---|---|
| `hugo-version` | `0.166.0` | Hugo extended version |
| `git-info` | `false` | Full history (blobs on demand) for sites using `enableGitInfo` |
| `base-url` | site config | Override Hugo's `baseURL`, e.g. `http://localhost:4173/` for a11y |
| `upload-artifact` | none | Artifact name for `public/`; empty skips the upload |

## a11y

Serves a built static site from an artifact and scans it with [pa11y-ci](https://github.com/pa11y/pa11y-ci). Its job reports as the `pa11y` check; called from a job named `a11y`, it appears as `a11y / pa11y`. It isn't Hugo-specific: any earlier job in the same run can upload the site.

| Input | Default | Purpose |
|---|---|---|
| `artifact` | required | Artifact holding the built site |
| `pa11y-config` | `.pa11yci.js` | pa11y-ci config path |
| `node-version` | `24` | Node.js version for pa11y-ci |
| `port` | `4173` | Local port to serve the site on |

The calling repo needs:

- `package.json` and `package-lock.json` with `pa11y-ci` and `serve` as dev dependencies. The workflow runs them with `npx --no-install`, so their versions come from your lockfile.
- A pa11y-ci config that scans the served site. The workflow sets `SITE_URL` (`http://localhost:4173/` by default) for the pa11y-ci step, so read it with `process.env.SITE_URL` in a `.pa11yci.js`. Build the site with the same address as its base URL. Set `chromeLaunchConfig.executablePath` from the `CHROME_PATH` environment variable; the workflow points it at the runner's Chrome and skips Puppeteer's download.

## Hugo site CI example

`.github/workflows/ci.yaml` in the calling repo, building once and scanning that build:

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
    uses: meridth/workflows/.github/workflows/hugo-ci.yaml@<sha> # v1.1.0
    with:
      base-url: http://localhost:4173/
      upload-artifact: site

  a11y:
    needs: build
    permissions:
      contents: read # Clone the repository for package.json and the pa11y config
    uses: meridth/workflows/.github/workflows/a11y.yaml@<sha> # v1.1.0
    with:
      artifact: site
```

Required checks: `build / build-site` and `a11y / pa11y`.
