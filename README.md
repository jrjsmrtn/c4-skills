<!-- SPDX-FileCopyrightText: 2026 Georges Martin <jrjsmrtn@gmail.com> -->
<!-- SPDX-License-Identifier: MIT -->

# C4 Architecture Skills

A Claude Code plugin providing skills for C4 architecture modeling and Structurizr DSL.

## Installation

### Claude Code

Install via the [jrjsmrtn-skills](https://github.com/jrjsmrtn/jrjsmrtn-skills) marketplace:

```
/plugin install github:jrjsmrtn/jrjsmrtn-skills
```

### Other agents (Copilot CLI, Cursor, Codex, Gemini CLI, Antigravity, Goose, …)

Use the GitHub CLI (`gh` ≥ 2.90.0) — it auto-detects the agent host and installs into the right skills directory:

```
gh skill install jrjsmrtn/c4-skills
```

Pin a single skill or version: `gh skill install jrjsmrtn/c4-skills <skill-name>[@v<version>]`. Update later with `gh skill update --all`.

For [Mistral Vibe](https://mistral.ai/products/vibe) (not yet in `gh skill`'s host detection), pass `--dir`:

```
gh skill install jrjsmrtn/c4-skills --dir ~/.vibe/skills
```

## Included Skills

| Skill | Description |
|-------|-------------|
| **c4-model** | Create and evolve C4 architecture models — abstractions, diagram types, notation rules |
| **c4-review** | Review models for notation compliance, completeness, and common mistakes (8-phase checklist incl. `validate` + `inspect`) |
| **structurizr-dsl** | Structurizr DSL syntax reference — elements, views, styles, directives, multi-workspace patterns |
| **c4-deployment** | Model deployment environments — nodes, instances, cloud and on-prem patterns |

All four skills include Elixir/OTP-specific guidance: OTP concepts mapped to C4 abstractions, Phoenix/LiveView/Ecto patterns, Ash Framework domain modeling, Oban/Broadway placement, umbrella projects, BEAM clustering, and deployment patterns.

## Usage

After installation, invoke skills using slash commands:

```
/c4-model
/c4-review
/structurizr-dsl
/c4-deployment
```

Or reference them naturally in conversation — Claude will activate the appropriate skill based on context.

## Companion Plugin

[AI-Assisted Project Orchestration Skills](https://github.com/jrjsmrtn/project-orchestration-skills) handles initial project setup (`setup-architecture-as-code` creates the directory structure, symlinks, and Makefile targets). This plugin focuses on the modeling work itself.

## License

MIT
