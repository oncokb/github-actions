# Check PR Label GitHub Action

Use this composite action to require pull requests to include one of the release
category labels: `major`, `fix`, `feature`, or `documentation`.

## Usage

```yaml
name: Check PR Label

on:
  pull_request:
    types: [opened, labeled, unlabeled, synchronize]

jobs:
  check-label:
    runs-on: ubuntu-latest

    steps:
      - name: Validate required label
        uses: oncokb/github-actions/check-pr-label-action@main
```

## Required Labels

The action reads labels directly from the `pull_request` event payload. It does
not require any inputs.
