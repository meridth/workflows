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

Builds a Hugo site on every PR and, optionally, scans it with [pa11y-ci](https://github.com/pa11y/pa11y-ci). Two jobs report as checks: `build-site` and `a11y`. Called from a job named `ci`, they appear as `ci / build-site` and `ci / a11y`.

`.github/workflows/ci.yaml` in the calling repo:

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
  ci:
    permissions:
      contents: read # Clone the repository
    uses: meridth/workflows/.github/workflows/hugo-ci.yaml@<sha> # v1.1.0
```

Inputs, all optional:

| Input | Default | Purpose |
|---|---|---|
| `hugo-version` | `0.166.0` | Hugo extended version |
| `git-info` | `false` | Full history (blobs on demand) for sites using `enableGitInfo` |
| `a11y` | `true` | Run pa11y-ci against the built site |
| `pa11y-config` | `.pa11yci.js` | pa11y-ci config path |
| `node-version` | `24` | Node.js version for pa11y-ci |

With `a11y: true`, the calling repo needs:

- `package.json` and `package-lock.json` with `pa11y-ci` and `serve` as dev dependencies. The workflow runs them with `npx --no-install`, so their versions come from your lockfile.
- A pa11y-ci config that scans `http://localhost:4173/`. The site is built with that `baseURL` and served there. Set `chromeLaunchConfig.executablePath` from the `CHROME_PATH` environment variable; the workflow points it at the runner's Chrome and skips Puppeteer's download.
