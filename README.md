<p align="center">
  <img src="./assets/header.svg" alt="Nico Ostermann, Data and Feature Infrastructure Engineer" width="100%" />
</p>

<p align="center">
  <b>I write down what I learn, every day. Right now I'm building a feature platform on Flink, Kafka, Redis and Cassandra.</b><br>
  Open to work in feature infrastructure and ML platforms.
</p>

<p align="center">
  <a href="./til/README.md"><img src="https://img.shields.io/badge/📓_Today_I_Learned-Daily_log-E50914?style=for-the-badge" /></a>
  <a href="https://github.com/oster-dev/featureforge"><img src="https://img.shields.io/badge/🚀_FeatureForge-Main_project-1a1a1a?style=for-the-badge" /></a>
  <a href="#the-roadmap"><img src="https://img.shields.io/badge/🗺️_Roadmap-8_months-222?style=for-the-badge" /></a>
  <a href="mailto:ostermann.dev@gmail.com"><img src="https://img.shields.io/badge/✉️_Contact-D14836?style=for-the-badge" /></a>
</p>

---

## My daily log

Since April 13, 2026 I write a Today I Learned entry. It's my notebook: what I learned, what I built, what broke. 

**[→ Read the full log](./til/README.md)**

---

## Activity

<p align="center">
  <img src="https://streak-stats.demolab.com?user=oster-dev&theme=dark&hide_border=true&background=0d1117&sideLabels=ffffff&sideNums=ffffff&dates=8b949e&hide_current_streak=true&hide_longest_streak=true&card_width=260&card_height=170" alt="Total contributions" />
</p>

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/oster-dev/oster-dev/output/github-snake-dark.svg" />
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/oster-dev/oster-dev/output/github-snake.svg" />
    <img alt="Contribution snake" src="https://raw.githubusercontent.com/oster-dev/oster-dev/output/github-snake.svg" />
  </picture>
</p>

---

## The roadmap

Eight months, April to November 2026. I planned it at the start and track it here. Every month ends with a milestone and a short review. The plan changed a little along the way, and the reviews say where.

| Month | Focus | Milestone | Status |
| :-- | :-- | :-- | :-- |
| 1 · Apr | Python, SQL, Git, Linux | Foundation, two pipeline projects | ✅ [Review](./monthly-updates/month-01.md) |
| 2 · May | AWS CLF-C02, Docker | First AWS certification | ✅ [Review](./monthly-updates/month-02.md) |
| 3 · Jun | AWS DEA-C01, Spark, Scala literacy | Data engineering foundation | ✅ [Review](./monthly-updates/month-03.md) |
| 4 · Jul | Kafka, Airflow, system design | Streaming fundamentals | ✅ [Review](./monthly-updates/month-04.md) |
| 5 · Aug | AWS SAA-C03, FeatureForge launch | FeatureForge v0.1.0 | ✅ [Review](./monthly-updates/month-05.md) |
| 6 · Sep | MLflow, Metaflow, open source | Lifecycle lab, merged MLflow PR | ✅ [Review](./monthly-updates/month-06.md) |
| 7 · Oct | FeatureForge streaming | Generic spec-driven Flink job, recovery tests | ✅ |
| 8 · Nov | FeatureForge spec platform, applications | Iceberg datasets, Cassandra, a serving service with measured latency | 🔄 In progress |

