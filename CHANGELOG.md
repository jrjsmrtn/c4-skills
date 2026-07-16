# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Community health files: `SECURITY.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md` (drafted, not yet tracked).

## [0.1.7] - 2026-07-16

### Added

- Documented the `structurizr/structurizr inspect` subcommand (lint-by-severity;
  exit code equals the number of violations) alongside `validate` in the Validation
  sections of `structurizr-dsl`, `c4-model`, and `c4-deployment` (`c4-review` already
  covered it in Phase 2).

### Changed

- Normalised the Structurizr `inspect` severity flag `-severity` → `-s`/`--severity`
  in `c4-review` to match the verified vNext CLI (`structurizr v2026.03.06`; both
  forms parse, `-s`/`--severity` is canonical).

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

The current version is `0.1.7` (see `.claude-plugin/plugin.json`).
