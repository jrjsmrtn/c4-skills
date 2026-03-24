---
# SPDX-FileCopyrightText: 2025 Georges Martin <jrjsmrtn@gmail.com>
# SPDX-License-Identifier: MIT
name: c4-deployment
description: Model deployment environments using C4 deployment diagrams and Structurizr DSL. Use when adding deployment views, modeling infrastructure, documenting production/staging environments, or mapping containers to infrastructure.
---

# C4 Deployment

Model deployment environments using C4 deployment diagrams and Structurizr DSL.

## When to Use

- Adding deployment views to an existing C4 model
- Documenting how containers map to infrastructure (servers, cloud services, Kubernetes)
- Modeling multiple environments (production, staging, development)
- Showing infrastructure components (load balancers, DNS, firewalls, CDNs)
- Communicating deployment architecture to operations teams

## Key Concepts

### Deployment Diagram Purpose

Deployment diagrams answer: **"Where does it run?"**

They show the mapping from C4 containers to physical or virtual infrastructure, for a **specific environment**. Create separate deployment views for production, staging, development, etc.

### Element Types

| Element | What It Represents | Examples |
|---------|-------------------|----------|
| Deployment Node | Something containers are deployed onto | Physical server, VM, Docker host, Kubernetes cluster, AWS region, browser |
| Infrastructure Node | Supporting infrastructure | Load balancer, DNS, firewall, CDN, API gateway |
| Software System Instance | A deployed instance of a software system | Only for external systems in deployment context |
| Container Instance | A deployed instance of a container | Your API running on a specific server |

### Deployment Nodes vs Infrastructure Nodes

**Deployment nodes** host containers — they run your code or store your data:
- Physical servers, VMs, cloud instances (EC2, Compute Engine)
- Container runtimes (Docker, Kubernetes pods)
- PaaS platforms (Heroku, Fly.io)
- Browsers (for client-side SPAs)
- Mobile devices

**Infrastructure nodes** support the system but don't host containers:
- Load balancers (ALB, HAProxy, Nginx)
- DNS services (Route 53, Cloudflare)
- Firewalls, WAFs
- CDNs
- API gateways
- VPNs, bastion hosts

## Structurizr DSL Syntax

### Deployment Environment

```dsl
model {
    # Define your system and containers first
    system = softwareSystem "System" {
        api = container "API" "REST API" "Go"
        db = container "Database" "Data store" "PostgreSQL"
        web = container "Web App" "SPA" "React"
    }

    # Then define deployment environments
    production = deploymentEnvironment "Production" {
        # deployment nodes, instances, infrastructure
    }

    staging = deploymentEnvironment "Staging" {
        # simpler version of production
    }
}
```

### Deployment Nodes

```dsl
deploymentNode <name> [description] [technology] [tags] [instances] {
    # nested nodes, instances, infrastructure
}
```

Nodes can be nested to show hierarchical infrastructure:

```dsl
deploymentEnvironment "Production" {
    aws = deploymentNode "AWS" "" "Amazon Web Services" {
        region = deploymentNode "eu-west-1" "" "AWS Region" {
            vpc = deploymentNode "VPC" "" "AWS VPC" {
                subnet = deploymentNode "Private Subnet" "" "AWS Subnet" {
                    ec2 = deploymentNode "EC2 Instance" "" "t3.medium" "" 2 {
                        containerInstance system.api
                    }
                }
            }
        }
    }
}
```

### Instances Count

```dsl
deploymentNode "Web Server" "" "Nginx" "" 4          # exactly 4
deploymentNode "Worker" "" "Go" "" "1..N"             # 1 to N (elastic)
deploymentNode "Standby" "" "PostgreSQL" "" "0..1"    # 0 or 1 (failover)
deploymentNode "Shard" "" "MongoDB" "" "3..6"         # 3 to 6
```

### Container Instances

```dsl
deploymentNode "App Server" "" "Ubuntu" {
    containerInstance system.api
    containerInstance system.worker
}
```

### Software System Instances

Used for external systems in deployment context:

```dsl
deploymentNode "External" {
    softwareSystemInstance externalPaymentSystem
}
```

### Infrastructure Nodes