**Next:** more spec operators and composite keys, operations (alerts, recovery drills, a soak run), practitioner tooling and a v1.0 release. The order and the proof for each step are in [`STATUS.md`](https://github.com/oster-dev/featureforge/blob/main/STATUS.md).

---

## FeatureForge

**[FeatureForge](https://github.com/oster-dev/featureforge)** is my main project. You describe a feature in a YAML file. The platform computes it from a Kafka stream with Flink, writes it to Redis or Cassandra, serves it through a service that returns a status for every value, and builds point-in-time training data from an Iceberg history. It runs on my laptop with Docker Compose and synthetic data, so it's a local build and not a production system.

<p align="center">
  <img src="./assets/pipeline.svg" alt="FeatureForge data flow: a YAML spec drives a generic Flink job. Kafka events pass a validation job, the Flink job and a materializer into Redis or Cassandra, and a feature service reads them. Iceberg tables hold the event facts and the publication history, and a Spark dataset builder writes training datasets." width="100%" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Redis-4,000_req%2Fs_at_p99_5.1_to_7.3_ms-1a1a1a?style=flat-square" />
  <img src="https://img.shields.io/badge/Cassandra-1,500_req%2Fs_at_p99_14.2_to_14.8_ms-1a1a1a?style=flat-square" />
  <img src="https://img.shields.io/badge/CI-GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white" />
  <img src="https://img.shields.io/badge/runs_on-Docker_Compose-2496ED?style=flat-square&logo=docker&logoColor=white" />
</p>

The latency numbers are local measurements: one laptop, one service process, Redis and Cassandra in Docker. They are not production numbers.

Here is what works today, and which test or document proves it:

| What works | Proof |
| :-- | :-- |
| One generic Flink job runs features from YAML specs, no code per feature | [`STATUS.md`](https://github.com/oster-dev/featureforge/blob/main/STATUS.md) |
| A feature keyed by content instead of user, from a spec alone, matching hand-calculated windows | `test_generic_job_content_play_count.py` |
| Event time, late events, dedupe, replays, conflicting events | `tests/integration/` |
| Savepoint resume and recovery after a TaskManager crash | `test_generic_job_recovery.py`, `test_generic_job_checkpoint_recovery.py` |
| Redis writes that stay correct under concurrency | `test_streaming_redis_online_store*.py` |
| Redis and Cassandra behind one store contract, and the same spec runs on both by changing one setting | `test_online_store_contract.py`, `test_cassandra_end_to_end.py` |
| A serving service with a status per value (`ok`, `missing`, `stale`, `backend_error`), never a silent zero, that survives a stopped or slow Redis | `test_feature_service_redis.py`, `test_feature_service_faults.py` |
| Training datasets in two explicit modes (event time and published as of) from an Iceberg history, with a pinned snapshot | `test_training_dataset_published.py`, `test_event_writer_kafka.py` |
| Measured latency and the saturation point on one laptop | [`docs/benchmarks.md`](https://github.com/oster-dev/featureforge/blob/main/docs/benchmarks.md) |
| `validate-spec` and `test-spec` with clear exit codes, no Docker needed | `test_cli_validate_spec.py`, `test_cli_test_spec.py` |

**What it can't do yet:** it runs locally on synthetic data, so I have no production numbers. Flink and Spark support count and ratio features only, JobManager failure is not proven, there are no alert rules or soak run yet, and the Kafka sinks are at-least-once, so I don't claim exactly-once.

<details>
<summary><b>Full architecture diagram</b></summary>

<br>

```mermaid
flowchart LR
  S[YAML feature spec] --> V[validate-spec · test-spec]
  S --> J[Generic Flink job]
  S --> BF[Spark backfill]
  E[Kafka events] --> J
  E --> W[Event writer]
  J --> F[Kafka feature topic]
  F --> M[Materializer]
  M --> R[(Redis)]
  M --> C[(Cassandra)]
  M --> H[(Iceberg history)]
  W --> EF[(Iceberg event facts)]
  R --> SV[Feature service]
  C --> SV
  H --> D[Spark dataset builder]
  EF --> D
  D --> T[Training dataset · Parquet + manifest]
```

</details>

### Where it stands

The percentages are my own estimate.

| What | Progress | Note |
| :-- | :-- | :-- |
| Streaming core (Kafka, Flink, Redis) | ![](https://geps.dev/progress/95?dangerouscolor=E50914&color=E50914) | Done |
| Declarative specs and generic job | ![](https://geps.dev/progress/65?dangerouscolor=E50914&color=E50914) | Count and ratio only, more operators next |
| Recovery | ![](https://geps.dev/progress/60?dangerouscolor=E50914&color=E50914) | TaskManager and savepoint done, JobManager open |
| Iceberg history and training datasets | ![](https://geps.dev/progress/85?dangerouscolor=E50914&color=E50914) | Both modes done, operators beyond count open |
| Cassandra as a second online store | ![](https://geps.dev/progress/90?dangerouscolor=E50914&color=E50914) | Behind one contract, a Compose service is open |
| Serving service with p99 latency | ![](https://geps.dev/progress/85?dangerouscolor=E50914&color=E50914) | Measured on one laptop, one process |
| Operations | ![](https://geps.dev/progress/30?dangerouscolor=E50914&color=E50914) | Metrics and a dashboard, no alerts or soak run |
| Catalog, embeddings, ML workflow | ![](https://geps.dev/progress/0?dangerouscolor=E50914&color=E50914) | Planned |

---

## Tools I use

<p align="center">
<img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
<img src="https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=postgresql&logoColor=white" />
<img src="https://img.shields.io/badge/Flink-E6526F?style=for-the-badge&logo=apacheflink&logoColor=white" />
<img src="https://img.shields.io/badge/Kafka-231F20?style=for-the-badge&logo=apachekafka&logoColor=white" />
<img src="https://img.shields.io/badge/Spark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white" />
<img src="https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white" />
<img src="https://img.shields.io/badge/Cassandra-1287B1?style=for-the-badge&logo=apachecassandra&logoColor=white" />
<img src="https://img.shields.io/badge/Iceberg-4A90D9?style=for-the-badge" />
<img src="https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white" />
<img src="https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white" />
<img src="https://img.shields.io/badge/Feast-FF6B35?style=for-the-badge" />
<img src="https://img.shields.io/badge/Airflow-017CEE?style=for-the-badge&logo=apacheairflow&logoColor=white" />
<img src="https://img.shields.io/badge/MLflow-0194E2?style=for-the-badge" />
<img src="https://img.shields.io/badge/Metaflow-FF8C00?style=for-the-badge" />
<img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" />
<img src="https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white" />
<img src="https://img.shields.io/badge/Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white" />
</p>

---

## Other projects

| Project | What it is |
| :-- | :-- |
| **[MLflow Model Lifecycle Lab](https://github.com/oster-dev/mlflow-model-lifecycle-lab)** | MLflow tracking and registry, Metaflow, a tested quality gate, CI |
| **[Kafka + Airflow + PostgreSQL](https://github.com/oster-dev/projects/tree/main/event-stream-pipeline)** | An event-stream pipeline with orchestration |
| **[PySpark Retail ETL](https://github.com/oster-dev/projects/tree/main/Retaildata%20PYSparkETL)** | A retail data ETL with PySpark |
| **[Jikan Feature Pipeline](https://github.com/oster-dev/projects/tree/main/jikan-feature-pipeline-dockerized)** | My first public project: API data in, validated features out |

---

## Certifications and open source

| | Status |
| :-- | :-- |
| AWS Cloud Practitioner · CLF-C02 | ✅ [Passed](https://www.credly.com/badges/2c2b32e6-4bc9-4e60-ad56-3cf7bc2775bd/public_url) |
| AWS Data Engineer Associate · DEA-C01 | ✅ [Passed](https://www.credly.com/badges/76633228-8b19-4fb6-81bd-ebea0e96f087/public_url) |
| AWS Solutions Architect Associate · SAA-C03 | ✅ [Passed](https://www.credly.com/badges/323635b9-4840-4a90-8900-a2a86b8082c1/public_url) |
| AWS ML Engineer Associate · MLA-C02 | ⏸️ I didn't pass the beta exam. I'll retake it when the official exam is out |
| MLflow documentation fix | ✅ Merged: [#26287](https://github.com/mlflow/mlflow/pull/26287) |

---

## About me

I'm Nico, 24, from Cologne. I started programming in March 2026 with a Python bootcamp and decided to aim at ML platform and feature infrastructure work. Since then I work on it every day: learn, build, write it down.

I like systems where the hard part is getting things right, like late events, duplicates and what happens after a crash. I like proving that with a test. I'm looking for a role where I can do this for real.

📍 Cologne, Germany · 🌍 Open to remote work worldwide
📬 [ostermann.dev@gmail.com](mailto:ostermann.dev@gmail.com) · 💼 [LinkedIn](https://www.linkedin.com/in/nico-ostermann-606a6b402)

<p align="center">
  <img src="./assets/footer.svg" alt="Built in public since April 2026" width="100%" />
</p>
