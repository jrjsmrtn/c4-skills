---
# SPDX-FileCopyrightText: 2025 Georges Martin <jrjsmrtn@gmail.com>
# SPDX-License-Identifier: MIT
name: structurizr-dsl
description: Structurizr DSL syntax reference and patterns for writing C4 architecture models. Use when writing or editing workspace.dsl files, looking up DSL syntax, implementing specific patterns, or troubleshooting DSL errors.
---

# Structurizr DSL

Comprehensive reference for the Structurizr DSL — the text-based language for defining C4 architecture models.

## When to Use

- Writing a new `workspace.dsl` from scratch
- Adding elements, relationships, or views to an existing model
- Looking up syntax for a specific DSL construct
- Implementing patterns (deployment, groups, themes, expressions)
- Troubleshooting parse errors or unexpected behavior

## Syntax Fundamentals

- Lines are processed **sequentially** — no forward references
- Keywords are **case-insensitive** (`softwareSystem` = `softwaresystem`)
- Double quotes are optional for single-word values without spaces
- Opening `{` must be on the **same line** as the statement
- Closing `}` must be on its **own line**
- Comments: `//` or `#` (single-line), `/* ... */` (multi-line)
- Line continuation: `\` at end of line
- Identifier characters: `a-zA-Z_0-9`

### Constants and Variables

```dsl
!const SYSTEM_NAME "My System"        # immutable
!var VERSION "1.0"                     # redefinable
# Reference: ${SYSTEM_NAME}, ${VERSION}
```

## Workspace Structure

```dsl
workspace [name] [description] {
    !identifiers hierarchical          # recommended for non-trivial models
    !impliedRelationships true         # default: auto-creates parent relationships

    model {
        # people, systems, relationships
    }

    views {
        # diagram definitions, styles, themes
    }

    configuration {
        # scope, visibility, users
    }
}
```

## Model Elements

### Person

```dsl
<id> = person <name> [description] [tags]
# or with block:
<id> = person <name> [description] [tags] {
    description "..."
    tags "Tag1" "Tag2"
    url "https://..."
    properties { key value }
    -> <target> [description] [technology] [tags]
}
```

Default tags: `Element`, `Person`

### Software System

```dsl
<id> = softwareSystem <name> [description] [tags] {
    !docs <path>                       # attach markdown documentation
    !adrs <path>                       # attach ADRs
    group <name> { ... }
    <id> = container <name> [description] [technology] [tags] { ... }
    -> <target> [description] [technology] [tags]
}
```

Default tags: `Element`, `Software System`

### Container

```dsl
<id> = container <name> [description] [technology] [tags] {
    !docs <path>
    !adrs <path>
    group <name> { ... }
    <id> = component <name> [description] [technology] [tags] { ... }
    -> <target> [description] [technology] [tags]
}
```

Default tags: `Element`, `Container`

### Component

```dsl
<id> = component <name> [description] [technology] [tags] {
    description "..."
    technology "..."
    tags "..."
    -> <target> [description] [technology] [tags]
}
```

Default tags: `Element`, `Component`

### Group

```dsl
group <name> {
    # elements of the same type
}
```

Style groups with the `Group:Name` tag. For nested groups:

```dsl
model {
    properties {
        "structurizr.groupSeparator" "/"
    }
    group "Organization" {
        group "Department" {
            system = softwareSystem "System"
        }
    }
}
```

## Relationships

```dsl
# Explicit
<source> -> <destination> [description] [technology] [tags]

# Within element block (implicit source = this)
-> <destination> [description] [technology] [tags]

# With properties
<source> -> <destination> [description] [technology] [tags] {
    tags "Async"
    url "https://..."
    properties { key value }
}