```dsl
deploymentNode "AWS" {
    route53 = infrastructureNode "Route 53" "DNS" "AWS Route 53"
    alb = infrastructureNode "ALB" "Load balancer" "AWS ALB"
    cdn = infrastructureNode "CloudFront" "CDN" "AWS CloudFront"

    # Infrastructure nodes can have relationships
    route53 -> cdn "resolves to"
    cdn -> alb "forwards to"
}
```

### Deployment Groups

Control which instances are linked when you have multiple replicas:

```dsl
deploymentEnvironment "Production" {
    primaryGroup = deploymentGroup "Primary"
    secondaryGroup = deploymentGroup "Secondary"

    deploymentNode "Primary Server" {
        containerInstance system.api primaryGroup
        containerInstance system.db primaryGroup
    }

    deploymentNode "Secondary Server" {
        containerInstance system.api secondaryGroup
        containerInstance system.db secondaryGroup
    }
}
```

Without deployment groups, all instances of container A would be linked to all instances of container B. Deployment groups restrict relationships to instances within the same group.

### Health Checks

```dsl
containerInstance system.api {
    healthCheck "API Health" "https://api.example.com/health" 30 5000
    # name, URL, interval (seconds), timeout (milliseconds)
}
```

## Views

### Deployment View

```dsl
views {
    deployment system "Production" "ProductionDeployment" {
        include *
        autoLayout lr
        title "Production Deployment"
        description "How the system is deployed in production"
    }

    deployment system "Staging" "StagingDeployment" {
        include *
        autoLayout lr
        title "Staging Deployment"
    }

    # Wildcard: show all systems in an environment
    deployment * "Production" "FullProductionDeployment" {
        include *
        autoLayout lr
    }
}
```

### Styling Deployment Elements

```dsl
styles {
    element "Deployment Node" {
        shape RoundedBox
        background #ffffff
        color #000000
        stroke #888888
    }
    element "Infrastructure Node" {
        shape RoundedBox
        background #ffffff
        color #000000
        stroke #888888
    }
}
```

## Common Patterns

### Single Server

```dsl
deploymentEnvironment "Production" {
    server = deploymentNode "Server" "Single production server" "Ubuntu 22.04" {
        nginx = infrastructureNode "Nginx" "Reverse proxy" "Nginx"
        appNode = deploymentNode "Application" "" "Systemd Service" {
            containerInstance system.api
        }
        dbNode = deploymentNode "Database" "" "PostgreSQL 16" {
            containerInstance system.db
        }
        nginx -> appNode "proxies to" "HTTP/localhost:4000"
    }
}
```

### Kubernetes

```dsl
deploymentEnvironment "Production" {
    cloud = deploymentNode "Cloud Provider" "" "GCP" {
        cluster = deploymentNode "GKE Cluster" "" "Kubernetes 1.29" {
            apiPod = deploymentNode "api Pod" "" "Kubernetes Pod" "" "1..N" {
                containerInstance system.api
            }
            workerPod = deploymentNode "worker Pod" "" "Kubernetes Pod" "" "1..N" {
                containerInstance system.worker
            }
        }
        cloudSql = deploymentNode "Cloud SQL" "" "GCP Cloud SQL" {
            containerInstance system.db
        }
        lb = infrastructureNode "Cloud Load Balancer" "Ingress" "GCP LB"
        lb -> apiPod "routes to" "HTTPS"
    }
}
```

### AWS with Themes

```dsl
views {
    theme amazon-web-services-2020.04.30
}

model {
    deploymentEnvironment "Production" {
        aws = deploymentNode "AWS" "" "" "Amazon Web Services - Cloud" {
            region = deploymentNode "eu-west-1" "" "" "Amazon Web Services - Region" {
                alb = infrastructureNode "ALB" "" "" "Amazon Web Services - Elastic Load Balancing"
                deploymentNode "ECS" "" "" "Amazon Web Services - Elastic Container Service" {
                    containerInstance system.api
                }
                deploymentNode "RDS" "" "" "Amazon Web Services - RDS" {
                    containerInstance system.db
                }
            }
        }
    }
}
```

Tag elements with the exact AWS/Azure/GCP theme tag names to get provider icons.

### Podman/Docker Compose (Development)

