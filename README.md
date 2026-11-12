# GitHub Actions

This repository contains workflows for GitHub actions used by the EO-DataHub. They can be used as per the following example:

```
jobs:
  security-scan:
    name: Call Security Scan
    permissions:
      contents: read
    uses: EO-DataHub/github-actions/.github/workflows/security.yaml@main

  qa:
    uses: EO-DataHub/github-actions/.github/workflows/qa-python.yaml@main

  unit-tests:
    name: Run unit tests
    permissions:
      contents: read
    uses: EO-DataHub/github-actions/.github/workflows/unit-tests-python-uv.yaml@main
    with:
      PYTHON_VERSION: "3.12"

  zizmor:
    permissions:
      contents: read
    uses: EO-DataHub/github-actions/.github/workflows/zizmor.yaml@main
```

## zizmor

`zizmor.yaml` runs [zizmor](https://docs.zizmor.sh/) over the repo's workflows and composite actions. Findings are shown as annotations and fail the job. It takes no inputs.

```
on:
  push:
    branches: [main]
  pull_request:

permissions: {}

jobs:
  zizmor:
    permissions:
      contents: read
    uses: EO-DataHub/github-actions/.github/workflows/zizmor.yaml@main
```

zizmor requires every `uses:` to be pinned to a commit SHA, so calling these workflows at `@main` is flagged. To allow that, add `.github/zizmor.yml` to the calling repo (the pattern is case-sensitive):

```
rules:
  unpinned-uses:
    config:
      policies:
        "EO-DataHub/*": ref-pin
```

The same file is where findings can be ignored (`rules.<id>.ignore`), or use an inline `# zizmor: ignore[<rule>]` comment. To fix findings locally: `uvx zizmor --fix .`.

## Secret scan

`secret-scan.yaml` scans git history with [gitleaks](https://github.com/gitleaks/gitleaks) and fails if it finds a secret. On pull requests and pushes it scans only the new commits. On other events, such as a `schedule` or `workflow_dispatch`, it scans the full history.

```
jobs:
  secret-scan:
    permissions:
      contents: read
    uses: EO-DataHub/github-actions/.github/workflows/secret-scan.yaml@main
    with:
      fail-on-findings: true # default
      config: "" # default: .gitleaks.toml if present, else built-in rules
```

Repos already calling `security.yaml` can set `ENABLE_SECRET_SCAN: true` instead.

If a real secret is found, rotate it. Removing it from the history is not enough.

To ignore a false positive, add a `gitleaks:allow` comment on the line, or add its fingerprint from the job log to `.gitleaksignore`. Old secrets in history don't block pull requests, but a full scan fails until they are rotated and ignored. Run one with `workflow_dispatch` when adding the workflow to a repo.

## Publishing images

`ecr-publish.yaml` builds the image on every run. Where it pushes depends on the run:

| Run | GitHub environment | Pushed to |
|---|---|---|
| Default branch or a tag | `release` | `<alias>/<image>` and `<alias>/dev/<image>`, tagged `latest`, the version, or the tag name |
| Any other branch | `dev` | `<alias>/dev/<image>`, tagged `<branch>` and `<branch>-<short sha>` |
| Pull request | none | nothing, build only |

The AWS roles only trust these two environments of each repository, and only when the job runs this workflow from `main`. So both environments must exist in the calling repo, the repo's OIDC subject must include `job_workflow_ref`, and the workflow must be called at `@main`. `release` should allow only the default branch and release tags. `dev` needs the variable `AWS_DEV_ROLE_ARN`.

`s3-publish.yaml` runs in the `release` environment.

## Renovate config

`renovate.json5` defines configuration of Renovate Bot to maintain and upgrade dependencies. This is easily reusable by extending the project's Renovate config (`.github/renovate.json`):

```
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>EO-DataHub/github-actions//renovate.json5"]
}
```
