---
name: c4-review
description: Review C4 architecture models for notation compliance, completeness, and common mistakes. Use when reviewing architecture PRs, auditing existing models, validating model quality, or before publishing architecture documentation.
license: MIT
metadata:
  author: "Georges Martin <jrjsmrtn@gmail.com>"
  version: "0.1.4"
---

# C4 Review

Review C4 architecture models for notation compliance, completeness, and common mistakes.

## When to Use

- Reviewing a PR that modifies `architecture/workspace.dsl`
- Auditing an existing C4 model for quality
- Before publishing or sharing architecture diagrams
- After significant model changes to catch regressions
- When onboarding to understand if an existing model is trustworthy

## Required Inputs

1. Path to the Structurizr DSL workspace file (typically `architecture/workspace.dsl`)
2. Optionally: the project's technology stack and deployment context for completeness checks

## Workflow

### Phase 1: Syntax Validation

Run structurizr/structurizr to catch syntax errors before manual review:

```bash
podman run --rm \
  -v "$(pwd)/architecture:/usr/local/structurizr" \
  structurizr/structurizr validate -workspace workspace.dsl
```

If validation fails, fix syntax errors first. Do not proceed with review until the model parses cleanly.

### Phase 2: Workspace Inspection

Run the `inspect` subcommand to surface Checkstyle-style violations (missing descriptions, duplicate names, duplicate view keys, duplicate relationship descriptions, etc.):

```bash
podman run --rm \
  -v "$(pwd)/architecture:/usr/local/structurizr" \
  structurizr/structurizr inspect -workspace workspace.dsl -severity error,warning
```

The exit code equals the number of violations. Severity levels can be tuned per element via `structurizr.inspection.*` workspace properties (`ignore`, `info`, `warning`, `error`).

### Phase 3: Element Review (manual)

Check every element in the model against these criteria:

#### People/Actors
- [ ] Every person has a clear, specific name (not "User" unless truly generic)
- [ ] Every person has a description stating their role/goal
- [ ] No duplicate people representing the same role
- [ ] External vs internal actors are distinguished (via tags or naming)

#### Software Systems
- [ ] Your system has a name and description
- [ ] External systems are clearly identified (tagged "External" or similar)
- [ ] Each external system has a description explaining what it provides
- [ ] No system is modeled that should actually be a container (e.g., a database you own)
- [ ] System names are unique and unambiguous