```dsl
deploymentEnvironment "Development" {
    devMachine = deploymentNode "Developer Machine" "" "macOS/Linux" {
        podman = deploymentNode "Podman" "" "Podman" {
            appContainer = deploymentNode "app" "" "Container" {
                containerInstance system.api
            }
            dbContainer = deploymentNode "db" "" "Container" {
                containerInstance system.db
            }
        }
        browser = deploymentNode "Browser" "" "Chrome/Firefox" {
            containerInstance system.web
        }
        browser -> appContainer "connects to" "HTTP/localhost:4000"
    }
}
```

### Elixir/OTP Release

A standard Phoenix/OTP release deployed as a systemd service with clustering:

```dsl
model {
    system = softwareSystem "System" {
        phoenix = container "Phoenix Application" "Web application and API" "Elixir/Phoenix"
        liveview = container "LiveView UI" "Real-time user interface" "Phoenix LiveView"
        oban = container "Oban Workers" "Background job processing" "Oban"
        db = container "Database" "Data store" "PostgreSQL"
    }

    deploymentEnvironment "Production" {
        server = deploymentNode "App Server" "" "Ubuntu 22.04" "" 2 {
            beam = deploymentNode "BEAM VM" "Erlang runtime" "Erlang/OTP 27" {
                release = deploymentNode "Release" "" "mix release" {
                    containerInstance system.phoenix
                    containerInstance system.liveview
                    containerInstance system.oban
                }
            }
        }
        dbServer = deploymentNode "Database Server" "" "Ubuntu 22.04" {
            deploymentNode "PostgreSQL" "" "PostgreSQL 16" {
                containerInstance system.db
            }
        }
        lb = infrastructureNode "Load Balancer" "Reverse proxy, SSL termination, WebSocket upgrade" "Nginx"
        lb -> server "proxies to" "HTTP/4000, WebSocket"
        server -> dbServer "queries" "TCP/5432"
        server -> server "cluster communication" "Erlang Distribution/EPMD"
    }
}
```

Key modeling decisions for Elixir/OTP:
- The **BEAM VM** is a deployment node — it's the runtime that hosts the release
- A **mix release** is a deployment node wrapping container instances — it's the packaged artifact
- **Phoenix, LiveView, and Oban** are separate C4 containers because they serve distinct roles, even though they run in the same BEAM instance. They share the process space but have different responsibilities.
- **Erlang distribution** between nodes is an infrastructure-level relationship (node-to-node clustering)
- **EPMD** (Erlang Port Mapper Daemon) or DNS-based discovery can be modeled as an infrastructure node if architecturally significant

### Elixir/OTP Clustering with libcluster

```dsl
deploymentEnvironment "Production" {
    deploymentNode "Kubernetes" "" "K8s 1.29" {
        pod1 = deploymentNode "app-0" "" "Kubernetes Pod" {
            beam1 = deploymentNode "BEAM VM" "" "Erlang/OTP 27" {
                containerInstance system.phoenix
            }
        }
        pod2 = deploymentNode "app-1" "" "Kubernetes Pod" {
            beam2 = deploymentNode "BEAM VM" "" "Erlang/OTP 27" {
                containerInstance system.phoenix
            }
        }
        pod3 = deploymentNode "app-2" "" "Kubernetes Pod" {
            beam3 = deploymentNode "BEAM VM" "" "Erlang/OTP 27" {
                containerInstance system.phoenix
            }
        }

        # libcluster forms an Erlang mesh
        pod1 -> pod2 "Erlang distribution" "BEAM Clustering/libcluster"
        pod2 -> pod3 "Erlang distribution" "BEAM Clustering/libcluster"
        pod1 -> pod3 "Erlang distribution" "BEAM Clustering/libcluster"
    }

    deploymentNode "Cloud SQL" "" "PostgreSQL 16" {
        containerInstance system.db
    }
}
```

### Elixir/OTP with Fly.io

