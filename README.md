# AI Hive — Local-First AI Infrastructure Suite

AI Hive is a local-first AI infrastructure suite made of seven independently deployable systems for model serving, observability, benchmarking, feature management, orchestration, deployment, and scheduling.

The goal of AI Hive is to go beyond building AI applications and instead build the infrastructure layer around production AI systems. Each project is designed to run locally with zero required cloud dependency while demonstrating production-style AI infrastructure patterns.

---

## Projects

| Project                                                               | Role in AI Hive | Description                                                                                                           |
| --------------------------------------------------------------------- | --------------- | --------------------------------------------------------------------------------------------------------------------- |
| **[Hermes](https://github.com/Gopal-Singh-Subramani-Singh/hermes)**   | Serve           | Distributed LLM inference gateway with routing, circuit breakers, rate limiting, streaming, and observability         |
| **[Argus](https://github.com/Gopal-Singh-Subramani-Singh/argus)**     | Monitor         | ML observability and drift detection platform with statistical tests, alerts, and dashboards                          |
| **[Pyrex](https://github.com/Gopal-Singh-Subramani-Singh/pyrex)**     | Benchmark       | Cross-backend inference benchmarking suite for Apple Silicon, PyTorch MPS, ONNX Runtime, MLX, and CPU                 |
| **[Strata](https://github.com/Gopal-Singh-Subramani-Singh/strata)**   | Store Features  | Online/offline feature store with Redis serving, DuckDB/Parquet storage, and point-in-time joins                      |
| **[Conduit](https://github.com/Gopal-Singh-Subramani-Singh/conduit)** | Orchestrate     | Event-driven ML pipeline orchestrator with Python DAG DSL, Redis Streams, retries, and dead letter queues             |
| **[Capsule](https://github.com/Gopal-Singh-Subramani-Singh/capsule)** | Deploy          | Container-native model deployment platform with Dockerfile generation, MinIO registry, Helm/K3s, and canary workflows |
| **[Lattice](https://github.com/Gopal-Singh-Subramani-Singh/lattice)** | Schedule        | Distributed ML job scheduler simulation with fair-share scheduling, gang scheduling, preemption, and backfill         |

---

## Lifecycle

```text
serve → monitor → benchmark → store features → orchestrate pipelines → deploy models → schedule jobs
```

AI Hive maps the production AI infrastructure lifecycle into seven focused systems:

```text
Hermes  → serve models
Argus   → monitor model behavior
Pyrex   → benchmark inference performance
Strata  → manage online/offline features
Conduit → orchestrate ML workflows
Capsule → package and deploy models
Lattice → schedule ML jobs
```

---

## Architecture

```text
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

See [ARCHITECTURE.md](./ARCHITECTURE.md) for detailed architecture documentation.

---

## Tech Stack

**Languages:** Python
**Frameworks:** FastAPI, Flask
**Databases:** Redis, TimescaleDB, DuckDB, SQLite
**Storage:** MinIO, Parquet
**Orchestration:** Docker, Docker Compose, Kubernetes/K3s, Helm
**Observability:** Prometheus, Grafana
**ML/AI:** PyTorch, ONNX Runtime, Apple MLX, Ollama, SHAP
**Protocols:** HTTP/REST, gRPC, SSE

---

## Design Principles

1. **Local-first** — Designed for Apple Silicon and local development with zero required cloud dependency.
2. **Independent deployability** — Each project is designed to run as a standalone system.
3. **No hard runtime coupling** — Hermes does not require Argus; Strata does not require Conduit.
4. **Production-style patterns** — Circuit breakers, rate limiting, dead letter queues, ASOF joins, canary deployments, drift detection, and fair-share scheduling.
5. **Observable by default** — Systems expose metrics and include observability documentation where applicable.
6. **Testable** — Projects include pytest-based test suites and local validation workflows.

---

## Optional Integrations

Each system is independently runnable, but the projects can be composed into larger workflows:

* **Argus** can monitor metrics and model behavior from **Hermes**.
* **Pyrex** can benchmark model backends used by **Hermes**.
* **Capsule** can package and deploy model services monitored by **Argus**.
* **Conduit** can orchestrate workflows that use **Strata** features.
* **Lattice** can schedule jobs triggered by **Conduit** pipelines.

These are optional integrations, not hard dependencies.

---

## Quick Start

Each system is published as an independent repository. Clone a project and follow its README.

Example:

```bash
git clone https://github.com/Gopal-Singh-Subramani-Singh/hermes.git
cd hermes
pip install -r requirements.txt
python -m pytest tests -v
```

For project-specific setup, see the individual repositories:

* [Hermes](https://github.com/Gopal-Singh-Subramani-Singh/hermes)
* [Argus](https://github.com/Gopal-Singh-Subramani-Singh/argus)
* [Pyrex](https://github.com/Gopal-Singh-Subramani-Singh/pyrex)
* [Strata](https://github.com/Gopal-Singh-Subramani-Singh/strata)
* [Conduit](https://github.com/Gopal-Singh-Subramani-Singh/conduit)
* [Capsule](https://github.com/Gopal-Singh-Subramani-Singh/capsule)
* [Lattice](https://github.com/Gopal-Singh-Subramani-Singh/lattice)

---

## Integration Demo

See [INTEGRATION_DEMO.md](./INTEGRATION_DEMO.md) for examples of independent demos, paired demos, and full ecosystem narratives.

---

## Resume Snippets

See [RESUME_SNIPPETS.md](./RESUME_SNIPPETS.md) for resume-ready descriptions of AI Hive and selected systems.

---

## LinkedIn Post

See [LINKEDIN_POST.md](./LINKEDIN_POST.md) for a professional launch post draft.

---

## Repository Structure

AI Hive is published as separate repositories:

```text
ai-hive    # Umbrella landing repository
hermes     # LLM inference gateway
argus      # ML observability platform
pyrex      # Inference benchmark suite
strata     # Feature store
conduit    # Pipeline orchestrator
capsule    # Model deployment platform
lattice    # Job scheduler
```

The `ai-hive` repository is the landing page and portfolio hub. Source code lives in the individual project repositories.

---

## License

Each project contains its own license. Refer to the individual project repositories for license details.

---

## Contact

Built by **Gopal Singh Subramani Singh**

GitHub: [@Gopal-Singh-Subramani-Singh](https://github.com/Gopal-Singh-Subramani-Singh)

---

## Acknowledgments

AI Hive was built as a learning-focused infrastructure suite to explore production AI systems patterns across model serving, observability, deployment, scheduling, benchmarking, orchestration, and feature management.

---

**Last Updated:** 2026-07-04
