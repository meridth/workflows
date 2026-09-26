# Mark Ready

Workflow: [`.github/workflows/mark-ready.yaml`](../.github/workflows/mark-ready.yaml)

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
    uses: meridth/workflows/.github/workflows/mark-ready.yaml@<sha> # v1.2.0
```

| Input | Default | Purpose |
|---|---|---|
| `label` | `mark-ready-when-ready` | Label that opts a draft PR in |

The calling repo needs a `mark-ready-when-ready` label, or pass another name with `with: { label: ... }`.

The caller must grant the permissions above: a reusable workflow can't exceed its caller's permissions.

Pin to a release commit SHA with the version in a comment, as shown. See [Releases](https://github.com/meridth/workflows/releases).