# Remove a relationship
<source> -/> <destination>
```

Default tags: `Relationship`

### Uniqueness

Relationships between the same source and destination must have **unique descriptions**. This is the only way the DSL distinguishes multiple relationships between the same pair.

## Identifiers

### Flat (default)

All identifiers are globally scoped. Every identifier must be unique across the entire workspace.

### Hierarchical (recommended)

```dsl
!identifiers hierarchical
```

Identifiers are scoped to their parent. Reference as `parent.child`:

```dsl
system = softwareSystem "System" {
    api = container "API"
    db = container "Database"
}
# Reference: system.api, system.db
```

## Views

### System Landscape

```dsl
systemLandscape [key] [description] {
    include *
    autoLayout [tb|bt|lr|rl] [rankSep] [nodeSep]
    title "Title"
    description "..."
}
```

### System Context

```dsl
systemContext <systemId> [key] [description] {
    include *
    autoLayout
    title "System Context for X"
}
```

### Container View

```dsl
container <systemId> [key] [description] {
    include *
    autoLayout
    title "Containers for X"
}
```

### Component View

```dsl
component <containerId> [key] [description] {
    include *
    autoLayout
    title "Components of X"
}
```

### Dynamic View

```dsl
dynamic <scope> [key] [description] {
    # scope: * (people/systems), systemId (+ containers), containerId (+ components)
    <source> -> <destination> [description] [technology]
    <source> -> <destination> [description] [technology]
    autoLayout
    title "Feature Y Flow"
}
```

Interactions are automatically numbered in order. To set explicit ordering:

```dsl
dynamic <scope> [key] [description] {
    1: user -> system.api "submits order" "HTTPS"
    2: system.api -> system.db "persists order" "SQL"
    3: system.api -> system.queue "publishes event" "AMQP"
}
```

### Deployment View

```dsl
deployment <systemId|*> <environment> [key] [description] {
    include *
    autoLayout
    title "Production Deployment"
}
```

### Filtered View

```dsl
filtered <baseViewKey> <include|exclude> <tags> [key] [description]
```

Creates a view on top of another, filtering elements by tags. The base view is hidden when a filtered view exists.

### Image View

```dsl
image <scope> [key] {
    plantuml <file|url>
    mermaid <file|url>
    image <file|url>
    title "Title"
}
```

### AutoLayout

```dsl
autoLayout [direction] [rankSeparation] [nodeSeparation]
# direction: tb (default), bt, lr, rl
# default separations: 300px each
```

### Include/Exclude with Expressions

#### Element Expressions

```dsl
include element.type==Container
include element.tag==Database
include element.tag!=External
include element.parent==<id>
include element.technology==PostgreSQL
include ->system.api->                 # elements connected to system.api (both directions)
include ->system.api                   # elements that depend on system.api (afferent)
include system.api->                   # elements system.api depends on (efferent)
```

Combine with `&&` (AND) and `||` (OR):

```dsl
include element.type==Container && element.tag!=External
```

#### Relationship Expressions

```dsl
include * -> *                         # all relationships between included elements
include user -> *                      # all relationships from user
include * -> system.db                 # all relationships to database
include relationship.tag==Async
include relationship==system.api->system.db
```

#### Reluctant Wildcard

```dsl
include *?                             # like * but excludes orphaned elements
```

## Styles

### Element Styles

```dsl
styles {
    element <tag> {
        shape <shape>                  # see shapes below
        icon <file|url>
        width <int>
        height <int>
        background <#rrggbb>
        color <#rrggbb>               # text color (alias: colour)
        stroke <#rrggbb>              # border color
        strokeWidth <1-10>
        fontSize <int>
        border <solid|dashed|dotted>
        opacity <0-100>
        metadata <true|false>          # show/hide metadata line
        description <true|false>       # show/hide description
    }
}
```

**Shapes**: `Box`, `RoundedBox`, `Circle`, `Ellipse`, `Hexagon`, `Diamond`, `Cylinder`, `Pipe`, `Person`, `Robot`, `Folder`, `WebBrowser`, `Window`, `Terminal`, `MobileDevicePortrait`, `MobileDeviceLandscape`, `Component`

### Relationship Styles

```dsl
styles {
    relationship <tag> {
        thickness <int>
        color <#rrggbb>
        style <solid|dashed|dotted>
        routing <Direct|Orthogonal|Curved>
        fontSize <int>
        width <int>
        position <0-100>               # label position along line
        opacity <0-100>
    }
}
```

### Styling All Elements/Relationships

Use the default tags:

```dsl
styles {
    element "Element" { ... }          # applies to ALL elements
    relationship "Relationship" { ... } # applies to ALL relationships
}
```

### Light/Dark Mode

```dsl
styles {
    light {
        element "Person" { background #08427B; color #ffffff }
    }
    dark {
        element "Person" { background #1a6bc4; color #ffffff }
    }
}
```

### Standard Style Recipe

```dsl
styles {
    element "Person" {
        shape Person
        background #08427B
        color #ffffff
    }
    element "Software System" {
        background #1168BD
        color #ffffff
    }
    element "External" {
        background #999999
        color #ffffff
    }
    element "Container" {
        background #438DD5
        color #ffffff
    }
    element "Component" {
        background #85BBF0
        color #000000
    }
    element "Database" {
        shape Cylinder
    }
    element "Web" {
        shape WebBrowser
    }
    element "Queue" {
        shape Pipe
    }
    relationship "Relationship" {
        routing Orthogonal
    }
    relationship "Async" {
        style dashed
    }
}
```

## Themes

```dsl
views {
    theme default                      # Structurizr default theme
    # or multiple:
    themes default "https://..." "file.json"
}
```

Built-in cloud themes:
- `amazon-web-services-2020.04.30`
- `microsoft-azure-2021.01.26`
- `google-cloud-platform`
- `kubernetes`

Custom styles override theme defaults.

## Directives

### !include

```dsl
!include <file>                        # inline a DSL fragment
!include <directory>                   # include all .dsl files in directory (alphabetically)
!include <url>                         # include from HTTPS URL
```

Content is **literally inlined** into the parent document. Relative paths resolve from the parent file's directory and **must stay within the same directory or a subdirectory** — `../` paths are not allowed.

Key behaviors:
- **Directory includes are recursive** — subdirectories are processed too
- **Alphabetical ordering** — files sorted with `Arrays.sort`, so use numeric prefixes for ordering control
- **Hidden files skipped** — `.DS_Store` and dotfiles are ignored
- **No deduplication** — the same file included from multiple places is inlined multiple times

**Duplicate identifier restriction**: if an included fragment defines an identifier that the including file also defines, Structurizr errors. There is no merge or override — plan fragment boundaries carefully.

### !docs and !adrs

```dsl
workspace "System" "Description" {
    !adrs docs/adr                     # attach ADRs at workspace level
    !docs docs/architecture             # attach documentation at workspace level
}
```

Can also be scoped to a softwareSystem or container:

```dsl
softwareSystem "System" {
    !docs docs/system-docs
    !adrs docs/adr
}
```

**Path restriction**: same as `!include` — paths must be within or below the DSL file's directory. Use a symlink (`architecture/docs -> ../docs`) if workspace.dsl is in a subdirectory.

#### !docs authoring conventions

**Do not point `!docs` at Diátaxis directories** (`explanation/`, `howto/`, etc.). Structurizr silently strips h1 headings — it auto-generates h1 from the element name. Diátaxis docs use h1 as their title, so those titles would vanish. Use a dedicated `docs/architecture/` directory for Structurizr-specific documentation, separate from Diátaxis content.

Files in the `!docs` directory are imported **alphabetically by filename** — use numeric prefixes for ordering:

```
docs/architecture/
├── 01-context.md
├── 02-containers.md
├── 03-deployment.md
└── images/
    └── overview.png
```

Heading level behavior in Structurizr's documentation renderer:
- **h1 (`#`)** — **ignored/hidden**; auto-generated from the element name
- **h2 (`##`)** — becomes the numbered section title in navigation (e.g., `1 Context`)
- **h3 (`###`)** — sub-sections (e.g., `1.1 Overview`), appear in navigation
- **h4+ (`####`)** — numbered but excluded from navigation

Images in the directory and subdirectories are automatically imported.

#### !adrs format requirements (adrtools default)

- Filename: **4-digit zero-padded** prefix (e.g., `0001-record-architecture-decisions.md`)
- Title line: `# N. Title Text` (number, period, space, title — adrtools format)
- Date: `Date: yyyy-MM-dd` (ISO 8601)
- Status section: `## Status` heading, status on next non-blank line (first word taken)
- Cross-references: `Superseded by [ADR-0005](0005-new-approach.md)` parsed as links between ADRs

**Pitfall**: the adrtools importer processes **every `.md` file** in the directory and expects a 4-digit prefix in the filename. A `README.md` or any non-ADR markdown file in `docs/adr/` will cause a parse error. Use YAML for the ADR index (`docs/adr/index.yml`) — non-`.md` files are ignored. The MADR importer is safer — it filters filenames to `\d{4}-.+?.md` and naturally skips non-conforming files.

ADR importers: `adrtools` (default), `madr`, `log4brains`:

```dsl
!adrs docs/adr madr
```

### !element and !relationship

Modify elements defined elsewhere:

```dsl
!element <identifier> {
    description "Updated description"
    tags "NewTag"
}
```

### Bulk Operations

```dsl
!elements element.tag==Container {
    tags "Monitored"
    properties { "team" "platform" }
}

!relationships relationship.tag==Async {
    tags "EventDriven"
}
```

## Terminology

Customize C4 terminology for your organization:

```dsl
views {
    terminology {
        person "Actor"
        softwareSystem "Application"
        container "Service"
        component "Module"
        deploymentNode "Server"
        relationship "Dependency"
    }
}
```

## Configuration

```dsl
configuration {
    scope <landscape|softwaresystem|none>
    visibility <private|public>
    users {
        "user@example.com" read
        "admin@example.com" write
    }
}
```

## Workspace Extension

Extend a parent workspace:

```dsl
workspace extends parent.dsl {
    model {
        !element existingSystem {
            newContainer = container "New Container" "Added later" "Go"
        }
    }
}
```

## Multiple Workspaces

When a system is large enough that a single workspace becomes unwieldy, split into focused workspaces — one master overview plus per-subsystem detail workspaces.

### Layout

```
architecture/
├── workspace.dsl                    # Master: all subsystems at container level
├── <project>-<subsystem>-workspace.dsl  # Focused: one subsystem at component level
├── shared/
│   └── _styles.dsl                  # Shared styles via !include
└── docs -> ../docs                  # Symlink for !adrs and !docs
```

### How focused workspaces relate to the master

The master workspace models all subsystems as **containers** within one softwareSystem. Each focused workspace **promotes** its subsystem to a top-level softwareSystem with full container/component detail, and declares other subsystems as one-line "Sibling" stubs:

```dsl
# In forge-build-workspace.dsl:
forgeSystem   = softwareSystem "Forge System"   "Requests package builds for Tier 2/3 hosts" "Sibling"
forgeRuntime  = softwareSystem "Forge Runtime"  "Policy engine, secrets, event log" "Sibling"

forgeBuild = softwareSystem "Forge Build" "Pluggable build system" {
    # ... full container/component detail ...
}
```

Sibling descriptions are **intentionally context-specific** — they explain how that subsystem relates to *this* workspace's focus, not a generic description.

### What can be shared via `!include`

| Fragment | Shareable? | Why |
|----------|-----------|-----|
| Styles | Yes | Identical across workspaces, no identifier clashes |
| People | No | Descriptions are context-specific per workspace |
| Sibling stubs | No | Identifier clashes with primary system; descriptions context-specific |
| External systems | No | Descriptions vary by workspace context |

Shared styles pattern:

```dsl
views {
    # ...
    styles {
        !include shared/_styles.dsl
    }
}
```

### Each workspace is self-contained

Every focused workspace must repeat `!adrs docs/adr` and `!docs docs/architecture` — these are per-workspace, not inherited. Structurizr Lite renders one workspace at a time.

### `workspace extends` limitations

`workspace extends parent.dsl` inherits all parent identifiers, but only supports **single-parent** inheritance. Child workspaces CAN override parent styles (merge semantics). It works well when the child uses the same modeling approach as the parent. It does **not** work when focused workspaces promote containers to top-level systems (different abstraction levels).

### Structurizr Lite with multiple workspaces

Structurizr Lite can serve multiple workspaces from numbered subdirectories. Create `structurizr.properties` in the mounted directory:

```properties
structurizr.workspaces=13
```

Then organize workspaces in numbered directories (`1-master/`, `2-build/`, etc.). Alternatively, use `STRUCTURIZR_WORKSPACE_FILENAME` to select a specific workspace file.

### Simon Brown's shared fragment pattern

For multi-domain organizations, share model fragments across independent workspaces:

```
shared/
  model.dsl              # shared model (systems, relationships)
domainA/
  workspace.dsl          # !include ../shared/model.dsl + domain-specific views
domainB/
  workspace.dsl          # !include ../shared/model.dsl + domain-specific views
```

This avoids `workspace extends` single-parent limitation and allows each workspace to compile independently.

## Validation

Always validate after changes:

```bash
# Mount project root so docs symlink resolves
podman run --rm \
  -v "$(pwd):/usr/local/structurizr" \
  -w /usr/local/structurizr/architecture \
  structurizr/cli validate -workspace workspace.dsl
```

## Common Patterns

### Multi-file Organization

```dsl
# workspace.dsl
workspace "System" {
    !identifiers hierarchical
    model {
        !include model/people.dsl
        !include model/systems.dsl
        !include model/relationships.dsl
    }
    views {
        !include views/
    }
}
```

### Tagging for Filtered Views

```dsl
model {
    system = softwareSystem "System" {
        api = container "API" "REST API" "Go" "Service,v2"
        legacyApi = container "Legacy API" "Old REST API" "Java" "Service,v1,Deprecated"
    }
}
views {
    container system "AllContainers" { include * }
    filtered "AllContainers" exclude "Deprecated" "CurrentContainers"
}
```

### Async/Sync Relationship Distinction

```dsl
model {
    system.api -> system.db "queries" "SQL"
    system.api -> system.queue "publishes order events to" "AMQP" "Async"
    system.worker -> system.queue "consumes order events from" "AMQP" "Async"
}
views {
    styles {
        relationship "Async" { style dashed }
    }
}
```

## Elixir/OTP Patterns

### Phoenix Application Workspace

Complete workspace template for a typical Phoenix application:

```dsl
workspace "MyApp" "Phoenix web application" {

    !identifiers hierarchical

    model {
        user = person "User" "Application user"
        admin = person "Administrator" "System administrator"

        system = softwareSystem "MyApp" "Description of what the system does" {

            phoenix = container "Phoenix Application" "Web application, API, and real-time UI" "Elixir/Phoenix" "Web"

            obanWorkers = container "Background Workers" "Processes async jobs" "Elixir/Oban" "Worker"

            postgres = container "PostgreSQL" "Primary data store" "PostgreSQL 16" "Database"

            redis = container "Redis" "Cache and PubSub adapter" "Redis 7" "Database"
        }

        # Relationships
        user -> system.phoenix "browses and interacts with" "HTTPS, WebSocket"
        admin -> system.phoenix "administers via" "HTTPS"
        system.phoenix -> system.postgres "reads from and writes to" "Ecto/TCP"
        system.phoenix -> system.redis "caches sessions in" "Redix/TCP"
        system.obanWorkers -> system.postgres "polls for and updates jobs in" "Ecto/TCP"
    }

    views {
        systemContext system "SystemContext" {
            include *
            autoLayout
        }

        container system "Containers" {
            include *
            autoLayout
        }

        styles {
            element "Person" {
                shape Person
                background #08427B
                color #ffffff
            }
            element "Software System" {
                background #6B4C9A
                color #ffffff
            }
            element "Container" {
                background #9B59B6
                color #ffffff
            }
            element "Database" {
                shape Cylinder
                background #336791
            }
            element "Web" {
                shape WebBrowser
            }
            element "Worker" {
                shape Robot
            }
        }
    }
}
```

### Phoenix with Components

When the internal structure of the Phoenix release matters:

```dsl
phoenix = container "Phoenix Application" "Web application and API" "Elixir/Phoenix" {

    # Phoenix contexts as components
    web = component "Web" "HTTP endpoint, controllers, LiveView" "Phoenix"
    accounts = component "Accounts" "User registration, authentication, authorization" "Elixir"
    orders = component "Orders" "Order lifecycle and processing" "Elixir"
    notifications = component "Notifications" "Email and push notifications" "Swoosh"
    repo = component "Repo" "Data access layer" "Ecto"

    web -> accounts "authenticates via" "Elixir function calls"
    web -> orders "delegates business operations to" "Elixir function calls"
    accounts -> repo "persists user data via" "Ecto queries"
    orders -> repo "persists order data via" "Ecto queries"
    orders -> notifications "triggers" "Elixir function calls"
}
```

### Phoenix with Ash Framework

When using Ash, domains replace Phoenix contexts as the component grouping:

```dsl
phoenix = container "Phoenix Application" "Web application and API" "Elixir/Phoenix" {

    # Web layer
    web = component "Web" "HTTP endpoint, controllers, LiveView" "Phoenix"
    api = component "API" "REST API endpoints" "AshJsonApi"

    # Ash domains as components
    accounts = component "Accounts" "User identity, authentication, authorization" "Ash Domain"
    helpdesk = component "Helpdesk" "Tickets, assignments, SLAs" "Ash Domain"
    billing = component "Billing" "Subscriptions and invoicing" "Ash Domain"

    # Supporting
    workers = component "Workers" "Background job processing" "Oban"

    web -> accounts "authenticates via" "Ash actions"
    web -> helpdesk "manages tickets via" "Ash actions"
    api -> accounts "exposes user endpoints from" "AshJsonApi"
    api -> helpdesk "exposes ticket endpoints from" "AshJsonApi"
    helpdesk -> accounts "looks up agents in" "Ash relationships"
    helpdesk -> billing "checks entitlements in" "Ash actions"
    workers -> helpdesk "processes SLA escalations via" "Ash actions"
}

# Ash handles persistence through its data layer
system.phoenix -> system.postgres "reads from and writes to" "Ash/Ecto/TCP"
```

### Umbrella Application (Single Release)

```dsl
system = softwareSystem "Platform" "Umbrella application" {

    release = container "Platform Release" "Umbrella release with all applications" "Elixir/OTP" {
        webApp = component "Web" "Phoenix endpoint and LiveView" "Phoenix"
        core = component "Core" "Business domain logic" "Elixir"
        ingestion = component "Ingestion" "Data pipeline" "Broadway"
        mailer = component "Mailer" "Transactional email" "Swoosh"

        webApp -> core "calls" "Elixir function calls"
        core -> ingestion "triggers pipelines in" "GenServer.cast"
        core -> mailer "sends email via" "Elixir function calls"
    }

    postgres = container "PostgreSQL" "Primary data store" "PostgreSQL 16" "Database"
    queue = container "RabbitMQ" "Message broker" "RabbitMQ" "Queue"

    release -> postgres "reads from and writes to" "Ecto/TCP"
    release.ingestion -> queue "consumes messages from" "AMQP/Broadway"
}
```

### Event-Driven Elixir System

```dsl
system = softwareSystem "EventPlatform" "Event-driven processing platform" {

    api = container "API Service" "REST API and WebSocket gateway" "Elixir/Phoenix" "API"
    eventStore = container "Event Store" "Append-only event log" "EventStoreDB" "Database"
    projector = container "Projector" "Builds read models from events" "Elixir/Broadway" "Worker"
    readDb = container "Read Database" "Optimized query store" "PostgreSQL 16" "Database"
    queue = container "Message Bus" "Event distribution" "NATS" "Queue"

    api -> eventStore "appends events to" "HTTP/gRPC"
    api -> readDb "queries read models from" "Ecto/TCP"
    eventStore -> queue "publishes events to" "NATS Streaming"
    projector -> queue "subscribes to events from" "NATS Streaming"
    projector -> readDb "updates read models in" "Ecto/TCP"
}
```

### Dynamic View: LiveView Request Flow

```dsl
dynamic system.phoenix "LiveViewFlow" "LiveView page load and interaction" {
    user -> web "requests page" "HTTPS"
    web -> accounts "checks session" "Elixir"
    web -> orders "loads data" "Elixir"
    orders -> repo "queries" "Ecto"
    web -> user "renders initial HTML" "HTTPS"
    user -> web "establishes LiveView connection" "WebSocket"
    user -> web "interacts with UI" "WebSocket"
    web -> orders "processes action" "Elixir"
    web -> user "pushes DOM diff" "WebSocket"
    autoLayout
}
```

### Oban-Powered Background Processing

When Oban is embedded in the Phoenix release (component-level):

```dsl
phoenix = container "Phoenix Application" "Web app with background processing" "Elixir/Phoenix" {
    web = component "Web" "HTTP and LiveView" "Phoenix"
    workers = component "Background Workers" "Scheduled and async jobs" "Oban"
    mailer = component "Mailer" "Email delivery" "Swoosh"

    web -> workers "enqueues jobs in" "Oban.insert"
    workers -> mailer "sends email via" "Elixir function calls"
}

# Oban uses PostgreSQL as its job queue
system.phoenix -> system.postgres "reads/writes data and polls jobs" "Ecto/TCP"
```

When Oban runs as a separate worker release (container-level):

```dsl
system = softwareSystem "System" {
    api = container "API Release" "Web API" "Elixir/Phoenix" "API"
    worker = container "Worker Release" "Background job processor" "Elixir/Oban" "Worker"
    postgres = container "PostgreSQL" "Data and job queue" "PostgreSQL 16" "Database"

    api -> postgres "reads/writes data and enqueues jobs" "Ecto/TCP"
    worker -> postgres "polls and processes jobs" "Oban/Ecto/TCP"
}
```

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| "X is not a valid identifier" | Forward reference | Move the definition before its first use |
| "X already exists" | Duplicate identifier | Use unique names or switch to `!identifiers hierarchical` |
| "Unexpected token" | `{` on next line | Move `{` to the same line as the statement |
| Implied relationship unwanted | Default behavior | Add `!impliedRelationships false` |
| View is empty | Elements not included | Add `include *` or specific `include` expressions |
| Duplicate relationship | Same source, dest, description | Use unique descriptions for each relationship |

## Related Skills

- `c4-model` — C4 abstractions and diagram types
- `c4-review` — Review models for correctness
- `c4-deployment` — Deployment modeling specifics
