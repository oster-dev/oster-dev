## Month 6 Review — September 2026  
### AWS SAA-C03 · MLflow · Metaflow · Model Lifecycle · Open Source Contribution

---

**Month 6 is officially complete.**

Month 6 was designed to turn the feature-infrastructure foundation from Month 5 into a broader ML Platform profile.

The roadmap goal was clear:

```text
Finish AWS SAA-C03
→ learn MLflow experiment tracking and model registry
→ learn Metaflow workflow orchestration
→ complete Project 1 documentation
→ make a real open-source contribution to Feast or MLflow
→ strengthen system design practice
```

The result became more substantial than a tool-learning month.

I completed the ML lifecycle layer around FeatureForge by building a public, tested `mlflow-model-lifecycle-lab`. The project connects model training, experiment tracking, artifacts, signatures, evaluation, quality-gate decisions, registry promotion, rejection paths, and inference from a registered model.

I also made my first upstream open-source contribution to MLflow.

While validating the official MLflow Model Registry tutorial end to end, I reproduced a failure caused by the new default `skops` serialization format for scikit-learn models. The tutorial’s `RandomForestRegressor` example required an explicitly reviewed trusted type that was not documented. I opened MLflow issue [#26256](https://github.com/mlflow/mlflow/issues/26256), provided a reproduction and validated fix direction, and the issue was implemented in [PR #26287](https://github.com/mlflow/mlflow/pull/26287), merged into `mlflow/mlflow:master` by maintainer `harupy`.

Month 6 therefore ended with more than local projects:

```text
Public model lifecycle project
→ tested and documented
→ real upstream MLflow documentation improvement
→ merged into master
```

---

**Roadmap Goals vs. Reality**

| Goal | Status | Note |
|---|---|---|
| AWS SAA-C03 certification | ✅ Done | Passed on 09.09.2026 and carried the cloud architecture perspective into ML platform work |
| MLflow fundamentals | ✅ Done | Built practical understanding of tracking, experiments, artifacts, signatures, model logging, registry workflows, and model URIs |
| Metaflow fundamentals | ✅ Done | Built a workflow with explicit steps, artifact passing, candidate evaluation, quality-gate routing, accepted and rejected outcomes |
| ML lifecycle project | ✅ Done | Built and published `mlflow-model-lifecycle-lab` with tests, CI, documentation, and a reproducible local flow |
| Model Registry workflow | ✅ Done | Trained, logged, evaluated, conditionally registered, loaded, and inferred from registered models |
| Model signatures and input examples | ✅ Done | Treated model input contracts as explicit artifacts rather than implicit assumptions |
| Quality-gate routing | ✅ Done | Implemented promotion or rejection based on measurable model-quality criteria |
| Open-source contribution | ✅ Done | Opened MLflow issue #26256; fix was implemented in PR #26287 and merged upstream |
| Feast open-source exploration | ✅ Done | Triaged real Feast issues and deliberately avoided duplicating work already claimed by other contributors |
| System Design routine | ✅ Established | Continued feature-store, training-system, and ML lifecycle design thinking |
| Month 6 roadmap milestone closed | ✅ Done | ML lifecycle capability and merged upstream OSS outcome achieved |

---

**Project That Proves Month 6**

**[mlflow-model-lifecycle-lab](https://github.com/oster-dev/mlflow-model-lifecycle-lab)**  
*A production-inspired local ML lifecycle workflow for reproducible experiments, quality-gated model promotion, registry loading, and inference.*

The project was not designed as a notebook showing a single `mlflow.log_metric()` call. Its purpose was to make the ML model lifecycle explicit and testable.

The implemented flow is:

```text
Load data
→ train candidate models
→ track parameters, metrics, artifacts, signatures, and input examples
→ evaluate candidates
→ apply a promotion-quality gate
→ register the accepted model
→ reject insufficient candidates
→ load the registered model
→ run inference
```

The project demonstrates:

- MLflow experiment tracking.
- Reproducible local tracking server configuration.
- SQLite backend storage for MLflow metadata.
- Local artifact storage.
- Multiple model candidates and evaluation metrics.
- Model logging with artifacts, signatures, and input examples.
- Explicit quality-gate logic for promotion decisions.
- Metaflow workflow steps and artifact transitions.
- Accepted and rejected model branches.
- MLflow Model Registry registration.
- Loading a model through a registry URI.
- Inference using a registered model artifact.
- Unit and integration-style tests.
- Ruff formatting and linting.
- Passing GitHub Actions CI.
- Documentation of setup, architecture, behavior, and validation.

The key outcome is that the project represents an **operational lifecycle contract**, not just model training:

```text
A model is not production-ready because it trains successfully.

It needs:
- reproducible inputs
- tracked parameters and metrics
- model artifacts
- a declared input/output contract
- a measurable quality threshold
- a promotion or rejection decision
- versioned registry identity
- recoverable inference behavior
```

---

**MLflow: Tracking, Artifacts, and Model Contracts**

This month made MLflow concrete as an ML Platform component.

Before building the lifecycle lab, experiment tracking could be described abstractly:

```text
parameters
metrics
artifacts
models
```

During the project, those became connected engineering contracts.

I learned that MLflow tracking is valuable not because it creates a user interface, but because it preserves the evidence required to understand, compare, reproduce, and promote a model.

The lifecycle project explicitly captures:

```text
Training configuration
→ model parameters
→ evaluation metrics
→ training artifacts
→ model signature
→ input example
→ registry version
→ promotion decision
```

That chain matters because it allows a model version to be traced back to the data, code path, configuration, and quality decision that created it.

Model signatures and input examples were especially important. A trained model without a declared interface creates an ambiguous inference contract. Recording these artifacts makes expected inputs visible and helps reduce training-serving skew.

---

**Metaflow: Workflow Boundaries and Quality Gates**

Metaflow introduced the orchestration layer that connects isolated ML operations into a controlled workflow.

The main lesson was that workflow design should make transitions explicit:

```text
Data preparation
→ training
→ evaluation
→ quality decision
→ register or reject
```

A quality gate is not merely an `if` statement. It is a platform policy boundary.

The workflow defines what conditions must be met before a candidate may become a registered, reusable model artifact. If the model does not satisfy the policy, the run still remains useful:

```text
Rejected model
→ metrics preserved
→ artifacts preserved
→ decision recorded
→ no unsafe promotion
```

This is a more credible design than always registering the latest trained model.

The project therefore demonstrates both paths:

```text
Accepted candidate
→ Model Registry

Rejected candidate
→ tracked evidence without promotion
```

That distinction is central to reliable ML platform engineering.

---

**Model Registry and Inference**

The Model Registry work turned model lifecycle theory into an executable end-to-end flow.

I validated the full path locally:

```text
MLflow Tracking Server
→ SQLite backend store
→ experiment and run logging
→ scikit-learn model logging
→ model registration
→ registered model version
→ models:/... URI resolution
→ successful model load
→ inference
```

This clarified the difference between several layers that are often collapsed into “MLflow setup”:

| Component | Responsibility |
|---|---|
| Tracking URI | Where a client sends tracking and registry requests |
| Backend Store | Metadata for experiments, runs, models, and registry records |
| Artifact Store | Model files, input examples, serialized artifacts, and other run outputs |
| Registry URI | A versioned or named reference to a registered model |
| Quality Gate | The policy that determines whether a candidate may enter the registry |

The important engineering conclusion was:

```text
Successful training does not prove that model registration works.

Successful registration does not prove that registry URI loading works.

The complete lifecycle must be validated end to end.
```

---

**Open Source: MLflow Documentation Contribution**

The most visible Month 6 outcome was the MLflow upstream contribution.

While validating the official Model Registry tutorial, I used the documentation as a real user would:

```text
Read tutorial
→ run example locally
→ configure tracking server
→ use SQLite backend
→ log RandomForest model
→ register model
→ load through models:/ URI
```

The tracking server and registry path worked, but the tutorial’s `RandomForestRegressor` logging step failed under the new default scikit-learn serialization behavior.

The root cause was:

```text
mlflow.sklearn.log_model()
→ default serialization_format="skops"
→ RandomForest contains sklearn.tree._tree.Tree
→ skops requires explicit reviewed trust for this type
→ existing tutorial omitted the required configuration
```

I separated potential causes rather than assuming the first error was the real problem:

```text
Direct SQLite tracking
→ HTTP tracking server
→ SQLite backend store
→ experiment logging
→ model logging
→ registry registration
→ registry URI loading
```

That process established that the failure was not caused by the tracking server, SQLite, registry, or URI configuration. It was caused by the serialization contract.

The working, security-aware approach was:

```python
skops_trusted_types=["sklearn.tree._tree.Tree"]
```

I also verified the full post-fix lifecycle:

```text
RandomForest training
→ model logging
→ model registration
→ models:/... load
→ successful inference
```

After inspecting MLflow source code, tests, existing documentation, Git history, and open issues, I opened:

**[MLflow Issue #26256](https://github.com/mlflow/mlflow/issues/26256)**  
`[DOC-FIX] Update Model Registry RandomForest tutorial for skops default serialization`

The issue proposed a deliberately minimal and security-aware documentation fix:

- Add the reviewed trusted tree type to the tutorial example.
- Explain why the example needs it.
- Warn users to trust only types they have reviewed.
- Link to the existing Pickle-Free Model documentation.

The result:

```text
Issue #26256
→ triaged by MLflow maintainer harupy
→ implemented through PR #26287
→ merged into mlflow/mlflow:master
→ issue closed as completed
```

**[MLflow PR #26287](https://github.com/mlflow/mlflow/pull/26287)**  
`[DOC-FIX] Update Model Registry tutorial for skops serialization`

The merged change updated the official MLflow Model Registry tutorial with:

```text
skops_trusted_types
→ explicit trusted Tree type
→ security guidance
→ Pickle-Free Model documentation link
```

This was a small documentation diff, but it represented a complete engineering contribution process:

```text
Reproduce
→ isolate
→ inspect source and tests
→ verify safe workaround
→ check duplicates
→ write a focused issue
→ upstream implementation
→ maintainer merge
```

The important result was not claiming a large MLflow feature. It was identifying a real break in an official user path and helping ensure that a secure, working solution reached the upstream documentation.

---

**Feast Open Source Exploration**

I also continued monitoring Feast as the more directly feature-infrastructure-aligned open-source target.

A strong candidate was Feast issue #6821:

```text
Feature View TTL is not applied on the standard online retrieval path
```

The issue concerned an important serving-correctness contract:

```text
FeatureView TTL
→ online retrieval
→ expired value returned as PRESENT
→ stale value indistinguishable from fresh value
```

This was highly relevant to FeatureForge because it connects freshness, online serving behavior, feature validity, and model-input correctness.

However, the issue was already actively claimed in the discussion by another contributor who had posted:

- a detailed understanding of the affected response assembly path;
- a proposal to use `FieldStatus.OUTSIDE_MAX_AGE`;
- a regression-test plan;
- a compatibility question about withholding expired values.

Although GitHub showed no formal assignee and no linked branch, I did not duplicate the work.

The lesson was important:

```text
No assignee does not always mean an issue is available.

Contributor etiquette requires reading the full discussion,
checking for active claims, proposed approaches, linked PRs,
and maintainer direction before beginning implementation.
```

This was valuable open-source practice even without a Feast PR.

---

**System Design and Platform Thinking**

Month 6 strengthened the connection between Feature Infrastructure and ML Platform design.

FeatureForge made feature data available and validated across offline and online paths. The ML lifecycle lab added the downstream control plane:

```text
Feature data
→ training input
→ candidate model
→ tracked experiment
→ evaluated artifact
→ promotion policy
→ registry version
→ inference consumer
```

Together, the two projects form a coherent platform story:

| Layer | Evidence |
|---|---|
| Feature Infrastructure | FeatureForge: event-time logic, offline/online serving, materialization, freshness, quality gates |
| ML Lifecycle | MLflow + Metaflow: training, tracking, signatures, evaluation, promotion or rejection, registry, inference |
| Cloud Architecture | AWS SAA-C03 plus documented AWS-oriented production profiles |
| Reliability | Tests, CI, runbooks, manifests, controlled failures, explicit decision boundaries |
| Open Source | MLflow issue #26256 resolved through merged upstream PR #26287 |

The central design principle across both projects became:

```text
Make important platform contracts explicit.

Data validity.
Feature freshness.
Training-serving compatibility.
Model input contracts.
Promotion criteria.
Artifact provenance.
Registry identity.
Failure and recovery paths.
```

---

**Personal Context**

Month 6 required a different form of discipline than Month 5.

Month 5 was about completing and releasing a large feature-infrastructure project. Month 6 was about avoiding superficial tool usage and instead proving that I understood the operational contracts around ML workflows.

It would have been easy to treat MLflow as a dashboard and Metaflow as a collection of decorators. Instead, I focused on the lifecycle boundaries that matter in production:

```text
What does the model consume?
What evidence supports promotion?
What happens when it fails the quality policy?
How is the promoted model identified?
Can the registered model actually be loaded?
What changes when a secure serialization default changes?
```

The MLflow contribution reinforced this mindset. I did not force a PR or rush to claim a random issue. I reproduced the behavior, tested the full lifecycle, examined the source and existing documentation, checked for duplicates, proposed a narrow fix, and let the upstream project decide the final implementation path.

That is the type of engineering behavior I want the portfolio to represent: not tool collection, but disciplined ownership at system boundaries.

---

**What Starts Next — Month 7**

- Begin focused AWS MLA-C01 preparation using Tutorial Dojo practice exams and a domain-based error log.
- Build Project 2: a streaming pipeline with Kafka, Flink, AWS-oriented architecture, Schema Registry, Prometheus, and Grafana.
- Implement idempotency, exactly-once processing, dead-letter handling, late-event handling, and watermarks.
- Continue system design practice around streaming ingestion, training-data systems, feature serving, and ML workflow infrastructure.
- Begin structured Netflix Culture preparation with Freedom & Responsibility and STAR stories.
- Continue a short daily Feast open-source issue scan without allowing it to block the primary roadmap.

---

Month 6 is closed.

The strongest outcome is not only that I learned MLflow and Metaflow. It is that I can now demonstrate a public ML lifecycle system with tracked evidence, explicit quality gates, model registry behavior, and reproducible inference — and that I contributed a validated documentation improvement to the MLflow upstream project.

Month 5 proved that I could build a feature platform.

Month 6 proved that I could connect that platform to a controlled model lifecycle and participate credibly in the open-source ecosystem around it.

The roadmap now moves into Month 7 with a more complete ML Platform foundation:

```text
Feature data
→ quality and freshness
→ model training
→ experiment tracking
→ evaluation
→ promotion policy
→ registry
→ inference
→ streaming systems next
```
