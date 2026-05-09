## Month 2 Review — May 2026
### AWS CLF-C02 · Docker · Advanced SQL · Functional Programming

---

**Month 2 is officially complete — 3 weeks ahead of schedule.**

What the roadmap planned for the entire month of May, I completed by May 9, 2026.
Every single required topic. Nothing skipped. Nothing half-done.
This was not luck — this was a deliberate, consistent, disciplined sprint.

---

**Pace — How Fast Each Topic Was Completed**

| Topic | Duration | Notes |
|---|---|---|
| Advanced SQL (Maven Course) | Apr 20 – Apr 24 · ~4 days | Window Functions, CTEs, Subqueries — done in one focused week |
| AWS CLF-C02 Preparation | Apr 25 – May 6 · ~11 days | René Fürst course + all 6 Maarek practice exams in German |
| AWS CLF-C02 Exam | May 6, 2026 | Passed — AWS Certified Cloud Practitioner ✅ |
| Docker (Schwarzmüller Course, Sections 2–6) | May 7 – May 8 · **2 days** | Images, Containers, Volumes, Networking, Compose — full send |
| Functional Programming (map, filter, reduce, Pure Functions, Immutability) | May 9 · **~2 hours** | Theory to practical refactor in the Jikan pipeline — same day |

Docker in 2 days. Functional Programming from zero to applied project refactor in under
2 hours. These are Statements.

---

**Roadmap Goals vs. Reality**

| Goal | Status | Note |
|---|---|---|
| AWS CLF-C02 — pass the exam | ✅ Done | Passed May 6, 2026 |
| Docker — Dockerfile, Compose, Networking, Volumes | ✅ Done | Completed May 8, 2026 |
| First Python app containerised on GitHub | ✅ Done | Jikan pipeline containerised with Dockerfile |
| Functional Programming — map, filter, reduce, Pure Functions, Immutability | ✅ Done | Completed May 9, 2026 |
| Advanced SQL — Window Functions, CTEs | ✅ Done | Completed April 20–24 |
| LinkedIn post published | ✅ Done | Weekly learning log active |

---

**Projects That Prove Month 2**

`jikan-feature-pipeline` — *Containerised + Functional Refactor*
The Jikan pipeline from Month 1 was extended significantly:
containerised with a production-style Dockerfile using `python:3.11-slim`,
and a functional refactor replacing
imperative loops with `map()` and `filter()` — committed with a clean message:
`refactor: apply functional style with map/filter`

This project now demonstrates API ingestion, validation, feature engineering,
containerisation, and functional programming — all in one public repository.

---

**What Was Learned — Key Concepts**

**Advanced SQL**
Window Functions with `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `PARTITION BY`,
`LEAD()`, `LAG()` — CTEs for readable multi-step query logic — Subqueries
and correlated subqueries for filtered aggregation.

**AWS CLF-C02**
Cloud fundamentals across all four exam domains: Cloud Concepts, Security & Compliance,
Technology, and Billing & Pricing. Passed after consistent daily practice
across 6 full Maarek practice exams. Highest score: 88%.

**Docker**
Images and Containers — Volumes and Bind Mounts — Networking between containers
— Multi-Container setups — Docker Compose for orchestration.
Core insight: Docker guarantees reproducible execution environments,
which is directly relevant to feature pipeline portability in ML infrastructure.

**Functional Programming**
`map()`, `filter()`, `reduce()` — Immutability as a core principle of Feature Stores
— Pure Functions: same input always produces same output, no side effects,
completely self-contained. Netflix requires this explicitly for ML pipeline
determinism and reproducibility at scale.

---

**Skill Progress vs. L5 Target (May 9, 2026)**

| Skill / Tool | Month 1 Level | Current Level | L5 Target | Progress |
|---|---|---|---|---|
| Python (Core) | OOP, APIs, Cleaning | Functional style, Pipeline architecture | Advanced, Type Hints, Testing | 55% |
| SQL | SELECT, JOIN, GROUP BY | Window Functions, CTEs, Subqueries | Optimisation, Performance | 55% |
| Git / GitHub | Commits, Repos | Clean commits, functional refactors | PRs, Actions, CI/CD | 45% |
| Docker | Concept only | Dockerfile, Compose, Networking, Volumes | Production deployments | 50% |
| AWS | Free Tier account | CLF-C02 certified ✅ | DEA + SAA + MLA | 20% |
| Functional Programming | Basic idea | map, filter, reduce, Pure Functions, applied | Production-level patterns | 70% |
| Spark | Not started | Not started | DataFrames, ETL, Scala reading | 0% |
| Kafka | Not started | Not started | Topics, Partitions, Exactly-Once | 0% |
| Airflow | Not started | Not started | DAGs, Scheduling, XComs | 0% |
| Feast | Concept only | Concept only | Offline/Online Serving | 5% |
| MLflow / Metaflow | Not started | Not started | Experiment Tracking, Registry | 0% |
| System Design | Basic | Basic | Designing Data-Intensive Applications | 5% |
| Scala (reading) | Not started | Not started | Read and adapt Spark-Scala code | 0% |

**Overall L5 Progress: ~28%**
*Month 2 complete in half the time. Ahead of schedule entering Month 3.*

---

**Personal Statement**

Month 2 was completed on May 9, 2026 — three weeks before the end of May.
Every required topic was covered. Nothing was skipped. That alone says something.

But this month carried more weight than just the curriculum.

On April 26, I was diagnosed with a double inguinal hernia — left side already present,
right side developing. Genetic. Unavoidable. Surgery is scheduled for May 19, 2026.
This means I had to temporarily leave Romania and reorient my entire situation in a
matter of days. That was not planned. None of it was.

My response was to accelerate.

I finished Docker in 2 days at full intensity. I completed Functional Programming —
from first concept to applied project refactor — in under 2 hours on the same day.
I passed the AWS CLF-C02 exam with consistent daily practice and walked out certified.
I did all of this knowing that surgery was coming, that recovery time would follow,
and that the buffer I was building now might be the only thing keeping Month 3 on track.

The CLF-C02 is my first AWS certification. I am aware that it is the entry-level credential —
but it is proof of something more important than the certificate itself: that I can learn
at pace, under pressure, and deliver a measurable result on a fixed deadline.
In under 11 days of preparation. That is a real signal.

I am not letting this slow me down. Surgery on May 19. Back at the desk May 20 or 21.
This roadmap does not stop. Neither do I.

---

**What Starts Next — Month 3**

- AWS DEA-C01 — Data Engineering Associate (Maarek course)
- Apache Spark — DataFrames, RDDs, Transformations, Actions
- First Spark ETL project in Python on GitHub
- Scala reading competency — read and adapt Spark-Scala code
- LeetCode — 1 Easy per day from Week 3 onwards

Month 2 is done. Month 3 starts next.

---
*Written: May 9, 2026 | Target: L5 — ML Platform / Data & Feature Infrastructure Engineer*
