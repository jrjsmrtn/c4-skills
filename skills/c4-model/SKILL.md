---
# SPDX-FileCopyrightText: 2025 Georges Martin <jrjsmrtn@gmail.com>
# SPDX-License-Identifier: MIT
name: c4-model
description: Create and evolve C4 architecture models with proper abstractions, diagram types, and notation. Use when adding elements to a C4 model, choosing diagram types, modeling relationships, or making architecture modeling decisions.
---

# C4 Model

Create and evolve C4 architecture models with correct abstractions, diagram types, and notation.

## When to Use

- Adding new elements (people, systems, containers, components) to a C4 model
- Deciding which diagram type to create for a given audience or purpose
- Modeling relationships between elements
- Evolving an existing model as the system grows
- Choosing the right level of abstraction for a diagram

## C4 Abstractions

The C4 model defines four levels of abstraction. Each has a precise meaning — do not conflate them.

### Level 1: Software System

- The highest level. Delivers value to its users (human or not).
- Typically owned by a single team, deployed together, lives in one repo.
- Is NOT: a product domain, bounded context, business capability, or team structure.

### Level 2: Container

- A separately running process or data store required for the system to work.
- A **runtime boundary** — not a code-organization unit.
- Examples: server-side web app, SPA (client-side), mobile app, database, message queue, serverless function, shell script, file/blob storage.
- **NOT containers**: JARs, DLLs, assemblies, libraries, modules — these are code-level constructs.
- **NOT Docker containers** — the name is coincidental.
- A single-page app is TWO containers: the server delivering the static files, and the client-side app running in the browser.
- Cloud services you own (S3 buckets, RDS instances) are containers, not external systems.

### Level 3: Component

- A grouping of related functionality behind a well-defined interface, within a single container.
- All components share the same process space — they are NOT separately deployable.
- Language-dependent: classes (OOP), modules (JS/Elixir), grouped functions (FP), files in a directory (C).
- NOT the same as packages, namespaces, or folder structures.
- Focus on architecturally significant groupings. Exclude utility classes and data models initially.

### Level 4: Code

- Classes, interfaces, functions, database tables.
- Almost always generated from code, never maintained manually.

## Diagram Types

### Core Diagrams (the "4 Cs")

| Diagram | Scope | Shows | Audience | Recommended? |
|---------|-------|-------|----------|-------------|
| System Context | One software system | System + users + external systems | Everyone | Yes, always |
| Container | One software system | Containers inside the system | Technical staff | Yes, always |
| Component | One container | Components inside the container | Developers | Only if valuable |
| Code | One component | Classes/modules/tables | Developers | No, generate if needed |

### Supporting Diagrams

| Diagram | Scope | Shows | When to Use |
|---------|-------|-------|-------------|
| System Landscape | Enterprise/org | All systems and their connections | Large organizations, enterprise views |
| Dynamic | Feature/use case | Runtime interactions with numbered sequence | Complex interaction flows, specific scenarios |
| Deployment | One+ systems in one environment | Infrastructure, nodes, deployed instances | Infrastructure documentation, ops audience |

### Diagram Selection Guide

**"Who is the audience?"**
- Everyone (including business) → System Context or System Landscape
- Technical staff → Container or Deployment
- Developers → Component (sparingly) or Dynamic

**"What question does the diagram answer?"**
- "What does the system do and who uses it?" → System Context
- "What are the main technical building blocks?" → Container
- "How is container X structured internally?" → Component
- "How does feature Y work at runtime?" → Dynamic
- "Where does it run?" → Deployment
- "What systems exist across the organization?" → System Landscape

## Notation Rules

Every C4 diagram must follow these rules:

### Required Elements
- **Title**: Every diagram has a title stating its type and scope
- **Legend/key**: Every diagram has a key explaining the notation
- **Element types**: Every element explicitly states its type (Person, Software System, Container, Component)
- **Descriptions**: Short descriptions (1-2 sentences) conveying key responsibilities

### Technology Labels
- All containers must specify their technology (e.g., "Python/FastAPI", "PostgreSQL 16")
- All components must specify their technology (e.g., "Elixir GenServer", "React Component")
- Inter-container relationships must include protocol/technology (e.g., "JSON/HTTPS", "JDBC", "gRPC")

### Relationship Lines
- Unidirectional only
- Every line labeled with intent matching the direction
- Avoid vague labels: "Uses" is almost always wrong
- Good: "sends customer update events to", "reads credentials from", "queries"
- Bad: "Uses", "Connects to", "Interacts with"

### Visual Consistency
- Colors must be consistent within and across diagrams
- Consider B&W printing and color blindness
- Acronyms must be understandable to the audience or explained in the legend

## Workflow: Adding Elements to a Model

### Step 1: Identify the Abstraction Level

