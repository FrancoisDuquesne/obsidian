# Pass Lifecycle

A pass goes through several phases, coordinated by three core services: **PSS**, **PPS**, and **PRS**.

---

## Bird's-eye view

```mermaid
flowchart TB
    subgraph PLAN["1 — Planning"]
        direction LR
        RES["Pass reserved"] --> QUARTZ["Quartz job scheduled"]
    end

    subgraph EXEC["2 — Execution"]
        direction LR
        POD["PPS Pod spawned"] --> PRE["Preprocessing"]
        PRE --> PROC["Procedure graph"]
        PROC --> POST["Postprocessing"]
        POST --> KILL["Pod terminated"]
    end

    subgraph REPORT["3 — Reporting"]
        direction LR
        INGEST["PRS ingests events"] --> STORE["Data persisted"]
        STORE --> QUERY["REST / SSE queries"]
    end

    PLAN ==> EXEC ==> REPORT

    style PLAN fill:#e8f5e9,stroke:#388e3c
    style EXEC fill:#fff8e1,stroke:#f9a825
    style REPORT fill:#e3f2fd,stroke:#1565c0
```

---

## Scheduling (PSS)

When a pass is **reserved, updated, imported, or cancelled**, the PSS creates/updates/cancels a Quartz scheduled job. When the job deadline fires:
1. Pass details are fetched from the database
2. PSS calls the **Kubernetes connector** to launch a PPS Pod

---

## Execution (PPS)

The [[05_MCS/Passes_Processor_Service|PPS]] is spawned as an **ephemeral Kubernetes Pod** for each pass.

### Execution phases
1. **Preprocessing** — load configuration, establish ground station connection
2. **Procedure execution** — run the procedure graph (scripted actions/flows)
3. **Postprocessing** — cleanup, finalize reports

### Pass states

```mermaid
stateDiagram-v2
    [*] --> starting
    starting --> executing
    executing --> success
    executing --> failure
    executing --> executing : stop & restart
    success --> [*]
    failure --> [*]
```

A pass can be **stopped and restarted** at any time within the ground station reservation window.

### Live operator interactions
During execution, the PSS acts as a **proxy** between the UI and the PPS Pod (using the Pod's internal Kubernetes address):

- Skip / unskip node or branch
- Override node parameters on the fly
- Create / delete dynamic nodes and links
- Re-run a single action or entire branch

Every interaction is persisted in the `scheduledpassaction` table.

---

## Reporting (PRS)

The [[05_MCS/Passes_Reporter_Service|PRS]] runs continuously and:
1. **Consumes** execution events published by PPS via RabbitMQ
2. **Stores** packets (TC/TM), events, and time correlation data
3. **Serves** this data via REST APIs and SSE streams

After the PPS Pod terminates, the PRS is the **sole access point** for historical pass data.

---

## Sequence overview

```mermaid
sequenceDiagram
    participant PSS as PSS (Scheduler)
    participant K8s as Kubernetes
    participant PPS as PPS Pod
    participant GS as Ground Station
    participant MQ as RabbitMQ
    participant PRS as PRS (Reporter)
    participant UI as Operator UI

    PSS->>K8s: Launch PPS Pod
    K8s-->>PSS: Pod UID
    PPS->>GS: Open TM/TC bridge
    PPS->>PPS: Execute procedure graph
    PPS->>GS: Send TC
    GS-->>PPS: Receive TM
    PPS->>MQ: Publish events & packets
    MQ-->>PRS: Consume events & packets
    PRS->>PRS: Store to database
    UI->>PSS: Operator interaction
    PSS->>PPS: Forward to Pod
    PPS-->>PSS: Response
    PSS-->>UI: Response
    PPS->>PPS: Postprocessing
    PPS->>K8s: Pod terminates
    UI->>PRS: Query historical data
    PRS-->>UI: REST / SSE response
```

---

## Links
- [[05_MCS/MCS_Overview]]
- [[05_MCS/Passes_Processor_Service]]
- [[05_MCS/Passes_Reporter_Service]]
