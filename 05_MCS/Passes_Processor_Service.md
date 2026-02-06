# Passes Processor Service (PPS)

> Not to be confused with [[04_Synchronization/PPS|PPS (Pulse Per Second)]] from the signal processing domain.

## What it is
The PPS is an **ephemeral batch service** that executes a single satellite pass. It is spawned as a **Kubernetes Pod** by the PSS when a scheduled pass begins, and terminated once the pass and post-processing are complete.

---

## Pass state machine

```mermaid
stateDiagram-v2
    [*] --> starting : PSS launches Pod
    starting --> executing : config loaded, GS connected
    executing --> success : procedure graph completed
    executing --> failure : error / timeout
    executing --> executing : stop & restart
    success --> [*] : Pod terminated
    failure --> [*] : Pod terminated
```

---

## Responsibilities

### Pass execution
- Runs the full pass lifecycle: preprocessing, procedure graph execution, postprocessing
- Manages pass state transitions: `starting` → `executing` → `success` / `failure`

### Ground station bridge
- Sends **telecommands (TC)** to the spacecraft via the ground station
- Receives **telemetry (TM)** packets from the spacecraft
- Supports multiple ground station providers (e.g. Leafspace via MQTT)

```mermaid
flowchart LR
    PPS["PPS Pod"]
    MQTT["MQTT broker"]
    GS["Ground Station<br/>(e.g. Leafspace)"]
    SAT["Spacecraft"]

    PPS -->|"TC packets"| MQTT --> GS -->|"RF uplink"| SAT
    SAT -->|"RF downlink"| GS --> MQTT -->|"TM packets"| PPS

    style PPS fill:#e8b84b,stroke:#333,color:#000
    style SAT fill:#e1f5fe,stroke:#0288d1
    style GS fill:#e8f5e9,stroke:#388e3c
```

### Procedure graph execution
- Executes a directed graph of scripted actions (Python scripts)
- Supports conditional branching, sequential and parallel execution
- Dynamic nodes can be added/removed at runtime by operators

```mermaid
flowchart TD
    START(("Start")) --> A["Pre-pass script"]
    A --> B["TC: set mode"]
    B --> C{"Check TM\nresponse"}
    C -->|OK| D["TC: payload on"]
    C -->|FAIL| E["TC: retry"]
    E --> C
    D --> F["Download data"]
    F --> G["Post-pass script"]
    G --> END(("End"))
    DYN["Dynamic node\n(operator-added)"]
    D -.->|"injected at runtime"| DYN -.-> F

    style DYN fill:#fff3e0,stroke:#e65100,stroke-dasharray: 5 5
    style START fill:#c8e6c9,stroke:#388e3c
    style END fill:#c8e6c9,stroke:#388e3c
```

### Live interactions
While executing, accepts operator commands proxied through PSS:
- Skip/unskip nodes or branches
- Override script parameters
- Add/remove dynamic nodes
- Re-run actions or branches

```mermaid
flowchart LR
    OP["Operator UI"] -->|"interaction"| PSS
    PSS -->|"proxy via Pod IP"| PPS["PPS Pod"]
    PPS -->|"response"| PSS -->|"response"| OP
    PSS -->|"persist action"| DB[(scheduledpassaction)]

    style OP fill:#e1f5fe,stroke:#0288d1
    style PPS fill:#e8b84b,stroke:#333,color:#000
```

### Security & timing
- Manages **encryption keys and IVs** for secure spacecraft communication
- Handles **time correlation** between ground and spacecraft clocks

### Event publishing
- Publishes all execution events, packets, and status updates to **RabbitMQ**
- Consumed downstream by [[05_MCS/Passes_Reporter_Service|PRS]]

---

## Deployment model

> **One pass = one Pod**

```mermaid
flowchart TB
    subgraph K8s["Kubernetes cluster"]
        PSS["PSS"]
        PSS -->|"create Pod"| P1["PPS Pod<br/>pass #101<br/>SAT-A"]
        PSS -->|"create Pod"| P2["PPS Pod<br/>pass #102<br/>SAT-B"]
        PSS -->|"create Pod"| P3["PPS Pod<br/>pass #103<br/>SAT-A"]
    end

    PSS -->|"store UID"| DB[(Database)]

    style P1 fill:#e8b84b,stroke:#333,color:#000
    style P2 fill:#e8b84b,stroke:#333,color:#000
    style P3 fill:#e8b84b,stroke:#333,color:#000
    style PSS fill:#6bc46b,stroke:#333,color:#000
```

- Pod is created by PSS via Kubernetes API
- Pod UID is stored in the database for addressing
- PSS uses Pod address to proxy live operator interactions
- Pod is cleaned up after pass completion

---

## Key classes
- `PassExecutionService` — main orchestrator
- `ProcedureExecutionService` — procedure graph runner
- `PassResolverService` — pass state and deadline management
- `PacketReceiver` / `PacketSender` — TM/TC exchange with ground station
- `LeafspaceGroundStationTtcBridge` — Leafspace-specific MQTT bridge

---

## Links
- [[05_MCS/Pass_Lifecycle]]
- [[05_MCS/Passes_Reporter_Service]]
- [[05_MCS/MCS_Overview]]
