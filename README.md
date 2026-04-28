<!-- SPDX-FileCopyrightText: 2026 Georges Martin <jrjsmrtn@gmail.com> -->
<!-- SPDX-License-Identifier: MIT -->

# C4 Architecture Skills

A Claude Code plugin providing skills for C4 architecture modeling and Structurizr DSL.

## Installation

Install via the [jrjsmrtn-skills](https://github.com/jrjsmrtn/jrjsmrtn-skills) marketplace:

```
/plugin install github:jrjsmrtn/jrjsmrtn-skills
```

## Included Skills

| Skill | Description |
|-------|-------------|
| **c4-model** | Create and evolve C4 architecture models — abstractions, diagram types, notation rules |
| **c4-review** | Review models for notation compliance, completeness, and common mistakes (7-phase checklist) |
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