```dsl
deploymentEnvironment "Production" {
    fly = deploymentNode "Fly.io" "" "Fly.io" {
        region1 = deploymentNode "cdg (Paris)" "" "Fly Region" {
            machine1 = deploymentNode "Fly Machine" "" "Firecracker VM" "" "1..N" {
                deploymentNode "BEAM VM" "" "Erlang/OTP 27" {
                    containerInstance system.phoenix
                }
            }
        }
        region2 = deploymentNode "ams (Amsterdam)" "" "Fly Region" {
            machine2 = deploymentNode "Fly Machine" "" "Firecracker VM" "" "1..N" {
                deploymentNode "BEAM VM" "" "Erlang/OTP 27" {
                    containerInstance system.phoenix
                }
            }
        }
        flyProxy = infrastructureNode "Fly Proxy" "Anycast routing, TLS termination" "Fly Proxy"
        dns = infrastructureNode "Fly DNS" "Internal DNS for clustering" "Fly DNS"
        flyProxy -> machine1 "routes to nearest" "HTTP/HTTPS"
        flyProxy -> machine2 "routes to nearest" "HTTP/HTTPS"
        region1 -> region2 "Erlang distribution" "WireGuard/6PN"

        pgFly = deploymentNode "Fly Postgres" "" "Fly Managed Postgres" {
            containerInstance system.db
        }
    }
}
```

### Elixir Umbrella (Multiple OTP Applications)

When an Elixir umbrella deploys as a single release, model each umbrella app as a C4 component within one container, not as separate containers:

```dsl
model {
    system = softwareSystem "System" {
        # The release is ONE container — umbrella apps are components
        release = container "Application Release" "Umbrella release" "Elixir/OTP" {
            webApp = component "Web" "Phoenix endpoint and LiveView" "Phoenix"
            core = component "Core" "Business logic and domain" "Elixir"
            ingestion = component "Ingestion" "Data pipeline" "Broadway"
            notifications = component "Notifications" "Email and push" "Swoosh"

            webApp -> core "calls" "Elixir function calls"
            core -> ingestion "triggers" "GenServer.call"
            core -> notifications "sends via" "Elixir function calls"
        }
        db = container "Database" "Data store" "PostgreSQL"
        queue = container "Message Queue" "Event streaming" "RabbitMQ"

        release -> db "queries" "Ecto/TCP"
        release.ingestion -> queue "consumes from" "AMQP/Broadway"
    }

    deploymentEnvironment "Production" {
        server = deploymentNode "App Server" "" "Ubuntu 22.04" "" 2 {
            deploymentNode "BEAM VM" "" "Erlang/OTP 27" {
                containerInstance system.release
            }
        }
        deploymentNode "Database Server" "" "Ubuntu 22.04" {
            containerInstance system.db
        }
        deploymentNode "Queue Server" "" "Ubuntu 22.04" {
            containerInstance system.queue
        }
    }
}
```

If umbrella apps are deployed as **separate releases** (separate BEAM instances), model each as a separate C4 container instead.

### High Availability / Failover

```dsl
deploymentEnvironment "Production" {
    primaryRegion = deploymentNode "Primary (eu-west-1)" "" "AWS Region" {
        deploymentNode "Primary DB" "" "PostgreSQL" {
            containerInstance system.db
        }
        deploymentNode "App Servers" "" "EC2" "" "2..4" {
            containerInstance system.api
        }
    }
    secondaryRegion = deploymentNode "Secondary (eu-west-2)" "" "AWS Region" {
        deploymentNode "Standby DB" "" "PostgreSQL (Read Replica)" "" "0..1" {
            containerInstance system.db
        }
    }
    primaryRegion -> secondaryRegion "replicates to" "PostgreSQL Streaming Replication"
}
```

## Checklist

When modeling deployment, verify:

- [ ] Each deployment environment is separate (`deploymentEnvironment` per env)
- [ ] Every container from the model has at least one `containerInstance` in production
- [ ] Deployment nodes reflect actual infrastructure (not invented structure)
- [ ] Instance counts match reality (or use ranges for elastic scaling)
- [ ] Infrastructure nodes are included for load balancers, DNS, firewalls
- [ ] Relationships between infrastructure and deployment nodes are specified
- [ ] Deployment groups are used when multiple replicas need isolated relationships
- [ ] Cloud provider themes are applied if using AWS/Azure/GCP
- [ ] `autoLayout lr` is used (left-to-right often works better for deployment)
- [ ] Each deployment view is scoped to one environment

## Related Skills

- `c4-model` — C4 abstractions and diagram types
- `structurizr-dsl` — Full DSL syntax reference
- `c4-review` — Review models for correctness
