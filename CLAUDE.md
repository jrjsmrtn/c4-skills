# CLAUDE.md

## Overview

Claude Code plugin providing C4 architecture modeling and Structurizr DSL skills. These skills support the ongoing work of creating, evolving, reviewing, and documenting software architecture using the C4 model and Structurizr DSL.

## Skills

| Skill | Purpose |
|-------|---------|
| `c4-model` | Create and evolve C4 architecture models — abstractions, diagram types, notation rules, modeling decisions |
| `c4-review` | Review models for notation compliance, completeness, and common mistakes (7-phase checklist) |
| `structurizr-dsl` | Structurizr DSL syntax reference — elements, views, styles, expressions, directives, patterns |
| `c4-deployment` | Model deployment environments — deployment/infrastructure nodes, instances, cloud and on-prem patterns |

All four skills include Erlang/Elixir/OTP-specific guidance: mapping OTP concepts (releases, applications, GenServers, supervisors) to C4 abstractions, Phoenix/LiveView/Ecto patterns, Ash Framework domain modeling, Oban/Broadway placement, umbrella projects, BEAM clustering, and deployment patterns (systemd, Kubernetes, Fly.io).

## Plugin Structure

```
c4-skills/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── c4-model/          # C4 abstractions, diagram types, OTP mapping
│   ├── c4-review/         # Notation compliance, OTP-specific checklist
│   ├── structurizr-dsl/   # DSL syntax, Phoenix/Ash workspace templates
│   └── c4-deployment/     # Deployment patterns incl. BEAM/Fly.io/K8s
├── CLAUDE.md
└── .gitignore
```

## Relationship to Other Plugins

- `project-orchestration-skills/setup-architecture-as-code` handles initial project setup (directory structure, Makefile targets, README)
- This plugin focuses on the modeling work itself: creating elements, writing DSL, reviewing models, modeling deployments

## Conventions

- Skills run in the target project's working directory
- Structurizr DSL files are expected in an `architecture/` directory
- Validation uses `structurizr/cli` container via podman
- Visualization uses `structurizr/lite` container via podman
