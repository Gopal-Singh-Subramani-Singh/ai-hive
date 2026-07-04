# AI Hive Projects

Detailed overview of each project in the AI Hive suite.

---

## Hermes — Distributed LLM Inference Gateway

### Problem It Solves
Running multiple LLM backends locally requires more than just a load balancer. You need fault isolation (circuit breakers), traffic shaping (rate limiting), quality-of-service (priority queues), and real-time observability.

### Core Systems Concepts
- **Circuit breakers**: CLOSED → OPEN → HALF_OPEN state machine
- **Rate limiting**: Redis token bucket with atomic Lua scripts
- **Routing strategies**: Latency-aware (EWMA), weighted round-robin, least-connections, priority-based
- **SSE streaming**: Real-time event streaming with Time-To-First-Token tracking
- **Health checking**: Async background monitoring with EWMA latency tracking

### Tech Stack
Python · FastAPI · Redis · Ollama · Prometheus · Grafana · Docker · Locust · asyncio · SSE

### Best Interview Topics
- Distributed systems patterns (circuit breakers, rate limiting)
- Load balancing algorithms
- Real-time streaming protocols (SSE)
- Observability and metrics

### Repository
[github.com/YOUR_USERNAME/hermes](https://github.com/YOUR_USERNAME/hermes)

---

## Argus — ML Observability & Drift Detection Platform

### Problem It Solves
Model accuracy degrades silently in production. Input distributions shift. Upstream data pipelines change. Without monitoring, you discover degradation through user complaints or business metrics — too late.

### Core Systems Concepts
- **Drift detection**: KS test, PSI, Jensen-Shannon divergence, Chi-squared, SHAP feature importance
- **Event ingestion**: Redis Streams for non-blocking SDK with at-least-once delivery
- **Time-series storage**: TimescaleDB for historical drift analysis
- **Alert engine**: YAML-based threshold rules with webhook actions
- **Analytics**: DuckDB for 30-day drift history queries

### Tech Stack
Python · FastAPI · Redis Streams · TimescaleDB · DuckDB · APScheduler · Prometheus · Grafana · Docker · SciPy · SHAP

### Best Interview Topics
- Statistical drift detection methods
- Time-series data storage and querying
- Event-driven architecture
- ML model monitoring in production

### Repository
[github.com/YOUR_USERNAME/argus](https://github.com/YOUR_USERNAME/argus)

---

## Pyrex — Cross-Backend ML Inference Benchmark Suite

### Problem It Solves
Choosing the wrong inference backend on Apple Silicon leaves performance on the table. MPS, MLX, ONNX Runtime, and CPU have different characteristics across kernel types — what is fastest for matmul may not be fastest for attention.

### Core Systems Concepts
- **Benchmark harness**: Systematic measurement across backends and kernels
- **Regression detection**: Z-score analysis against saved baselines
- **Telemetry**: MPS memory, CPU RSS, CPU %, optional power via powermetrics
- **Result storage**: DuckDB + Parquet for queryable history
- **CI integration**: GitHub Actions workflow gates PRs on regressions

### Tech Stack
Python · PyTorch (MPS + CPU) · ONNX Runtime · Apple MLX · DuckDB · Parquet · Jinja2 · matplotlib · Typer · GitHub Actions

### Best Interview Topics
- Performance benchmarking methodology
- Memory and CPU profiling
- Statistical regression detection
- Cross-platform optimization

### Repository
[github.com/YOUR_USERNAME/pyrex](https://github.com/YOUR_USERNAME/pyrex)

---

## Strata — Distributed Feature Store

### Problem It Solves
Training-serving skew is one of the most common sources of silent model degradation. It happens when training data uses feature values that would not have been available at prediction time — a form of future leakage.

### Core Systems Concepts
- **Dual-store architecture**: Redis online store + DuckDB/Parquet offline store
- **Point-in-time correctness**: ASOF joins ensure no future leakage
- **Feature materialization**: Offline → online sync on APScheduler schedule
- **Consistency validation**: Samples online vs offline, flags mismatches
- **Feature lineage**: DAG tracking upstream sources and downstream consumers

### Tech Stack
Python · FastAPI · Redis · DuckDB · Parquet · MinIO · SQLite · APScheduler · Prometheus · Grafana · gRPC · Docker

### Best Interview Topics
- Feature stores and ML infrastructure
- Point-in-time correctness and ASOF joins
- Online/offline serving patterns
- Data consistency in distributed systems

### Repository
[github.com/YOUR_USERNAME/strata](https://github.com/YOUR_USERNAME/strata)

---

## Conduit — Event-Driven ML Pipeline Orchestrator

### Problem It Solves
ML pipelines have hard requirements: task ordering, retries on transient failures, forensics on permanent failures, and resource guards to prevent OOM. 

### Core Systems Concepts
- **DAG scheduling**: Kahn's algorithm for topological sort with cycle detection
- **Event-driven execution**: Redis Streams task queue with at-least-once delivery
- **Retry logic**: Exponential backoff + jitter, configurable per-task
- **Dead letter queue**: Exhausted tasks parked with full forensics
- **Resource quotas**: psutil-based CPU and memory caps

### Tech Stack
Python · FastAPI · Redis Streams · SQLite · APScheduler · Prometheus · Grafana · Docker · asyncio · Typer

### Best Interview Topics
- Workflow orchestration and DAG scheduling
- At-least-once delivery semantics
- Dead letter queues and error handling
- Resource management in distributed systems

### Repository
[github.com/YOUR_USERNAME/conduit](https://github.com/YOUR_USERNAME/conduit)

---

## Capsule — Container-Native Model Deployment Platform

### Problem It Solves
Deploying an ML model to production requires solving half a dozen infrastructure problems that have nothing to do with the model itself: containerization, image registry, Kubernetes manifests, traffic routing, version management, and rollback.

### Core Systems Concepts
- **Model packaging**: Auto-detect framework, generate Dockerfile and FastAPI server
- **ONNX optimization**: Convert PyTorch → ONNX with optional INT8 quantization
- **Model registry**: MinIO-backed versioned artifact storage
- **Helm deployment**: Generate Kubernetes manifests, deploy to K3s
- **Canary rollout**: Traefik weighted traffic splitting with auto-rollback

### Tech Stack
Python · Typer · Docker · Helm · Kubernetes (K3s) · MinIO · ONNX Runtime · FastAPI · Prometheus · SQLite · Rich

### Best Interview Topics
- Model deployment workflows
- Container orchestration (Kubernetes)
- Canary deployments and rollback strategies
- Model registry and versioning

### Repository
[github.com/YOUR_USERNAME/capsule](https://github.com/YOUR_USERNAME/capsule)

---

## Lattice — Distributed ML Job Scheduler

### Problem It Solves
Production ML clusters face scheduling problems that naive FIFO queues cannot solve: large jobs starving small ones, teams monopolizing shared resources, multi-GPU jobs needing atomic allocation, preempting low-priority jobs to unblock critical ones.

### Core Systems Concepts
- **Fair-share scheduling**: Dominant Resource Fairness (DRF) for per-team allocation
- **Gang scheduling**: Atomic all-or-nothing multi-worker reservation
- **Preemption**: SIGUSR1-based checkpoint/restore for low-priority job eviction
- **Backfill**: Fill idle capacity while large jobs wait in queue
- **Simulation**: Simulates multi-worker cluster with Docker containers and resource constraints

### Tech Stack
Python · FastAPI · gRPC · Redis · SQLite · Docker · Prometheus · Grafana · asyncio · Pydantic

### Best Interview Topics
- Job scheduling algorithms (FIFO, DRF, gang, preemption, backfill)
- Resource allocation in distributed systems
- Fair-share scheduling and multi-tenancy
- Cluster simulation and modeling

### Repository
[github.com/YOUR_USERNAME/lattice](https://github.com/YOUR_USERNAME/lattice)

---

## Comparison Matrix

| Feature | Hermes | Argus | Pyrex | Strata | Conduit | Capsule | Lattice |
|---------|--------|-------|-------|--------|---------|---------|---------|
| **Domain** | Serving | Monitoring | Benchmarking | Features | Orchestration | Deployment | Scheduling |
| **Primary Language** | Python | Python | Python | Python | Python | Python | Python |
| **Primary Framework** | FastAPI | FastAPI | Typer | FastAPI | FastAPI | Typer | FastAPI |
| **Storage** | Redis | TimescaleDB, Redis | DuckDB | Redis, DuckDB | Redis, SQLite | MinIO, SQLite | Redis, SQLite |
| **Observability** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **REST API** | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ✅ |
| **CLI** | ❌ | ❌ | ✅ | ❌ | ✅ | ✅ | ❌ |
| **gRPC** | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ✅ |
| **Docker Required** | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ | ✅ |
| **K8s Integration** | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| **Apple Silicon** | ✅ | ✅ | Required | ✅ | ✅ | ✅ | ✅ |

---

**Note**: Replace `YOUR_USERNAME` with your actual GitHub username after creating repositories.
