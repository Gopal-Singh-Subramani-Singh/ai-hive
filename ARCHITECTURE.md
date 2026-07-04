# AI Hive — Architecture

AI Hive is a collection of seven independently deployable infrastructure systems. Each project solves a distinct layer of the production ML infrastructure stack and can be run, tested, and deployed on its own.

---

## System Overview

```
                         AI Hive
        Local-First AI Infrastructure Suite

 ┌─────────────────────────────────────────────────────┐
 │                                                     │
 │  Hermes   →  LLM inference serving and routing      │
 │  Argus    →  ML observability and drift detection   │
 │  Pyrex    →  Inference benchmarking and profiling   │
 │  Strata   →  Online/offline feature serving         │
 │  Conduit  →  Event-driven ML pipeline orchestration │
 │  Capsule  →  Model packaging and deployment         │
 │  Lattice  →  Distributed job scheduling             │
 │                                                     │
 └─────────────────────────────────────────────────────┘
```

---

## ML Infrastructure Lifecycle

Each project maps to a stage in the production AI infrastructure lifecycle:

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│  Hermes │    │  Argus  │    │  Pyrex  │    │  Strata │
│  serve  │    │ monitor │    │benchmark│    │  store  │
│   LLM   │    │  drift  │    │backends │    │features │
│requests │    │+ alerts │    │+ regress│    │ + ASOF  │
└─────────┘    └─────────┘    └─────────┘    └─────────┘

┌─────────┐    ┌─────────┐    ┌─────────┐
│ Conduit │    │ Capsule │    │ Lattice │
│orchestr-│    │ package │    │schedule │
│ate DAGs │    │+ deploy │    │+ alloc  │
│+ queues │    │+ canary │    │ML jobs  │
└─────────┘    └─────────┘    └─────────┘
```

---

## Individual System Architectures

### Hermes — LLM Inference Gateway

```
Client Request
     │
     ▼
Rate Limiter (Redis token bucket, Lua atomic)
     │
     ▼
Priority Queue (Redis Sorted Set: premium/standard/batch)
     │
     ▼
Router (latency-aware / weighted round-robin / least-conn / priority)
     │
     ├── Circuit Breaker A → Ollama Backend A
     ├── Circuit Breaker B → Ollama Backend B
     └── Circuit Breaker C → Ollama Backend C
                                     │
                              SSE Streaming Proxy
                                     │
                              Prometheus Metrics
                                     │
                              Grafana Dashboard
```

### Argus — ML Observability Platform

```
SDK (argus.log)
    │
    ▼
FastAPI /ingest
    │
    ▼
Redis Streams ──► StreamConsumer ──► TimescaleDB
                                         │
                                         ▼
                               DriftEngine (APScheduler)
                                    │    │    │    │    │
                                   KS   PSI  Chi  JSD  SHAP
                                         │
                                         ▼
                               AlertEngine (YAML rules)
                                         │
                                         ▼
                               Webhooks / Retraining Trigger
                                         │
                                         ▼
                               Prometheus Gauges ──► Grafana
```

### Pyrex — Benchmark Suite

```
CLI (Typer)
  └── BenchmarkRunner
        ├── Backends: MPS | CPU | ONNX Runtime | MLX
        ├── Kernels: attention | ffn | matmul | layernorm | conv2d | embedding
        ├── Telemetry: powermetrics | MPS memory | psutil
        └── Store: DuckDB + Parquet
  └── RegressionDetector (z-score vs baseline)
  └── RooflineAnalyser (arithmetic intensity chart)
  └── ReportGenerator (HTML + Jinja2)
```

### Strata — Feature Store

```
SDK (strata.get / strata.log)
    │
    ├── Online Path:   FastAPI → Redis Hash → <low-latency response>
    │
    └── Offline Path:  DuckDB ASOF JOIN → Parquet → MinIO → training dataset
                                │
                        Materialiser (APScheduler)  ← syncs offline → online
                                │
                        Consistency Validator
                                │
                        Lineage DAG Tracker
                                │
                        Prometheus Metrics (10 metrics)
```

### Conduit — ML Pipeline Orchestrator

```
┌─────────────────────────────────────────┐
│  Client (CLI / REST / SDK)              │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│  PipelineEngine  (Kahn's algorithm)     │
└──────┬───────────────────────┬──────────┘
       │                       │
┌──────▼──────┐       ┌────────▼────────┐
│ Redis Stream│       │  SQLite Store   │
│ (task queue)│       │  (run history)  │
└──────┬──────┘       └─────────────────┘
       │
┌──────▼──────────────────────────────────┐
│  WorkerPool  (asyncio workers)          │
│  - ResourceQuotaManager (psutil)        │
│  - RetryWithBackoff (exponential+jitter)│
│  - DeadLetterQueue (exhausted tasks)    │
└─────────────────────────────────────────┘
```

### Capsule — Model Deployment Platform

```
CLI (Typer)
  ├── Packager
  │     └── Auto-detect framework → Dockerfile gen → Docker build
  │                              → ONNX optimise → push to MinIO registry
  │
  ├── Deployer
  │     └── Helm chart gen → K3s deploy → canary Traefik weights
  │
  ├── CanaryController
  │     └── Prometheus query → auto-rollback on error rate threshold
  │
  └── ManifestStore
        └── SQLite → package history / deployment log / events
```

### Lattice — ML Job Scheduler

```
REST API / gRPC
    │
    ▼
Scheduler Engine
    ├── FIFO (4 priority tiers: critical/high/normal/batch)
    ├── DRF  (Dominant Resource Fairness — per-team fair-share)
    ├── Gang (atomic all-or-nothing worker reservation)
    ├── Preemption (SIGUSR1 → checkpoint/restore)
    └── Backfill (fill idle capacity while large jobs wait)
    │
    ▼
Redis Sorted Set Queue ──► SQLite Job State Store
    │
    ▼
Worker Pool
    └── Docker containers with cgroup resource constraints
    │
    ▼
Prometheus (12 metrics) ──► Grafana
```

---

## Optional Integrations

These integrations show how the systems can work together. **They are optional — no project requires another to run.**

```
Argus can monitor Hermes inference metrics.
  → Point Argus at Hermes /metrics; track token counts, latency distributions.

Pyrex can benchmark Hermes backends.
  → Run Pyrex against Ollama backends that Hermes routes to.

Capsule can package and deploy Hermes-like model services.
  → Use Capsule to package a custom inference server, then route via Hermes.

Conduit can orchestrate ML workflows that use Strata features.
  → A Conduit DAG task can call strata.get() to fetch features at inference time.

Lattice can schedule training or inference jobs used by Conduit.
  → Conduit pipeline submits a training job to Lattice via the REST API.

Argus can receive drift signals from Strata-served features.
  → Log Strata feature distributions to Argus for drift tracking.
```

**Important**: These are optional integration patterns for portfolio demonstrations. In production use, each system operates independently.

---

## Design Principles

1. **Local-first** — designed for Apple Silicon, zero cloud dependency
2. **Independent deployability** — each project runs standalone with `docker compose up`
3. **No cross-project runtime dependencies** — Hermes does not require Argus; Strata does not require Conduit
4. **Production-style patterns** — circuit breakers, rate limiting, dead letter queues, ASOF joins, canary deployments
5. **Observable by default** — Prometheus metrics and Grafana dashboards across all projects that serve traffic
6. **Testable without dependencies** — every project's test suite runs without external services (mocked)