Before adding anything, determine what you're modeling:
- Is it a separately running process? → Container
- Is it a logical grouping within a process? → Component
- Is it an external system you don't own? → Software System (external)
- Is it a user or role? → Person

### Step 2: Define the Element

For each new element, determine:
1. **Name**: Clear, unambiguous (unique within scope)
2. **Description**: 1-2 sentences on its responsibility
3. **Technology**: What it's built with (containers and components)
4. **Tags**: For styling (e.g., "Database", "Web", "External")

### Step 3: Define Relationships

For each relationship:
1. **Direction**: Source → Destination (who initiates?)
2. **Description**: What is the intent? (not "uses" — be specific)
3. **Technology**: Protocol or mechanism (for inter-container)

### Step 4: Create or Update Views

- Add the element to relevant views using `include`
- Decide if a new view is needed
- Validate with `structurizr/cli`

## Common Modeling Decisions

### When to Split a Container

Split when:
- Parts deploy independently
- Parts scale independently
- Parts use different technology stacks
- Parts have different availability requirements

Do NOT split when:
- It's just different modules in the same process
- You want to show internal structure (use components instead)

### When to Add Components

Add components when:
- The container's internal structure is architecturally significant
- You need to show how responsibilities are distributed
- There are important interfaces between internal parts

Skip components when:
- The container is simple enough to understand from its description
- Components would just mirror the folder structure
- The diagram would have too many elements to be useful

### When to Model External Systems

Model as external software system when:
- You don't own or control it
- It's operated by a different team/organization
- You interact with it through a defined API/protocol

