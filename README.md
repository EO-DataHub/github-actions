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

## Renovate config

`renovate.json5` defines configuration of Renovate Bot to maintain and upgrade dependencies. This is easily reusable by extending the project's Renovate config (`.github/renovate.json`):

```
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>EO-DataHub/github-actions//renovate.json5"]
}
```
