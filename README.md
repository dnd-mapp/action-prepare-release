# dnd-mapp/action-prepare-release

[![push main](https://github.com/dnd-mapp/action-prepare-release/actions/workflows/push-main.yaml/badge.svg?branch=main)](https://github.com/dnd-mapp/action-prepare-release/actions/workflows/push-main.yaml)
[![license](https://img.shields.io/github/license/dnd-mapp/action-prepare-release)](LICENSE)

Composite GitHub Action that opens the `chore: release X.Y.Z` pull request of the next release, with `CHANGELOG.md` and `package.json` prepared.

The D&D Mapp packages are released from a tag push, after their release pull request merges. A maintainer starts this action from a manual workflow with the part of the version to bump. It opens the release pull request, which then goes through review and CI like any other. See [The prepare release workflow](#the-prepare-release-workflow).

## Requirements

- A workflow that a maintainer starts by hand, with `workflow_dispatch`, on the base branch.
- A checkout of the head of the base branch, which is what `actions/checkout` gives a run on that branch.
- `@dnd-mapp/changelog-tools` 1.1.0 or later as a dev dependency of the repository, installed before the action runs. The action calls `pnpm exec changelog`.
- A GitHub App that is installed on the repository with read and write access to contents and pull requests. Its client ID and private key are inputs of the action.
- Auto-merge allowed in the settings of the repository.
- A Linux runner, because the action uses GNU `base64`, `jq`, and the `gh` CLI of the runner image.

## Usage

Pin the action to a commit SHA and note the version in a comment, like the third-party actions in the D&D Mapp workflows. Take the SHA from the commit that the release tag points to.

```yaml
- name: Open the release pull request
  uses: dnd-mapp/action-prepare-release@<commit-sha> # v1.0.0
  with:
      bump: ${{ inputs.bump }}
      client-id: ${{ vars.GH_APP_CLIENT_ID }}
      private-key: ${{ secrets.GH_APP_PRIVATE_KEY }}
```

Run it after the dependencies are installed.

## What it does

1. Checks that the checked out commit is the head of `<base-branch>` on the remote. The release branch starts from that commit, so the pull request is up to date when it opens.
2. Runs `changelog release --bump <bump>`, which edits the changelog and the manifest in place. It fails without changes when the changelog cannot be released, for example when `[Unreleased]` is empty. See [Preparing a release](https://github.com/dnd-mapp/changelog-tools#preparing-a-release) for every check.
3. Reads the new version from the manifest, and writes its release notes with `changelog notes`.
4. Creates a token for the GitHub App, narrowed to writing contents and pull requests.
5. Fails when the `chore/release-X.Y.Z` branch already exists. Otherwise it creates the branch from the checked out commit.
6. Commits the changelog and the manifest to the branch with the GraphQL `createCommitOnBranch` mutation. GitHub signs a commit made this way, so it passes a rule that requires signed commits. If the commit fails, the action deletes the branch again.
7. Opens the `chore: release X.Y.Z` pull request with the release notes as its body, and turns on auto-merge with a merge commit. Auto-merge is pinned to the release commit, and the action never merges the pull request itself.

## Inputs

| Input         | Default        | Description                                                                           |
|:--------------|:---------------|:--------------------------------------------------------------------------------------|
| `bump`        | (required)     | Part of the version to bump, one of `major`, `minor`, or `patch`                      |
| `client-id`   | (required)     | Client ID of the GitHub App that creates the branch, the commit, and the pull request |
| `private-key` | (required)     | Private key of the GitHub App                                                         |
| `changelog`   | `CHANGELOG.md` | Path to the changelog, relative to the repository root                                |
| `manifest`    | `package.json` | Path to the package manifest, relative to the repository root                         |
| `base-branch` | `main`         | Branch that the release pull request targets, and whose head must be checked out      |

## Outputs

| Output             | Description                                         |
|:-------------------|:----------------------------------------------------|
| `version`          | The version of the release, without the leading `v` |
| `pull-request-url` | URL of the release pull request                     |

## The prepare release workflow

Every package repository has this `prepare-release.yaml`. Start it from the Actions tab, or with `gh workflow run prepare-release.yaml -f bump=minor`.

```yaml
name: Prepare release

on:
    workflow_dispatch:
        inputs:
            bump:
                description: Part of the version to bump
                required: true
                type: choice
                options:
                    - patch
                    - minor
                    - major

permissions: {}

jobs:
    prepare:
        name: Prepare release
        runs-on: ubuntu-26.04
        timeout-minutes: 5
        permissions:
            contents: read
        concurrency:
            group: ${{ github.workflow }}
            cancel-in-progress: false
        steps:
            - name: Checkout repository
              uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

            - name: Set up Node.js and pnpm and install dependencies
              uses: pnpm/setup@703c52620218391530e48b9e8870d5c0082e1b9b # v2.1.0
              with:
                  cache: true
                  runtime: node
                  require-lockfile: true

            - name: Open the release pull request
              uses: dnd-mapp/action-prepare-release@<commit-sha> # v1.0.0
              with:
                  bump: ${{ inputs.bump }}
                  client-id: ${{ vars.GH_APP_CLIENT_ID }}
                  private-key: ${{ secrets.GH_APP_PRIVATE_KEY }}
```

After the pull request merges, a maintainer creates the annotated tag `vX.Y.Z` on the merge commit and pushes it. That starts the release workflow of the repository.

### Why it looks like this

- A GitHub App token and not `GITHUB_TOKEN`, because a pull request opened with `GITHUB_TOKEN` does not start the pull request workflow. The required `CI` check would then never run.
- The commit goes through the API and not `git push`, because GitHub signs it. The runner holds no signing key.
- `contents: read` for `GITHUB_TOKEN` only, for the checkout. The app token does all the writing, and it exists only inside the action.
- A composite action cannot read secrets, so the workflow passes the app credentials as inputs.
- A bump and not a version as the input, so a release can never skip or repeat a version. The release date is the UTC date of the run.
- One run at a time through `concurrency`, so two runs cannot race for the same version.

### One-time setup

The D&D Mapp organization shares the credentials of its GitHub App with the repositories that use this action.

1. Install the app on the repository.
2. Share the `GH_APP_CLIENT_ID` organization variable with the repository.
3. Share the `GH_APP_PRIVATE_KEY` organization secret with the repository.

## Versioning

This repository is released with `vX.Y.Z` tags and GitHub Releases, like the packages. It prepares its own releases with its [prepare release workflow](.github/workflows/prepare-release.yaml), which runs the action from the checkout. Consumers pin a commit SHA, so a new release never changes a workflow until the pin is updated. Renaming or removing an input or output, or changing a default, is a breaking change.

## Changelog

Notable changes for consumers of this action are listed in the [changelog](CHANGELOG.md).

## Contributing

Contributions are welcome. See the [contributing guide](CONTRIBUTING.md) for details.

## License

[MIT](LICENSE) © D&D Mapp
