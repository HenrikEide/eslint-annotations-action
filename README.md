# ESLint Annotate from Report JSON


## Description

Fork of [ESLint Annotate from Report JSON](https://github.com/marketplace/actions/eslint-annotate-from-report-json)

## Inputs

| Name | Description | Required | Default Value |
|---|---|---|---|
| `GITHUB_TOKEN` | The [`GITHUB_TOKEN` secret](https://docs.github.com/en/actions/configuring-and-managing-workflows/authenticating-with-the-github_token#about-the-github_token-secret) | No | `${{ github.token }}` |
| `report-json` | Path or [glob pattern](https://github.com/actions/toolkit/tree/master/packages/glob) to locate the ESLint report JSON file. Use multiple lines to specify multiple glob patterns. | No | `eslint_report.json` |
| `only-pr-files` | Only annotate files changed when run on the `pull_request` event | No | `true` |
| `fail-on-warning` | Fail the GitHub Action when ESLint warnings are detected. Set to `true` to enable. | No | `false` |
| `fail-on-error` | Whether to fail the Github action when ESLint errors are detected. If set to false, the check that is created will still fail on ESLint errors. | No | `true` |
| `check-name` | The name of the GitHub status check created. | No | `ESLint Report Analysis` |
| `markdown-report-on-step-summary` | Whether to show a markdown report in the step summary. | No | `false` |

## Outputs

| Name | Description |
|---|---|
| `summary` | A short description of the error and warning count |
| `errorCount` | The amount of errors ESLint reported on |
| `warningCount` | The amount of warnings ESLint reported on |

## Usage Example

In `.github/workflows/nodejs.yml`:

```yml
name: Example NodeJS Workflow

on: [pull_request]

jobs:
  node_test:
    permissions:
      # Default permissions (matching what would be set if the permissions section was missing at all)
      contents: read
      packages: read

      # Need to add these 2 for eslint-annotate-action
      pull-requests: read
      checks: write
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - name: Install Node Dependencies
        run: npm ci
        env:
          CI: TRUE
      - name: Test Code Linting
        run: npm run lint
      - name: Save Code Linting Report JSON
        # npm script for ESLint
        # eslint --output-file eslint_report.json --format json src
        # See https://eslint.org/docs/user-guide/command-line-interface#options
        run: npm run lint:report
        # Continue to the next step even if this fails
        continue-on-error: true
      - name: Annotate Code Linting Results
        uses: ataylorme/eslint-annotate-action@v3
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          report-json: "eslint_report.json"
      # OPTIONAL: save a copy of the usage report for download or use in another job
      # - name: Upload ESLint report
      #   uses: actions/upload-artifact@v4
      #   with:
      #     name: eslint_report.json
      #     path: eslint_report.json
      #     retention-days: 5
```
