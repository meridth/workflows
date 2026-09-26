# workflows

Reusable GitHub Actions workflows shared across Meridth, LLC repositories.

## Reusable workflows available

- [Mark Ready](docs/mark-ready.md): marks a labeled draft PR ready once its checks pass
- [Hugo CI](docs/hugo-ci.md): builds a Hugo site, optionally uploading it for later jobs
- [Accessibility (a11y)](docs/a11y.md): scans a built static site with pa11y-ci
- [Cloudflare Pages Deploy](docs/cloudflare-pages-deploy.md): deploys a built static site to Cloudflare Pages in a GitHub environment
- [Resolve Deploy Run](docs/resolve-deploy-run.md): finds the deploy run to promote and outputs its commit

> [!CAUTION]
> A reusable workflow can't exceed its caller's permissions. Each doc lists the permissions its caller job must grant.

> [!TIP]
> Pin callers to a release commit SHA with the version in a comment, for example `@<sha> # v1.2.0`. Dependabot's `github-actions` ecosystem bumps these pins. See [Releases](https://github.com/meridth/workflows/releases).

## Releasing

Releases are created when a PR carrying the `release` label merges to `main`. The version comes from `major`, `minor`, or `patch`, or from the auto-labeler (`feat:` titles get `feature`, a minor bump; `fix:` titles get `fix`, a patch). To release changes that already merged without the label, run the [Release workflow](https://github.com/meridth/workflows/actions/workflows/release.yaml) manually.
