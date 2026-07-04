# AI Hive — Resume Snippets

Three ready-to-use resume versions. Pick the one that fits your target role and space constraints.

---

## Version 1 — One Umbrella Project (recommended for most roles)

**AI Hive — Local-First AI Infrastructure Suite** | Python · FastAPI · Redis · Docker · Prometheus · Grafana · DuckDB · MinIO · K3s · Ollama

- Built a 7-project AI infrastructure suite covering LLM inference serving, ML observability, benchmarking, feature storage, pipeline orchestration, model deployment, and distributed job scheduling.
- Developed independently deployable systems using Python, FastAPI, Redis, Docker, Prometheus, Grafana, DuckDB, MinIO, Kubernetes/K3s, and Ollama.
- Implemented production-style infrastructure patterns including circuit breakers, rate limiting, drift detection, point-in-time joins, DAG scheduling, canary deployment, and fair-share scheduling.

---

## Version 2 — Umbrella + Selected Systems (for ML/MLOps roles)

**AI Hive — Local-First AI Infrastructure Suite** | Python · FastAPI · Redis · Docker · Prometheus · DuckDB · MinIO · K3s

- Built an independently deployable AI infrastructure ecosystem spanning serving, monitoring, benchmarking, feature management, orchestration, deployment, and scheduling.
- **Hermes**: Built a production-style LLM inference gateway routing requests across multiple Ollama backends using latency-aware routing, circuit breakers, Redis-backed token bucket rate limiting, and SSE streaming.
- **Argus**: Built an ML observability platform for monitoring model health and feature drift using statistical drift tests (KS, PSI, JS divergence), event-driven ingestion via Redis Streams, and Prometheus/Grafana dashboards.
- **Strata**: Built a local-first feature store with Redis online serving, DuckDB/Parquet offline storage, point-in-time correct ASOF joins, and a Python SDK for batch and low-latency feature retrieval.
- **Capsule**: Built a container-native model deployment platform with model manifests, Dockerfile generation, MinIO-backed versioning, Helm/K3s deployment workflows, and rollback support.
- **Lattice**: Built a distributed ML job scheduler with simulated multi-worker execution, fair-share scheduling (DRF), gang scheduling, preemption, backfill, Redis-backed state, and Prometheus observability.

---

## Version 3 — Short One-Liner (for summary sections or limited space)

Built **AI Hive**, a local-first AI infrastructure suite of 7 independently deployable systems covering LLM serving, observability, benchmarking, feature stores, pipeline orchestration, model deployment, and distributed scheduling.

---

## Per-Project Resume Bullets

Use these individually when highlighting specific systems.

**Hermes**
> Built a production-style LLM inference gateway routing requests across multiple Ollama backends using latency-aware routing, circuit breakers, Redis-backed token bucket rate limiting, and SSE streaming.

**Argus**
> Built an ML observability platform for monitoring model health and feature drift using statistical drift tests, event-driven ingestion, historical analytics, and Prometheus/Grafana dashboards.

**Pyrex**
> Built a cross-backend ML inference benchmark suite comparing PyTorch MPS, ONNX Runtime, MLX, and CPU execution with latency, throughput, memory, and regression analysis on Apple Silicon.

**Strata**
> Built a local-first feature store with Redis online serving, DuckDB/Parquet offline storage, point-in-time correct joins, and a Python SDK for batch and low-latency feature retrieval.

**Conduit**
> Built an event-driven ML pipeline orchestrator with a Python DAG DSL, Redis Streams task queue, dependency-aware scheduling, retries, dead letter queues, and Prometheus-based execution monitoring.

**Capsule**
> Built a container-native model deployment platform with model manifests, Dockerfile generation, MinIO-backed versioning, Helm/K3s deployment workflows, and rollback support.

**Lattice**
> Built a distributed ML job scheduler with simulated multi-worker execution, fair-share scheduling, gang scheduling, preemption, backfill, Redis-backed state, and Prometheus observability.

---

## Interview Talking Points

**"Walk me through AI Hive."**

> AI Hive is a 7-project local-first infrastructure suite I built to go beyond building AI apps and instead implement the infrastructure layer around production AI systems. Each project solves a distinct problem: Hermes handles LLM inference routing, Argus detects model drift, Pyrex benchmarks backends, Strata provides point-in-time correct features, Conduit orchestrates ML pipelines as DAGs, Capsule handles model packaging and deployment to Kubernetes, and Lattice schedules jobs fairly across a simulated cluster. Every system is independently deployable and tested.

**"What was the hardest part?"**

> The circuit breaker in Hermes required careful state machine design — CLOSED, OPEN, and HALF_OPEN states with configurable thresholds and EWMA-weighted latency tracking. The point-in-time join in Strata was also non-trivial — using DuckDB's native ASOF JOIN to guarantee that training data only uses feature values available at the time of the label event, preventing future leakage.

**"Why local-first?"**

> I wanted to focus on systems architecture, not cloud billing. Running everything on Apple Silicon with Docker Compose means I can demonstrate real infrastructure patterns — Redis streams, circuit breakers, Kubernetes deployments — without any cloud dependency or cost. It also makes everything reproducible for anyone with a modern MacBook.

**"How does the latency-aware routing in Hermes work?"**

> The router builds an EWMA over each backend's health check ping latency using alpha=0.1, then picks `min(backends, key=ewma_latency)` per request. Alpha=0.1 is conservative by design — it avoids overreacting to a single slow health check in production. On a single-machine demo all backends share one Ollama instance so routing decisions are cosmetic, but in a real multi-GPU deployment each backend is a separate Ollama instance and the EWMA meaningfully reflects backend saturation. The circuit breaker, rate limiter, and queue are all exercised regardless.

**"Why does the Hermes dashboard show Degraded on startup?"**

> The health checker fires immediately on startup and marks backends unhealthy if Ollama hasn't loaded the model into memory yet. It self-heals within one health check interval (10 seconds by default). In production you'd configure a readiness probe delay or use Kubernetes liveness/readiness probes to prevent traffic before the first successful health check — which is exactly what Capsule's K3s deployment manifests do.

**"What would you change about Hermes if you were building it for production?"**

> Three things: First, I'd track inference latency in the EWMA rather than health check latency — that requires wrapping the actual forward pass timer and reporting it back to the router asynchronously. Second, I'd separate backend health (can it respond at all?) from routing weight (how fast is it right now?), because today OPEN circuit breaker and high EWMA are conflated into "don't use this backend." Third, I'd add shadow traffic — duplicate a small percentage of requests to a canary backend and compare latency distributions before shifting weight.

---

## Verified Live Metrics

The following metrics were verified on real running instances using `run_live_metrics.py`:

- **Hermes**: 3.7ms p50 health check latency, 7.3ms p50 end-to-end chat latency, verified circuit breaker trip state.
- **Argus**: 4.75ms p50 health check latency.
- **Pyrex**: Executed real multi-backend benchmarks with successful status in 4.5s.
- **Strata**: 2.98ms p50 health check latency.
- **Conduit**: 5.78ms p50 health check latency.
- **Capsule**: 59.85s model package duration, successful Dockerfile and ONNX generation.
