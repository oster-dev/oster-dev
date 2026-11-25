## Month 7 Review — October 2026  
### FeatureForge Stream · Kafka · Flink · Event Time · Idempotent Writes · One Platform Instead of Three Repos

---

**Month 7 is complete, and it did not go as planned.**

The roadmap for Month 7 looked like this:

```text
AWS MLA-C01 preparation and exam
→ Project 2: a streaming pipeline with Kafka and Flink
→ idempotency, dead-letter handling, late events, watermarks
→ Schema Registry, Prometheus and Grafana
→ system design and culture preparation
```

The streaming core got built. Two parts changed shape: the streaming project became part of FeatureForge instead of a repository of its own, and it makes no exactly-once claim. The AWS exam did not happen. I explain why below.

---

**Roadmap Goals vs. Reality**

| Goal | Status | Note |
|---|---|---|
| AWS MLA certification | ❌ Not achieved | I did not pass the MLA-C02 beta exam, and the paid exam is lost, see below |
| Project 2: streaming pipeline with Kafka and Flink | ✅ Done, in a different shape | Built as FeatureForge Stream (v0.1.0), then merged into FeatureForge instead of living in its own repo |
| Event time and windows | ✅ Done | Event time, never ingestion time. Windows are right-closed: the lower bound is excluded, the upper bound is included |
| Idempotent writes | ✅ Done | Guarded Redis writes with WATCH/MULTI, including a controlled conflict test against a live Redis |
| Dead-letter handling | ✅ Done | Invalid events become rejection records with a reason code on a separate topic and never block valid events |
| Exactly-once processing | 🎯 Deliberately not claimed | The sinks are at-least-once. Correctness comes from dedupe and idempotent writes |
| Schema Registry | ⚠️ Replaced | Executable Pydantic contracts instead. A registry stays an open decision |
| Prometheus and Grafana | ⏭️ Moved to November | Built for the serving service, not for this month's streaming job |
| AWS-oriented architecture | ⚠️ Documentation only | An AWS production profile (S3, DynamoDB, managed Kafka and Flink) exists as design. Nothing is deployed |
| System design and culture preparation | [CONFIRM] | [One sentence on what you actually did in October] |

---

**The Plan Change: One Platform Instead of Three Repos**

The original roadmap planned three portfolio projects in three repositories:

| Project | Stack | Main proof |
|---|---|---|
| Project 1: Open Mini Feature Store | Feast, Spark, AWS S3, Python, tests, CI/CD | Data and feature infrastructure |
| Project 2: Streaming pipeline | Kafka, Flink, AWS, monitoring, Schema Registry | Real-time infrastructure |
| Project 3: End-to-end ML workflow | Metaflow, MLflow, AWS, real data | ML platform thinking |

I changed that. Instead of three separate repositories, everything gets bundled into one large and ambitious flagship project: **FeatureForge**.

The reason is in ADR-006. A separate streaming repo would have stayed smaller, but it would have left streaming as a parallel prototype and not as part of the platform. It would have made it harder to enforce shared contracts, to test the joined Kafka-to-Redis path, and to evolve offline and streaming semantics together.

How the bundling was planned:

```text
One platform boundary
→ the streaming code is imported in four documented steps
→ computation, validation and store semantics are preserved in every import
→ a provenance document records what came from where
→ one CI, one set of contracts, one ADR history
```

On top of that sits a written readiness plan. Every requirement of the target role gets one verifiable artifact: a test, a measurement, a drill report or a document. The rule is "no claim without proof". The plan splits the work into work packages: a declarative spec platform, Iceberg datasets, Cassandra as a second online store, a serving service with a load test, operations, a catalog, a notebook, performance work, and the end-to-end ML workflow as the last package.

Project 3 therefore does not disappear. The end-to-end ML workflow with Metaflow, MLflow, a registry and rollback is the last planned package of FeatureForge. The lab from Month 6 stays a separate public project.

The trade-off is real. One repository means more moving services in one place, a heavier local setup with Docker Compose, and an extra Flink job in CI. I accepted that for one coherent platform.

---

**Project That Proves Month 7**

**[FeatureForge Stream](https://github.com/oster-dev/featureforge-stream)** (v0.1.0)  
*The streaming core: Kafka, Flink and guarded Redis writes, merged into FeatureForge in four imports.*

The flow:

```text
Kafka events
→ Flink validation job (invalid events go to a rejected topic)
→ Flink play-count job (event time, windows)
→ Kafka feature topic
→ materializer (guarded Redis write)
→ offset committed only after the write
→ Redis
```

What the four imports brought in:

- **Import 1:** event validation, right-closed timestamp mapping, the play-count reference, unit tests and a golden fixture.
- **Import 2:** event routing, a guarded Redis play-count store and the Kafka-to-Redis materializer.
- **Import 3:** live Redis integration tests, including a controlled WATCH conflict.
- **Import 4:** the Flink runtime configuration, the validation and play-count jobs and a joined Kafka-to-Redis test.

The joined test showed one play event going from Kafka through Flink into Redis, and the stored feature was read back and checked. The provenance document is just as clear about what that did **not** show: checkpoint recovery, sustained operation and all late-event behaviors. Those came later and are part of the November work.

---

**What I Understood About Streaming Correctness**

```text
Use event time, never ingestion time.
Make window boundaries explicit.
Treat invalid input as data, not as an exception.
Commit offsets only after the write succeeded.
Do not call it exactly-once.
```

A Redis write and a Kafka offset commit are not one transaction. A failure between them causes a redelivery. That is why the writes have to be idempotent, and why replay, routing and correction tests must never be described as end-to-end exactly-once. I wrote the limit next to the result.

---

**AWS MLA-C02: What Happened**

The roadmap named the AWS machine learning certification as the Month 7 goal.

The exam I ended up sitting was the beta of MLA-C02. I did not pass it. It was very close, Beta retakes are not allowed so i need to wait for the official release of MLA-C02 and to sit the real exam then.

I also paid for an exam. A problem with Udemy and AWS, has not been resolved. As of November 25, neither Udemy nor AWS has given me a solution, so the paid exam is lost.

The result is simple. There is no MLA certificate in Month 7, and my profile says so openly. The retake is planned for after the official release of MLA-C02.

What I take from it:

- A certificate is an external dependency. I should not put the critical path of a roadmap on it.
- The ML platform proof does not sit in the certificate. It sits in the lifecycle lab and in FeatureForge.
- I keep the three certificates I have: CLF-C02, DEA-C01 and SAA-C03.

---

**Personal Context**

Month 7 asked for a different kind of discipline than Month 6. Month 6 was about proving contracts around a model. Month 7 was about changing my own plan when the plan was wrong, and about reporting a miss without dressing it up.

I would rather write down "I did not pass, and the paid exam is gone" than hide it. A roadmap that only records successes is not a roadmap, it is a highlight reel.

---

**What Starts Next — Month 8**

- FeatureForge as the spec platform: Iceberg training datasets, Cassandra as a second online store, and a serving service with a measured p99.
- Applications, with the repository and its status page as the evidence.
- The daily log continues.

---

Month 7 is closed.

The strongest outcome is not a certificate. It is that the streaming core exists, that it runs inside one platform instead of next to it, and that I know where it is proven and where it is not.
