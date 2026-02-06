# Mission Control Software (MCS) Overview

## What it is
The MCS is a **microservice-based backend** for operating spacecraft. It handles ground planning, pass execution, telemetry/telecommand processing, and spacecraft lifecycle management.

> This is the **control plane** for satellite operations — configuration, orchestration, and state management.

---

## Architecture layers

```mermaid
graph TB
    subgraph Gatekeepers
        OG["Operator Gatekeeper<br/>(Web GUI)"]
        TG["Tasking Gatekeeper<br/>(API key)"]
        AG["Admin Gatekeeper<br/>(Proxy)"]
    end

    subgraph Core Backend
        PSS["PSS<br/>Passes Scheduler"]
        PPS["PPS<br/>Passes Processor"]
        PRS["PRS<br/>Passes Reporter"]
        EMS["EMS<br/>Equipment Mgmt"]
        SLS["SLS<br/>Spacecraft Lifecycle"]
        OAP["OAP<br/>Onboard Activity Planner"]
        NMS["NMS<br/>Notifications"]
    end

    subgraph Extension Backend
        EXT1["Mission-specific<br/>service A"]
        EXT2["Mission-specific<br/>service B"]
    end

    subgraph Infrastructure
        DB[(PostgreSQL)]
        MQ["RabbitMQ"]
        K8s["Kubernetes"]
    end

    OG --> PSS & PRS & EMS
    TG --> OAP & EMS
    PSS -->|K8s API| K8s -->|spawns| PPS
    PPS -->|AMQP| MQ -->|AMQP| PRS
    PPS -->|REST| EMS
    PPS -->|REST| SLS
    PSS --> DB
    PRS --> DB
    EMS --> DB
    SLS --> DB
    PSS -->|deploys| EXT1 & EXT2

    style PPS fill:#e8b84b,stroke:#333,color:#000
    style PRS fill:#4b9ee8,stroke:#333,color:#fff
    style PSS fill:#6bc46b,stroke:#333,color:#000
```

### Core backend
Mission-agnostic microservices with well-delimited responsibilities:

| Service | Abbr | Role |
|---|---|---|
| Passes Scheduler Service | PSS | Ground planning, triggers pass execution |
| [[05_MCS/Passes_Processor_Service\|Passes Processor Service]] | PPS | Executes a single pass (ephemeral batch) |
| [[05_MCS/Passes_Reporter_Service\|Passes Reporter Service]] | PRS | Stores & serves pass execution data |
| Equipment Management Service | EMS | Satellite, constellation & ground station config |
| Spacecraft Lifecycle Service | SLS | Spacecraft state tracking & history |
| Onboard Activity Planner | OAP | Mission planning, activity-to-TC transformation |
| Notification Management Service | NMS | Event routing & operator notifications |

### Extension backend
Mission-specific services for spacecraft-specific orchestration (error recovery, payload handling, etc.).

### Gatekeepers
Security layer — no backend service is directly exposed. Gatekeepers handle authentication, authorization, and actor-specific concerns.

| Actor | Auth | Notes |
|---|---|---|
| Operator | JWT / OIDC | Web GUI |
| Tasking Software | API key | Headless API |
| System admin | Third-party redirect | Proxy |

---

## Communication patterns
- **Synchronous REST** — when the caller must fail or block on the target's response
- **Asynchronous AMQP (RabbitMQ)** — for decoupled event-driven flows (e.g. PPS publishing events, PRS consuming them)

```mermaid
flowchart LR
    subgraph sync["Synchronous (REST)"]
        direction LR
        A["PPS"] -->|"GET config"| B["EMS"]
        A -->|"GET keys"| C["SLS"]
    end

    subgraph async["Asynchronous (AMQP)"]
        direction LR
        D["PPS"] -->|"publish"| E["RabbitMQ"] -->|"consume"| F["PRS"]
        E -->|"consume"| G["NMS"]
    end

    style sync fill:#f5f5f5,stroke:#999
    style async fill:#f0f7ff,stroke:#999
```

---

## Key concept: the pass

A **pass** is a time window during which a ground station has line-of-sight to a satellite.

```mermaid
flowchart LR
    GS["Ground Station"] <-->|"RF link"| SAT["Satellite"]

    subgraph pass["During a pass"]
        direction TB
        TC["TC (telecommands) ➜ uplink"]
        TM["TM (telemetry) ➜ downlink"]
        PROC["Procedure graph execution"]
    end

    GS --- pass

    style pass fill:#fff8e1,stroke:#f9a825
    style SAT fill:#e1f5fe,stroke:#0288d1
    style GS fill:#e8f5e9,stroke:#388e3c
```

See [[05_MCS/Pass_Lifecycle]] for the full flow.

---

## Links
- [[05_MCS/Pass_Lifecycle]]
- [[05_MCS/Passes_Processor_Service]]
- [[05_MCS/Passes_Reporter_Service]]
