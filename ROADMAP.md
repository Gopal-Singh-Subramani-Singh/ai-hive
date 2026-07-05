co# AI Hive Roadmap

## Current Status: v1.0 (July 2026)

All seven systems are functionally complete, tested, and documented. This is the initial public release.

---

## Completed (v1.0)

### Hermes
- ✅ 4 routing strategies (latency-aware, weighted RR, least-conn, priority)
- ✅ Per-backend circuit breakers
- ✅ Redis token bucket rate limiting
- ✅ Priority queue (implementation complete, not default routing path)
- ✅ SSE streaming proxy
- ✅ Live dashboard UI
- ✅ Prometheus + Grafana observability

### Argus
- ✅ 5 drift detection methods (KS, PSI, JS, Chi-squared, SHAP)
- ✅ Redis Streams ingestion pipeline
- ✅ TimescaleDB time-series storage
- ✅ DuckDB analytics API
- ✅ YAML alert rules engine
- ✅ Python SDK
- ✅ Prometheus + Grafana dashboards

### Pyrex
- ✅ 4 backends (PyTorch MPS, ONNX RT, MLX, CPU)
- ✅ 6 kernels (attention, FFN, matmul, layernorm, conv2d, embedding)
- ✅ Regression detection (z-score analysis)
- ✅ DuckDB result storage
- ✅ HTML report generation
- ✅ GitHub Actions CI integration

### Strata
- ✅ Redis online store
- ✅ DuckDB/Parquet offline store
- ✅ ASOF joins for point-in-time correctness
- ✅ Feature materialization
- ✅ Consistency validator
- ✅ Feature lineage DAG
- ✅ Python SDK
- ✅ gRPC batch API

### Conduit
- ✅ Python DAG DSL (@conduit.task / @conduit.dag)
- ✅ Topological scheduling (Kahn's algorithm)
- ✅ Redis Streams task queue
- ✅ Retry with exponential backoff
- ✅ Dead letter queue
- ✅ DLQ replay
- ✅ Resource quota manager
- ✅ REST API + Typer CLI

### Capsule
- ✅ Framework auto-detection
- ✅ Dockerfile generation
- ✅ FastAPI server generation
- ✅ ONNX optimization (INT8 quantization)
- ✅ MinIO model registry
- ✅ Helm chart generation
- ✅ K3s deployment support (implementation complete, local demo verification pending)
- ✅ Canary deployment
- ✅ Automatic rollback
- ✅ Deployment history

### Lattice
- ✅ FIFO scheduling (4 priority tiers)
- ✅ Dominant Resource Fairness (DRF)
- ✅ Gang scheduling
- ✅ Preemption with checkpoint/restore
- ✅ Backfill
- ✅ Worker pool simulation
- ✅ Redis + SQLite state management
- ✅ REST API + gRPC
- ✅ 200-job stress simulation
- ✅ FIFO vs DRF utilization report

---

## Planned Enhancements

### Short Term (Next 3-6 months)

#### Hermes
- [ ] Wire priority queue into default routing path
- [ ] Add request tracing (OpenTelemetry)
- [ ] Per-model-id rate limiting
- [ ] Support non-Ollama backends (OpenAI-compatible endpoints)
- [ ] Add HTTPS / mTLS support

#### Argus
- [ ] Alert deduplication and notification channels (Slack, PagerDuty)
- [ ] Replace TimescaleDB with DuckDB for lighter deployment
- [ ] Add online drift monitoring (streaming detection)
- [ ] Add data quality checks (null rates, type mismatches)
- [ ] Multi-tenant model registry with API key auth

#### Pyrex
- [ ] Add LLM inference benchmarks (Ollama or llama.cpp backends)
- [ ] Add throughput (tokens/sec) as first-class metric
- [ ] Add memory bandwidth analysis
- [ ] Support non-macOS backends (CUDA via torch.cuda)
- [ ] Add time-series charting for benchmark trends

#### Strata
- [ ] Schema evolution support (feature versioning)
- [ ] Streaming feature ingestion (Kafka / Flink compatibility)
- [ ] Feature serving SDK for multiple languages (Go, Java)
- [ ] Distributed online store (Redis Cluster)
- [ ] Feature monitoring integration with Argus

#### Conduit
- [ ] Distributed workers (multi-machine worker pool via gRPC)
- [ ] DAG versioning and rollback
- [ ] Web dashboard for pipeline visualization
- [ ] Per-task resource limits via Docker containers
- [ ] Sensor tasks (external triggers: S3, webhooks, time windows)
- [ ] DAG import from YAML config

#### Capsule
- [ ] End-to-end K3s demo verification with reproducible setup script
- [ ] Support ONNX Runtime as a serving backend
- [ ] Add model A/B testing with Prometheus-based automatic traffic promotion
- [ ] Multi-cluster deploy support
- [ ] Model performance regression gate during packaging

#### Lattice
- [ ] Real compute execution (actual Python subprocess or Docker exec)
- [ ] cgroup-based resource enforcement
- [ ] Multi-machine worker distribution (gRPC worker agents on remote hosts)
- [ ] Kubernetes operator mode (Lattice as a CRD controller)
- [ ] Job checkpointing storage in MinIO
- [ ] Priority inversion detection and resolution

---

## Long Term (6-12 months)

### Cross-Project Integrations
- [ ] **Hermes → Argus**: Live LLM gateway metrics streaming to drift detection
- [ ] **Pyrex → Hermes**: Automated backend selection based on benchmark results
- [ ] **Capsule → Argus**: Deploy models with automatic drift monitoring wiring
- [ ] **Conduit → Strata**: Pipeline templates that auto-wire feature retrieval
- [ ] **Lattice → Conduit**: Schedule jobs triggered by pipeline completion events

### Infrastructure Improvements
- [ ] Unified observability dashboard across all systems
- [ ] Shared authentication and authorization layer
- [ ] Multi-tenancy support across all systems
- [ ] Unified CLI (`aihive <command>`)
- [ ] Terraform / Pulumi deployment templates

### Documentation & Community
- [ ] Video tutorials for each system
- [ ] End-to-end integration tutorials
- [ ] Architecture decision records (ADRs)
- [ ] Contributing guide
- [ ] Code of conduct
- [ ] Issue templates
- [ ] PR templates

---

## Research & Experimental

These are exploratory ideas, not commitments:

- [ ] **WebAssembly model serving**: Package models as WASM for edge deployment
- [ ] **Federated feature store**: Multi-organization feature sharing with privacy
- [ ] **Auto-scaling based on drift**: Trigger retraining when drift exceeds threshold
- [ ] **LLM-powered query optimization**: Natural language → SQL for feature queries
- [ ] **Chaos engineering toolkit**: Automated failure injection across all systems
- [ ] **Cost optimization**: Track and optimize compute/storage costs in cloud deployment

---

## Non-Goals

AI Hive will **not**:
- Become a cloud-hosted service (local-first is core principle)
- Require Kubernetes for basic functionality (K3s optional)
- Support Windows (macOS/Linux only)
- Include model training (inference and infra only)
- Implement AutoML or hyperparameter tuning
- Become a monolith (independent repos maintained separately)

---

## Contributing

Contributions are welcome! See individual project repositories for contributing guidelines.

If you want to propose a new feature:
1. Open an issue in the relevant project repo
2. Describe the problem it solves
3. Explain how it fits the project's scope
4. Discuss implementation approach

---

## Version History

- **v1.0** (July 2026): Initial public release, all 7 systems functional

---

**Last Updated**: 2026-07-04
