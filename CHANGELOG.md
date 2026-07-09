# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Community health files: `SECURITY.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`.

## [0.1.6] - 2026-07-10

### Fixed

- Portfolio-review quality pass: corrected the Structurizr CLI flag `-workspace` → `-w` for the
  consolidated `structurizr/structurizr` (vNext) image and mounted the project root with `:z`
  (`c4-review`, `structurizr-dsl`); added one-line `structurizr/structurizr validate` Validation
  pointers to the C4 reference skills; portability polish.

---

Releases prior to this changelog are recorded in the repository's git history.
Run `git tag --sort=-creatordate` to list released versions and
`git log <previous-tag>..<tag>` to see what changed in each.

The current version is `0.1.6` (see `.claude-plugin/plugin.json`).
