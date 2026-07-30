## Month 4 Review — July 2026  
### Kafka · Airflow · System Design Start · First Streaming Pipeline

---

**Month 4 is officially complete.**

This phase ended up being shorter than expected in practice, because the DEA-C01 effort had already started back in Month 3 and was carried all the way through to passing. Even though the passing itself was still part of the Month 4 milestone, the hardest and most time-consuming part of that exam had already been absorbed earlier, which meant Month 4 could move faster once the certification pressure was gone. After that, I went all in with maximum motivation to finish the streaming project properly. The result is now public proof that Month 4 is done: Kafka + Airflow + System Design were started, learned, and applied in a real project through `event-stream-pipeline`.

---

**Roadmap Goals vs. Reality**

| Goal | Status | Note |
|---|---|---|
| Kafka fundamentals | ✅ Done | Topics, partitions, consumer groups, offsets, semantics applied in practice |
| Airflow fundamentals | ✅ Done | DAGs, scheduling, retries, XComs, backfill explored and used |
| System Design start | ✅ Done | Started with notes, reading, and structured thinking |
| First streaming pipeline project | ✅ Done | `event-stream-pipeline` completed as the Month 4 proof project |
| Month 4 roadmap milestone closed | ✅ Done | Main goals covered and validated in a real project |

---

**Projects That Prove Month 4**

**event-stream-pipeline**

I completed a event streaming project built around Kafka ingestion, Airflow orchestration, raw event persistence, validation, dead-letter handling, and daily metrics aggregation.

I worked intensely on this project, read documentation, fixed issues, and built it piece by piece until the pipeline functioned end to end. It is not perfect yet, and there is still room to improve some parts, but the core behavior works, and I am satisfied with the result. More importantly, it gave me a much better understanding of Kafka and Airflow in practice.

---

**What Was Covered**

**Kafka**  
The Month 4 project made Kafka concepts concrete: topics, partitions, consumer groups, offsets, and the practical effects of consumer configuration. That knowledge moved from theory into a working pipeline.

**Airflow**  
Airflow became real through DAG execution, task ordering, scheduling behavior, retries, and the operational side of getting a DAG to show up and run correctly. The project gave me a much clearer picture of how Airflow behaves in a real containerized setup.

**System Design**  
Month 4 also marked the start of system design work. It was the beginning of a broader foundation phase, not a full mastery phase yet, but it is now clearly in motion and no longer just a roadmap label.

---

**Personal Context**

This month was shaped by very intense work after passing DEA-C01 on 15.07.2026. That gave me a strong second wind, and I used it to go full speed on the streaming project instead of slowing down after the exam. The result is that Month 4 ended with a real finished project, not just notes or half-built drafts. That matters because the month was not easy, but it still turned into a strong deliverable.

---

**What Starts Next — Month 5**

- Feast and feature store fundamentals.
- Offline / online feature store design.
- Feature definitions and backfill logic.
- A new public project with documentation, tests, and CI/CD.
- Stronger GitHub hygiene and project presentation.

---

Month 4 is closed. Month 5 begins from a stronger base: Kafka and Airflow are no longer abstract, and the first streaming pipeline is now a finished proof project.
