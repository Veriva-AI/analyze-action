# Veriva Code Scan — GitHub Action

Analyze pull request diffs for AI-generated code patterns, security vulnerabilities, hallucinated dependencies (slopsquatting), and code quality. Uploads findings to GitHub Code Scanning as SARIF.

## Usage

```yaml
name: Veriva
on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read
  pull-requests: read
  security-events: write   # required when upload-sarif is true

jobs:
  veriva:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0   # full history so the action can diff against base

      - uses: Veriva-AI/analyze-action@v1
        with:
          sarif-file: veriva.sarif
          upload-sarif: true
          fail-on-critical: false
          min-score: 0
```

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `sarif-file` | no | `veriva-results.sarif` | Path to write the SARIF output file. |
| `upload-sarif` | no | `true` | Upload SARIF to GitHub Code Scanning. Needs `security-events: write`. |
| `fail-on-critical` | no | `false` | Fail the action if any CRITICAL or HIGH findings are detected. |
| `min-score` | no | `0` | Minimum trust score (0–100). Fails the action if the computed score is below this threshold. |

## Outputs

| Name | Description |
|---|---|
| `trust-score` | Trust score as an integer 0–100. |
| `grade` | Letter grade A–F. |
| `findings-count` | Total number of findings. |
| `critical-count` | Number of CRITICAL / HIGH findings. |
| `sarif-file` | Path to the generated SARIF file (echoed from the input). |

## Examples

### Gate PRs on trust grade

```yaml
- uses: Veriva-AI/analyze-action@v1
  with:
    fail-on-critical: true
    min-score: 70
```

### SARIF upload with Code Scanning

```yaml
- uses: Veriva-AI/analyze-action@v1
  id: veriva
  with:
    upload-sarif: true
- uses: github/codeql-action/upload-sarif@v3
  if: always()
  with:
    sarif_file: ${{ steps.veriva.outputs.sarif-file }}
```

## What gets analyzed

The action analyses the diff between the PR head and the base ref (or the push's `before..after` range). Layer 1 runs entirely inside the runner — no network calls, no cost. Deeper Layer 2/3 analysis runs on the Veriva platform for signed-in orgs via the [Veriva GitHub App](https://github.com/marketplace/veriva).

## Source

This repo is a generated mirror — published artefacts only (`action.yml`, bundled `dist/`, README, LICENSE). For bug reports and feature requests, open an issue here. For deeper integration questions, contact us at [aria@veriva.dev](mailto:aria@veriva.dev).

## Links

- [veriva.dev](https://veriva.dev)
- [Documentation](https://veriva.dev/docs/github-action)
- [Veriva CLI](https://www.npmjs.com/package/@veriva/cli)

## License

Apache-2.0. See [LICENSE](./LICENSE).
