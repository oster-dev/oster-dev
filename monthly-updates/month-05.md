## Month 5 Review — August/September 2026  
### AWS SAA-C03 · FeatureForge · Feature Infrastructure · First Public Release

---

**Month 5 is officially complete.**

Month 5 became much bigger than a certification month. The original plan was to prepare for AWS SAA-C03 while starting Project 1: a public, production-inspired feature store. In practice, both became serious delivery milestones.

I passed the AWS Certified Solutions Architect – Associate exam on 09.09.2026 and completed **FeatureForge**, the first flagship portfolio project in my roadmap. What started as a focused feature-store project developed into a complete local feature platform with deterministic data generation, feature computation, point-in-time correctness, offline and online serving, quality controls, freshness checks, failure simulations, operational runbooks, CI, and a documented AWS production profile.

The result is now public and versioned:

**[FeatureForge v0.1.0](https://github.com/oster-dev/featureforge/releases/tag/v0.1.0)**

Month 5 did not end with a half-finished prototype or a collection of disconnected tools. It ended with a released, reproducible platform project that I can explain, run locally, test, and defend architecturally.

---

**Roadmap Goals vs. Reality**

| Goal | Status | Note |
|---|---|---|
| AWS SAA-C03 preparation | ✅ Done | Studied architecture, security, resilience, storage, networking, and cost trade-offs |
| AWS SAA-C03 certification | ✅ Done | Passed on 09.09.2026 |
| Feature store fundamentals | ✅ Done | Learned offline/online stores, feature definitions, materialization, historical retrieval, and serving contracts |
| Feast fundamentals | ✅ Done | Implemented entities, sources, feature views, feature services, historical retrieval, and online serving |
| Spark-based feature computation | ✅ Done | Added a PySpark feature-computation path and parity-tested it against the Pandas reference implementation |
| Project 1: FeatureForge | ✅ Done | Public repository completed and released as v0.1.0 |
| Quality, freshness, and failure handling | ✅ Done | Implemented correctness gates, freshness checks, manifests, failure simulations, and runbooks |
| GitHub Actions CI | ✅ Done | Added reproducible linting and test validation in clean CI environments |
| AWS production profile | ✅ Done | Documented S3, DynamoDB, IAM, storage ownership, and architecture trade-offs |
| Month 5 roadmap milestone closed | ✅ Done | Project 1 released and SAA-C03 passed |

---

**Project That Proves Month 5**

**[FeatureForge](https://github.com/oster-dev/featureforge)**  
*A production-inspired feature platform for reproducible offline training data and low-latency online ML feature serving.*

FeatureForge became the strongest project in my portfolio so far because it required more than implementing one library or one pipeline. The goal was to build a small but coherent platform with explicit contracts between data generation, feature computation, offline storage, Feast definitions, materialization, online serving, validation, and operational behavior.

The project now demonstrates:

- Deterministic synthetic data generation for users, content, events, and labels.
- Explicit event-time and ingestion-time modeling.
- Data contracts and validation for records, configuration, source data, and feature outputs.
- Point-in-time feature computation with explicit temporal boundaries.
- Deterministic and idempotent date-range feature backfills.
- Partitioned Parquet offline feature datasets.
- A readable Pandas reference implementation and a parity-tested PySpark implementation.
- Feast entities, sources, feature views, feature services, and historical retrieval.
- Full and incremental materialization into Redis for local online serving.
- Online feature lookup and deterministic ranking behavior.
- Offline/online parity validation.
- Pre-materialization feature correctness gates.
- Feature freshness checks based on business-time partitions.
- Machine-readable generation, backfill, and materialization manifests.
- Controlled failure simulations and operator runbooks.
- GitHub Actions CI.
- A documented AWS production profile using S3 and DynamoDB as the production-oriented storage model.

The most important outcome is not that I “used Feast” or “used Spark.” The important outcome is that I built a platform flow where the same feature contracts remain visible across training retrieval, online serving, materialization, correctness validation, freshness validation, and future AWS deployment.

---

**What Was Covered**

**AWS SAA-C03**

This month included the final preparation and exam attempt for AWS Certified Solutions Architect – Associate. The exam required much more than memorizing AWS services. It reinforced architecture trade-offs around security, resilience, storage, networking, identity, cost, and failure handling.

Passing the exam on 09.09.2026 was an important milestone because it strengthened the cloud architecture layer behind the projects I am building. It also changed how I think about FeatureForge: not only as a local Python project, but as a system with a credible path toward managed AWS services and explicit operational trade-offs.

---

**Feature Infrastructure**

FeatureForge made feature infrastructure concrete.

Before this month, terms such as offline store, online store, historical retrieval, materialization, feature view, point-in-time correctness, and training-serving skew were mostly concepts. During this project, they became implementation and design constraints.

I learned why feature platforms need more than a feature table:

- Training data must not include future information.
- Offline and online values need explicit consistency contracts.
- Materialization should not move invalid data into a serving store.
- Freshness is different from correctness.
- Backfills need deterministic paths, idempotency, and auditability.
- A feature platform needs visible ownership of source paths, timestamps, contracts, and operational failure behavior.

The project treats these concerns as executable contracts rather than documentation-only intentions.

---

**PySpark and Parity**

One of the most important technical lessons came from maintaining both a readable Pandas reference implementation and a PySpark implementation of the same feature logic.

The work was not only about translating code into Spark. The real requirement was proving that both engines produced equivalent outputs under the same temporal rules. This introduced a concrete parity-testing discipline:

```text
Pandas reference logic
→ PySpark implementation
→ explicit timestamp handling
→ feature-by-feature output comparison
→ parity test
```

This also exposed the importance of timezone handling across Python and JVM boundaries. Event-time semantics cannot depend on an implicit local timezone. The project therefore treats timestamps carefully and tests the behavior rather than assuming the runtime will behave consistently.

---

**Reliability, Quality, and Freshness**

The final FeatureForge work moved beyond the happy path.

I added a pre-materialization correctness gate so that invalid persisted feature data blocks Feast materialization before it reaches Redis. I also separated correctness from freshness:

```text
Correctness:
Are persisted feature values structurally and semantically valid?

Freshness:
Are the newest feature snapshots recent enough for the intended serving workflow?
```

That distinction became one of the most important architecture lessons of the month. A dataset can be structurally valid but too old for a current serving workflow. Conversely, historical re-materialization can be correct even if the source data is not “fresh” relative to the current date.

The project now includes:

- Persisted feature correctness validation.
- Feature freshness SLO checks based on observation-date partitions.
- Blocked materialization behavior.
- Completed and blocked materialization manifests.
- Controlled stale-feature, failed-backfill, and failed-materialization simulations.
- Operational runbooks with diagnosis, recovery, verification, and prevention steps.

This was the point where FeatureForge started feeling like platform engineering rather than a feature-store tutorial.

---

**CI and AWS Production Profile**

I also added a reproducible GitHub Actions baseline that validates the repository in a clean environment. The CI pipeline runs formatting, linting, and tests without relying on my local machine.

The AWS production profile documents how the local architecture could evolve without rewriting its core contracts:

| Local development | AWS-oriented production profile |
|---|---|
| Local Parquet offline store | Amazon S3 partitioned Parquet |
| Redis via Docker Compose | DynamoDB online store profile |
| Local filesystem paths | Environment-owned shared storage URI |
| Local execution | Scheduled or managed compute profile |
| Local manifests | Durable lineage and operational metadata |

The important design decision was to preserve the same invariant across environments:

```text
Backfill output
→ correctness gate input
→ freshness check input
→ Feast source
→ materialization source
→ serving parity-test source
```

All of these must resolve to the same canonical feature-store location. Otherwise, a system can validate one dataset while serving another, which defeats the point of quality controls.

---

**Release**

FeatureForge was officially released as **v0.1.0**.

The release was intentionally treated as an engineering milestone rather than just a tag:

- Validated the local platform flow.
- Verified tests, formatting, and linting.
- Created and pushed an annotated Git tag.
- Published the GitHub release.
- Marked the repository as a completed portfolio artifact.
- Preserved future improvement scope for deliberate follow-up releases such as `v0.1.1` or `v0.2.0`.

Project 1 is no longer unfinished roadmap scope. It is now a public, versioned artifact that represents my current feature-infrastructure engineering work.

---

**Personal Context**

Month 5 demanded sustained focus because it combined certification preparation with the largest project I had built so far.

The SAA-C03 preparation was challenging, especially when practice exams exposed weaknesses in security and resilient architecture. Instead of treating weak scores as failure, I used them to identify specific gaps, review the underlying concepts, and keep moving forward.

After passing the certification, I put full energy into finishing FeatureForge properly. The project expanded because I kept asking what would make it more credible: stronger contracts, better tests, explicit failure behavior, reproducible CI, an AWS operating profile, clearer documentation, and a real release.

That work took more effort than a smaller demo would have required, but it produced something much more valuable: a project that demonstrates end-to-end ownership, not just tool familiarity.

---

**What Starts Next — Month 6**

- MLflow experiment tracking, model artifacts, signatures, and model registry.
- Metaflow workflow orchestration, steps, branching, and quality-gate routing.
- A focused ML lifecycle lab with reproducible training and explicit promotion policy.
- An open-source contribution to Feast or MLflow.
- Continued system design practice focused on feature platforms, training systems, and streaming infrastructure.

---

Month 5 is closed.

The strongest outcome is not just one AWS certification or one project release. It is that I now have a public feature platform that connects data generation, event-time logic, point-in-time training retrieval, feature computation, offline storage, online serving, quality gates, freshness, failure handling, CI, and cloud architecture into one coherent system.

FeatureForge v0.1.0 is the first major proof project in my roadmap. Month 6 starts from a stronger base: I have moved from learning individual data tools to building and releasing a platform-shaped system around them.
