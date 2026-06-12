# Create Release GitHub Action

Use this composite action to prepare a Release Drafter release from a shared
workflow. The action checks out the caller repository, generates the shared
Release Drafter config from `release-labels.json`, looks at merged pull request
labels since the latest GitHub release, aligns the template to use
`NEXT_MAJOR_VERSION`, `NEXT_MINOR_VERSION`, or `NEXT_PATCH_VERSION`, commits
that config change when needed, and then runs Release Drafter to create or
update the release draft.

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

| Input                                                                                       | Required | Description                                                                                                                         |
| ------------------------------------------------------------------------------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `github-token`                                                                              | Yes      | Token with permission to push commits and create or update the release draft.                                                       |
| `version-level`                                                                             | No       | Explicit release version level for manual runs. Supports `major`, `minor`, and `patch`. Leave empty to infer from merged PR labels. |
| `release-drafter-config-path`                                                               | No       | Path where the shared Release Drafter config should be written. Defaults to `.github/release-drafter.yml`.                          |
| `release-drafter-config-name`                                                               | No       | Config name passed to Release Drafter. Defaults to `release-drafter.yml`.                                                           |
| `release-drafter-major-token`, `release-drafter-minor-token`, `release-drafter-patch-token` | No       | Placeholder values used in the Release Drafter config. Defaults match the OncoKB release config.                                    |
| `default-version-level`                                                                     | No       | Version level to use when no merged PR labels match. Defaults to `patch`.                                                           |
| `git-user-name`, `git-user-email`                                                           | No       | Git identity used when committing the config update.                                                                                |
| `target-branch`                                                                             | No       | Branch to push config updates to. Defaults to the current ref name.                                                                 |
| `run-release-drafter`                                                                       | No       | Set to `false` to only align and commit the Release Drafter config.                                                                 |
| `publish`                                                                                   | No       | Set to `true` to have Release Drafter publish the release instead of leaving it as a draft.                                         |

## Version Level

If `version-level` is provided, the action uses it directly. This is useful for
manual `workflow_dispatch` runs.

If `version-level` is empty, the action inspects merged pull requests after the
latest published GitHub release. Version precedence is:

1. labels listed in `majorLabels`
2. category labels with `versionLevel: minor`
3. category labels with `versionLevel: patch`

If no previous release exists, all merged pull requests are inspected. If no
matching labels are found, the action uses `default-version-level`.

The label mapping is fixed by the root-level `release-labels.json` file so
release inference stays in sync with the shared Release Drafter categories.

## Release Drafter Config

The action generates the shared Release Drafter config from `release-labels.json`
and writes it into the caller repository before Release Drafter runs:

```yaml
name-template: "$NEXT_MINOR_VERSION"
tag-template: "$NEXT_MINOR_VERSION"
categories:
  # Generated from release-labels.json
change-template: "- $TITLE @$AUTHOR (#$NUMBER)"
template: |
  ## Changes

  $CHANGES

  ## 🕵️‍♀️ Full commit logs

  - https://github.com/<caller-repo>/compare/$PREVIOUS_TAG...$NEXT_MINOR_VERSION
```

When the inferred level is `major`, the action rewrites the config to use
`NEXT_MAJOR_VERSION`. `minor` uses `NEXT_MINOR_VERSION`, and `patch` uses
`NEXT_PATCH_VERSION`.
