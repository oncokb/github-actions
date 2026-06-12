# Create Release GitHub Action

Use this composite action to create or update a GitHub release draft from a
shared release-note definition stored in this repo. The action reads
`create-release-action/release-labels.json` from the same action repo commit/ref
the caller used, infers the next version from merged PR labels, builds
categorized release notes, and creates or updates the release through the GitHub
API.

The caller repository does not need `.github/release-drafter.yml`.

## Usage

```yaml
name: After master commit

on:
  push:
    branches:
      - master
  workflow_dispatch:
    inputs:
      version-level:
        description: "Release version level"
        required: false
        type: choice
        options:
          - major
          - minor
          - patch

permissions:
  contents: write
  pull-requests: read
  issues: read

jobs:
  create-release:
    if: github.repository == 'oncokb/oncokb-public'
    runs-on: ubuntu-latest
    steps:
      - name: Create release
        uses: oncokb/github-actions/create-release-action@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          version-level: ${{ github.event.inputs['version-level'] }}
```

## Key Inputs

| Input | Required | Description |
| --- | --- | --- |
| `github-token` | Yes | Token with permission to create or update releases. |
| `version-level` | No | Explicit release version level for manual runs. Supports `major`, `minor`, and `patch`. Leave empty to infer from merged PR labels. |
| `default-version-level` | No | Version level to use when no merged PR labels match. Defaults to `patch`. |
| `target-branch` | No | Target branch or commitish for the release. Defaults to the repository default branch. |
| `publish` | No | Set to `true` to publish the release instead of leaving it as a draft. |

## Version Level

If `version-level` is provided, the action uses it directly. This is useful for
manual `workflow_dispatch` runs.

If `version-level` is empty, the action inspects merged pull requests after the
latest published GitHub release. Version precedence is:

1. category labels with `versionLevel: major`
2. category labels with `versionLevel: minor`
3. category labels with `versionLevel: patch`

If no previous release exists, all merged pull requests are inspected. If no
matching labels are found, the action uses `default-version-level`.

## Release Notes

Release note categories and accepted labels are defined in
`create-release-action/release-labels.json`. Updating that one file changes both
release-note generation and PR label validation.

The generated release body uses:

```markdown
## Changes

### <category title>
- <PR title> @<author> (#<number>)

## 🕵️‍♀️ Full commit logs

- https://github.com/<caller-repo>/compare/<previous-tag>...<next-version>
```
