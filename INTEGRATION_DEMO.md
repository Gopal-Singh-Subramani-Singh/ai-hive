# AI Hive — Integration Demo Guide

This guide shows how the AI Hive projects can be demonstrated independently, in pairs, and as a full ecosystem narrative.

> **Note**: No single "run everything" script exists yet. The three levels below reflect honest demo maturity.

---

## Level 1 — Independent Demos

Each project runs standalone. This is the most reliable and recommended starting point for any demo or interview.

### Hermes — LLM Inference Gateway

```bash
cd hermes
pip install -r requirements.txt
docker run -d -p 6379:6379 redis:7-alpine

# Start gateway (works without Ollama — returns graceful errors)
uvicorn gateway.main:app --reload --host 0.0.0.0 --port 8000

# View dashboard
open http://localhost:8000/ui

# Health check
curl http://localhost:8000/health

# Metrics
curl http://localhost:8000/metrics

# Run tests (no deps needed)
pytest tests/ -v
```

**What this proves**: Circuit breaker logic, routing strategies, rate limiting, priority queue, SSE streaming architecture.

---

### Argus — ML Drift Detection

```bash
cd Argus/argus
cp .env.example .env
docker compose up timescaledb redis prometheus grafana -d
uvicorn argus_core.main:app --port 8001 --reload

# Register a model
curl -X POST http://localhost:8001/models \
  -H "Content-Type: application/json" \
  -d '{"model_id": "demo_model", "name": "Demo", "version": "1.0",
       "features": [{"name": "score", "type": "numeric"}]}'

# Run synthetic drift demo
python demo/synthetic_drift_demo.py

# View Grafana drift dashboard
open http://localhost:3000   # admin / argus
```

**What this proves**: KS test, PSI, Jensen-Shannon divergence, alert rules, Prometheus metrics.

---

### Pyrex — Inference Benchmarking

```bash
cd Pyrex/pyrex
pip install -r requirements.txt && pip install -e .

# Quick benchmark (no special hardware needed)
pyrex run --quick

# Save as baseline
pyrex baseline --name baseline

# Run full benchmark
pyrex run

# Generate HTML report
pyrex report <run_id>

# Run tests
pytest tests/ -v
```

**What this proves**: Cross-backend comparison, regression detection, roofline analysis, DuckDB result storage.

---

### Strata — Feature Store

```bash
cd Strata/strata
pip install -r requirements.txt
docker compose up redis minio prometheus grafana -d
uvicorn strata_core.main:app --port 8003 --reload

# Run fraud feature store demo
python demo/fraud_feature_store.py

# API docs
open http://localhost:8003/docs

# Run tests
pytest tests/ -v
```

**What this proves**: Online Redis serving, DuckDB ASOF JOIN, feature materialization, lineage tracking.

---

### Conduit — Pipeline Orchestrator

```bash
cd conduit/conduit
pip install -r requirements.txt
docker compose up redis prometheus grafana -d
uvicorn conduit_api.main:app --port 8004 --reload

# Run ML training pipeline demo
python demo/ml_training_pipeline.py

# CLI usage
conduit dags
conduit run my_dag
conduit dlq

# Run tests
pytest tests/ -v
```

**What this proves**: DAG DSL, topological scheduling, Redis Streams queue, dead letter queue, retry backoff.

---

### Capsule — Model Deployment

```bash
cd Capsule/capsule
pip install -r requirements.txt && pip install -e .
brew install helm
docker compose up minio registry -d

# Train demo model
python examples/fraud_detector/train_model.py

# Package model
capsule package --manifest examples/fraud_detector/capsule.yaml

# List packages
capsule list

# Run tests (no K3s needed)
pytest tests/ -v
```

**What this proves**: Model manifest, Dockerfile generation, ONNX optimization, MinIO registry, Helm chart generation.

> ⚠️ **K3s deployment note**: K3s deployment support is implemented. Local K3s demo verification is pending — requires K3s installation (`curl -sfL https://get.k3s.io | sh -`).

---

### Lattice — Job Scheduler

```bash
cd Lattice/lattice
pip install -r requirements.txt
docker compose up redis prometheus grafana -d
uvicorn lattice.api.rest_api:app --port 8002 --reload

# Submit a job
curl -X POST http://localhost:8002/jobs \
  -H "Content-Type: application/json" \
  -d '{"team": "team_A", "name": "bert-finetune",
       "priority": 2, "cpu_cores": 2.0, "memory_gb": 4.0,
       "num_workers": 1, "estimated_duration_seconds": 300}'

# Run 200-job stress simulation
python simulation/stress_test.py

# Generate FIFO vs DRF utilisation chart
python simulation/utilisation_report.py

# Run tests (no Docker needed)
pytest tests/ -v
```

**What this proves**: FIFO, DRF, gang scheduling, preemption, backfill, Redis queue, Prometheus metrics.

---

## Level 2 — Optional Paired Demos

These are integration scenarios that work well together. They require both systems to be running.

### Hermes + Pyrex — Benchmark the Gateway

Run Pyrex benchmarks against the Ollama backends that Hermes routes to.

```bash
# Terminal 1: Start Hermes
cd hermes && uvicorn gateway.main:app --host 0.0.0.0 --port 8000

# Terminal 2: Benchmark with Pyrex
cd Pyrex/pyrex
pyrex run --backends pytorch_mps,pytorch_cpu --kernels matmul,attention

# Compare results against baseline
pyrex compare baseline <run_id>
```

**Story**: Pyrex gives you empirical data to choose which Ollama backend Hermes should prefer. Run before configuring routing weights.

---

### Hermes + Argus — Monitor the Gateway

Point Argus at Hermes metrics to track inference distribution drift.

