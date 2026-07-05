# AI Hive — Architecture

AI Hive is a collection of seven independently deployable infrastructure systems. Each project solves a distinct layer of the production ML infrastructure stack and can be run, tested, and deployed on its own.

---

## System Overview

```mermaid
flowchart LR
    H["🔀 **Hermes**\nLLM Inference Gateway"]
    AR["🔍 **Argus**\nDrift Detection"]
    PY["⚡ **Pyrex**\nBenchmark Suite"]
    ST["🗄️ **Strata**\nFeature Store"]
    CO["🔄 **Conduit**\nPipeline Orchestrator"]
    CA["📦 **Capsule**\nModel Deployment"]
    LA["🗓️ **Lattice**\nJob Scheduler"]

    H --> AR --> PY --> ST --> CO --> CA --> LA
```

---

## ML Infrastructure Lifecycle

```mermaid
flowchart TD
    subgraph ROW1 [Serve · Monitor · Benchmark · Store]
        direction LR
        H["**Hermes**\nserve LLM requests\nrouting + circuit breakers"]
        AR["**Argus**\nmonitor drift\n+ fire alerts"]
        PY["**Pyrex**\nbenchmark backends\n+ regression detection"]
        ST["**Strata**\nstore features\n+ ASOF joins"]
    end

    subgraph ROW2 [Orchestrate · Deploy · Schedule]
        direction LR
        CO["**Conduit**\norchestrate DAGs\n+ queues"]
        CA["**Capsule**\npackage + deploy\n+ canary rollout"]
        LA["**Lattice**\nschedule + allocate\nML jobs"]
    end
```

---

## Individual System Architectures

### Hermes — LLM Inference Gateway

```mermaid
flowchart TD
    Client([Client Request]) --> RL[Rate Limiter\nRedis token bucket · atomic Lua]
    RL --> PQ[Priority Queue\nRedis Sorted Set\npremium › standard › batch]
    PQ --> Router[Router\nlatency-aware EWMA\nweighted RR · least-conn · priority]
    Router --> CB_A[Circuit Breaker A]
    Router --> CB_B[Circuit Breaker B]
    Router --> CB_C[Circuit Breaker C]
    CB_A --> OA[Ollama Backend A]
    CB_B --> OB[Ollama Backend B]
    CB_C --> OC[Ollama Backend C]
    OA & OB & OC --> SSE[SSE Streaming Proxy\nTTFT tracking]
    SSE --> OBS[Prometheus Metrics ──► Grafana\nLive Dashboard /ui]

    style Client fill:#4A90D9,color:#fff
    style OBS fill:#2E8B57,color:#fff
```

---

### Argus — ML Observability Platform

```mermaid
flowchart TD
    SDK([SDK argus.log / argus.init]) --> API[FastAPI /ingest · /ingest/batch]
    API --> RS[(Redis Streams)]
    RS --> SC[StreamConsumer\nconsumer groups]
    SC --> TSDB[(TimescaleDB\ntime-series store)]
    TSDB --> DE[DriftEngine\nAPScheduler]
    DE --> KS[KS Test]
    DE --> PSI[PSI]
    DE --> CHI[Chi-squared]
    DE --> JSD[JS Divergence]
    DE --> SHAP[SHAP Drift]
    KS & PSI & CHI & JSD & SHAP --> AE[AlertEngine\nYAML rules]
    AE --> WH[Webhooks]
    AE --> RT[Retraining Trigger]
    AE --> PROM[Prometheus Gauges ──► Grafana]
    TSDB --> DUCK[(DuckDB\n30-day analytics)]

    style SDK fill:#4A90D9,color:#fff
    style PROM fill:#2E8B57,color:#fff
```

---

### Pyrex — Benchmark Suite

```mermaid
flowchart TD
    CLI([CLI Typer]) --> BR[BenchmarkRunner]

    BR --> B1[PyTorch MPS]
    BR --> B2[PyTorch CPU]
    BR --> B3[ONNX Runtime\nCoreML EP]
    BR --> B4[Apple MLX\noptional]

    BR --> K1[attention · ffn\nmatmul · layernorm\nconv2d · embedding]
    BR --> TEL[Telemetry\nMPS mem · psutil RSS\nCPU% · powermetrics]

    BR --> RD[RegressionDetector\nz-score vs baseline]
    BR --> RA[RooflineAnalyser\nTFLOPS vs arithmetic intensity]
    BR --> RG[ReportGenerator\nHTML · Jinja2 + matplotlib]
    BR --> RS[(DuckDB + Parquet\nfull run history)]

    RD --> CI[GitHub Actions CI\nblock PRs on regression]

    style CLI fill:#4A90D9,color:#fff
    style RD fill:#C0392B,color:#fff
    style CI fill:#2E8B57,color:#fff
```

---

### Strata — Feature Store

