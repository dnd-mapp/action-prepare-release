# Agent instructions

## Project

This repository is the composite GitHub Action `dnd-mapp/action-prepare-release`. The action in `action.yaml` prepares the next release with the `changelog` bin from `@dnd-mapp/changelog-tools`, and opens its `chore: release X.Y.Z` pull request with a GitHub App token. Read the [shared contributing guide](https://github.com/dnd-mapp/.github/blob/main/CONTRIBUTING.md) for the conventions that every D&D Mapp repository follows, and [docs/contributing/README.md](docs/contributing/README.md) for the layout, the checks, and the release steps of this repository.

- Keep the action to opening the release pull request. Tagging, staging the package, and creating the GitHub Release belong elsewhere.
- Commit through the GraphQL `createCommitOnBranch` mutation, never with `git push`, so GitHub signs the commit for the signed-commits rule.
- Run `format-check`, `lint-md`, and `actionlint` before you commit.
