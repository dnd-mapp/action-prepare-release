# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- The action. It checks that the head of the base branch is checked out, prepares the release with `changelog release`, and commits the changelog and the manifest to a `chore/release-X.Y.Z` branch through the API with a GitHub App token. It then opens the `chore: release X.Y.Z` pull request with the release notes as its body, and turns on auto-merge. It outputs `version` and `pull-request-url`.

[Unreleased]: https://github.com/dnd-mapp/action-prepare-release/commits/main