```mermaid
flowchart TD
    SDK([SDK strata.get / strata.log\nstrata.get_historical]) --> ONLINE & OFFLINE & GRPC

    subgraph ONLINE [Online Path — real-time inference]
        OA[FastAPI /features/get] --> REDIS[(Redis Hash\nlow-latency · TTL)]
    end

    subgraph OFFLINE [Offline Path — training data]
        OB[FastAPI /features/historical] --> DUCK[DuckDB ASOF JOIN\npoint-in-time correct]
        DUCK --> PARQ[(Parquet · MinIO)]
    end

    GRPC[gRPC Batch API]

    MAT[Materialiser\nAPScheduler] -->|reads| PARQ
    MAT -->|writes| REDIS

    CV[Consistency Validator] --> REDIS & PARQ

    REDIS & DUCK --> PROM[Prometheus · 10 metrics ──► Grafana]

    style SDK fill:#4A90D9,color:#fff
    style PROM fill:#2E8B57,color:#fff
    style CV fill:#E67E22,color:#fff
```

---

### Conduit — ML Pipeline Orchestrator

```mermaid
flowchart TD
    Client([CLI / REST / SDK]) --> PE[PipelineEngine\nKahn's topological sort\ncycle detection · parallel dispatch]
    PE --> RQ[(Redis Streams\ntask queue · consumer groups)]
    PE --> SS[(SQLite Store\nrun history · task state)]
    RQ --> WP[WorkerPool\nasyncio]
    WP --> RQM[ResourceQuotaManager\npsutil CPU + memory]
    WP --> RWB[RetryWithBackoff\nexponential + jitter]
    WP --> DLQ[DeadLetterQueue\nexhausted tasks + forensics]
    DLQ -->|replay| RQ
    WP --> OBS[Prometheus · 10 metrics ──► Grafana]

    style Client fill:#4A90D9,color:#fff
    style DLQ fill:#C0392B,color:#fff
    style OBS fill:#2E8B57,color:#fff
```

---

### Capsule — Model Deployment Platform

```mermaid
flowchart TD
    CLI([CLI Typer\npackage · deploy · status · rollback]) --> PKG & DEP & CC & MS

    subgraph PKG [Packager]
        DET[Auto-detect framework] --> PACK[Dockerfile + FastAPI gen]
        PACK --> ONNX[ONNX optimise\nINT8 quantisation]
        ONNX --> REG[Push to MinIO registry]
    end

    subgraph DEP [Deployer]
        HELM[Helm chart gen] --> K8S[K3s deploy\nkubernetes client]
    end

    subgraph CC [CanaryController]
        TRF[Traefik weighted traffic split] --> PROM_Q[Prometheus error rate query]
        PROM_Q --> ROLL[Auto-rollback]
    end

    MS[(ManifestStore\nSQLite history)]

    style CLI fill:#4A90D9,color:#fff
    style ROLL fill:#C0392B,color:#fff
```

---

### Lattice — ML Job Scheduler

```mermaid
flowchart TD
    API([REST API :8002 / gRPC :50051]) --> SE[Scheduler Engine]

    SE --> FIFO[FIFO\n4 priority tiers]
    SE --> DRF[DRF\nDominant Resource Fairness\nper-team fair-share]
    SE --> GANG[Gang\natomic all-or-nothing reservation]
    SE --> PRMP[Preemption\nSIGUSR1 → checkpoint → evict]
    SE --> BF[Backfill\nfill idle capacity]

    SE --> RQ[(Redis Sorted Set Queue)]
    SE --> SS[(SQLite Job State Store)]
    SE --> WP[Worker Pool\nDocker containers]

    WP --> OBS[Prometheus · 12 metrics ──► Grafana]

    style API fill:#4A90D9,color:#fff
    style OBS fill:#2E8B57,color:#fff
    style PRMP fill:#C0392B,color:#fff
```

---

## Optional Integrations

These integrations show how the systems can work together. **They are optional — no project requires another to run.**

```mermaid
flowchart TD
    H[Hermes\nLLM Gateway]
    AR[Argus\nDrift Detection]
    PY[Pyrex\nBenchmarks]
    ST[Strata\nFeature Store]
    CO[Conduit\nOrchestrator]
    CA[Capsule\nDeployment]
    LA[Lattice\nScheduler]

    AR -.->|monitor gateway metrics| H
    PY -.->|benchmark Ollama backends| H
    CA -.->|package model services routed via| H
    CA -.->|deploy models monitored by| AR
    ST -.->|log feature distributions to| AR
    CO -.->|strata.get in DAG tasks| ST
    LA -.->|schedule jobs triggered by| CO
```

> Dashed arrows = optional integrations. All systems run independently.

---

## Design Principles

1. **Local-first** — designed for Apple Silicon, zero cloud dependency
2. **Independent deployability** — each project runs standalone with `docker compose up`
3. **No cross-project runtime dependencies** — Hermes does not require Argus; Strata does not require Conduit
4. **Production-style patterns** — circuit breakers, rate limiting, dead letter queues, ASOF joins, canary deployments
5. **Observable by default** — Prometheus metrics and Grafana dashboards across all projects that serve traffic
6. **Testable without dependencies** — every project's test suite runs without external services (mocked)