Model as a container within your system when:
- You deploy and manage it (even if it's third-party software like PostgreSQL)
- It's part of your system's infrastructure

## Erlang/Elixir/OTP Modeling Guide

The BEAM runtime has unique characteristics that affect C4 modeling decisions.

### Mapping OTP Concepts to C4 Abstractions

| OTP Concept | C4 Abstraction | Rationale |
|-------------|---------------|-----------|
| OTP Release | Container | A release is a standalone, deployable BEAM instance |
| OTP Application | Component | An application runs within a release, not separately |
| Umbrella app (single release) | Component per app | All apps share one BEAM VM, one deployment unit |
| Umbrella app (separate releases) | Container per release | Each release is a separate runtime boundary |
| GenServer / Agent | Code | Implementation detail, not architecturally significant |
| Supervisor tree | Code | Internal structure of an OTP application |
| Phoenix Endpoint | Component | The HTTP/WebSocket entry point within the release |
| Phoenix LiveView | Component or Container | Container if it's a separately served SPA; component if part of the Phoenix app |
| Ecto Repo | Component | Data access layer within the release |
| Oban | Component or Container | Component if embedded in the release; container if deployed as a separate worker release |
| Broadway | Component | A data pipeline within the release |
| Ash Domain | Component | A domain boundary grouping related resources |
| Ash Resource | Code | An individual resource within an Ash domain |
| Phoenix PubSub | Code | Internal messaging mechanism, not a separate container |
| Erlang Distribution | Relationship technology | Communication between BEAM nodes, label on deployment relationships |
| Mnesia / ETS | Code or Container | ETS is in-process (code); Mnesia across nodes could be modeled as a data store container if architecturally significant |
| Nerves firmware | Container | A deployed runtime on embedded hardware |

### Key Modeling Decisions for OTP

**GenServers are NOT containers.** A GenServer is a process within the BEAM VM. It doesn't have its own deployment boundary. It's a code-level construct — analogous to a class instance in OOP. Only model a GenServer as a component if it represents an architecturally significant subsystem (e.g., a connection pool, a rate limiter, a state machine for a core domain concept).

**Supervisor trees are NOT components.** They're an implementation mechanism for fault tolerance. The C4 component level should model functional groupings ("Authentication", "Order Processing"), not supervision hierarchies.

**One release = one container.** Even if a release contains multiple OTP applications, it's one container in C4 because it deploys as a single unit. Use components to show the internal applications if that structure matters.

**LiveView can go either way.** If Phoenix serves LiveView from the same endpoint, LiveView is a component within the Phoenix container. If you want to emphasize that LiveView provides a distinct real-time UI separate from traditional request/response, you can model it as a separate container — the key question is whether your audience needs to see them as separate building blocks.

**Oban workers: embedded or separate?** If Oban runs within your main Phoenix release (the common case), it's a component. If you deploy a separate release specifically for background job processing (e.g., a worker-only release), it's a separate container.

**Ash domains are components, not containers.** An Ash domain groups related resources and defines their API. It runs within the same BEAM process as everything else — it's a logical boundary, not a runtime one. Model each Ash domain as a component. Individual Ash resources are code-level constructs (like classes) and shouldn't appear in C4 diagrams. When using Ash, domains replace Phoenix contexts as the natural component grouping.

**Ash and data access.** With Ash, the application talks to the database through Ash's data layer, not directly through Ecto queries. The relationship from your container to the database should still say "reads from and writes to" with technology "Ash/Ecto/TCP" or simply "Ecto/TCP" — Ash wraps Ecto but the wire protocol is the same.

### Container-Level Patterns for Elixir Systems

**Typical Phoenix application:**
- Phoenix Application (container) — "Web application and REST API" / "Elixir/Phoenix"
- Database (container) — "Primary data store" / "PostgreSQL 16"
- Cache (container, if separate) — "Session and cache store" / "Redis"

**Phoenix with real-time features:**
- Phoenix Application (container) — includes LiveView, Channels, PubSub
- Relationship: User → Phoenix "interacts with" "HTTPS, WebSocket"

**Event-driven Elixir system:**
- Phoenix Application (container) — API and web interface
- Message Queue (container) — "Event bus" / "RabbitMQ" or "NATS"
- Worker Release (container) — "Event consumers and processors" / "Elixir/Broadway"
- Database (container)

**Nerves IoT system:**
- Firmware (container) — "Embedded application" / "Elixir/Nerves"
- Cloud API (container) — "Device management API" / "Elixir/Phoenix"
- Database (container)
- Firmware → Cloud API "reports telemetry to" "MQTT" or "HTTPS"

### Component-Level Patterns for Elixir Systems

When modeling the internals of a Phoenix release:

**With Phoenix contexts:**

| Component | Description | Technology |
|-----------|-------------|------------|
| Web | HTTP endpoint, controllers, LiveView | Phoenix |
| API | REST/GraphQL API | Phoenix |
| Accounts | User authentication and authorization | Elixir |
| Core | Business logic and domain | Elixir |
| Workers | Background job processing | Oban |
| Ingestion | Data pipeline and ETL | Broadway |
| Notifications | Email, SMS, push notifications | Swoosh |
| Repo | Data access and queries | Ecto |

**With Ash Framework:**

| Component | Description | Technology |
|-----------|-------------|------------|
| Web | HTTP endpoint, controllers, LiveView | Phoenix |
| API | REST/GraphQL API | AshJsonApi / AshGraphql |
| Accounts | User identity, authentication, authorization | Ash Domain |
| Helpdesk | Tickets, assignments, SLAs | Ash Domain |
| Notifications | Email, SMS, push notifications | Ash Domain / Swoosh |
| Workers | Background job processing | Oban |

With Ash, domains replace Phoenix contexts as the component grouping. The Repo/data-access component disappears — Ash resources handle persistence internally through the Ash data layer.

### Relationship Labels for Elixir Systems

Use specific labels, not "uses":

| From → To | Good Label | Technology |
|-----------|-----------|------------|
| User → Phoenix | "browses and interacts with" | "HTTPS, WebSocket" |
| Phoenix → Database | "reads from and writes to" | "Ecto/TCP" |
| Phoenix → Redis | "caches sessions in" | "Redix/TCP" |
| Phoenix → Queue | "publishes domain events to" | "AMQP" |
| Worker → Queue | "consumes jobs from" | "AMQP" or "Oban/PostgreSQL" |
| Phoenix → External API | "fetches exchange rates from" | "HTTP/JSON" |
| LiveView → Phoenix | "connects via" | "WebSocket" |
| Node A → Node B | "replicates state via" | "Erlang Distribution" |

## Common Mistakes

1. **Confusing containers with Docker containers** — C4 containers are runtime boundaries
2. **Treating libraries as containers** — JARs, packages, modules are code-level, not containers
3. **Vague relationship labels** — "Uses" says nothing; describe the intent
4. **Missing technology on inter-container relationships** — always specify protocol
5. **Too many components** — only model architecturally significant groupings
6. **Mixing abstraction levels** — don't show components and external systems at the same level in a container diagram
7. **Creating component/code diagrams that won't be maintained** — if it'll go stale, don't create it
8. **Missing titles or legends** — every diagram needs both
9. **Modeling organizational structure** — C4 models software, not teams
10. **Modeling GenServers as containers** — GenServers are BEAM processes within a VM, not separate runtime boundaries
11. **Modeling supervisor trees as components** — Supervision is a fault-tolerance mechanism, not an architectural grouping
12. **Treating each OTP application as a container** — If they deploy in one release, it's one container with multiple components
13. **Confusing Phoenix PubSub with a message queue** — PubSub is in-process (or across BEAM nodes); it's not a separate container like RabbitMQ

## Diagram Staleness

| Level | Change Rate | Maintenance Effort |
|-------|------------|-------------------|
| System Context | Slow | Low — create and maintain |
| Container | Moderate | Low — create and maintain |
| Component | Fast during development | Medium — create only if valuable |
| Code | Very fast | None — generate on demand, don't maintain |

## Related Skills

- `structurizr-dsl` — DSL syntax for implementing models
- `c4-review` — Review models for correctness
- `c4-deployment` — Model deployment environments
