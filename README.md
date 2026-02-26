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
```

## Renovate config

`renovate.json5` defines configuration of Renovate Bot to maintain and upgrade dependencies. This is easily reusable by extending the project's Renovate config (`.github/renovate.json`):

```
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>EO-DataHub/github-actions"]
}
```