```bash
# Terminal 1: Hermes running on :8000
# Terminal 2: Argus running on :8001

# Log inference feature distributions to Argus
# (manual API calls or SDK integration)
curl -X POST http://localhost:8001/ingest \
  -H "Content-Type: application/json" \
  -d '{"model_id": "hermes_gateway", "features": {"latency_ms": 45.0,
       "tokens": 120}, "prediction": 1}'

# View drift dashboard
open http://localhost:3000
```

**Story**: Argus watches whether the latency distribution and token counts from Hermes requests shift over time — a proxy for backend degradation.

---

### Conduit + Strata — Feature-Driven Pipeline

A Conduit DAG task fetches features from Strata during pipeline execution.

```bash
# Terminal 1: Strata running on :8003
# Terminal 2: Conduit running on :8004

# Conduit pipeline task using Strata features:
# import sdk as strata
# strata.init("http://localhost:8003")
# features = strata.get("user_001", ["tx_count_1h", "avg_amount_7d"])

python demo/ml_training_pipeline.py
```

**Story**: The ML training pipeline (Conduit) fetches point-in-time correct features from Strata before running model training. No feature leakage.

---

### Lattice + Conduit — Scheduled Job Submission

Conduit pipelines submit compute-heavy tasks to Lattice for fair scheduling.

```bash
# Terminal 1: Lattice running on :8002
# Terminal 2: Conduit running on :8004

# A Conduit DAG task submits a training job to Lattice
curl -X POST http://localhost:8002/jobs \
  -H "Content-Type: application/json" \
  -d '{"team": "ml_team", "name": "conduit-training-job",
       "priority": 2, "cpu_cores": 4.0, "memory_gb": 8.0,
       "num_workers": 2, "estimated_duration_seconds": 600}'

# Monitor scheduling
curl http://localhost:8002/cluster
```

**Story**: Conduit orchestrates the pipeline DAG; Lattice allocates cluster resources fairly across competing training jobs.

---

### Capsule + K3s — Full Deploy Cycle

Package a model with Capsule and deploy to a local K3s cluster.

```bash
# Prerequisites: K3s installed, minio running
cd Capsule/capsule
python examples/fraud_detector/train_model.py
capsule package --manifest examples/fraud_detector/capsule.yaml
capsule deploy fraud-detector:1.0
capsule status fraud-detector
capsule deploy fraud-detector:2.0 --canary 10   # canary at 10%
capsule rollback fraud-detector --yes           # rollback if needed
```

**Story**: Full model lifecycle from training → packaging → canary deployment → rollback.

---

## Level 3 — Full Ecosystem Narrative

This is the high-level architecture story for resume, LinkedIn, and interviews. Not a single runnable demo, but a coherent narrative backed by independent implementations.

### The Story

> "I built AI Hive — a local-first AI infrastructure suite that covers the complete lifecycle of production ML systems."

**Step 1 — Choose your serving infrastructure with Pyrex**

Before deciding which Ollama backends to use, run Pyrex benchmarks across PyTorch MPS, ONNX Runtime, MLX, and CPU. Compare latency, throughput, and memory. Save a baseline. Detect regressions on every PR via GitHub Actions CI.

**Step 2 — Build your feature store with Strata**

Define features using the Python DSL. Strata serves them via Redis for real-time inference (online) and via DuckDB ASOF JOINs for training data (offline). Point-in-time correctness is enforced — no future leakage. Features materialize on schedule from offline to online.

**Step 3 — Orchestrate your training pipeline with Conduit**

Define your ML workflow as a DAG using `@conduit.task` and `@conduit.dag`. Conduit resolves dependencies topologically, dispatches tasks to workers via Redis Streams, retries with exponential backoff, parks unrecoverable failures in a dead letter queue.

**Step 4 — Schedule training jobs fairly with Lattice**

When the pipeline submits compute-heavy jobs, Lattice allocates worker capacity fairly across teams using Dominant Resource Fairness. Gang scheduling ensures multi-worker jobs start atomically. Backfill fills idle gaps while large jobs wait.

**Step 5 — Package and deploy models with Capsule**

After training, Capsule auto-detects the model framework, generates a Dockerfile, optimises to ONNX, pushes to the MinIO registry, generates a Helm chart, and deploys to K3s with canary traffic splitting. Rollback is one command.

**Step 6 — Serve LLM inference with Hermes**

Route inference requests across multiple Ollama backends using latency-aware routing and weighted round-robin. Each backend has an independent circuit breaker. Redis-backed token buckets rate-limit per tier. Priority queues ensure premium requests are never starved.

**Step 7 — Monitor everything with Argus**

Point Argus at any model producing predictions. Ingestion flows through Redis Streams into TimescaleDB. The drift engine runs KS tests, PSI, Chi-squared, Jensen-Shannon divergence on a schedule. YAML alert rules fire webhooks and retraining triggers. Grafana dashboards surface everything.

---

### What This Demonstrates

| Pattern | Where |
|---------|-------|
| Circuit breakers | Hermes |
| Token bucket rate limiting | Hermes |
| Statistical drift detection | Argus |
| Event-driven streaming | Argus, Conduit |
| Point-in-time correct joins | Strata |
| Feature materialization | Strata |
| DAG orchestration | Conduit |
| Dead letter queues | Conduit |
| Fair-share scheduling | Lattice |
| Gang scheduling | Lattice |
| Canary deployment | Capsule |
| Model versioning | Capsule |
| Cross-backend benchmarking | Pyrex |
| Regression detection | Pyrex |
| Prometheus observability | All (except Pyrex CLI) |

---

*Each project is independently deployable. These integrations are optional demonstrations, not hard runtime dependencies.*