#### Containers
- [ ] Every container represents a **runtime boundary** (separately running process or data store)
- [ ] No libraries, modules, or packages modeled as containers
- [ ] Every container has a technology specified
- [ ] Every container has a meaningful description (not just restating the name)
- [ ] Database containers use the "Database" tag for cylinder shape
- [ ] No containers that should be external systems (services you don't own/deploy)

#### Components (if present)
- [ ] Components represent architecturally significant groupings, not just folder structure
- [ ] Every component has technology specified
- [ ] Components are within a single container (not spanning containers)
- [ ] The component diagram adds genuine value (if not, recommend removing it)

### Phase 4: Relationship Review

Check every relationship:

#### Labels
- [ ] Every relationship has a label describing **intent** (what is communicated, not just "uses")
- [ ] Labels match the arrow direction (source does X **to** destination)
- [ ] No vague labels: flag "Uses", "Connects to", "Interacts with", "Communicates with"

Suggest specific alternatives for vague labels:
| Vague | Ask | Better Example |
|-------|-----|----------------|
| "Uses" | What does it use it for? | "queries order history from" |
| "Connects to" | What data flows? | "sends email notifications via" |
| "Interacts with" | What's the interaction? | "submits search queries to" |
| "Communicates with" | What's communicated? | "publishes order events to" |

#### Technology
- [ ] Inter-container relationships specify protocol/technology (e.g., "JSON/HTTPS", "SQL/TCP", "gRPC", "AMQP")
- [ ] Intra-container relationships (component-to-component) may omit technology
- [ ] Person-to-container relationships specify the interface technology (e.g., "HTTPS", "WebSocket")

#### Direction
- [ ] Arrows point in the direction of dependency or data flow
- [ ] No bidirectional relationships (split into two if needed)
- [ ] Implied relationships make sense (e.g., user → container implies user → system)

### Phase 5: View Review

Check each view:

#### System Context View
- [ ] Exists (required for all projects)
- [ ] Shows the system at center with all users and external systems
- [ ] Uses `include *` or explicitly includes all relevant elements
- [ ] Has a descriptive title
- [ ] Does NOT include internal containers or components
- [ ] Does NOT include technology details (keep it non-technical)

#### Container View
- [ ] Exists (required for all projects)
- [ ] Shows all containers within the system boundary
- [ ] Includes people and external systems that interact with containers
- [ ] Has a descriptive title
- [ ] Does NOT include deployment details (clustering, load balancing, replication)
- [ ] Does NOT show components (those belong in component views)

#### Component View (if present)
- [ ] Scoped to a single container
- [ ] Only created for containers where internal structure matters
- [ ] Not just mirroring the code's package structure

#### Dynamic View (if present)
- [ ] Clearly scoped to a specific feature or use case
- [ ] Interactions are numbered in sequence
- [ ] Includes only elements involved in the specific flow

#### Deployment View (if present)
- [ ] Scoped to a specific environment (production, staging, etc.)
- [ ] Shows deployment nodes, not just containers
- [ ] Infrastructure nodes (load balancers, DNS, firewalls) are included where relevant

### Phase 6: Style Review

- [ ] People use `shape Person`
- [ ] Databases use `shape Cylinder`
- [ ] Colors are consistent across diagram types
- [ ] External elements are visually distinct from internal ones
- [ ] Tags are used for styling (not hardcoded per element)
- [ ] Consider color blindness — don't rely on color alone to convey meaning

### Phase 7: Completeness Check

- [ ] All known users/personas are represented
- [ ] All external system integrations are modeled
- [ ] All major containers are present (web app, API, database, queues, etc.)
- [ ] Relationships cover all known communication paths
- [ ] No orphaned elements (elements with no relationships)
- [ ] System context and container views exist at minimum

### Phase 8: Structural Issues

- [ ] `!identifiers hierarchical` is used (recommended for non-trivial models)
- [ ] Identifiers are descriptive (`api`, `database`, not `c1`, `c2`)
- [ ] Tags are used consistently for element types
- [ ] Groups are used for logical organization where helpful
- [ ] `!adrs` directive points to ADR directory (if ADRs exist)
- [ ] `!docs` directive points to documentation (if applicable)

## Output Format

Produce a review summary with:

1. **Validation result**: Pass/fail from structurizr/structurizr `validate`
2. **Inspection violations**: Count and categorization from structurizr/structurizr `inspect`
3. **Issues found**: Categorized as:
   - **Error**: Incorrect abstraction, missing required elements, wrong notation
   - **Warning**: Vague labels, missing descriptions, potential confusion
   - **Suggestion**: Style improvements, optional views, organizational improvements
4. **Completeness assessment**: What's missing relative to the known system
5. **Positive observations**: What the model does well (helps calibrate quality)

## Common Anti-Patterns

### The "Everything Uses Everything" Model
- Too many relationships with vague labels
- Fix: remove implied relationships, make labels specific

### The "Code Structure Mirror"
- Components that exactly match package/module structure
- Fix: model architecturally significant groupings, not code organization

### The "Missing Middle"
- System context exists but no container view, jumping straight to components
- Fix: always create the container view

### The "Stale Component Diagram"
- Component diagram that hasn't been updated as code evolved
- Fix: delete it or regenerate it; stale diagrams are worse than none

### The "Docker Confusion"
- Modeling Docker containers as C4 containers, or modeling C4 containers as Docker containers
- Fix: C4 containers are runtime boundaries; Docker is a deployment mechanism (belongs in deployment view)

### The "Technology Soup"
- Container descriptions that are just technology names ("Elixir GenServer Phoenix")
- Fix: descriptions should state responsibilities ("Handles user authentication and session management")

### The "GenServer Container"
- Individual GenServers modeled as C4 containers
- Fix: GenServers are BEAM processes within a VM — they're code-level, not runtime boundaries. Model the release as the container.

### The "Supervision Tree Diagram"
- Component diagram that mirrors the supervision tree structure
- Fix: components should represent functional groupings (Accounts, Orders, Notifications), not supervision hierarchies (Application → Supervisor → Worker)

### The "Umbrella = Microservices" Model
- Each umbrella app modeled as a separate C4 container or even software system
- Fix: if the umbrella deploys as one release, it's one container. Model umbrella apps as components within that container. Only split into containers if they deploy as separate releases.

## Erlang/Elixir/OTP Review Checklist

When reviewing a C4 model for an Elixir/OTP system, check these additional items:

### Abstraction Correctness
- [ ] OTP releases are modeled as containers (not individual applications or GenServers)
- [ ] GenServers, Agents, Tasks are NOT modeled as containers or components (unless architecturally significant)
- [ ] Supervisor trees are NOT reflected in the component structure
- [ ] Umbrella apps deployed as a single release are modeled as ONE container with components, not multiple containers
- [ ] Phoenix PubSub is NOT modeled as a separate message queue container
- [ ] ETS tables are NOT modeled as separate database containers (they're in-process)

### Container Review
- [ ] Phoenix application container specifies "Elixir/Phoenix" as technology, not just "Elixir"
- [ ] If Oban is embedded in the Phoenix release, it's a component, not a separate container
- [ ] If Broadway is embedded, it's a component, not a separate container
- [ ] Mnesia is only a separate container if it's architecturally significant and spans nodes
- [ ] Nerves firmware is modeled as a container, not a software system

### Relationship Review
- [ ] Ecto connections specify "Ecto/TCP" or "SQL/TCP", not just "Ecto"
- [ ] WebSocket connections are labeled (for LiveView, Channels)
- [ ] Erlang distribution between nodes is modeled as a deployment-level relationship, not a container-level one
- [ ] Oban job queues via PostgreSQL are labeled "Oban/PostgreSQL", not just "PostgreSQL"

### Component Review (if present)
- [ ] Components represent functional domains (Accounts, Orders, Notifications), not OTP applications or supervision trees
- [ ] Phoenix contexts or Ash domains map naturally to components (they're the intended grouping mechanism)
- [ ] Ecto.Repo is either a component ("Data Access") or omitted — not a separate container
- [ ] Technology labels use Elixir ecosystem terms: "Phoenix", "Ecto", "Oban", "Broadway", "Swoosh", "LiveView", "Ash Domain", "AshJsonApi", "AshGraphql"

### Ash Framework Review (if applicable)
- [ ] Ash domains are modeled as components, not containers (they're logical boundaries within the release)
- [ ] Individual Ash resources are NOT modeled as components (they're code-level, like classes)
- [ ] No separate "Data Access" or "Repo" component — Ash handles persistence internally
- [ ] AshJsonApi / AshGraphql are reflected in the API component technology, not as separate containers
- [ ] Ash domain components don't duplicate the resource list — describe domain responsibility instead

### Deployment Review (if present)
- [ ] BEAM VM is modeled as a deployment node hosting the release
- [ ] Erlang distribution / clustering is shown between BEAM nodes
- [ ] EPMD or DNS-based discovery is an infrastructure node (if architecturally relevant)
- [ ] Hot code upgrades, if used, are noted in deployment node properties

## Related Skills

- `c4-model` — C4 abstractions and diagram types
- `structurizr-dsl` — DSL syntax reference
- `c4-deployment` — Deployment modeling specifics
