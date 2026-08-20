# Today I Learned! Daily Log

A running log of things learned on the journey to becoming an
ML Platform | Data and Feature Infrastructure Engineer.

TIL Started: April 13, 2026

---

## August 20, 2026

**SAA-C03 Exam Prep | Day 21 · StackLessions — Load Balancing, Auto Scaling & Disaster Recovery**

Only four episodes today, but each one was dense enough that a fifth would have added noise rather than value. Better to stop while retention is still high and pick up Episode 20 fresh tomorrow.

**What I Covered**

*Episode 16 – Elastic Load Balancing: ALB vs NLB vs GWLB:*
- ALB (Layer 7): routes by path/host/header, attaches WAF, offloads auth to Cognito/OIDC, but has no static IP.
- NLB (Layer 4): ultra-low latency TCP/UDP/TLS, static/Elastic IP per AZ, the only ELB type that can back PrivateLink.
- GWLB (Layer 3): transparently inserts third-party appliances (firewalls, IDS/IPS) inline via GENEVE on port 6081, preserving original packet addresses.
- The classic trap: "HTTP" doesn't automatically mean ALB — a static IP requirement means NLB, or NLB in front of ALB.
- Cross-zone load balancing: always on for ALB, off by default (and billed) for NLB/GWLB.

*Episode 17 – EC2 Auto Scaling Strategies:*
- ASG fundamentals: min/desired/max capacity, launch templates (versioned, modern) vs launch configurations (deprecated since Oct 2024).
- The health check grace period trap: defaults to 300s in the console but 0 via CLI/SDK, killing instances before they finish booting.
- Four scaling policy types matched to demand shape: target tracking (unpredictable/gradual), step scaling (multi-threshold bursts), scheduled (known time-based spikes), predictive (recurring cyclical demand).
- Predictive scaling only scales out — a dynamic policy is still needed to scale back in.
- Warm pools (stopped state recommended) to cut scale-out latency without paying for idle running instances.

*Episode 18 – Relational Resilience: RDS Multi-AZ, Read Replicas, and Aurora:*
- The canonical trap: "route reads to the Multi-AZ standby" is always wrong — the standby is never readable.
- Multi-AZ (synchronous, automatic failover, same endpoint) vs Read Replicas (asynchronous, manually promoted, no auto-failover) as two independent dials: availability vs read scaling.
- Aurora's 6-copy/3-AZ storage redundancy exists automatically, even with zero replicas.
- Aurora Replicas (shared storage, sub-100ms lag, double as failover targets) vs standard RDS Read Replicas (separate async copies, seconds-to-minutes lag).
- Aurora Global Database for cross-region DR (RTO in minutes, RPO in seconds) and RDS Proxy for Lambda connection pooling (cuts failover from ~24s to ~3.1s).

*Episode 19 – Disaster Recovery Strategies and RTO/RPO:*
- RTO (downtime tolerance) and RPO (data-loss tolerance) as two independent axes that drive every DR decision.
- The four strategies on the cost-vs-speed spectrum: Backup & Restore (hours), Pilot Light (tens of minutes, data replicates but compute is off), Warm Standby (minutes, reduced capacity already running), Multi-Site Active/Active (near-zero, all regions serve live traffic).
- The critical distinction: pilot light can't serve traffic until compute is turned on; warm standby serves immediately at reduced capacity.
- Even active/active needs point-in-time backups — replication instantly copies corruption to every region.
- AWS Backup as the centralized policy layer (Vault Lock for WORM immutability, cross-account copies for ransomware isolation, Audit Manager for SOC 2 evidence).

**What This Means**

This block ties together nearly every resilience concept from the exam's second-heaviest domain (26%): load balancer selection, scaling strategy matching, and the RDS/Aurora availability-vs-read-scaling distinction that trips up even experienced engineers. The RTO/RPO decision framework in particular gives a repeatable method for any disaster recovery scenario question, regardless of the specific services involved.

> **What I understood**
> - Choosing between ALB, NLB, and GWLB comes down to OSI layer and specific requirements like static IP or PrivateLink support, not just "HTTP vs. not HTTP."
> - The health check grace period default trap (300s console vs. 0s CLI/SDK) is exactly the kind of operational detail that silently breaks a working setup.
> - Availability (Multi-AZ) and read scaling (Read Replicas) are two independent dials, not a single "resilience" setting, and Aurora's shared-storage replicas behave differently from standard RDS replicas.
> - RTO and RPO as two independent axes give a repeatable framework for any DR question, and knowing that even active/active setups still need point-in-time backups closes a subtle but important gap.
> - Stopping at four dense episodes instead of pushing a fifth protected retention rather than just maximizing volume.

**Result**

Four StackLessions episodes completed (Elastic Load Balancing, EC2 Auto Scaling Strategies, Relational Resilience, Disaster Recovery Strategies), each with practice quizzes. Picking up with Episode 20 tomorrow with a clear head rather than pushing a fifth episode tonight.

---

## August 19, 2026

**SAA-C03 Exam Prep | Day 20 · StackLessions — Web Protection, Threat Detection, Decoupling & Orchestration**

Today covered another dense StackLessions block: web application protection, threat detection and governance, a full retrieval checkpoint on the entire Secure Architectures domain, and two more episodes on decoupling and application integration.

**What I Covered**

*Episode 11 – Web Application Protection: WAF, Shield, and Network Firewall:*
- The layer map: WAF for Layer 7 (HTTP/HTTPS content — SQLi, XSS, rate-limiting), Shield for Layers 3/4 (volumetric DDoS), Network Firewall for Layers 3–7 inside the VPC (any protocol, DPI via Suricata), Firewall Manager for org-wide enforcement.
- Web ACL scope: Global (us-east-1, CloudFront only) vs Regional (ALB, API Gateway, AppSync).
- Shield Standard (free, automatic, L3/L4) vs Shield Advanced (paid, cost protection, 24/7 response team, pairs with WAF for L7 DDoS mitigation).
- Network Firewall for domain-based egress filtering (SNI/HTTP host) — a capability neither security groups nor WAF provide.

*Episode 12 – Threat Detection and Governance: GuardDuty, Macie, and CloudTrail:*
- The core distinction: GuardDuty (behavior — "is someone acting suspicious?"), Macie (content — "where does sensitive data live in S3?"), CloudTrail (the immutable record of every API call).
- AWS Config (state — "was this compliant over time?") vs CloudTrail (action — "who did it?").
- Security Hub as the aggregation layer — requires AWS Config enabled or most controls show INSUFFICIENT_DATA.
- Amazon Inspector for proactive CVE scanning across EC2, ECR, and Lambda.
- GuardDuty only detects, never remediates — automated response requires EventBridge → Lambda.

*Episode 13 – Secure Architectures Retrieval Checkpoint:*
- Five pure-retrieval scenarios covering the entire security block: EC2-to-S3 access (IAM role via instance profile), blocking a CIDR at the subnet level (NACL, not security group), RDS credential rotation with least effort (Secrets Manager, not Parameter Store), lowest-cost private S3 access (Gateway Endpoint, free), and discovering unprotected PII across buckets (Macie).

*Episode 14 – Decoupling Architectures: SQS, SNS, and EventBridge:*
- SQS as point-to-point buffering (Standard: high-throughput, at-least-once; FIFO: strict order, exactly-once).
- SNS as push-based fan-out to many subscribers at once.
- EventBridge as content-based routing with native SaaS integrations (45+ partner sources).
- The flagship pattern: one event triggering many services → SNS topic with one SQS queue per subscriber.
- SQS vs Kinesis Data Streams: SQS deletes on consumption (one consumer group), Kinesis retains and allows replay for multiple independent consumers.
- Scaling consumers by backlog-per-instance, not raw queue depth or CPU.

*Episode 15 – Application Integration: API Gateway and Step Functions:*
- API Gateway as the synchronous HTTP front door (REST API for usage plans/API keys/WAF/caching vs cheaper HTTP API for simple Lambda proxying).
- Step Functions as the serverless workflow orchestrator with built-in Retry and Catch.
- Standard Workflows (exactly-once, up to 1 year, for anything non-idempotent like payments) vs Express Workflows (at-least-once, high-volume, cheaper).
- The 29-second API Gateway integration timeout ceiling and the async 202-Accepted-then-poll fix for long-running workflows.
- API keys identify and throttle clients but never authenticate or authorize — that needs IAM, Cognito, or a Lambda authorizer.

**What This Means**

Today closed out the entire "Design Secure Architectures" domain (30% of the exam) with a dedicated retrieval checkpoint, then pivoted into Application Integration patterns (SQS/SNS/EventBridge/API Gateway/Step Functions) that show up heavily across both the Resilient and High-Performing domains — directly targeting the 33% weak spot flagged in the earlier Tutorials Dojo mock exam.

> **What I understood**
> - WAF, Shield, and Network Firewall map to distinct OSI layers, and the exam expects picking the tool by the layer and traffic type, not by generic "security" framing.
> - GuardDuty, Macie, CloudTrail, and Config each answer a fundamentally different question (behavior, content, action, state), and confusing them is a classic distractor pattern.
> - The retrieval checkpoint format (recalling answers cold, not just recognizing them) is a much better test of exam-day readiness than passive review.
> - SQS, SNS, EventBridge, and Kinesis are frequently confused but solve different decoupling problems, and the "one event, many services" pattern (SNS fan-out to per-subscriber SQS queues) is a recurring exam scenario.
> - Standard vs. Express Step Functions workflows hinge entirely on idempotency requirements, a distinction that's easy to overlook under time pressure.

**Result**

Five StackLessions episodes completed (Web Application Protection, Threat Detection & Governance, Secure Architectures Checkpoint, Decoupling Architectures, Application Integration), each with practice quizzes, continuing the parallel Tutorials Dojo review cycle.

---

## August 18, 2026

**SAA-C03 Exam Prep | Day 19 · StackLessions — Secrets, S3 Security & VPC Networking Deep Dive**

Today continued the StackLessions series with five dense episodes covering runtime secrets management, the full S3 security stack, VPC networking foundations, traffic control between security groups and NACLs, and VPC endpoints with PrivateLink.

**What I Covered**

*Episode 6 – Runtime Secrets: Secrets Manager vs Parameter Store:*
- Parameter Store (free standard tier, SecureString via KMS, no built-in rotation) vs Secrets Manager (paid, purpose-built for credentials with automatic rotation).
- The decision rule: need automatic database credential rotation → Secrets Manager; just need cheap encrypted config → Parameter Store SecureString.
- Common trap: forgetting `WithDecryption=true` returns ciphertext, not plaintext.
- Canonical pattern: EC2 instance role fetches secrets at runtime, never stored in user-data.

*Episode 7 – Securing Data in Amazon S3:*
- The six layers of S3 defense: Block Public Access, bucket policies/disabled ACLs, server-side encryption, deny-unencrypted-puts policy, presigned URLs/VPC endpoints, and Object Lock (WORM).
- Object Ownership "Bucket owner enforced" as the fix for cross-account upload ownership problems.
- SSE-S3 vs SSE-KMS vs SSE-C vs DSSE-KMS, and why SSE-KMS with a customer-managed key is the compliance answer (CloudTrail audit trail).
- Presigned URLs (max 7 days via CLI/SDK) and their hidden expiry tied to the credentials that signed them.
- Object Lock Governance mode (bypassable) vs Compliance mode (no bypass, ever — not even root).

*Episode 8 – VPC Foundations: Subnets, Route Tables, Internet and NAT Gateways:*
- VPC scoped to one region across AZs; a subnet lives in exactly one AZ.
- Public vs private subnet is purely a route table decision — no toggle exists.
- Internet Gateway (two-way traffic for public IPs) vs NAT Gateway (outbound-only for private subnets, must sit in a public subnet, one per AZ for HA).
- VPC Peering is non-transitive (point-to-point only) vs Transit Gateway as the transitive hub for many VPCs.

*Episode 9 – VPC Traffic Control: Security Groups vs NACLs:*
- Security Groups: instance-level, allow-only, stateful (return traffic automatic), evaluated as one combined ruleset.
- NACLs: subnet-level, allow AND deny, stateless (return traffic must be explicit), numbered rules evaluated in order with first-match-wins.
- The classic ephemeral-port trap: NACLs need an explicit outbound rule for ports 1024–65535 or replies get silently dropped.
- Only a NACL can block a specific IP/CIDR — security groups cannot express deny.
- Security group referencing (by group ID) for maintenance-free multi-tier architectures.

*Episode 10 – Private Connectivity: VPC Endpoints and PrivateLink:*
- Gateway Endpoints: free, route-table-based, S3 and DynamoDB only, same-region/same-VPC traffic only.
- Interface Endpoints (PrivateLink): ENI with a private IP, covers 100+ services, billed hourly + per-GB, works cross-region and on-premises.
- Gateway endpoints cannot serve on-premises, peered VPCs, or cross-region traffic — that requires an interface endpoint.
- Endpoint policies are additive on top of IAM and bucket policies, useful for blocking data exfiltration to external buckets.

**What This Means**

Today's material lines up almost perfectly with the two heaviest exam domains (Security 30%, Resilient Architectures 26%) and directly reinforces the VPC networking I covered in the Maarek course, but now framed as decision rules rather than a feature tour — exactly the kind of reinforcement needed to convert the earlier 33%-on-High-Performing weak spot into real exam-day confidence.

> **What I understood**
> - The Secrets Manager vs. Parameter Store decision boils down to one question: does this need automatic rotation, or just cheap encrypted storage?
> - S3 security is genuinely layered defense-in-depth (six distinct controls), not a single setting, and Object Lock Compliance mode is the one true point of no return, even for root.
> - Security Groups and NACLs solve overlapping but distinct problems — stateful allow-only at the instance level vs. stateless allow/deny at the subnet level — and the ephemeral-port trap is a classic exam gotcha.
> - Gateway Endpoints and Interface Endpoints are not interchangeable: only PrivateLink-based interface endpoints support cross-region, on-premises, and the 100+ service coverage beyond S3/DynamoDB.

**Result**

Five StackLessions episodes completed (Runtime Secrets, S3 Security, VPC Foundations, Security Groups vs NACLs, VPC Endpoints & PrivateLink), each with built-in practice quizzes, continuing in parallel with the Tutorials Dojo review cycle.

---

## August 17, 2026

**SAA-C03 Exam Prep | Day 18 · StackLessions Series — A Genuine Game Changer**

Today I worked through the full StackLessions "Cloud Architect Path" video series for SAA-C03, and it lived up to the hype from DEA-C01: the way this content is structured for exam-day thinking, not just service memorization, is genuinely one of the best study formats I've used.

**What I Covered**

*Episode 1 – Exam Format, Domain Weights & How to Study:*
- Scaled scoring (100–1000, pass at 720), compensatory scoring model, pretest questions.
- Domain weight breakdown: Secure Architectures 30%, Resilient 26%, High-Performing 24%, Cost-Optimized 20% — Security and Resilience alone make up 56% of the exam.
- The "decathlon, not pass/fail gates" mental model and the four-step reading method: read the qualifier, list hard requirements, eliminate violations, apply the qualifier.
- The three traps for experienced engineers: over-engineering, technically-valid-but-suboptimal answers, and near-identical service name pairs.

*Episode 2 – IAM Core: Users, Groups, Roles, and Policy Evaluation:*
- Default-deny model, explicit Deny always wins over Allow.
- Users vs Groups vs Roles, and why groups can only contain users (no nesting, no roles).
- The canonical EC2 pattern: IAM role via instance profile, never stored access keys.
- Identity-based vs resource-based policies, spotting a resource-based policy via the Principal element.
- Root account hygiene: MFA always on, never create root access keys.

*Episode 3 – IAM Roles Assumed: STS, Cross-Account Access, SCPs, and Permission Boundaries:*
- STS AssumeRole and temporary credentials as the tie-breaker heuristic.
- Trust policy (who) vs permissions policy (what) on every role.
- External ID for defending against the confused deputy problem.
- SCPs as org-wide guardrails that only restrict, never grant; permission boundaries as the per-identity equivalent.
- IAM Roles Anywhere for on-premises/non-AWS workloads via X.509 certificates.

*Episode 4 – Workforce and Application Identity: IAM Identity Center and Cognito:*
- The bright-line rule: workforce and multi-account access → IAM Identity Center; application end users → Cognito.
- SAML for authentication vs SCIM for provisioning/directory sync.
- Cognito user pools (authentication, "who you are") vs identity pools (authorization, "what you can touch").
- Why IAM users are the wrong tool for customer-facing apps (5,000 user quota, long-term credential risk).

*Episode 5 – Encryption and Key Management: KMS and CloudHSM:*
- Envelope encryption: KMS wraps a data key, the data key does the bulk encryption locally.
- The three KMS key types (AWS-owned, AWS-managed, customer-managed) on a control spectrum.
- The KMS key policy as a separate authorization gate from IAM — the most common AccessDenied cause.
- Automatic key rotation rules, and why imported (BYOK) key material can only rotate manually.
- When CloudHSM is the correct answer: single-tenant, sole-custody hardware mandates only.
- KMS (at rest) vs ACM (in transit), and the us-east-1 quirk for CloudFront certificates.

**What This Means**

This series reframes the entire exam around decision rules instead of feature lists — exactly the shift needed after weeks of service-by-service course content. The heavy emphasis on Security (30% of the exam) tracks with my current domain mastery gap from the Tutorials Dojo mock exams, so front-loading this material now is well-timed rather than redundant.

> **What I understood**
> - The exam rewards decision rules (qualifier reading, requirement elimination) far more than raw service recall, which is a genuinely different skill than what the Maarek course built.
> - Security and Resilience together make up 56% of the exam weight, which validates prioritizing the security-heavy episodes right now given my mock exam weak spots.
> - Small distinguishing rules — explicit Deny always wins, groups can't be nested, KMS key policies as a separate gate from IAM — are exactly the kind of near-identical trap the exam is designed to test.
> - Running StackLessions in parallel with the Tutorials Dojo review cycle, rather than replacing it, keeps both the conceptual reframing and the steady score-trend momentum intact.

**Result**

Five full StackLessions episodes completed today, each with built-in practice quizzes. This series becomes the primary study format for the coming days, run in parallel with the ongoing Tutorials Dojo review cycle.

---

## August 16, 2026

**SAA-C03 Exam Prep | Strategy Shift — StackLessions YouTube Series Discovered**

Big find today: StackLessions has finally uploaded a dedicated video series for SAA-C03 on YouTube, mirroring what they did for DEA-C01. Since that format had the single biggest learning impact on me during DEA-C01 prep, I'm shifting focus for the next several days to work through this new series in full.

**What This Means**

This isn't a random course-hop — it's a deliberate return to the resource format that historically produced my best retention. The plan is to treat the next few days primarily as a StackLessions video block, layered on top of (not replacing) the Tutorials Dojo question-review cycle that's been steadily pushing my 30-day score trend upward (43% → 59% this past week).

> **What I understood**
> - Switching resources deliberately, based on a format that already proved effective for DEA-C01, is different from random course-hopping.
> - The StackLessions block is meant to layer on top of the existing Tutorials Dojo review cycle, not replace the momentum it already built.
> - Protecting the upward 30-day trend (43% → 59%) means keeping the question-review cycle running in parallel rather than pausing it for new content.

**Result**

Study focus pivots starting now: StackLessions SAA-C03 video series becomes the primary content source for the next few days, while Tutorials Dojo practice questions continue in parallel to keep reinforcing weak domains.

**SAA-C03 Exam Prep | Day 17 · Reviewing Yesterday's Questions — Steady Upward Climb**

Today I went back through yesterday's fresh questions and did a full review pass, including reading the explanations for every answer I got wrong again. It's a smaller, more focused session, but the trend line keeps moving in the right direction.

**Session Numbers**

- Questions answered today: 96, at 67% correct.
- Overall activity: 5 active study days, 336 questions total in the last 12 weeks.
- Score trend (last 30 days): now at a latest 59%, continuing the climb from 43% (Aug 13) → 56% (Aug 14) → 56% (Aug 15) → 59% today. Still flagged "Needs Work" against the 72% pass line, but the trajectory has been consistently upward since the Aug 13 low point.

**What This Means**

Going from 57% on yesterday's fresh attempt to 67% on today's review of the exact same question set shows the explanation-reading habit is doing real work — it's converting first-pass mistakes into retained knowledge rather than just repeated guessing. More importantly, the 30-day trend line itself has now risen for three consecutive check-ins, which is a much stronger signal than any single day's score.

> **What I understood**
> - Re-reviewing the same question set with explanations converted a meaningful share of yesterday's mistakes into retained knowledge, shown by the 57% → 67% jump.
> - Three consecutive upward check-ins on the 30-day trend line is a stronger signal of real progress than any single day's score.
> - The repeating cycle of fresh questions followed immediately by a review-with-explanations pass appears to be the actual mechanism moving the 30-day average, not just raw question volume.
> - Still being flagged "Needs Work" at 59% against a 72% threshold means there's real gap left, but the direction and consistency of the trend now matter more than the absolute number.

**What I Understood**

The pattern across this week is becoming clear: fresh questions expose the actual gap, and the immediate review-with-explanations pass closes a meaningful chunk of it right away. Repeating this cycle — new batch, then review — seems to be the mechanism actually moving the 30-day average, rather than volume alone.

**Result**

96 questions reviewed at 67% correct, pushing the 30-day trend to 59%. Progress is steady and visible; next step is continuing the new-question-then-review cycle to keep closing the gap toward the 72% pass threshold.

---

## August 15, 2026

**SAA-C03 Exam Prep | Day 16 · Fresh Questions Reveal Real Progress**

Today I deliberately worked through a batch of completely new, previously unseen questions in Tutorials Dojo to get an honest read on my actual knowledge level — separate from the boost that reviewing already-seen questions tends to give. The result confirms a genuine learning curve is forming.

**Session Numbers**

- Questions answered today: 96, at 57% correct.
- Overall activity: 4 active study days, 240 questions total in the last 12 weeks.
- Score trend (last 30 days): climbed from 43% (Aug 13) to 56% (Aug 14) to a latest 56% today, still flagged "Needs Work" against the 72% pass line but clearly trending upward since the Aug 13 low point.

**What This Means**

Testing myself on entirely fresh questions rather than repeating known ones is the more rigorous signal here, and scoring 57% on unfamiliar material — up from the 42% low earlier this week — shows the improvement isn't just an artifact of memorizing specific question-answer pairs from Smart Review. The underlying concept understanding is genuinely strengthening.

> **What I understood**
> - Testing on fresh, unseen questions is a more honest measure of readiness than repeating known ones, since it isolates real concept understanding from memorized answers.
> - A 42% → 57% jump on unfamiliar material this week is meaningful evidence that Smart Review is building durable knowledge, not just pattern-matching on specific questions.
> - The trend line curving upward since the Aug 13 low point matters more than any single day's percentage, since it shows sustained direction rather than a one-off spike.
> - Still being flagged "Needs Work" against the 72% threshold is expected at this stage; the goal now is closing the remaining gap through the same review discipline that worked earlier this week.

**What I Understood**

Comparing this session against the past few days makes the trajectory clear: Aug 12 baseline mock at 50%, a dip to 42% on Aug 13 after hitting harder material, recovery to 68% on Aug 14 through Smart Review of known mistakes, and now 57% on brand-new questions today. That pattern is consistent with real skill consolidation rather than noise.

**Result**

96 new questions completed at 57% correct, confirming an upward learning curve. Tomorrow's plan is to go back and review today's specific mistakes and their explanations before introducing the next batch of fresh questions.

---

## August 14, 2026

**SAA-C03 Exam Prep | Day 15 · Smart Review Pays Off — Score Trend Turns "Improving"**

Today I used Tutorials Dojo's Smart Review feature for the first time, which resurfaces previously missed questions and lets me read through their explanations again. The impact on today's numbers was immediate and clear.

**Session Numbers**

- Questions answered today: 69, at 68% correct — just shy of the 72% pass line.
- Overall activity: 3 active study days, 144 questions total in the last 12 weeks.
- Score trend (last 30 days): moved from 43% (Aug 13) up to a latest score of 55%, with the trend now officially flagged as "Improving" instead of "Needs Work."

**What This Means**

The jump from 42% correct yesterday to 68% correct today validates the review approach directly: instead of grinding fresh questions blindly, revisiting exactly the questions I got wrong and reading their explanations closes knowledge gaps far more efficiently. Smart Review essentially turns every mistake into a targeted micro-lesson rather than a one-off wrong answer, which is exactly the kind of deliberate practice that moves a score trend from declining to improving in a single session.

> **What I understood**
> - Smart Review's targeted resurfacing of missed questions closed gaps far faster than answering fresh, unrelated questions would have.
> - A single well-designed review session can shift a 30-day trend line from "Needs Work" to "Improving," showing that method matters more than raw question volume.
> - Being close to the 72% pass line on a daily score, even if not yet there, is a meaningful signal that the underlying gaps are shrinking, not just noise.
> - The next real test is whether these gains hold up on fresh, previously unseen questions, not just on material I've already reviewed.

**What I Understood**

This confirms the plan from yesterday was the right one. The daily score itself (68%) is already close to the 72% pass threshold, and the 30-day trend line curving upward suggests that consistent Smart Review sessions, rather than just volume of new questions, is the more effective lever for closing the remaining gap to exam-readiness.

**Result**

69 questions completed at 68% correct, pushing the 30-day trend to "Improving" at 55% latest. Next step is continuing Smart Review cycles on remaining flagged questions while gradually mixing in fresh questions to confirm the retained gains transfer beyond just previously-seen material.

---

## August 13, 2026

**SAA-C03 Exam Prep | Day 14 · Weaker Session — 65 Questions at 42% Correct**

Today was a noticeably weaker study day compared to the recent streak. I worked through 65 questions in Tutorials Dojo and only scored 42% correct, pulling my 30-day score trend further below the 72% pass line to a latest score of 43%.

**Session Numbers**

- Questions answered today: 65, at 42% correct.
- Overall activity so far: 2 active study days, 75 questions total in the last 12 weeks.
- Score trend (last 30 days): dropped from roughly 50% on August 12 to 43% on August 13, still well below the 72% pass threshold, flagged as "Needs Work."

**What This Means**

A dip like this right after finishing the full course and taking the first mock exam is a normal part of the process — it usually reflects genuine knowledge gaps surfacing under exam-style pressure rather than a step backward. What matters now is how I close those gaps, not the raw score itself.

> **What I understood**
> - A score dip right after course completion is a common pattern, since exam-style questions surface gaps that passive video content doesn't reveal.
> - Volume of questions answered isn't the same as progress; the trend line matters more than any single session's score.
> - Reading explanations for every missed question, not just the correct answer, is the mechanism that actually converts weak recall into durable understanding.
> - Staying at "Needs Work" status this early is expected and shouldn't be read as a setback, since the review process hasn't started yet.

**My Plan Going Forward**

I'm staying confident because I have a concrete follow-up strategy: bookmarking every missed question, then carefully reading through the explanations for each one rather than just noting the correct answer. This kind of deliberate review — understanding *why* an option is right or wrong — tends to convert weak recall into durable understanding much faster than re-answering fresh questions blindly.

**Result**

65 questions completed at 42% today; next step is a dedicated review pass over all flagged incorrect questions and their explanations before attempting further mock exams, working toward exam-readiness at the 72%+ pass level.

---

## August 12, 2026

**SAA-C03 Exam Prep | Day 13 · First Tutorials Dojo Mock Exam — Reality Check**

Today I purchased the Tutorials Dojo SAA-C03 practice exam bundle and immediately took a 100-question mock exam to get an honest baseline now that the Maarek course is fully completed. The result: not passed, but a clear and actionable map of where the remaining work needs to go.

**Mock Exam Results**

- Score: 50% (passing threshold is 70%)
- Correct: 50 / Incorrect: 50
- Status: Not Passed — "Keep going, review the missed answers and target your weak domains"

**Domain Performance**

| Domain | Score |
|---|---|
| Design Cost-Optimized Architectures | 75% |
| Design Resilient Architectures | 50% |
| Design Secure Architectures | 50% |
| Design High-Performing Architectures | 33% |

**What This Means**

Cost-Optimized Architectures is clearly my strongest domain right now, while Design High-Performing Architectures is the weakest by a wide margin at 33%. This domain covers compute, storage, database, and networking solutions for performance — directly overlapping with the EC2, VPC, and storage sections I worked through hard in the past week, so the gap suggests I need deeper review rather than just re-watching, likely through targeted question review and hands-on reinforcement of performance-oriented service choices (e.g., placement groups, caching, read replicas, storage classes).

Design Resilient and Secure Architectures sitting at 50% each are close behind and represent the next priority tier, since both are large, heavily-weighted domains on the actual exam.

> **What I understood**
> - Finishing the full course does not automatically translate into a passing score; the mock exam exposed a real gap between watching content and applying it under exam conditions.
> - Design High-Performing Architectures needs targeted question review and hands-on reinforcement, not just a re-watch of the course sections.
> - Cost-Optimized Architectures being the strongest domain suggests trade-off reasoning transfers well when the decision criteria are more clear-cut (pricing models, storage tiers).
> - Resilient and Secure Architectures are close behind at 50% and should be treated as the next priority tier after Performance.

**Result**

First Tutorials Dojo mock exam completed with a 50% baseline score. Next step is reviewing all missed answers in detail and building a focused study plan around Design High-Performing Architectures first, then Resilient and Secure Architectures, before attempting the next mock exam.

---

## August 11, 2026

**SAA-C03 Exam Prep | Day 12 · COURSE COMPLETE**

Today marks the finish line: I completed the entire Maarek SAA-C03 course from Section 28 all the way through to the "Congratulations" lecture. Six sections in one day, closing out disaster recovery, advanced architectures, miscellaneous services, whitepapers, and exam preparation itself.

**What I Covered**

*Section 28 – Disaster Recovery & Migrations (completed):*
- Disaster Recovery in AWS, Elastic Disaster Recovery (DRS).
- Database Migration Service (DMS) and Hands On, RDS/Aurora Migrations.
- On-Premises Strategies with AWS, AWS Backup and Hands On.
- Application Migration Service (MGN), Transferring Large Datasets into AWS.
- VMware Cloud on AWS.
- Completed the Disaster Recovery & Migration Quiz (10 questions).

*Section 29 – More Solution Architectures (completed):*
- Event Processing in AWS, Caching Strategies in AWS.
- Blocking an IP Address in AWS, High Performance Computing (HPC) on AWS.
- EC2 Instance High Availability.
- Completed the More Solution Architectures Quiz (3 questions).

*Section 30 – Other Services (completed):*
- CloudFormation Intro, Hands On, Service Role.
- Amazon SES, Amazon Pinpoint.
- SSM Session Manager and Other Services.
- AWS Cost Explorer, AWS Cost Anomaly Detection.
- AWS Outposts, AWS Batch, Amazon AppFlow, AWS Amplify, Instance Scheduler on AWS.
- Completed the Other Services Quiz (5 questions).

*Section 31 – Whitepapers and Architectures (completed):*
- AWS Well-Architected Framework & Well-Architected Tool.
- AWS Trusted Advisor Overview and Hands-On.
- Examples of Architecture for the SAA exam.
- Completed the Whitepapers & Architectures Quiz (1 question).

*Section 32 – Preparing for the Exam (completed):*
- Exam Preparation Introduction, State of Learning Checkpoint.
- Exam Tips, Links to Whitepapers (article).
- Exam Walkthrough and Signup, saving on exam cost, extra time for non-native English speakers.
- How does the exam work, plus the built-in practice exam.

*Section 33 – Congratulations (completed):*
- AWS Certification Paths.
- Congratulations lecture and Bonus Lecture.

**What This Means**

This closes the full instructional portion of the SAA-C03 prep. Disaster recovery and migration services (DMS, DRS, AWS Backup) round out the resilience domain, while the Well-Architected Framework and Trusted Advisor tie together the architectural best practices that show up across nearly every exam scenario. The exam-prep section itself now gives me a clear checklist for signup, timing, and logistics.

> **What I understood**
> - DMS, DRS, and AWS Backup each target a different point in the migration/resilience lifecycle, and the exam expects precise matching of scenario to service.
> - The Well-Architected Framework and Trusted Advisor function as the connecting thread across nearly every domain covered so far, not as a standalone topic.
> - Finishing the exam-logistics section removes uncertainty about signup, timing, and accommodations, so the only remaining unknowns are content-related.
> - Completing all 33 sections confirms the course backlog is fully closed; the next phase is entirely about targeted practice and weak-domain review, not new material.

**Result**

Course officially finished end-to-end — all 33 sections and their quizzes completed. Next milestone: shifting fully into practice exams and targeted review of weak domains (Compute and Networking, per the baseline exams) ahead of scheduling the real SAA-C03 exam.

---

## August 10, 2026

**SAA-C03 Exam Prep | Day 11 · Monitoring, IAM Advanced, Security & Full VPC Networking**

Back to maximum intensity as planned — today I processed a massive amount of material, completing four full sections that together cover operational visibility, advanced identity management, security services, and the entire VPC networking deep dive.

**What I Covered**

*Section 24 – AWS Monitoring & Audit: CloudWatch, CloudTrail, Config (completed):*
- CloudWatch Metrics, Logs, Logs Hands On, Live Tail Hands On, Agent.
- CloudWatch Alarms and Hands On, Network Synthetic Monitor.
- Amazon EventBridge Overview and Hands On.
- CloudWatch Insights and Operational Visibility.
- CloudTrail Overview, Hands On, and EventBridge Integration.
- AWS Config Overview and Hands On.
- CloudTrail vs CloudWatch vs Config comparison.
- Completed the Monitoring & Auditing Quiz (16 questions).

*Section 25 – Identity and Access Management (IAM) Advanced (completed):*
- AWS Organizations Overview, Hands On, Tag Policies.
- IAM Advanced Policies, Resource-based Policies vs IAM Roles, Policy Evaluation Logic.
- AWS IAM Identity Center.
- AWS Directory Services and Hands On.
- AWS Control Tower.
- Completed the IAM Advanced Quiz (5 questions).

*Section 26 – AWS Security & Encryption: KMS, SSM Parameter Store, Shield, WAF (completed):*
- Encryption 101, KMS Overview, Hands On with CLI, Multi-Region Keys.
- S3 Replication with Encryption, Encrypted AMI Sharing Process.
- SSM Parameter Store Overview and Hands On CLI.
- AWS Secrets Manager Overview and Hands On.
- AWS Certificate Manager (ACM), AWS CloudHSM.
- WAF, Shield DDoS Protection, Firewall Manager, WAF/Shield Hands On.
- DDoS Protection Best Practices.
- Amazon GuardDuty, Amazon Inspector, Amazon Macie.
- Completed the AWS Security & Encryption Quiz (25 questions).

*Section 27 – Networking: VPC (completed):*
- CIDR, Private vs Public IP, Default VPC Overview, VPC Overview and Hands On.
- Subnet Overview and Hands On.
- Internet Gateways & Route Tables, and Hands On.
- Bastion Hosts, NAT Instances, NAT Gateways, Regional NAT Gateway (all with Hands On).
- NACLs & Security Groups and Hands On.
- VPC Peering and Hands On.
- VPC Endpoints and Hands On.
- VPC Flow Logs and Hands On with Athena.
- Site-to-Site VPN, Virtual Private Gateway/Customer Gateway and Hands On.
- Direct Connect, Direct Connect Gateway, Transit Gateway.
- VPC Traffic Mirroring, IPv6 for VPC and Hands On.
- Egress-Only Internet Gateway and Hands On, Section Cleanup, VPC Section Summary.
- Networking Costs in AWS, AWS Network Firewall.
- Completed the VPC Quiz (23 questions).

**What This Means**

This was one of the heaviest and most exam-critical days yet. VPC networking alone is consistently one of the largest, most tested domains on SAA-C03, and combining it with the security/encryption stack (KMS, WAF, Shield, GuardDuty) and monitoring/auditing tools (CloudWatch, CloudTrail, Config) covers nearly the entire "secure and monitor architectures" pillar of the exam in one sitting.

> **What I understood**
> - CloudWatch, CloudTrail, and Config each answer a different question (metrics/alarms, API activity history, and configuration compliance), and mixing them up is a common exam trap.
> - IAM policy evaluation logic and resource-based policies vs. roles are precise, rule-based mechanics that need to be memorized exactly, not approximated.
> - KMS, Secrets Manager, and SSM Parameter Store overlap in purpose but differ in rotation, cost, and integration depth — exactly the kind of distinction SAA-C03 tests.
> - VPC networking, my confirmed weakest baseline domain, got its most complete single-day coverage yet, from CIDR and subnets through NAT, peering, endpoints, VPN, and Direct Connect.

**Result**

Four full sections completed end-to-end (Monitoring & Audit, IAM Advanced, Security & Encryption, VPC Networking) with all four section quizzes finished, confirming the return to full study intensity after vacation.

---

## August 9, 2026

**SAA-C03 Exam Prep | Day 10 · Quiet Day — Data & Analytics and Machine Learning Overview**

Today was a calmer, lower-volume day compared to the recent streak, but I still closed out two content-rich sections covering NoSQL/analytics extras, the full data & analytics toolset, and a broad AWS machine learning services overview. Starting tomorrow, I'm going back to maximum intensity.

**What I Covered**

*Section 21 – Databases & Analytics Extras (tail end):*
- Amazon Keyspaces (for Apache Cassandra).
- Amazon Timestream.

*Section 22 – Data & Analytics (completed):*
- Amazon Athena and Athena Hands On.
- Amazon Redshift.
- OpenSearch (formerly ElasticSearch).
- Amazon EMR.
- Amazon QuickSight.
- AWS Glue and Lake Formation.
- Amazon Managed Service for Apache Flink and its Hands On.
- MSK (Managed Streaming for Apache Kafka).
- Big Data Ingestion Pipeline overview.
- Completed the Data & Analytics Quiz (14 questions).

*Section 23 – Machine Learning (completed):*
- Rekognition, Transcribe, Polly, and Translate overviews.
- Lex + Connect Overview.
- Comprehend and Comprehend Medical overviews.
- SageMaker AI Overview.
- Kendra, Personalize, and Textract overviews.
- Machine Learning Summary.
- Completed the Machine Learning Quiz (13 questions).

**What This Means**

Even on a lighter day, this material matters: analytics services like Athena, Redshift, and Glue frequently show up on the SAA-C03 exam in data pipeline and reporting scenarios, while the AI/ML overview section builds the "which service does what" recognition needed for scenario-based questions, even though deep ML expertise isn't required for this exam.

> **What I understood**
> - Athena, Redshift, EMR, and Glue each fit different points in a data pipeline, and the exam tests recognizing which one fits a given scenario rather than deep implementation detail.
> - MSK sitting alongside Kinesis and Data Firehose shows there are multiple valid streaming approaches on AWS, chosen based on existing tooling and control needs.
> - The ML section is about service recognition (Rekognition, Comprehend, Textract, Lex, SageMaker, etc.), not deep machine learning expertise, since SAA-C03 tests architecture decisions, not ML theory.
> - A lighter day still made real progress, closing two full sections while leaving room to reset before increasing intensity again tomorrow.

**Result**

Section 21 wrapped up, Section 22 (Data & Analytics) and Section 23 (Machine Learning) both completed in full including their quizzes, with a heavier pace resuming tomorrow.

---

## August 8, 2026

**SAA-C03 Exam Prep | Day 9 · Storage Extras, Messaging, Containers & Serverless**

Today continued the increased study pace from Cologne, covering five full sections: AWS storage extras, application decoupling with messaging services, containers, and a deep serverless block ending in solutions architecture discussions.

**What I Covered**

*Section 16 – AWS Storage Extras (completed):*
- AWS Snow Family Overview and Hands On, Snowball into Glacier architecture.
- Amazon FSx and Hands On.
- Storage Gateway Overview and Hands On.
- AWS Transfer Family, DataSync Overview.
- All AWS Storage Options Compared.
- Completed the AWS Storage Extras Quiz (17 questions).

*Section 17 – Decoupling Applications: SQS, SNS, Kinesis, Active MQ (completed):*
- Introduction to Messaging.
- SQS Standard Queues Overview and Hands On, Message Visibility Timeout, Long Polling, FIFO Queues.
- SQS + Auto Scaling Group integration.
- Amazon SNS Overview and Hands On, SNS + SQS Fan Out Pattern.
- Amazon Kinesis Data Streams and Hands On.
- Amazon Data Firehose and Hands On.
- SQS vs SNS vs Kinesis comparison.
- Amazon MQ.
- Completed the Messaging & Integration Quiz (12 questions).

*Section 18 – Containers on AWS: ECS, Fargate, ECR & EKS (completed):*
- Docker Introduction.
- Amazon ECS, ECS Cluster Hands On, ECS Service Hands On.
- ECS Auto Scaling, ECS Solutions Architectures, ECS Clean Up.
- Amazon ECR.
- Amazon EKS Overview and Hands On.
- Completed the Containers on AWS Quiz (7 questions).

*Section 19 – Serverless Overviews from a Solutions Architect Perspective (completed):*
- About the Serverless Section (article), Serverless Introduction.
- Lambda Overview, Hands-On, Limits, Concurrency and Hands On.
- Lambda SnapStart, Lambda@Edge & CloudFront Functions, Lambda in VPC.
- RDS Invoking Lambda & Event Notifications.
- Amazon DynamoDB, Hands-On, and Advanced Features.
- API Gateway Overview and Basics Hands-On.
- Step Functions, Amazon Cognito Overview.
- Completed the Serverless Overview Quiz (17 questions).

*Section 20 – Serverless Solutions Architecture Discussions (completed):*
- Mobile Application case study: MyTodoList.
- Serverless Website case study: MyBlog.com.
- MicroServices Architecture.
- Software Updates Distribution.
- Completed the Serverless Solutions Architecture Discussions Quiz (8 questions).

**What This Means**

This day covered a huge swath of modern application architecture: decoupled messaging patterns (SQS/SNS/Kinesis), container orchestration (ECS/EKS), and the full serverless stack (Lambda, DynamoDB, API Gateway, Cognito). These topics are heavily represented on the SAA-C03 exam under application integration and modern architecture design, so closing five sections in one sitting is a major dent in the remaining course backlog.

> **What I understood**
> - SQS, SNS, and Kinesis solve different decoupling problems (queueing, pub/sub fan-out, and streaming), and the exam expects precise use-case matching.
> - ECS, Fargate, ECR, and EKS represent a spectrum from managed orchestration to full Kubernetes control, each with different operational trade-offs.
> - The serverless stack (Lambda, DynamoDB, API Gateway, Cognito, Step Functions) forms a cohesive architecture pattern rather than isolated services.
> - Covering five sections in one day shows the storage, messaging, container, and serverless domains are now much more consolidated than before this study block.

**Result**

Five sections completed end-to-end (Storage Extras, Messaging & Integration, Containers, Serverless Overview, Serverless Solutions Architecture Discussions), with all five section quizzes finished.

---

## August 7, 2026

**SAA-C03 Exam Prep | Day 8 · Massive Volume Increase — RDS Through CloudFront**

Today I finally delivered on the promise to increase study volume now that I'm back in Cologne, and it shows: I worked through 7 full course sections in a single day, from database fundamentals all the way to global content delivery.

**What I Covered**

*Section 9 – AWS Fundamentals: RDS + Aurora + ElastiCache (completed):*
- Amazon RDS Overview, RDS Read Replicas vs Multi-AZ, RDS Hands On.
- RDS Custom for Oracle and Microsoft SQL Server.
- Amazon Aurora, Aurora Hands On, Aurora Advanced Concepts.
- RDS & Aurora Backup and Monitoring, RDS Security, RDS Proxy.
- ElastiCache Overview, ElastiCache Hands On, ElastiCache for Solutions Architects.
- List of Ports to be familiar with (article).
- Completed the RDS, Aurora, & ElastiCache Quiz (24 questions).

*Section 10 – Route 53 (completed):*
- What is a DNS, Route 53 Overview, registering a domain, creating first records.
- Route 53 EC2 Setup, TTL, CNAME vs Alias.
- Routing Policies: Simple, Weighted, Latency, Failover, Geolocation, Geoproximity, IP-based, Multi-Value.
- Route 53 Health Checks and Hands On.
- 3rd Party Domains & Route 53, Route 53 Resolvers & Hybrid DNS, Section Cleanup.
- Completed the Route 53 Quiz (7 questions).

*Section 11 – Classic Solutions Architecture Discussions (completed):*
- Solutions Architecture Discussions Overview.
- Case studies: WhatsTheTime.com, MyClothes.com, MyWordPress.com.
- Instantiating applications quickly, Beanstalk Overview, Beanstalk Hands On.
- Completed the Classic Solutions Architecture Discussions Quiz (6 questions).

*Section 12 – Amazon S3 Introduction (completed):*
- S3 Overview and Hands On.
- S3 Security: Bucket Policy and Hands On.
- S3 Website Overview and Hands On.
- S3 Versioning, Replication, Replication Notes and Hands On.
- S3 Storage Classes Overview and Hands On, S3 Express One Zone.
- Completed the Amazon S3 Quiz (7 questions).

*Section 13 – Advanced Amazon S3 (completed):*
- S3 Lifecycle Rules (with S3 Analytics) and Hands On.
- S3 Requester Pays, Event Notifications and Hands On.
- S3 Performance, Batch Operations, Storage Lens.
- Completed the Amazon S3 Advanced Quiz (9 questions).

*Section 14 – Amazon S3 Security (completed):*
- S3 Encryption and Hands On, About DSSE-KMS (article), S3 Default Encryption.
- S3 CORS and Hands On, S3 MFA Delete and Hands On.
- S3 Access Logs and Hands On, S3 Pre-signed URLs and Hands On.
- Glacier Vault Lock & S3 Object Lock, S3 Access Points, S3 Object Lambda.
- Completed the Amazon S3 Security Quiz (12 questions).

*Section 15 – CloudFront & AWS Global Accelerator (completed):*
- CloudFront Overview, CloudFront with S3 Hands On.
- CloudFront ALB/EC2 as an Origin, Geo Restriction, Cache Invalidation.
- AWS Global Accelerator Overview and Hands On.
- Completed the CloudFront & AWS Global Accelerator Quiz (5 questions).

**What This Means**
This was by far the highest-volume day so far, closing out the database fundamentals section and pushing all the way through DNS, classic architecture patterns, and a very deep dive into Amazon S3 across three dedicated sections plus CDN/global routing. S3 alone spans four sections in this course, which reflects how heavily SAA-C03 weights storage architecture, security, and performance decisions.

> **What I understood**
> - RDS, Aurora, and ElastiCache each solve different reliability and performance needs, and the exam tests exactly when to choose which.
> - Route 53 routing policies (weighted, latency, failover, geolocation, geoproximity) are architectural decisions, not just DNS trivia.
> - S3 is disproportionately weighted in this course across four sections, matching how central Storage is to the SAA-C03 exam.
> - CloudFront and Global Accelerator solve different problems — caching/CDN vs. global network-layer routing — and shouldn't be confused.

**Result**

Six sections completed in one day (RDS/Aurora/ElastiCache, Route 53, Classic Solutions Architecture Discussions, S3 Introduction, Advanced S3, S3 Security) plus CloudFront & Global Accelerator, with every section quiz completed along the way.

---

## August 6, 2026

**SAA-C03 Exam Prep | Day 7 · Finishing EC2 Storage and Advancing Through ELB & ASG**

Today I continued through the Maarek SAA-C03 course and made strong progress across EC2 storage, high availability, and scaling topics. I officially returned from vacation and am back in Cologne, which means starting tomorrow I can increase the study volume again and move through the course with more intensity.

**What I Covered**

*Section 7 – EC2 Instance Storage:*
- EBS Overview and EBS Hands On.
- EBS Snapshots and EBS Snapshots Hands On.
- AMI Overview and AMI Hands On.
- EC2 Instance Store.
- EBS Volume Types.
- EBS Multi-Attach.
- EBS Encryption.
- Amazon EFS and Amazon EFS Hands On.
- EFS vs EBS.
- EBS & EFS Section Cleanup.
- Reached the EC2 Data Management Quiz.

*Section 8 – High Availability and Scalability: ELB & ASG:*
- High Availability and Scalability.
- Elastic Load Balancing Overview.
- Application Load Balancer (ALB).
- Application Load Balancer Hands On Part 1 and Part 2.
- Network Load Balancer (NLB) and NLB Hands On.
- Gateway Load Balancer (GWLB).
- Sticky Sessions.
- Cross Zone Load Balancing.
- SSL Certificates and SSL Certificates Hands On.
- Connection Draining.
- Auto Scaling Groups Overview.
- Auto Scaling Groups Hands On.
- Auto Scaling Groups Scaling Policies.
- Auto Scaling Groups Scaling Policies Hands On.
- Reached the High Availability & Scalability Quiz.

**What This Means**

This was one of the most relevant study days so far because it directly targeted two of my biggest weakness clusters from the three baseline practice exams: Compute and Storage. EBS, EFS, AMIs, load balancers, and Auto Scaling are core SAA-C03 building blocks, so getting through these sections gives me more of the infrastructure-level thinking that the exam has clearly been demanding from the start.

> **What I understood**
> - EBS, EFS, and Instance Store each serve a fundamentally different access pattern and need to be chosen based on the specific use case, not interchangeably.
> - ALB, NLB, and GWLB are not interchangeable either — each targets a different layer and traffic type, and the exam tests precisely these distinctions.
> - Auto Scaling policies (scheduled, target tracking, step scaling) are one of my confirmed weak spots from the baseline, so working through them in the course is directly closing that gap.
> - Being back in Cologne means the next phase can have the consistency and volume SAA-C03 actually requires to get through the remaining sections cleanly.

**Result**

Section 7 effectively completed, Section 8 advanced through load balancers and Auto Scaling, and from tomorrow onward the study pace increases again now that the România vacation is officially over.

---

## August 5, 2026

**SAA-C03 Exam Prep | Day 6 · Completing EC2 Fundamentals & Advancing into EC2 Associate Level**

Today I continued through the Maarek SAA-C03 course, finishing Section 5 – EC2 Fundamentals entirely and progressing well into Section 6 – EC2 Solutions Architect Associate Level.

**What I Covered**

*Section 5 – EC2 Fundamentals (completed):*
- AWS Budget Setup, EC2 Basics, and creating an EC2 instance with EC2 User Data to serve a website (hands-on).
- EC2 Instance Types Basics.
- Security Groups & Classic Ports Overview, plus a hands-on lab.
- SSH Overview, and SSH connection walkthroughs for Linux/Mac, Windows, and Windows 10.
- SSH Troubleshooting (article).
- EC2 Instance Connect and EC2 Instance Roles Demo.
- EC2 Instance Purchasing Options (On-Demand, Reserved, Spot, etc.).

*Section 6 – EC2 - Solutions Architect Associate Level (in progress):*
- Private vs Public vs Elastic IP, including the hands-on lab.
- EC2 Placement Groups, plus hands-on.
- Elastic Network Interfaces (ENI) — overview, hands-on, and extra reading.
- EC2 Hibernate, plus hands-on.
- Reached the EC2 SAA Level Quiz (5 questions) at the end of the section.

**What This Means**

This was a dense, practical day covering the full EC2 lifecycle: from basic instance creation and SSH access through to associate-level architecture concepts like IP addressing strategy, placement groups for performance/compliance, and ENIs for advanced networking setups. Since Compute has consistently been one of my weakest domains across all three baseline practice exams, this section is directly closing one of my highest-priority gaps.

> **What I understood**
> - EC2 fundamentals (instance types, security groups, SSH access, purchasing options) form the base layer that all associate-level compute concepts build on.
> - Private vs. Public vs. Elastic IP choices are a recurring architectural decision, not just a networking detail.
> - Placement Groups and ENIs address specific performance, resilience, and compliance requirements rather than being general-purpose defaults.
> - Compute is one of my confirmed baseline weaknesses, so this section is directly targeted rework, not just course progression.

**Result**

Section 5 (EC2 Fundamentals) completed in full; Section 6 (EC2 - Solutions Architect Associate Level) advanced through lesson 55, with the EC2 SAA Level Quiz.

---

## August 4, 2026

**SAA-C03 Exam Prep | Day 5 · Course Start — IAM & AWS CLI Section**

Today I watched through an entire section of Stephane Maarek's course from the beach at Olimp, Romania — a fitting contrast between coastal downtime and dense IAM content.

**What I Covered**

I completed Section 4 – IAM & AWS CLI in full, from the introduction through the hands-on labs and the section quiz. The lessons covered:

- IAM fundamentals: Users, Groups, Policies, and how Root accounts should never be used or shared.
- Groups only contain Users, not other Groups, and Users can belong to multiple Groups at once.
- Hands-on labs for IAM Users & Groups, IAM Policies, and IAM MFA setup.
- AWS Access Keys, CLI, and SDK setup across Windows, Mac OS X, and Linux.
- AWS CloudShell as a browser-based CLI alternative, plus its Region availability constraints.
- IAM Roles for AWS services and their hands-on configuration.
- IAM Security Tools and IAM Best Practices, closing out with an IAM & AWS CLI quiz (9 questions).

**What This Means**

This section builds the identity and access foundation that underpins almost every other AWS service tested on SAA-C03, so getting IAM Users, Groups, Policies, and Roles solid early is a good structural start before moving into networking and compute, which remain my weakest baseline areas.

> **What I understood**
> - IAM is the identity layer that every other AWS service and exam domain builds on top of.
> - Groups only ever contain Users, never other Groups — a small but exam-relevant detail.
> - CLI/SDK setup and CloudShell give practical alternatives to console-based work, each with their own constraints.
> - Starting the course with IAM was the right structural choice before tackling my weaker Networking and Compute domains.

**Result**

Section 4 (IAM & AWS CLI) of the Maarek SAA-C03 course completed, including hands-on labs and the section quiz.

---

## August 3, 2026

**SAA-C03 Exam Prep | Day 4 · Course Decision After 3-Exam Baseline**

Today was mostly travel, driving from Balea Lac down to Mamaia, Constanța on the Black Sea coast, but I used the downtime to make a key decision about my SAA-C03 preparation path.

**Decision**

After mapping three full baseline practice exams (41%, 46%, 33% correct), I've decided to commit to Stephane Maarek's "Ultimate AWS Certified Solutions Architect Associate" course as my structured path forward for SAA-C03. It's the most established SAA-C03 prep course on the market, with roughly 27 hours of video, hands-on labs, over 800 slides, and a full practice exam, and it's built specifically around the architectural depth this certification demands.

**Why This Makes Sense**

My hope was that DEA-C01 and CLF-C02 knowledge would carry over significantly, but the three-exam baseline made it clear that SAA-C03 is a fundamentally different exam with a different approach. It requires deep AWS cloud architecture thinking rather than data-platform-centric reasoning, and the weakest domains from my baseline (Networking, Compute, Database) are exactly the architecture-heavy areas this course is structured to build up systematically. The course covers VPC networking, compute scaling design, database architecture trade-offs, security, and the AWS Well-Architected Framework in the depth needed to actually close these gaps.

**What This Means**

> - Self-study through practice exams alone wasn't sufficient to close the architectural depth gap.
> - The Maarek course is widely regarded as the de facto standard for SAA-C03 preparation, with a 4.7 rating across hundreds of thousands of reviews.
> - My plan: work through the course systematically, prioritizing Networking, Compute, and Database sections first based on my baseline weaknesses.

**Result**

Committed to the Stephane Maarek SAA-C03 course as the structured study path, informed directly by the combined weakness mapping from Practice Exams 1–3.

---

## August 2, 2026

**SAA-C03 Exam Prep | Day 3 · Practice Exam 3 and Full 3-Exam Priority Mapping**

Today I finished the third and final SAA-C03 mapping practice exam, this time from the top of the Transfagarasan near Balea Lac in Romania, and the view genuinely stole focus from the questions. The result was **33% correct (22/65)**, the lowest of the three baseline exams, with Networking & Content Delivery scoring only 22% correct across 18 questions, making it the single largest and weakest domain overall.

**Result Overview — All Three Exams (% Correct)**

| Topic Area | Exam 1 | Exam 2 | Exam 3 |
|---|---|---|---|
| Networking & Content Delivery | 55% | 58% | 22% |
| Compute | 42% | 36% | 36% |
| Database | 33% | 58% | 25% |
| Security, Identity & Compliance | 40% | 22% | 50% |
| Storage | 44% | 44% | 41% |
| Application Integration | 29% | 0% | 67% |
| Migration & Transfer | 0% | 100% | 100% |
| Management & Governance | 75% | 0% | 100% |

**Combined Priority List (All 3 Exams)**

1. Networking & Content Delivery — largest and now weakest domain overall.
2. Compute — consistently around 36% correct across all three exams.
3. Database — unstable, weakest in the most recent exam.
4. Security, Identity & Compliance — fluctuating, never reliably passed.
5. Storage — consistently medium-weak.
6. Application Integration — highly volatile between exams.
7. Migration & Transfer, Analytics, Management & Governance — lowest priority, either consistently strong or too few questions to be statistically meaningful.

**What I Understood**

My hope was that DEA-C01 and CLF-C02 knowledge would carry over significantly into SAA-C03, but across all three baseline exams it's clear that SAA-C03 is a fundamentally different exam with a completely different approach. It tests deep AWS cloud architecture thinking, infrastructure trade-offs, and design reasoning rather than the data-platform-centric knowledge that DEA-C01 rewarded, and this gap shows up hardest in Networking, Compute, and Database, which are architecture-heavy domains by nature. Based on this full three-exam overview, I've concluded that self-study alone isn't going to close this gap efficiently, and I need a structured SAA-C03 course to properly rebuild the architectural mental models this exam actually demands.

> **What I understood**
> - SAA-C03 rewards architecture and trade-off reasoning, not data-platform knowledge like DEA-C01 did.
> - Networking, Compute, and Database are consistently the weakest domains across all three baseline exams.
> - Fluctuating results in Security and Application Integration point to unstable, shallow understanding rather than real mastery.
> - Self-study alone is not sufficient here; a structured course is needed to rebuild the correct mental models.

**Result**

Three-exam baseline mapping complete: 41%, 46%, 33% correct respectively, with Networking, Compute, and Database confirmed as the top rework priorities before moving into structured course-based study.

---

## August 1, 2026

**SAA-C03 Exam Prep | Day 2 · Practice Exam 2/3 Result and Combined Weakness Picture**

Today I completed the second SAA-C03 practice exam and scored **46% correct (30/65)**, a slight improvement over Exam 1, but Security, Compute, and Storage remain clearly the biggest problem areas.

**Result Overview — Practice Exam 2/3**

| Topic Area | Questions | Correct | Wrong |
|---|---|---|---|
| Migration & Transfer | 1 | 100% | 0% |
| Application Integration | 1 | 0% | 100% |
| Analytics | 1 | 100% | 0% |
| Networking & Content Delivery | 12 | 58% | 42% |
| Database | 12 | 58% | 42% |
| Storage | 18 | 44% | 56% |
| Compute | 11 | 36% | 64% |
| Security, Identity & Compliance | 9 | 22% | 78% |

Total: 46% correct, 1h 51m test time, August 1, 2026.

**What Stands Out**

- Security is clearly the weakest area in this exam with a 78% error rate, with repeated mistakes on permissions boundaries, Security Lake, PrivateLink, presigned URLs, and encryption strategies.
- Storage remains the largest topic block with 18 questions and still shows gaps around FSx variants (Lustre vs. Windows File Server), lifecycle policies, and EBS/EFS/S3 decisions.
- Compute also has a high error rate at 64%, mainly around Auto Scaling strategies, ECS/EKS decisions, and scaling based on SQS queue length.
- Database and Networking improved moderately to 58% correct each, but are not yet reliable.
- Migration & Transfer, Application Integration, and Analytics each had only one question, so these results are not statistically meaningful and should not be overweighted.

**Rework Priority for Exam 2/3**

1. Security, Identity & Compliance (78% wrong, biggest gap): IAM permissions boundaries, Security Lake, PrivateLink, Secrets Manager vs. Parameter Store, S3 encryption options.
2. Compute (64% wrong, 11 questions): Auto Scaling (scheduled vs. target tracking), ECS/EKS networking and IAM design, SQS-based scaling.
3. Storage (56% wrong, largest topic block with 18 questions): FSx variants, lifecycle transitions, EBS vs. EFS vs. S3 by use case.
4. Networking & Content Delivery (42% wrong): Transit Gateway, Direct Connect resilience, VPC endpoints.
5. Database (42% wrong): Aurora endpoints (custom/reader/writer), RDS Proxy, backup strategies.

**Combined Picture from Exam 1 + Exam 2**

Migration & Transfer looks strong in Exam 2, but that is only because it had a single question, so it should not be treated as a confirmed strength. Security is now a consistently weak area across both exams, and the same is true for Compute and Storage — these three are the real core topics to prioritize before Practice Exam 3.

> **What I understood**
> - A single improvement in overall score does not mean the weak areas have actually closed; Security got worse relative to Exam 1.
> - Topics with very few questions in an exam can create misleading signals of strength or weakness.
> - Security, Compute, and Storage are now confirmed as the persistent weak areas across two exams, not one-off mistakes.
> - The rework plan should now focus on these three areas specifically before attempting Exam 3.

---

## July 31, 2026

**SAA-C03 Exam Prep | Day 1 · Practice Exam 1 Baseline and Priority Mapping**

Today I finished the first SAA-C03 practice exam and used it as a pure knowledge map rather than as a final judgment. The result was **41% correct (27/65)**, which shows that I already have some transfer from DEA-C01, but SAA-C03 clearly tests a different set of architectural decisions and trade-offs.

**Result Overview**

| Topic Area | Questions | Correct | Wrong |
|---|---|---|---|
| Migration & Transfer | 4 | 0% | 100% |
| Management & Governance | 4 | 75% | 25% |
| Networking & Content Delivery | 11 | 55% | 45% |
| Analytics | 4 | 50% | 50% |
| Compute | 12 | 42% | 58% |
| Storage | 9 | 44% | 56% |
| Security, Identity & Compliance | 5 | 40% | 60% |
| Database | 9 | 33% | 67% |
| Application Integration | 7 | 29% | 71% |

Total: 41% correct, 1h 24m test time.

**What This Means**

- Migration & Transfer did not come out as a strength in this exam, so it should not be treated as a confirmed strong area from this result.
- The biggest gaps are Application Integration, Database, and Security, which are exactly the areas where SAA-C03 is about different use-case decisions, not just broader versions of the same topics.
- Compute also needs priority because it had the largest question weight in the exam.
- This first exam is useful as a baseline for targeted study, not as a final performance verdict.

**Rework Priority**

1. Application Integration: SQS, SNS, EventBridge, Kinesis decisions, pub/sub patterns, queue prioritization.
2. Database: Aurora Global Database, Read Replicas, RDS encryption migration, multi-region database strategies.
3. Security, Identity & Compliance: IAM Roles Anywhere, SCPs, Tag Policies, cross-account access, PrivateLink.
4. Storage: EFS vs. FSx vs. Storage Gateway, EBS types, IOPS sizing.
5. Compute: EKS, ECS, Fargate decisions, Auto Scaling, EC2 vs. container vs. serverless trade-offs.

Networking & Content Delivery is already above 50% correct, so it only needs targeted cleanup. Analytics and Management & Governance are moderate or low-priority refresh areas. Migration & Transfer should be revisited only as needed, but it is not the area to prioritize from this exam.

**Result**

The first practice exam worked exactly as intended: it mapped out where the real SAA-C03 gaps are and where DEA-C01 knowledge does not transfer as cleanly. The next two practice exams will provide more evidence, and only after all three will I draw the final conclusion and start the targeted rework phase.

> **What I understood**
> - SAA-C03 is a different exam with different architectural choices compared to DEA-C01.
> - A practice exam is most valuable when used as a diagnostic tool, not as a final scorecard.
> - Application Integration, Database, Security, and Compute are the highest-value areas to revisit first.
> - My migration background is not the main story here; architectural reasoning still needs sharpening.

---

## July 30, 2026

**Month 4 · Officially Finished + Month 4 project uploaded**

Today I learned that finishing a project is often less about writing the last line of code and more about closing all the loose ends around it. The big takeaway from today was that my Month 4 project is truly finished now: the Kafka + Airflow event-stream pipeline works, the validation and metric layers behave as expected, and the project can be treated as the official Month 4 proof point in my roadmap.

I also learned how important it is to clean up the repository before publishing. I spent time adjusting the GitHub structure, removing unnecessary empty folders, and making the project look like a deliberate final version rather than a raw work-in-progress. That matters because a good GitHub repo is not just about working code — it is also about readability, structure, and how clearly the project communicates what it is.

A big technical lesson today was related to Docker and Airflow state. I hit an authentication problem during `airflow-init` because the Postgres volume still contained old state, and the fix was to reset the stack with `docker compose down -v` so the metadata database could be rebuilt cleanly. After that, the initialization succeeded, which reminded me again that containerized data projects often fail because of persistent state rather than because the code itself is wrong.

Another useful lesson was that `airflow tasks test` is extremely valuable as a quick verification step. Once the environment was clean again, I used it to confirm that the `create_tables` task worked before trusting the full pipeline run. That gave me confidence that the DAG import, SQL loading, and database connectivity were all healthy before I moved on to the rest of the pipeline.

From a project perspective, today was a strong closing day. I manually prepared the project for GitHub, restructured the repository layout, validated the pipeline end to end, and treated the project as the finished Month 4 milestone. That also means the transition into Month 5 is now clean: Month 4 is done, the public project exists, and the next focus can move fully toward Feast and the feature store direction.

> **What I understood**
> - Finishing a data infrastructure project means closing the environment, repository, and validation gaps too, not just the code gaps.
> - Persistent Docker state can cause misleading Airflow errors even when the code is fine.
> - `airflow tasks test` is a fast, practical confidence check before trusting the full DAG run.
> - Repo cleanup is part of shipping a project because presentation communicates engineering maturity.



---

## July 29, 2026

**Event Stream Pipeline · Full Reset, Docker Compose Debugging & End-to-End Recovery**

Today I spent more than 11 hours working on my event-stream pipeline project and started the entire setup from scratch in a practical way. The main focus was debugging Docker Compose, environment variables, a wrong Postgres password lingering in the shell environment, and a persisted Postgres volume that had credential issues.

**What I Debugged**

- Built the complete local stack from scratch! 
- Tracked down a stale `POSTGRES_PASSWORD=airflow_secure_pwd` shell variable that was overriding the `.env` value
- Learned that `docker compose down` does not remove named volumes, while `docker compose down -v` removes volumes and therefore also old data and Kafka topics
- Used `down -v --remove-orphans` to fully clean the environment before bringing the stack back up again

**What I Learned**

- Shell variables can override values from `.env`, so a local shell can silently keep the wrong credentials active
- Persisted Docker volumes can preserve old database state long after the config file has changed
- A clean reset is sometimes faster and safer than trying to salvage a broken dev environment
- Infrastructure debugging is part of the project itself, not just an annoyance around the project

**What Worked**

- Postgres was freshly initialized after the cleanup
- The Airflow database migration completed successfully
- The `event_stream_pipeline` DAG ran without errors
- I was able to produce an event into Kafka and verify that the DAG consumed it correctly
- The raw record appeared in `raw_events` in Postgres, confirming the full end-to-end flow

**Result**

Today was a hard debugging day, but also a very productive learning day. The project is now built on a clean foundation, and I understand the interaction between Docker Compose, volumes, environment variables, Airflow, Kafka, and Postgres much more deeply.

> **What I understood**
> - `docker compose down` and `docker compose down -v` have very different effects, especially when persistent state is involved
> - Shell environment variables can override `.env` values and cause confusing credential bugs
> - Seeing the full event flow work again proved that the pipeline is now based on a much more solid foundation

---

## July 28, 2026

**Apache Airflow · Assets, Executors, Task Groups, XCom & Branching**

Second Airflow day with a focus on advanced concepts: data-driven scheduling via assets, the three executor architectures, task organization, and data exchange between tasks.

**Assets as Data-Driven Scheduling**

- Understood assets via the `@asset` decorator: an asset (e.g. `user`) encapsulates an external data source with a URI and schedule, and automatically pushes its result as an XCom
- Learned asset chaining — a second asset (`user_location`) can set `schedule=user` and gets triggered automatically once the source asset is materialized, including `xcom_pull` with `include_prior_dates=True`
- Learned `@asset.multi` with multiple outlets to produce several downstream assets (`user_location`, `user_login`) from one asset at once, plus the `airflow assets materialize` CLI command for manual triggering

**The Three Executor Types**

| Executor | How It Works | Use Case |
|---|---|---|
| SequentialExecutor | Runs exactly one task at a time, uses SQLite | Debugging, default on first install |
| LocalExecutor | Runs multiple tasks in parallel on one machine | Small to medium setups, easy to configure |
| CeleryExecutor | Distributes tasks via a broker (Redis) and result backend to multiple worker nodes | Production scaling across multiple machines |

- Understood the SQLite limitation: only one writer at a time, which is why it is not compatible with the Local or Celery executor, confirmed by quiz
- Practically set up Celery queues with dedicated worker pools (`high_cpu`, `ml_model`, `default`) to route tasks by resource need, and learned monitoring via Flower
- Internalized key concurrency parameters: `parallelism` (max tasks per scheduler, default 32), `max_active_tasks_per_dag` (default 16), and `max_active_runs_per_dag` (default 16)

**Task Groups for DAG Organization**

- Used task groups with `@task_group` to visually bundle related tasks and apply `default_args` (e.g. `retries`) to a group instead of the whole DAG
- Built nested task groups and understood how values are passed between groups and individual tasks via parameters

**XCom and Branching**

- Fully worked through XCom mechanics: `xcom_push`/`xcom_pull` for explicit data exchange, plus implicit passing of return values via the TaskFlow API (`val = t1(); t2(val)`)
- Implemented branching with `@task.branch`: depending on the return value, only one path executes while the other is automatically marked "skipped," clearly visible in the UI
- Learned `@task.sql` as a specialized SQL task decorator that dynamically generates SQL strings from Python code and executes them via `SQLExecuteQueryOperator`

**Custom Provider SDK**

As the most advanced point of the day, I built a small custom SDK (`my-sdk`) with its own `@task.sql` decorator and integrated it into an Airflow instance via `pyproject.toml` and a custom Dockerfile — an important step toward extensibility and platform engineering.

> **What I understood**
> - Assets shift scheduling from purely time-based to data-driven, which is a much more natural model for real pipelines
> - Choosing the right executor is an infrastructure decision, not a code decision, and directly determines how far a setup can scale
> - Task groups and nested groups are what keep large DAGs readable and maintainable as complexity grows
> - Building a custom provider SDK is the first real platform-engineering move, since it turns Airflow into something extensible for a team rather than just a personal tool

---

## July 27, 2026

**Apache Airflow · Fundamentals, Architecture & First Practical DAG**

Today was my first real contact with Apache Airflow: why it exists, all core components, architecture variants, and a complete practical example (the user_processing DAG) with an API sensor, PostgreSQL, and CSV export.

**Why Airflow and What It Delivers**

- Internalized the four core benefits: organization (controlling task order and timing), visibility (a dashboard for all workflows), flexibility/scalability (from simple queries to ML training), and extensibility via provider packages.
- Clearly separated what Airflow is NOT: not a data-processing framework, not a real-time streaming solution, not a data storage system — so it doesn't fit high-frequency sub-minute scheduling or direct bulk data processing.
- Understood Airflow explicitly as an orchestrator rather than a processing engine, which cleanly separates it from Kafka and Spark from the last few weeks.

**Core Components in Detail**

- Worked through seven core components: Metadata Database, Scheduler, DAG File Processor, Executor, API Server, Worker, Queue, and Triggerer (manages deferrable tasks waiting on external events).
- Compared single-node vs. multi-node architecture: single-node is simple but not scalable, while multi-node distributes scheduler, worker, and API server across multiple machines for scalability and fault tolerance, at the cost of setup complexity.

**Core Concepts: DAG, Operator, Task, Workflow**

- Understood a DAG as an acyclic task structure — cycles are explicitly not allowed, confirmed via quiz.
- Grasped the operator as the blueprint for a single task, with task/task instance as the concrete execution of an operator at a specific point in time.
- Learned the important standard operators: PythonOperator, BashOperator, SQLExecuteQueryOperator, FileSensor, plus S3KeySensor and HttpSensor from the Astronomer Registry.
- Correctly understood task dependencies both via `set_upstream`/`set_downstream` and via bitshift operators (`>>`, `<<`), confirmed by quiz.
- Internalized the sensor concept: a long-running task that waits for an external event at fixed intervals via a poke function, e.g. `@task.sensor` with `PokeReturnValue`.

**Practical Example: user_processing DAG**

- Built a complete pipeline: `create_table` (SQLExecuteQueryOperator against PostgreSQL) → `is_api_available` (sensor on an external API) → `extract_user` (PythonOperator/TaskFlow) → `process_user` (CSV creation) → `store_user` (PostgresHook with `copy_expert` for bulk insert).
- Learned the TaskFlow API with `@dag` and `@task` decorators as an alternative to classic operator instances, including implicit data passing between tasks without manual XCom handling.
- Correctly set up Airflow connections (e.g. `postgres`) and provider packages (`apache-airflow-providers-postgres`), and tested individual tasks in isolation via `airflow tasks test`.
- First contact with assets/datasets: understood how one DAG (e.g. `daily_report`) produces an asset via outlets and a second DAG (`monthly_report`) listens to it and gets triggered automatically.

**Result**

A very dense first day: solid architecture understanding plus a fully functional end-to-end DAG example with a real database connection.

> **What I understood**
> - Airflow is an orchestrator, not a processing engine, which makes its role next to Spark and Kafka very clear.
> - The core components (scheduler, executor, worker, triggerer) each solve a distinct part of reliably running scheduled workflows.
> - The TaskFlow API removes a lot of boilerplate compared to classic operator instances, especially around passing data between tasks.
> - Sensors and deferrable tasks are the mechanism for waiting on external events without wasting resources.

---

## July 26, 2026

**Kafka · Real-World Case Studies, Enterprise Admin & Advanced Topic Configurations** 

Today I worked through the final three sections of the Kafka course: real-world case studies for system-design thinking, enterprise admin fundamentals, and deep internal topic behavior. This officially completes the full Kafka beginners course, from producer basics all the way to enterprise administration. 

**Real-World Case Studies**

- Went through four architecture patterns with Kafka as the backbone: GetTaxi (IoT/position tracking using Kafka Streams for surge pricing), MySocialMedia (CQRS pattern with separated command/query paths for posts, likes, and comments), MyBank (CDC via Debezium combined with Kafka Streams for real-time alerts), and logging/metrics aggregation plus big-data ingestion as a "speed layer" in front of batch systems. 
- Core design heuristic: topic key choice (e.g. user_id, post_id) and partition count need to be planned correctly early, since both are very hard to change safely later. 
- Understood why state changes in event streams should be modeled as events (e.g. "User X liked Post Y at time Z") instead of as raw state. 

**Kafka in the Enterprise for Admins**

- Learned the most important monitoring metrics: under-replicated partitions, request handler utilization, and request timing, exposed via JMX and typically visualized in Prometheus, Datadog, or ELK. 
- Internalized the security pillars: encryption (SSL between client and broker), authentication (SSL certificates, SASL/PLAINTEXT, SASL/SCRAM, Kerberos, OAuth), and authorization via ACLs. 
- Understood advertised listeners in detail: why misconfigured private/public IPs or localhost settings can prevent clients outside the network from reaching the broker. 
- Compared multi-cluster replication strategies: active/passive (simple, but no clean failover) vs. active/active (better latency and redundancy, but conflict risk with asynchronous writes). 

**Advanced Topic Configurations**

- Understood segments and their two indexes (offset-to-position, timestamp-to-offset), along with the `log.segment.bytes` and `log.segment.ms` levers. 
- Clearly distinguished the two cleanup policies: `delete` (default, time/size based) vs. `compact` (keeps only the latest value per key, used for example in `__consumer_offsets`). 
- Traced log compaction practically through an employee-salary topic: only the last salary value per employee key survives compaction, order is preserved, and deleted records remain visible for `delete.retention.ms`. 
- Understood unclean leader election as a tradeoff between availability and data loss when all in-sync replicas are offline. 
- Internalized two strategies for large messages (>1MB): external storage (S3/HDFS) with a reference in Kafka vs. directly raising `message.max.bytes`, `replica.fetch.max.bytes`, `max.partition.fetch.bytes`, and `max.request.size`. 

**Result**

This completes the entire Kafka course, from producer basics to enterprise administration — a very solid foundation for the Month 4 streaming-pipeline project in the roadmap. 

> **What I understood**
> - Topic key and partition design decisions are effectively permanent, so system design thinking has to happen before the first record is written. 
> - Enterprise Kafka is as much about monitoring, security, and multi-cluster strategy as it is about producers and consumers. 
> - Log compaction and segment internals explain a lot of Kafka's "magic" behavior, like how `__consumer_offsets` stays efficient at scale. 
> - Finishing the full course from basics to admin-level topics closes the loop needed before starting the actual streaming pipeline project. 

---

## July 25, 2026

**Kafka · OpenSearch Consumer, Delivery Semantics & Extended APIs**

Today I completed OpenSearch Consumer and Advanced Consumer Configurations and Kafka Extended APIs. It was an especially dense day with a very direct connection to the streaming-pipeline direction of the roadmap.

**OpenSearch Consumer Implementation**

- OpenSearch cluster set up with Docker Compose (`opensearch` and `opensearch-dashboards`), including access to the Dev Tools console
- `OpenSearchConsumer` project built with the Kafka client and the OpenSearch REST High Level Client, including handling for both secured and unsecured REST clients
- Wikimedia index created programmatically in OpenSearch and individual Kafka records indexed via `IndexRequest`

**Idempotence and Delivery Semantics**

- Compared two idempotence strategies for the consumer: IDs based on Kafka coordinates (`topic_partition_offset`) versus IDs extracted from the JSON payload itself; the payload-based approach prevents duplicates even across consumer restarts over multiple runs
- Fully internalized the three delivery semantics: at-most-once (commit offset before processing, data loss possible), at-least-once (commit offset after processing, duplicates possible, so idempotence is needed), exactly-once (only with the Transactional API or an idempotent consumer)
- Reinforced that at-least-once plus idempotent processing is the standard recommendation for most real-world applications

**Offset Commit Strategies and Batching**

- Understood auto-commit behavior: with `enable.auto.commit=true`, offsets are committed every `auto.commit.interval.ms` on the next `.poll()`, which can lead to data loss if processing has not finished before the next poll
- Implemented manual offset committing with `consumer.commitSync()` after a successful bulk insert, including batching multiple records into one `BulkRequest` instead of sending each record individually
- Understood `auto.offset.reset` behavior for `latest`, `earliest`, and `none`

**Kafka Extended APIs**

- Understood Kafka Connect as the high-level API for source and sink integrations, solving External Source → Kafka and Kafka → External Sink without custom code, with built-in fault tolerance, idempotence, and scaling
- Connected a Kafka Connect Wikimedia source and ElasticSearch sink in practice and configured them via Connect Standalone properties
- Learned Kafka Streams as a Java library for Kafka → Kafka transformations and built three own stream processors: bot-vs-human counter, website counter with one-minute time windowing, and an event-count timeseries with 10-second windows
- Understood the core Kafka Streams model: one-record-at-a-time processing, exactly-once capable, no separate cluster required, runs as a normal Java application

**Schema Registry**

- Understood the core problem without Schema Registry: Kafka only handles bytes without validation, so producer schema changes can silently break consumers
- Internalized the role of Schema Registry: producers register schemas, Kafka validates via Avro, consumers fetch the schema for deserialization; supports backward, forward, and full compatibility
- Saw schema evolution demonstrated live in Conduktor: from one field (`f1`) to two fields (`f1`, `f2`), including how incompatible messages are rejected by the schema rules

**Result**

This completes both the practical producer/consumer reliability side and the high-level ecosystem overview across Connect, Streams, and Schema Registry. That is exactly the foundation this part of Month 4 is supposed to build for the streaming-pipeline project.

> **What I understood**
> - Reliability in Kafka is not only about the producer; the consumer side matters just as much through offset strategy, batching, and idempotent writes
> - At-least-once plus idempotent processing is the practical default because it balances correctness and operational simplicity
> - Kafka Connect, Streams, and Schema Registry form the real ecosystem layer that turns Kafka from a log into a platform
> - Today was the first full view of how a production-style Kafka pipeline is assembled end to end

---

## July 24, 2026

**Kafka · Real-World Wikimedia Producer & Advanced Producer Configurations**

Today I worked through sections 10 and 11 of the Kafka course: from the real-world project concept through a complete Wikimedia producer to the critical producer reliability settings. This was exactly the part I expected to be most relevant to my streaming pipeline goal.

**Real-World Project**

- Target architecture understood: Wikimedia Recent-Change stream → Kafka producer → Kafka topic → consumer → OpenSearch
- Docker Compose setup with ZooKeeper and a Kafka broker configured (KRaft-adjacent Confluent images, ports, listener config)
- Java project structure with Gradle set up for the Wikimedia producer, including OkHttp3 and okhttp-eventsource as dependencies for the SSE stream

**Wikimedia Producer Implementation**

- Implemented `WikimediaChangeHandler` as an `EventHandler` reacting to the SSE stream and sending every message asynchronously via `kafkaProducer.send()` to the `wikimedia.recentchange` topic
- Understood that the event stream runs on its own thread (`eventSource.start()`) while the producer keeps running in the background and the main thread blocks
- Ran the producer live against real Wikimedia traffic and verified messages arriving in the topic via the Conduktor Console UI

**Producer Acknowledgements Deep Dive**

- Fully internalized the three acks levels: `acks=0` (no waiting, highest throughput, data loss possible), `acks=1` (only the leader confirms, limited loss possible), `acks=all` (leader plus ISR confirm, no loss)
- Understood `min.insync.replicas` as the lever between availability and durability: with `acks=all`, RF=3, and `min.insync.replicas=2`, exactly one broker can fail without write operations failing
- Took away the rule of thumb: `acks=all` combined with `min.insync.replicas=2` is the most common compromise between resilience and availability

**Producer Retries and Idempotence**

- Understood why retries without an idempotent producer can cause reordering under parallel in-flight processing, especially relevant for key-based ordering
- Learned `delivery.timeout.ms` as the overarching timeout budget that wraps `linger.ms`, `retries`, and `request.timeout.ms`
- Internalized the idempotent producer as the Kafka 3.0 standard: prevents duplicates from network errors via server-side duplicate detection, automatically combining `acks=all`, high retries, and controlled in-flight requests
- Worked fully through the safe-producer summary: `acks=all`, `min.insync.replicas=2`, `enable.idempotence=true`, `retries=MAX_INT`, `delivery.timeout.ms=120000`, `max.in.flight.requests.per.connection=5`

**Performance Tuning**

- Understood message compression: snappy/lz4 as a good tradeoff between CPU cost and compression ratio, more effective with larger batches
- Internalized `linger.ms` and `batch.size` as the levers for throughput vs. latency, applied practically in the high-throughput producer demo with snappy, a 20ms linger, and a 32KB batch size
- Compared default partitioner behavior: round-robin (Kafka ≤2.3) vs. sticky partitioner (Kafka ≥2.4), and why sticky partitioning lowers p99 latency through larger batches
- Understood `max.block.ms` and `buffer.memory` as the backpressure mechanism when the producer produces faster than the broker can absorb

**Result**

Completed the "Quiz on Producer Configurations" with 6 out of 6 correct answers.

> **What I understood**
> - Reliability in Kafka is a tunable spectrum, not a fixed guarantee, and acks plus min.insync.replicas define exactly where a system sits on that spectrum
> - Idempotent producers solve a real correctness problem, not just a convenience issue, once retries and parallel in-flight requests are in play
> - Throughput and latency tuning (batching, linger, compression, sticky partitioning) are concrete, measurable levers, not vague performance advice
> - The Wikimedia producer made all of these settings feel like production decisions instead of abstract configuration flags

---

## July 23, 2026

**Kafka · Java Producer & Consumer API**

Today I worked through the Java programming part of the Kafka course and moved from project setup to the actual producer and consumer APIs. The focus was never really Java syntax itself, but the transferable Kafka logic behind the examples, which made the material feel directly relevant to the streaming layer I want to build later.

**Producer side**

- Kafka project set up with Gradle and the first Java producer implemented
- Producer callbacks learned as asynchronous confirmations that reveal which partition and offset a message was actually written to
- Producer with keys implemented, reinforcing that a key is consistently hashed to the same partition and preserves ordering for related events

**Consumer side**

- Java consumer implemented and the pull model for reading from partitions understood
- Graceful shutdown covered so the consumer can stop cleanly without losing processing state
- Consumer behavior inside a consumer group explored, including exclusive partition assignment per group member
- Incremental cooperative rebalancing and static group membership reviewed as modern strategies that cause less interruption than classic rebalancing
- Auto offset commit behavior understood, including when offsets are committed automatically and why that matters for delivery semantics

**Result**

The chapter "Kafka Java Programming 101" is now fully complete, including the final quiz with 6 out of 6 correct answers. The next section in the course is already the "Kafka Real World Project", which moves from isolated examples into a connected, realistic use case.

> **What I understood**
> - Kafka producers are not just about sending records; callbacks and keys control observability, partitioning, and ordering.
> - Consumer groups are the core scaling mechanism, and rebalancing behavior directly affects stability.
> - Auto commit is convenient, but it also changes the risk profile for delivery semantics.
> - The Java examples matter because they expose the Kafka mechanics cleanly, even if the long-term goal is broader than Java itself.

---

## July 22, 2026

**Kafka · Fundamentals & CLI**

Today I started my Kafka block and finally built a clean mental model for why Kafka exists at all: it replaces brittle N×M point-to-point integrations with a decoupled log-based system where sources and targets can evolve independently. The architecture started to click through brokers, topics, partitions, replication, and the CLI tools that make the whole system practical.

The most important idea for me was that topics are like tables without constraints, while partitions are the real unit of ordering and scale. Ordering is only guaranteed within a partition, keys are hashed to a partition so that related records stay together, and replication gives the cluster resilience through leaders and in-sync replicas. That also made delivery semantics and acks feel much more concrete, because the tradeoff between speed and durability is now easy to read in the configuration.

I also worked through consumer groups, offsets, and the internal `__consumer_offsets` topic, which made pull-based consumption and group balancing much clearer. On top of that, I learned the practical Kafka CLI flow for topics, producers, consumers, and consumer-group management, plus the important shift away from ZooKeeper toward KRaft and the standard `--bootstrap-server` approach.

This was exactly the right start for the next phase of the roadmap, because Kafka is not just theory here — it is the streaming ingestion layer I want to connect to the existing PySpark ETL project. The Java + Gradle setup will be the bridge into real producer and consumer work, and today gave me the architecture vocabulary to move into that work with more confidence.

> **What I understood**
> - Kafka exists to decouple systems cleanly instead of forcing direct point-to-point integrations.
> - Ordering, replay, and fault tolerance all depend on partitions, replication, and the chosen ack strategy.
> - Consumer groups, offsets, and CLI tools are not side topics; they are the operational core of Kafka.
> - KRaft is the modern direction, so ZooKeeper should no longer be treated as the default mental model.

---

## July 21, 2026

**Month 3 · Officially Completed**

Today I officially closed Month 3 of the roadmap, and the milestone feels real because it now has a visible public artifact behind it. The PySpark ETL project was finished from scratch in the planned style, intentionally small in scope but technically meaningful, and published on GitHub as the Month 3 deliverable.

That choice mattered. Instead of trying to make the project bigger than necessary, I kept it focused on proving that Spark fundamentals are not just understood in theory, but can be applied in a real repository with a clean pipeline, clear output, and proper presentation. I also cleaned up my GitHub profile and repositories so the portfolio feels consistent and complete rather than scattered.

What this month taught me is that small projects can still be strong evidence when the scope is intentional and the execution is clean. Spark becomes much clearer once it is turned into an actual pipeline, and roadmap progress is not only about learning new topics, but also about finishing and publishing them properly. Month 3 is now officially complete, and the next phase will move into Kafka, Airflow, and system design foundations.

> **What I understood**
> - A compact project can still be strong proof when it is well scoped and well presented.
> - Spark understanding becomes more concrete once it is used in a real ETL pipeline.
> - Portfolio hygiene, documentation, and cleanup are part of the proof, not optional extras.
> - Progress is measured not only by learning, but by shipping and publishing.

---

## July 20, 2026

**Scala + Spark Scala Basics — Day 5: Spark ML, Streaming, and GraphX with Pregel**

Today spanned an impressive arc from Spark ML through streaming (DStream and Structured Streaming)
all the way to GraphX with Pregel, and the course material itself already flags which parts are
outdated. It was a dense but very rewarding day that closed several loops at once.

**Spark ML**

- The ML library covers feature extraction (TF/IDF), basic statistics, linear/logistic regression, SVM, Naive Bayes, decision trees, K-Means, PCA/SVD, and recommendations via Alternating Least Squares
- Explicit note in the course itself: MLlib (the RDD-based library) is deprecated in Spark 3, and the new ML library works exclusively with DataFrames
- `ALS().setMaxIter(5).setRegParam(0.01).setUserCol().setItemCol().setRatingCol()` as a compact API, trained with `.fit(ratings)`
- Critical self-reflection built right into the course material: recommendation quality is called questionable ("putting your faith in a black box is dodgy"), noting that the previously built movie-similarity solution often performs better

**Linear Regression and Decision Trees with Spark ML**

- `VectorAssembler` combines multiple feature columns into a single vector, followed by a train/test split via `randomSplit()`
- `LinearRegression().setRegParam().setElasticNetParam().setMaxIter().setTol()` uses stochastic gradient descent, but is sensitive to feature scaling, assuming normal distribution and defaulting the y-intercept to 0
- Practice exercise: real-estate valuation on the Taiwan dataset using `DecisionTreeRegressor` instead of linear regression, since decision trees handle differing scales more robustly

**Classic Spark Streaming (DStream)**

- Concept: continuous data streams are aggregated in intervals, with sources like Kinesis, HDFS, Kafka, and Flume; checkpointing saves state to disk for fault tolerance
- Windowed operations (`window()`, `reduceByWindow()`, `reduceByKeyAndWindow()`) and `updateStateByKey()` for ongoing state management across batches
- Practice example: Twitter hashtag counting using `flatMap()` to split text, `filter()` for hashtags, `map()` to key/value pairs, and `reduceByKeyAndWindow()` over a 300-second window

**Structured Streaming as the Successor**

- The course material itself clarifies that Spark 2 introduced Structured Streaming, which uses DataSets as the primary API and is no longer based on micro-batches
- Practice example: `spark.readStream.text()` reads Apache access logs, `regexp_extract()` parses host, timestamp, method, endpoint, status, and content size, followed by `groupBy("status").count()` with `.writeStream.outputMode("complete")`
- Windowed aggregation with event time: `groupBy(window(col("eventTime"), "30 seconds", "10 seconds"), col("endpoint"))` to find top URLs in a sliding time window

**GraphX and Pregel**

- GraphX works with `VertexRDD` and `EdgeRDD`, useful for connectivity, degree distribution, path lengths, triangle counting, and PageRank, but without native support for something like "Degrees of Separation"
- Reimplemented the BFS example once more, this time via the Pregel API: vertices iteratively send messages to neighbors in "supersteps" until convergence
- Core logic: `(id, attr, msg) => math.min(attr, msg)` as the vertex program preserving the shortest known distance, plus a reduce function that also keeps the minimum when multiple messages arrive

**Important note on current status**

Both DStream and GraphX are now officially deprecated: DStream since Spark 3.4.0 and GraphX since Spark 4.0.x, both being replaced by DataFrame-native successors, Structured Streaming and GraphFrames. Notably, the course material already flags this exact caveat for MLlib ("MLlib is deprecated in Spark 3") but not for DStream and GraphX, simply because the course predates their official deprecation.

**Why this was still worthwhile**

Today's Pregel-based BFS implementation was the third time solving the same algorithm, now in a
third philosophy (message-passing instead of RDD-reduce or DataFrame-join), which rounds out a solid
understanding of the tradeoffs between RDD, DataFrame, and graph-native APIs. The most important
transfer for the roadmap is that Spark ML is built entirely on DataFrames, and Structured Streaming
follows the same architectural line, exactly the direction production feature pipelines are built
in today.

>**What I understood**
>- MLlib and DStream/GraphX represent an older RDD-centric era of Spark that is actively being phased out
>- DataFrame-native APIs (ML, Structured Streaming, GraphFrames) are the clear direction for production systems
>- Seeing BFS solved three times across three paradigms made the tradeoffs between RDD, DataFrame, and graph APIs concrete rather than abstract

---

## July 19, 2026

**Scala + Spark Scala Basics — Day 4: Collaborative Filtering, Caching, SBT Packaging, and AWS EMR**

Today's focus was item-based collaborative filtering, caching strategy, packaging with SBT, and
running a real production-style cluster job on AWS EMR. This was the first day that closed the full
loop from local development to actual cluster deployment.

**Item-Based Collaborative Filtering (Movie Similarities)**

- Strategy: find every movie pair rated by the same user, measure the similarity of their ratings across all shared users, group by movie, and sort by similarity strength
- Technical core is a self-join on the same ratings DataSet: `ratings.as("ratings1").join(ratings.as("ratings2"), $"ratings1.userId" === $"ratings2.userId" && $"ratings1.movieId" < $"ratings2.movieId")`, where the `<` condition avoids duplicate pairs
- Cosine similarity implemented manually: computing `xx`, `yy`, `xy` columns, then per movie pair `sum(xy)` as the numerator and `sqrt(sum(xx)) * sqrt(sum(yy))` as the denominator, the classic vector similarity formula translated directly into Spark SQL
- Result filtering by a score threshold (0.97) and a minimum co-occurrence count (`numPairs`) to exclude weak or noisy similarities

**Caching Concept**

- Clear rule: as soon as more than one action runs on the same DataSet, it should be cached with `.cache()` or `.persist()`, otherwise Spark re-evaluates the full lineage every time
- Difference: `.persist()` additionally allows caching to disk instead of only memory, acting as a fallback if a node fails

**Packaging with SBT**

- `build.sbt` with `name`, `version`, `organization`, `scalaVersion`, and `libraryDependencies`, marking Spark-Core and Spark-SQL as `"provided"` so they are not bundled into the JAR
- `sbt-assembly` plugin wired in via `assembly.sbt` to build a fat JAR with all dependencies included
- The command `sbt assembly` produces a self-contained JAR in the `target/scala-x.xx` folder, runnable directly with `spark-submit <jar>` without specifying a class
- Important practical note: never use local filesystem paths in the script; use HDFS or S3 instead, since the cluster cannot access the local machine

**spark-submit and Cluster Parameters**

- Syntax: `spark-submit --class <MainClass> --jars <deps> --files <files> <jar>`
- Key parameters: `--master` (yarn, hostname:port, mesos), `--num-executors` (must be set explicitly on YARN, since the default is only 2), `--executor-memory`, `--total-executor-cores`
- Architecture concept: the Spark Driver communicates with the Cluster Manager, which coordinates multiple cluster workers and executors

**Amazon EMR (Elastic MapReduce)**

- EMR as a fast way to spin up a cluster with Spark, Hadoop, and YARN pre-installed, billed by instance hours plus network and storage I/O
- Practical setup workflow: AWS account, EC2 key pair, SSH access (PuTTY on Windows including .pem to .ppk conversion), IAM roles for service and instance profiles, and configuring S3 bucket access
- Concrete cluster workflow: place scripts and data on S3, launch the cluster via the AWS Console, SSH into the master node, download the JAR from S3 with `aws s3 cp s3://bucket/file .`, then run `spark-submit`
- Important warning: always terminate the cluster after use, otherwise costs keep accruing; roughly $30 was spent for a few hours of test runtime
- Practical proof: a successful run of the "Star Wars" similarity example on the real 1M dataset, producing scores like 0.99 for "The Empire Strikes Back"

**Partitioning as a Performance Factor**

- Central insight: Spark does not automatically distribute expensive operations like self-joins optimally; explicit `.repartition()` (DataFrame) or `.partitionBy()` (RDD) is required
- Operations that preserve partitioning in the result: `join()`, `cogroup()`, `groupWith()`, `leftOuterJoin()`, `rightOuterJoin()`, `groupByKey()`, `reduceByKey()`, `combineByKey()`, `lookup()`
- Practical rule of thumb: at least as many partitions as cores/executors, but not too many to avoid shuffle overhead; `repartition(100)` as a solid starting point for large operations

**Cluster Debugging and Dependency Management**

- Spark Web UI (port 4040) shows jobs, stages, shuffle read/write, and DAG visualization, though it is hard to reach externally on EMR
- Recommendation: avoid hardcoding Spark configuration including the master in the driver script, so EMR defaults and spark-submit parameters can take effect
- Executor memory can be adjusted on error directly via `--executor-memory` in the spark-submit call
- Broadcast variables serve as a way to share data outside RDDs/DataSets between driver and executors, important since executors don't run on the same machine as the driver
- Missing dependencies can either be bundled into the JAR via `sbt assembly` or passed at submit time via `--jars`

**Why this mattered for the roadmap**

Today was the first time the complete chain from local development to fat-JAR packaging to cluster
deployment on AWS EMR was run end to end. This is exactly the skillset that matters for an L5
Data/Feature Infrastructure role, since real deployment and scaling questions like partitioning,
executor memory, cluster cost, and dependency management showed up, not just algorithm logic. The
partitioning concept is especially relevant since it is the kind of performance tuning regularly
needed in production feature pipelines.

>**What I understood**
>- Caching is not optional once a DataSet is reused across multiple actions; it directly affects cost and speed
>- Fat-JAR packaging with SBT is the standard bridge between local Scala code and cluster execution
>- EMR makes cluster setup fast, but cost discipline (terminating clusters) is a real operational responsibility
>- Partitioning is a manual performance decision, not something Spark solves automatically for expensive joins

---

## July 18, 2026

**Scala + Spark Scala Basics — Day 3: RDD vs. DataSet Tradeoffs and BFS with Accumulators**

Today's focus was implementing the same problems in parallel RDD and DataSet form, culminating in a
BFS-based "Degrees of Separation" algorithm using accumulators. This was the clearest day yet for
understanding when each API actually fits better.

**Spark SQL with DataSets (case class + SparkSession)**

- `SparkSession.builder.appName(...).master("local[*]").getOrCreate()` as the new entry point instead of `SparkContext`
- `case class Person(id, name, age, friends)` for schema definition, combined with `.as[Person]` on CSV read
- `createOrReplaceTempView("people")` plus `spark.sql("SELECT ...")` for direct SQL queries on DataSets

**DataFrame Operations vs. RDD Style**

- `.select()`, `.filter()`, `.groupBy().count()`, and column arithmetic directly on the DataFrame, e.g. `people("age") + 10`
- FriendsByAge reimplemented using `.groupBy("age").avg("friends")` instead of manual `reduceByKey` logic
- Refined with `agg(round(avg(...), 2).alias("friends_avg"))` for clean output formatting

**Key Insight: DataSets Are Not Always the Better Choice**

- DataSets work best with structured data; on unstructured text like a book, the DataSet version gets stuck with generic Row objects and a single "value" column, while RDDs stay more direct
- WordCount solved additionally with SQL functions like `explode()`, `split()`, `lower()`, and the special comparison operators `=!=` and `===`
- RDDs and DataSets can be mixed, for example loading data as an RDD and converting it into a DataSet with `.toDS()`

**Schema Definition with StructType**

- Explicit schema via `new StructType().add("col", Type, nullable)` instead of inference, important for weather data (MinTemperatures) and customer orders
- `withColumn()` to create new or transformed columns, such as a Fahrenheit conversion directly in the DataFrame

**Broadcast Variables and UDFs**

- Motivation: a large lookup table (movie ID to name) should be transferred once per executor, not repeatedly per task
- `sc.broadcast(loadMovieNames())` to distribute the table, `.value` to access it inside the executor
- A UDF built with `udf(lookupName)` to display movie names via the broadcast map in a new column

**MostPopularSuperhero: RDD vs. DataSet Comparison**

- RDD version: `flatMap(parseNames)` returning `Option` to discard malformed rows, `reduceByKey`, `map` to swap key/value, and `.max()`
- DataSet version: `split()`, `size()`, `withColumn()`, `groupBy().agg(sum(...))`, and `.sort($"connections".desc).first()`, noticeably more compact than the RDD solution
- Bonus task "MostObscureSuperheroes": `.agg(min("connections")).first().getLong(0)` and `.join(names, usingColumn="id")` to resolve names of the least-connected heroes

**BFS and Degrees of Separation — the Central Tradeoff Showcase**

- Classic breadth-first search with color coding (WHITE for unvisited, GRAY for marked to explore, BLACK for fully processed) and distance values
- Data model: `BFSNode = (heroID, (connections, distance, color))`, initialized with distance 9999 and color WHITE, except for the start node
- RDD implementation: iterative `flatMap(bfsMap)` to expand gray nodes plus `reduceByKey(bfsReduce)` to merge duplicates per hero ID, keeping the shortest distance and darkest color
- Accumulator concept: `sc.longAccumulator("Hit Counter")` as a global, driver-readable counter that executors can increment in parallel, crucial for detecting when the target node is found
- Important pitfall: the accumulator only updates when an action (`mapped.count()`) actually forces evaluation of the RDD; lazy transformations alone are not enough
- DataSet implementation of the same BFS: using `explode()`, `join()` with `left_outer`, and `when().otherwise()` constructs instead of manual reduce logic to update color and distance each iteration

>**What I understood**
>- DataSets shine on structured data, but RDDs are still more direct for unstructured or highly custom logic
>- Mixing RDDs and DataSets in the same pipeline is a normal and practical pattern, not a compromise
>- Accumulators only reflect real state once an action triggers evaluation, which is a subtle but critical Spark behavior
>- The BFS exercise made the RDD-vs-DataSet tradeoff concrete instead of theoretical

---

## July 17, 2026

**Scala + Spark Scala Basics — Day 2: RDDs, Key/Value Transformations, DataFrames and DataSets**

Today went deep into RDDs, key/value transformations, and the conceptual transition toward
DataFrames and DataSets in Spark-Scala. It was a dense but coherent day that moved from raw
functional transformations to real architectural understanding of where Spark is heading.

**Functional Programming and RDD Basics**

- `sc.parallelize(...)` to create an RDD from a list
- `.map(x => x*x)` as a basic example of functional transformation
- A named function like `def squareIt` can be passed to `.map()` just like a lambda
- RDD actions: `collect`, `count`, `countByValue`, `take`, `top`, `reduce`
- RDD transformations: `map`, `flatMap`, `filter`, `distinct`, `sample`, `union`, `intersection`, `subtract`, `cartesian`

**RDD Creation and Data Sources**

- Creating RDDs from lists, text files (`sc.textFile`), S3, and HDFS
- `HiveContext` and running SQL queries directly on RDDs via `hiveCtx.sql(...)`
- Additional sources: JDBC, Cassandra, HBase, Elasticsearch, JSON, CSV, Sequence Files, Object Files, and compressed formats
- Lazy evaluation: nothing runs until an action is called

**Practice Example: RatingsCounter**

- Full Scala code using `SparkContext`, `textFile`, `map`, `countByValue`, `sortBy`, and `foreach(println)`
- Concept of stages and tasks: Spark builds an execution plan from RDD operations and splits it into stages and tasks for cluster distribution

**Key/Value RDDs**

- RDDs can hold key/value pairs, such as (age, number of friends)
- SQL-style joins: `join`, `rightOuterJoin`, `leftOuterJoin`, `cogroup`, `subtractByKey`
- Specialized key/value operations: `reduceByKey`, `groupByKey`, `sortByKey`, `keys()`, `values()`
- Creating key/value RDDs by mapping to tuples, e.g. `rdd.map(x => (x, 1))`
- `mapValues()` and `flatMapValues()` for more efficient transformations when only values change

**Practice Example: FriendsByAge**

- `parseLine` function to parse CSV rows into `(age, numFriends)` tuples
- `mapValues(x => (x, 1))` and `reduceByKey` to compute sum and count per key
- Average calculation with `mapValues(x => x._1 / x._2)`

**Practice Example: MinTemperatures / MaxTemperatures**

- Parsing weather data into `(stationID, entryType, temperature)`
- `filter()` to isolate specific entry types, such as only "TMIN"
- Temperature conversion handled directly in the parsing step
- `reduceByKey` with `min()` / `max()` to aggregate per station

**flatMap() vs. map()**

- `map()` transforms each element one to one
- `flatMap()` can produce multiple new elements from a single input, such as splitting sentences into words

**Practice Example: WordCount / WordCountBetter / WordCountBetterSorted**

- Reading text, splitting into words with the regex `\W+`, and normalizing everything to lowercase
- Word counting with `map(x => (x,1)).reduceByKey(_+_)`
- Sorting results by frequency by swapping key and value and using `sortByKey()`

**Practice Example: TotalSpentByCustomer**

- `extractCustomerPricePairs` function to parse customer ID and amount
- `reduceByKey` to sum spending per customer
- Results sorted and printed by amount

**Transition to DataFrames and DataSets**

- DataFrames extend RDDs and add automatic optimization
- DataFrames contain Row objects, support SQL queries, carry a schema, read/write JSON, Hive, and Parquet, and connect to JDBC, ODBC, and Tableau
- DataSets are typed (`DataSet[Person]`, `DataSet[(String, Double)]`), with the schema known at compile time rather than only at runtime like DataFrames
- DataSets can only be used in compiled languages like Java and Scala, not in Python
- RDDs can be converted to DataSets with `.toDS()`
- The Spark trend is moving away from RDDs toward DataSets, since they serialize more efficiently, allow better compile-time execution plans, and simplify development
- MLLib and Spark Streaming are moving toward DataSets rather than RDDs as their primary API
- `SparkSession` replaces `SparkContext` for Spark SQL and DataSets, including operations like `.select()`, `.filter()`, `.groupBy().mean()`, and `.rdd().map(...)`
- Spark SQL exposes a JDBC/ODBC server via `start-thriftserver.sh`, accessible through `beeline`
- User-Defined Functions (UDFs) via `org.apache.spark.sql.functions.udf`

**Why today's observation mattered**

The key insight today was recognizing exactly what makes Scala indispensable for Spark: DataSets are
type-safe and checked at compile time, which is simply not possible in Python because Python is not
compiled. This is not a nice-to-have detail, it is the core reason production Spark codebases at
companies like Netflix lean so heavily on Scala, which is precisely the target environment for this
roadmap.

>**What I understood**
>- This was a dense but highly coherent day, moving from pure RDD transformations through key/value
>pair operations to a conceptual grasp of DataFrames and DataSets

---

## July 16, 2026

**Scala + Spark Scala Basics — Day 1**

Today I started my Scala and Spark-Scala learning block, and I went much deeper than syntax alone. The first crash course sections made it clear why Scala matters so much inside the Spark ecosystem, and the transition into Spark already felt like the right next step for my roadmap.

What stood out most today was how Scala feels familiar coming from Python, but also much more explicit and strongly typed. That difference makes the language feel stricter at first, but it also explains why Scala + Spark is such a powerful combination in real-world codebases.

I covered immutable values with `val`, basic string concatenation, primitive data types, string interpolation with `s` and `f`, regex extraction, boolean logic, control flow, typed functions, higher-order functions, tuples, lists, maps, and safe fallbacks with `Try(...).getOrElse(...)`. I also reached the Spark introduction, Spark basics, and the beginning of the RDD section, which made the whole path feel more concrete and engineering-focused.

For my roadmap, this is exactly the right direction. I do not need to become a full-time Scala engineer; I need to become comfortable enough to read and adapt Spark-Scala code, and today felt like a strong first step toward that goal.

> **What I understood**
> - Scala is not about replacing Python for me; it is about unlocking Spark code comprehension and adaptation.
> - Strong typing and explicitness make the language feel more rigid, but also more predictable.
> - The real objective is practical Spark-Scala literacy, not language mastery for its own sake.

---

## July 15, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 15: Passed**

Today I officially passed the AWS Certified Data Engineer – Associate exam, finishing in under an hour and coming out with the clear feeling that the preparation had paid off. This result validated the entire training arc from late March to mid-July, including the CLF-C02 pass shortly before DEA-C01.

The path to this point was not lightweight. It cost stress, energy, repetition, and focus, and it also happened alongside private challenges, a hernia, moving back to Germany, and the experience of living and following work in Seattle. But that difficulty is exactly what made the knowledge durable, practical, and real instead of shallow memorization.

Starting from no IT experience at the end of March 2026, my understanding of AWS data engineering best practices has changed completely. The goal is no longer just passing an exam; it is building pipelines correctly, choosing the right storage solution, and applying security and governance patterns with confidence. This certification now stands as a concrete proof point of that growth and a real anchor for the long-term L5 goal.

Tomorrow starts the next phase: more power, more depth, and a stronger focus on reading and adapting Spark-Scala code without needing to write everything from scratch. The momentum is here, and now it is about turning the certification into practical engineering strength.

> **What I understood**
> - Real competence comes from repeated retrieval, correction, and pressure-tested study, not from cramming.
> - Stress can be converted into durable skill when the work is treated seriously and consistently.
> - The certification is not the finish line; it is evidence that the engineering path is now becoming tangible and earned.

---

## July 14, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 14: Final Trap Review**

Today I ran one final 20-question pre-exam quiz focused on the highest-value DEA-C01 traps plus the
newer topics that were added more recently. The result was **20/20**, which made it seven perfect quiz
results in a row across domain-specific quizzes, a mixed-domain quiz, an active recall quiz, and the
final trap review.

**What I reinforced**

- Kinesis Data Streams is the right answer when a scenario requires low-latency processing with custom consumers, while Firehose remains a buffered delivery service
- Rising IteratorAge points to consumer lag, so the fix is consumer-side scaling or parallelism rather than changing producers
- S3 Intelligent-Tiering is the right choice for unknown access patterns, while explicit lifecycle transitions are better when the pattern is known
- One Zone-IA remains a trap whenever the prompt also requires multi-AZ resilience or high availability
- Deep Archive is the right answer when retrieval can take many hours and minimum storage cost is the priority
- Macie discovers and classifies sensitive data, Lake Formation enforces governed access, and S3 Object Lambda masks data dynamically on read
- Glue Data Catalog stores lake table metadata, while Glue Schema Registry handles streaming schemas and compatibility
- Redshift Spectrum is for querying S3 data, while Redshift Federated Query is for operational sources like RDS or Aurora
- Secrets Manager is the answer when automatic secret rotation is required, while Parameter Store fits general configuration values
- CloudTrail captures API calls, AWS Config tracks configuration history, and CloudWatch Logs handles application logs
- MWAA is the correct migration path when existing Airflow DAGs must be preserved with minimal code changes

**Newer topics that locked in**

- AWS Transfer Family is the managed answer for SFTP, FTPS, and FTP ingestion into AWS storage
- AWS CDK is the high-level programmable IaC layer that synthesizes CloudFormation, while AWS SAM is the specialized framework for serverless stacks
- Amazon Bedrock with Knowledge Bases is the native fit for managed LLM integration, summarization, and retrieval-augmented generation over enterprise data
- HNSW is a strong vector index choice for high-recall semantic search, and pgvector in Aurora PostgreSQL enables vector storage and approximate nearest neighbor search
- SageMaker Unified Studio, domains, and projects form the governed collaboration model for modern ML workspaces
- SageMaker Catalog combined with IAM and Lake Formation enables project-scoped access to governed datasets and ML assets

**Main takeaway**

The knowledge is no longer just familiar, it is stable under recall pressure. The exam is tomorrow, and
the job now is not to learn more but to protect energy, sleep well, and show up calm.

---

## July 13, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 13: Active Recall Quiz and Confidence Check**

Today I ran another active recall quiz across the DEA-C01 blueprint and scored **20/20** again.
That confirmed the material is stable across single-domain, mixed-domain, and pure recall formats.

**Active Recall Quiz**

- Kinesis Data Streams for real-time personalization with multiple consumers and custom logic
- Consumer-side scaling and Enhanced Fan-Out to address rising IteratorAge when producers are within throughput limits
- S3 One Zone-IA as the class disqualified by high-availability requirements because it stores data in a single AZ
- S3 Intelligent-Tiering for unknown and variable access patterns
- Glacier Instant Retrieval for compliance archives that still need millisecond retrieval
- S3 Object Lambda to intercept and transform object data at read time without modifying the stored object
- Lake Formation to enforce fine-grained column and row-level permissions across Athena, Redshift Spectrum, and EMR
- Macie to automatically discover and classify sensitive data in S3 using machine learning
- EMR Master and Core On-Demand with Task Spot to protect HDFS stability while still saving cost
- Redshift Federated Query for querying RDS or Aurora data without copying it into the warehouse
- Redshift Spectrum for querying S3 data directly using the existing cluster as the query engine
- Redshift distribution keys for minimizing data movement during frequent joins on a specific key
- Athena for ad hoc pay-per-query analytics on S3
- Glue Data Catalog for centralizing technical metadata for tables used by Athena, Spectrum, and EMR
- Glue Schema Registry for centralizing streaming payload schemas and enforcing compatibility rules across producers and consumers
- Secrets Manager when automatic secret rotation is required
- AWS Config for tracking configuration state and compliance over time
- CloudTrail for capturing the full audit trail of API calls across an account
- Parameter Store for configuration and secrets management without automatic rotation
- MWAA for migrating existing Airflow DAGs with minimal code changes

**What I reinforced**

- Kinesis Data Streams versus Firehose depends on custom consumer needs and latency requirements
- IteratorAge problems are usually solved by scaling consumers or improving parallelism, not by changing producers first
- Storage class selection depends on availability, retrieval time, and access pattern, not just the storage family name
- Macie discovers, Lake Formation governs, and S3 Object Lambda masks or transforms at read time
- EMR Spot placement should protect HDFS by keeping master and core nodes On-Demand
- Redshift feature selection depends on source system and data location, especially when comparing federated access and Spectrum
- Glue Data Catalog and Glue Schema Registry solve different metadata layers
- Secrets Manager, Config, and CloudTrail each answer a different governance question

**Main takeaway**

Six consecutive perfect quiz results now confirm that the material is stable across single-domain, mixed-domain,
and pure recall formats. The exam date move to July 15 is protecting focus without costing any retention,
since today's recall accuracy matched every previous result exactly.

---

## July 12, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 12: Active Recall Session and Exam Date Decision**

Today was focused on active recall against the master battle plan, plus a strategic decision to move
the retake date. The session confirmed strong retention across all four domains, and the exam booking
was shifted to July 15 to protect performance quality.

**Active Recall Session**

- Read through the full master battle plan PDF twice as an encoding pass to refresh all four domains
- Ran a third pass using active recall by covering the answers and testing memory before checking each one
- Confirmed that this blurting-style method (cover the answer, recall, then verify) is one of the most effective techniques for long-term retention, stronger than passive rereading alone
- Noticed that most answers came back correctly on the first recall attempt, which signals strong retention across all domains rather than just recognition memory

**Exam Date Decision**

- A personal incident came up that affects readiness for the originally planned exam date
- Decided to move the exam booking to July 15 instead of pushing through under added stress
- This is a deliberate, strategic choice, not a setback, since the mixed-domain quiz already scored 20/20 and the extra days can be used for spaced recall instead of new material

**What I reinforced**

- Active recall combined with spaced repetition builds stronger memory pathways than rereading, even when a recall attempt is not fully successful
- Confidence under exam conditions comes from repeated retrieval practice, not from cramming more content
- Rescheduling an exam under personal stress is a valid strategy to protect performance quality

**Main takeaway**

The best use of the remaining days is short, spaced active recall sessions focused on weak points and
exam traps, not additional reading. Protecting mental capacity now matters more than adding new material.

>**What I understood**
>- Blurting-style active recall exposes true retention far better than a second or third rereading pass
>- Rescheduling is a controlled decision made from a position of strength, not weakness
>- The extra days should be spent on spaced recall and trap review, not new domain content

---

## July 11, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 11: Master Battle Plan Expansion**

Today I expanded the master battle plan again because a few important DEA-C01 topics were still
missing. The new version now covers all four domains with keyword-to-answer mappings and trap-focused
patterns for faster recall under exam pressure.

**What changed**

- A full English version was created with keyword-to-answer mappings for all four domains
- The document was restructured around exam traps instead of only topic summaries
- Cross-domain trap patterns were added so the same keyword logic can be reused across question styles
- The missing DEA-C01 topics were filled in to make the battle plan more complete for revision

**Why this helped**

The new structure is more useful than a simple topic list because it trains the mind to map a keyword
to the exact AWS responsibility layer. That matters when a service is relevant but not actually the
right answer category.

Examples of the trap logic include:

- Discovery vs. governance: Macie finds sensitive data, Lake Formation controls access
- Ingestion vs. delivery: Kinesis Data Streams supports multiple consumers, Firehose delivers data downstream
- Storage class vs. lifecycle intent: Glacier retrieval time and access pattern decide the choice
- Service capability vs. exam qualifier: the last sentence often decides the correct answer

**Main takeaway**

The biggest lesson today was that DEA-C01 rewards precise reading of the qualifier and the service
boundary. A service can be relevant without being the actual answer category, so the question must be
mapped to the exact responsibility layer before selecting the option.

>**What I understood**
>- The battle plan now works as a fast recall sheet, not just a study summary
>- Trap-based structure is more valuable than raw topic coverage for exam performance
>- Service boundary thinking is the real filter that prevents wrong but plausible answers

---

## July 10, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 10: Mixed-Domain Trap Drill**

Today was a mixed-domain quiz across all DEA-C01 areas, focused on qualifiers, service boundaries,
and exam traps. The result was **20/20**, which confirmed that the concepts hold up across domains
and not just inside one topic.

**Key answers from the drill**

- Real-time or near-real-time processing with multiple consumers: Kinesis Data Streams or MSK, depending on replay and Kafka compatibility
- Automatic discovery and classification of sensitive data in S3: Amazon Macie
- Long-term archival with up to 12 hours retrieval: Glacier Flexible Retrieval
- EMR Spot placement that keeps HDFS stable while saving cost: Master and Core On-Demand, Spot only on Task nodes
- Unpredictable access pattern: S3 Intelligent-Tiering
- Rising IteratorAge in Kinesis: add shards and increase parallelization, or use Enhanced Fan-Out
- Predictable BI dashboard workloads: Redshift provisioned or Redshift Serverless depending on elasticity needs
- Column-level and row-level access control in a data lake: Lake Formation
- Sporadic ad hoc analytics on S3: Athena
- High-cardinality joins and date filtering: Redshift distribution and sort keys aligned to access patterns
- Near-real-time delivery to S3: Kinesis Data Firehose
- Known access pattern over time: S3 Lifecycle rules
- Mask data on read without changing the source object: S3 Object Lambda
- Serverless SQL exploration over S3: Athena
- Macie-style discovery plus Lake Formation-style governed access: Macie + Lake Formation together
- Parquet plus partitioning to reduce scan cost: Athena or Spectrum
- Strict performance and concurrency control: provisioned services when serverless simplicity is not enough

**Main takeaway**

The biggest lesson was to read the last sentence first, identify the qualifier, and eliminate
answers that violate it. That approach turned the mixed drill into a clean **20/20** result
and confirmed that the concepts now hold up across the full exam blueprint.

>**What I understood**
>- Qualifier-first reading is now the main exam pattern, not a backup trick
>- The difference between discovery, masking, governance, and storage tier selection is now clearer across domains
>- The final mixed drill showed that the full blueprint is stable enough for the retake

---

## July 9, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 9: Domain 3 and 4 Baseline Assessment**

Today I ran a 15-question baseline quiz for Domain 3 and another 15-question baseline quiz
for Domain 4. Both came back **15/15**, which confirmed that the operational and security
patterns are still stable and that full-domain coverage is now in place across the exam.

**Domain 3 — Data Operations and Support**

The quiz reinforced these operational choices:

- AWS Step Functions for multi-service orchestration with retries, branching, and human-in-the-loop approval
- Amazon EventBridge + AWS Lambda for low-overhead scheduled checks and validation tasks
- Amazon MWAA for managed Apache Airflow DAGs when existing workflows already live in Airflow
- AWS DataBrew for code-free data preparation and rule-based quality checks
- Amazon Athena for rolling averages, log analysis, mismatch detection, and skew investigation using SQL over S3
- Amazon QuickSight SPICE for fast dashboarding over large datasets while keeping source data in S3
- Amazon Redshift Serverless vs. provisioned tradeoffs based on elasticity, cost predictability, and workload spikes
- CloudWatch Logs, Glue console history, and Athena-over-S3-log analysis for pipeline troubleshooting
- EMR log analysis plus S3 access logs for intermittent timeout debugging

**Domain 4 — Data Security and Governance**

The quiz reinforced these security and governance choices:

- Cross-account S3 access using IAM roles, STS temporary credentials, trust policies, and bucket policies in the target account
- ABAC using resource tags and condition keys for scalable attribute-based authorization
- AWS Secrets Manager for sensitive values with rotation; Systems Manager Parameter Store for non-sensitive configuration
- SSE-S3 for low-overhead encryption at rest, SSE-KMS with customer-managed keys for tighter control and auditability
- TLS/HTTPS to protect data in transit, including explicit enforcement in clients
- AWS Lake Formation for fine-grained governance across table-, column-, and row-level access with LF-Tags
- Amazon Macie for discovering PII and sensitive data in S3
- Amazon S3 Object Lambda for dynamic masking and transformation at read time
- AWS CloudTrail as the primary audit-log service, supported by AWS Config and CloudWatch alarms
- Residency concerns handled by keeping data and processing in the correct Regions, not by IAM alone

**Milestone | Full DEA-C01 Domain Coverage**

All four DEA-C01 domains now have baseline assessments completed:

- Domain 1: 15/15
- Domain 2: 15/15
- Domain 3: 15/15
- Domain 4: 15/15

This is the strongest signal so far that the core exam patterns are stable across ingestion,
storage, operations, and security.

>**What I understood**
>- The exam is no longer about discovering missing major themes, but about staying sharp on traps and wording
>- Full-domain baseline coverage is now complete, so the remaining work can shift to mixed practice and qualifier heuristics

---

## July 8, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 8: Domain 1 and 2 Baseline Assessment**

Today I ran a 15-question baseline quiz for Domain 1 and another 15-question baseline quiz
for Domain 2. Both came back **15/15**, which confirmed that the core patterns are still
solid and that the remaining prep can stay focused on the final blueprint coverage.

**Domain 1 — Data Ingestion & Transformation**

The quiz reinforced the main architectural choices:

- MSK for fan-out and replayability across independent consumer groups
- Kinesis Data Firehose for near-real-time delivery with minimal custom code
- Kinesis Data Streams retention + sequence numbers for ordered, replayable pipelines
- Enhanced Fan-Out for dedicated per-consumer throughput
- AppFlow for low-code SaaS ingestion without custom API clients
- Lambda reserved concurrency as the direct lever against throttling from Kinesis-driven invocations
- Step Functions for multi-service orchestration with human-in-the-loop approval via Wait states
- AWS CDK in TypeScript for type-safe, reusable IaC
- CodeCommit + CodeBuild + CodePipeline as the native AWS CI/CD pattern
- Parquet as the default transformation target for analytics cost and performance

**Deep dive: EMR Spot placement**

The safe exam default remains **On-Demand Master, On-Demand Core, Spot Task**. Core nodes
hold HDFS state, so Spot there is only safe when the scenario explicitly makes data-loss
risk acceptable or resilience is already engineered. The question wording matters a lot here.

**Domain 2 — Data Store Management**

The quiz reinforced these choices:

- RDS for OLTP workloads that need ACID guarantees
- Lake Formation for fine-grained governance in multi-source data lakes
- Redshift Spectrum for hybrid queries joining S3 external data with Redshift tables
- Redshift materialized views for repeated aggregations on refreshed data
- Partitioning to reduce scanned data and lower Athena / Spectrum cost
- DynamoDB TTL for automatic expiry of transient records
- S3 Versioning + Object Lock for compliant and recoverable protection
- SageMaker Feature Store for ML-specific lineage and cataloging
- Apache Iceberg for ACID transactions, row-level deletes, and schema evolution on S3

**What I understood**

> The foundational and classic decision patterns for DEA-C01 are still strong, and the
baseline results show that Domains 1 and 2 are ready for the next phase of mixed trap drills.

**Progress check**

- Domain 1 baseline: **15/15**
- Domain 2 baseline: **15/15**
- Next up: Domain 3 and Domain 4 baseline assessment

---

## July 7, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 7: LLM Integration and Data Governance**

Today focused on LLM integration in data pipelines and on governance frameworks for
controlled data sharing. The goal was to understand how AWS services keep LLMs inside
governed, auditable boundaries instead of letting them act as uncontrolled automation.
This was the last of the newly added exam-skill areas in the current gap list, so the
focus was on closing the final new concepts cleanly.

**What I learned**

- PII classification and governance metadata belong in catalog and tagging systems before data ever reaches an LLM
- AWS Lake Formation is the main control point for fine-grained access, so LLM-driven workflows must still respect user permissions
- The AWS Glue Data Catalog acts as the authoritative metadata source for data classification, ownership, and allowed usage
- LLMs should suggest policy templates, masking rules, or classifications, but deterministic AWS services and human stewards must enforce them
- Asynchronous patterns like SQS and EventBridge are safer than synchronous per-record LLM calls because they support retries, throttling, and centralized checks
- Row-level security, SCP guardrails, versioned source documents, and detailed redacted logs all help make AI-assisted data processing compliant and auditable

**Quiz result**

- LLM Integration and Governance Quiz: **15/15 — 100%**

**The core pattern**

The quiz reinforced one central idea: the **LLM advises, AWS enforces**. That means the
model can classify, summarize, or draft policies, but actual access control, masking, and
routing must be handled by governed AWS services. Whenever compliance, audit, or data
residency came up, the correct answer was always the deterministic AWS control, not the
LLM itself.

>**What I understood**
>- The final new exam-skill area is now covered and no longer needs separate discovery work
>- Governance metadata has to exist before AI touches the data, not after the fact
>- The answer pattern is consistent: LLM for suggestion, AWS for enforcement

**Progress check**

All newly added exam-skill gaps are now covered. The remaining work is repetition, trap
recognition, and mixed practice rather than new topic discovery.

---

## July 6, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 6: AWS Transfer Family and Infrastructure as Code**

Today focused on AWS Transfer Family and Infrastructure as Code with SAM and CDK. The goal
was to understand how to build secure file transfer workflows into S3 or EFS and how to
define them cleanly in code.

**What I learned**

- AWS Transfer Family provides managed SFTP, FTPS, and FTP endpoints for secure file transfer into AWS
- Transfer Family can use S3 or EFS as the backend storage layer
- For custom authentication, Transfer Family uses a Lambda-based identity provider rather than reading users directly from Secrets Manager
- IAM roles and prefix-level permissions are central to controlling what each Transfer Family user can access
- AWS SAM is useful for compact serverless templates, especially when wiring S3 events to Lambda
- AWS CDK is better when the infrastructure becomes more complex and you want reusable constructs in TypeScript or Python
- CDK also supports testing infrastructure logic with normal unit test frameworks

**Quiz result**

- AWS Transfer Family and IaC Quiz: **13/15 — 86%**

**What the misses taught me**

The first mistake was thinking Secrets Manager could act as the identity provider. It cannot;
the correct pattern is a Lambda custom identity provider. The second mistake was assuming
the identity provider Lambda is triggered by S3 events. In reality, Transfer Family calls
it directly during authentication. That clarified the key exam idea: identity for Transfer
Family is an auth-time callback, not an event-driven workflow.

**Key exam pattern**

- If the question asks about custom authentication for Transfer Family, think Lambda identity provider
- If the question asks about automation and repeatable infrastructure, think SAM or CDK depending on complexity
- If the question asks about auditing or logging, combine CloudTrail data events with Transfer Family logs in CloudWatch

>**What I understood**
>- Transfer Family auth is a direct authentication callback, not a trigger-based workflow
>- SAM is the concise option for simple serverless wiring; CDK is the better choice when the stack grows and needs reusable logic
>- The repeated trap is now identified clearly enough to avoid on the next quiz

**Next step**

Keep moving through the remaining gaps instead of revisiting already stable topics.

---

## July 5, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 5: SageMaker Unified Studio, Catalog, and ML Lineage**

Today I focused on the remaining DEA-C01 gap area around SageMaker Unified Studio,
SageMaker Catalog, domains, domain units, projects, and ML lineage tracking. I also
removed another false gap from the study plan: the miss was not a technical weakness,
but a wording trap around compliance and audit.

**What I learned**

- SageMaker Unified Studio is the browser-based interface that brings together authoring, experimentation, deployment, monitoring, governance, and the catalog in one place
- A SageMaker domain is the managed control plane for users, access, and shared resources, while domain units are logical sub-divisions inside a domain for team or environment isolation
- SageMaker Projects provide standardized templates for pipelines, source repos, IAM roles, and CI/CD-style workflows
- SageMaker Catalog is the curated business catalog for approved ML assets such as pipelines, models, datasets, and project templates
- ML Lineage Tracking records the chain from datasets and code to training jobs, models, and endpoints, which makes it useful for reproducibility, impact analysis, auditability, and compliance
- The key exam trap is that "compliance" and "audit" can point to ML lineage, not only to logging or encryption features

**Quiz result**

- SageMaker Unified Studio / Catalog / ML Lineage Quiz: **14/15 — 93%**

**What the miss taught me**

The wrong-answer trap was confusing SageMaker Training Compiler with SageMaker ML Lineage Graph.
Training Compiler is a performance tool, while ML lineage is a governance and traceability tool.
If a question mentions reproducibility, audit, compliance, or tracing dataset-to-endpoint
relationships, the answer is lineage.

>**What I understood**
>- The real topic is not just SageMaker features, but knowing whether the question is about speed or about traceability
>- Lineage is the correct answer whenever the exam asks about auditability, reproducibility, or model history
>- This was another false gap: the topic is mostly solid, and the miss was a wording issue rather than a knowledge hole

**Next steps**

Move to the next remaining gap block instead of re-reading already known material. Keep
using active recall and short quizzes to separate performance tools from governance tools.
Continue closing the remaining DEA-C01 gaps one topic at a time.

---

## July 4, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 4: Bedrock KBs, Vector Indexes, and Iceberg**

Today focused on the first major remaining gap area: Amazon Bedrock Knowledge Bases,
vectorization, HNSW vs IVF, and Apache Iceberg. The plan is now cleaner because I’m no
longer repeating topics that are already solid, and I’m keeping this day centered on the
remaining exam-only gaps.

**What I learned**

- Vectorization in Bedrock Knowledge Bases turns unstructured text into dense embeddings for semantic similarity search
- HNSW is a graph-based approximate nearest-neighbor index with strong recall and low latency on moderate-sized datasets
- IVF partitions vectors into clusters and trades recall vs latency through centroid probing
- Apache Iceberg adds schema evolution, snapshot tracking, and structured table semantics on top of lake data, which makes it useful as a source for knowledge-base ingestion
- Chunking and metadata matter: each chunk gets embedded separately, and stable row identifiers should be kept as metadata for traceability

**Practice result**

- Bedrock KBs and Vector Index Quiz: **15/15 — 100%**

**What changed**

The remaining scope gaps are now being treated as separate exam-only topics instead of
mixing them into broad review. That makes the next days much more efficient, because the
plan can now focus only on the actual missing pieces instead of redoing material that is
already solid.

>**What I understood**
>- Vectorization, HNSW, IVF, and Iceberg are now in a good enough state to move forward
>- The quiz confirmed that the topic is strong enough to leave the first gap area behind
>- Active recall and short quizzes are the right way to verify each topic before moving on

**Next steps**

Move to the next remaining gap day instead of redoing already-solved material. Keep using
active recall and short quizzes to verify each topic before moving on.

---

## July 3, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 3: Highest-Priority Gap Mapping**

Today was about translating the corrected gap analysis into a clear action plan. The highest-priority gaps are now identified, and tomorrow the work begins to actively close them.

**Highest-priority gaps**

- **Vector search / vector index / vectorization**: questions around vector indexes, Bedrock Knowledge Base, and vectorization concepts were not anchored strongly enough yet
- **B-tree / index creation**: the exam exposed a gap around choosing the right index type with the least operational overhead
- **Watermark / streaming state**: the wording around adjusting only the watermark after ingestion is now flagged as a special trap
- **Blank table copy patterns**: `WHERE 1=0` style zero-row table copies need to be fully automatic, not a hesitation point
- **Level aggregation**: rollup / cube-style aggregations need to be recognized instantly
- **Apache Iceberg**: open table format behavior must be more solid, especially around schema evolution and table management

>**What I understood**
>- The exam is now narrowed down to precision gaps, not broad knowledge gaps
>- The highest-priority list is small enough to attack directly and thoroughly
>- Tomorrow starts the actual repair phase, beginning with these exact items


**Plan for tomorrow**

Tomorrow’s prep starts with fixing the highest-priority gaps 

---

## July 2, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 2: Domain 1 + 2 Re-Review and Flight Prep**

Today was about narrowing the focus even further. The plan is to review Domain 1 and
Domain 2 again in StackLessions today, and if needed continue on the flight. I also went
through the topics that felt new to me from the exam questions I saw after the exam, so
that there are no knowledge gaps left before the next retake.

**What was reviewed**

- Domain 1 and Domain 2 re-review in StackLessions
- The exam topics that felt unfamiliar after the last attempt
- The goal was not broad repetition, but closing the last small gaps that still existed

**Why this matters**

The first two domains are still the decisive ones, and reviewing them again in a calmer,
more deliberate way helps turn earlier recognition into real recall. Going back through the
post-exam unfamiliar topics also removes the chance of leaving a hidden gap behind.

>**What I understood**
>- The next retake needs zero ambiguity in Domain 1 and Domain 2
>- Re-reviewing the same material with a fresh mindset is more valuable now than adding new volume
>- Anything that felt new on the actual exam has to be made familiar before July 13

**Travel note**

If the review is not finished today, the flight is the backup study block. That keeps the
prep moving even while traveling and makes sure the next retake is approached with a clean
knowledge base.

---

## July 1, 2026

**AWS DEA-C01 Retake Prep 3.0 — Day 1: Material Gap Analysis**

Day 1 of a new prep cycle, and it started with an honest audit instead of jumping straight
back into studying. Actively searched for updated study material today, targeting
specifically the topics that felt unfamiliar on the real exam.

**Material gap analysis**

- StackLessons course structure and pedagogy are genuinely strong and still recommendable —
  the core teaching approach holds up well
- However, the content was not fully up to date with the current DEA-C01 exam scope, which
  cost real points on the actual exam
- Key takeaway: supplement any single course with additional, more current sources to close
  the gaps that naturally exist when AWS updates the exam faster than course providers can

**Seattle trip wrap-up**

Return flight booked for tomorrow: Seattle → Frankfurt → home to Cologne. Seattle was a
genuine experience, but realistically not the right long-term fit as a German-Romanian
without a strong existing connection to the city. On top of everything else, a fire alarm
went off at 1 AM in the accommodation — loud, triggered across all floors, with an actual
fire one floor below. A lot to process today, stacked directly on top of processing the
failed exam result.

**Mindset moving forward**

Setbacks noted, but confidence remains that the retake will go better with sharper, updated
preparation. Trusting the roadmap to sort itself out over time — no need to force
conclusions today. Closing this Seattle chapter on good terms: valuable experience, clear
decision to move on.

>**What I understood**
>- The exam gap was never a knowledge problem — it is a material-currency problem, and
  Prep 3.0 needs to actively correct for that by mixing in updated sources
>- StackLessons remains a good foundation; it just cannot be the only source going forward
>- A rough night and a hard result on the same day do not need to be processed into a
  final verdict immediately — some things are allowed to just settle

---

## June 30, 2026

**AWS DEA-C01 — Exam Result — Attempt 3**

Third attempt. Score: **698/1000** — Fail. Passing threshold: **720**. Missed by just 22 points.

**Domain breakdown**

| Domain | Weight | Result |
|---|---:|---|
| D1 — Data Ingestion & Transformation | 34% | Needs Improvement |
| D2 — Data Store Management | 26% | Needs Improvement |
| D3 — Data Operations & Support | 22% | Meets Competencies |
| D4 — Data Security & Governance | 18% | Meets Competencies |

Domain 3 and 4 are solid. The gap is isolated to Domain 1 and Domain 2, which together
make up 60% of the exam — that is exactly where the next round of effort needs to go.

**Honest reflection: study material vs real exam gap**

A frustrating mismatch became clear across seven different study sources, all advertised
as DEA-C01 exam content. On exam day, roughly 20 questions took a completely different
direction than anything covered across all of those sources. This is not a lack-of-effort
problem — it is a real misalignment between marketed prep content and the actual current
exam scope.

>**What I understood**
>- 22 points away from passing with Domain 3 and 4 already at Meets Competencies means
  the path forward is narrow and specific, not a full restart
>- Seven sources missing the same ~20 questions' worth of content is a structural signal,
  not bad luck — the next prep cycle needs sources closer to the live exam pool, not more volume
>- Pausing Domain 3 and 4 entirely and going all-in on Domain 1 and 2 is the correct,
  disciplined use of the next two weeks

**Next steps**

- Retake booked for **July 13, 2026** — the earliest date allowed under the mandatory 14-day wait
- Until then: focused grinding limited strictly to Domain 1 (Ingestion/Transformation) and Domain 2 (Data Store Management)
- Domain 3 and 4 paused — no need to re-review what is already solid

```
Progress | AWS DEA-C01
- Attempt 1 — Jun 1:  708/1000 ❌ (threshold: 720, delta: -12)
- Attempt 2 — Jun 15: 680/1000 ❌ (threshold: 720, delta: -40)
- Attempt 3 — Jun 29: 698/1000 ❌ (threshold: 720, delta: -22)
- D1 Ingestion & Transformation (34%): Needs Improvement
- D2 Data Store Management (26%): Needs Improvement
- D3 Data Operations & Support (22%): Meets Competencies ✅
- D4 Data Security & Governance (18%): Meets Competencies ✅
- Retake: July 13, 2026 — focused on D1 + D2 only
```

---

## June 29, 2026

**AWS DEA-C01 Retake Prep 2.0 — Exam Day**

Today was the exam day. Before the test, the keyword mapping list and the exam-trap notes
from the Master Battle Plan PDF were reviewed once more, and the feeling going in was
solid and calm. One last pass on the rules, one last pass on the traps, then straight into
the exam.

**Two last-minute clarifications**

- `CREATE TABLE ... AS SELECT ... WHERE 1=0` is the trick for creating a **zero-row copy**
  of a table structure without loading data
- EventBridge cannot directly target DataBrew; if orchestration is needed, Step Functions
  sits in between

**What stood out before the exam**

The last review was not about learning new material — it was about locking in the final
precision rules and trusting the work that had already been done over the previous 2.0
prep phase. The keyword map, the trap list, and the repeated recall passes all came together
in one final review session.

>**What I understood**
>- The final value now is not more volume, but calm recall and exact trigger mapping
>- The `WHERE 1=0` pattern is the right mental shortcut for a table-structure-only copy
>- EventBridge → DataBrew is not a direct path; orchestration belongs to Step Functions

---

## June 28, 2026

**AWS DEA-C01 Retake Prep 2.0 — Day 13: Trigger Phrases and Precision Mapping**

Today was a pure keyword-mastery day. Three quiz rounds tested trigger phrases under exam-like
pressure, and the main lesson was clear: the problem is not broad service knowledge — it is
the exact mapping from wording to the right AWS service.

**Quiz results**

- Quiz 1 at Madrona Park Beach: **17/20**
- Quiz 3A at Madrona Park Beach: **5/5**
- Quiz 3B at Madison Park Beach: **15/20**

Quiz 3A was a complete hit and confirmed that the trigger phrases from the first keyword block
are fully locked in. Quiz 3B was harder and more distracting, but that was useful because it
exposed the last precision gaps.

**Today’s key learning points**

- CloudTrail logs Management Events by default; Data Events like `S3 GetObject` must be enabled explicitly
- CloudTrail Lake is the right choice for long-term immutable audit history with SQL access across accounts and regions
- IAM Trust Policies define who can assume a role; Permission Policies define what the role can do
- Glue Job Bookmarks work with DynamicFrames, not cleanly with Spark DataFrames
- S3 Event Notifications do not support SQS FIFO directly; EventBridge is the correct workaround
- EMR Spot is only safe when Master and Core stay On-Demand and Spot is used only for Task nodes
- SSE-S3 is for simple, low-cost encryption; SSE-KMS is the choice for auditability and key control
- Lake Formation is for governance and fine-grained access, not for data prep
- DataBrew is for visual no-code data preparation, not governance
- Athena cost drops most through Parquet, partitioning, and Snappy compression
- Redshift Spectrum is read-only; writes belong in S3 or a regular Redshift table

**Quiz errors**

- Quiz 1: CloudTrail Data Events, Trust Policy vs Permission Policy, and Glue Job Bookmarks
- Quiz 3A: 5/5 correct, no errors
- Quiz 3B: S3 Event Notifications → SQS FIFO, EMR Spot rolling, SSE-S3 vs SSE-KMS, and CloudTrail Data Events

**Exam readiness**

No more quizzes today. Tomorrow should be a short review block with the error list, trigger
mappings, and the most important trap patterns. The focus is now on retrieval, not new material.

>**What I understood**
>- The remaining issue is not broad knowledge, but precision in trigger-word mapping
>- Quiz 3A confirmed the first keyword block is fully internalized
>- Quiz 3B exposed the last small gaps, which is exactly what the final prep day should do

---

## June 27, 2026

**AWS DEA-C01 Retake Prep 2.0 — Day 12: First Cold Quiz on untested Material**

Day 12 was about one thing: verifying that known material holds up under real quiz pressure.
The full study PDF had been worked through — today was the first time that exact content
got tested in a cold, unseen 20-question quiz format. Completed on a bench directly outside
**William H. Gates Hall at the University of Washington, Seattle**.

Result: **18/20**.

The distinction matters: this was not new material. This was a verification that what
was studied actually converted into reliable recall under exam conditions. It did.

**Topics verified under quiz pressure for the first time**

- SNS + SQS Fan-out pattern vs EventBridge Rules vs Pipes
- S3 Event Notifications — SQS FIFO trap via EventBridge
- LSI vs GSI — creation-time constraint, consistency difference
- DynamoDB Streams, DAX, TTL, and hot partition fixes
- OpenSearch UltraWarm vs Cold — Cold is not directly queryable
- AWS Batch, Neptune, Amazon Keyspaces, AppFlow
- Step Functions Standard vs Express — no `.sync` / `.waitForTaskToken` in Express
- Firehose Parquet compression — GZIP not supported for columnar formats in Athena
- Lake Formation LF-Tags, Cell-Level Security, IAMAllowedPrincipals
- Redshift Distribution Styles, COPY vs INSERT, Concurrency Scaling
- Glue Bookmark Pause Mode, `CRAWL_EVENT_MODE`, FLEX execution class

**Two misses — precision-level traps, not knowledge gaps**

**Q1 — SNS+SQS vs EventBridge:**
Knew both options. Chose the wrong one under pressure. Rule now locked:
fan-out with per-consumer DLQ and filter = SNS + SQS, not EventBridge Rule.

**Q15 — GZIP vs Snappy for Parquet in Athena:**
Knew Snappy was supported, got briefly confused by GZIP.
Rule now locked: GZIP is not supported for columnar formats (Parquet/ORC) in Athena.

Both errors were precision-level. Zero service confusion. Zero category mistakes.

**Full Quiz History**

| Session | Result |
|---|---|
| Checkpoint 1 | 12/12 ✅ |
| Checkpoint 2 | 15/15 ✅ |
| StackLessions Final Walkthrough | 18/20 ✅ |
| Standard Quiz (Jun 25, Pocket Beach) | 15/15 ✅ |
| Trap Quiz (Jun 25, MOHAI) | 13/15 ✅ |
| Trap Quiz 2.0 (Jun 26, Elliott Bay Park) | 13/13  ✅ | out of 15, two completely wrong questions were recognized and rejected logically |
| Cold Quiz (Jun 27, UW Gates Hall) | 18/20 ✅ |

>**What I understood**
>- 18/20 on a first-time cold quiz across all DEA-C01 domains is a clear green light for June 29
>- Both errors were pressure-level precision traps, not knowledge gaps — that is the
  best possible type of wrong answer two days before an exam
>- The remaining hours before the exam are spent enjoying Seattle sunshine —
  the city that is the end of the roadmap 🌤️

---

## June 26, 2026

**AWS DEA-C01 Retake Prep 2.0 — Day 11: Trap Training with Flashcards and Quiz Verification**

Today was fully about active knowledge testing instead of passive review. First the worst exam
traps were drilled with flashcards, then a hard quiz with new questions tested whether the
understanding really held up under pressure.

**What helped most today**

The combination of flashcards and quiz worked well because it covered two layers: first
clean recognition of traps, then recall under pressure. Some quiz questions were
intentionally or accidentally flawed, and that actually helped — it forced AWS rules to be
verified actively instead of being accepted blindly.

**Key lessons of the day**

- Redshift Spectrum can read Glacier Instant Retrieval, but not Glacier Flexible Retrieval
  or Deep Archive
- Step Functions Express does not accept `.sync` or `.waitForTaskToken`; that belongs to
  Standard workflows

**Why this matters**

Today was the real proof that DEA-C01 is not just about memorization, but about correctly
recognizing wording traps, service boundaries, and incorrect AWS language. That skill is
what decides whether an answer is only "kind of right" or truly the safe exam answer.

**Checkpoint status**

| Session | Result |
|---|---|
| Trap Quiz | 13/15, but the wrong questions were recognized and rejected logically |
| Overall takeaway | Exam-trap understanding confirmed |

---

## June 25, 2026

**AWS DEA-C01 Retake Prep 2.0 — Day 10: Quiz Training in Seattle**

First prep day in Seattle. Instead of structured reading, knowledge was
tested in two quiz rounds under real exam conditions — one at Pocket Beach on a log, one
on a bench at the Seattle Museum of History & Industry.

**Method of the day**

Active recall under real conditions: no looking things up, qualifier in the last sentence
read first, two options eliminated immediately, then answer. Environment: relaxed but
focused — exactly the combination that matters before an exam.

**Quiz Round 1 — Standard Quiz (Pocket Beach)**

- 15 questions across all 4 domains
- Result: **15/15 ✅** 

**Quiz Round 2 — Trap Quiz (Seattle Museum of History & Industry)**

- 15 pure trap questions from the hardest DEA-C01 pitfalls
- Result: **13/15 ✅**
- Wrong: GSI question + one additional trap question

**New insight today: GSI independent capacity**

GSIs have their own provisioned capacity, independent of the base table. That means:

- A GSI with a low-cardinality partition key (e.g. `order_status` with 4 values) creates hot partitions in the GSI
- GSI throttling propagates back to the base table
- The only real fix is redesigning the GSI partition key to a high-cardinality column
- Increasing base WCU, switching to On-Demand, or adding DAX do not fix the root cause

**Glue FLEX language precision**

Today's quiz had a misleading phrasing using "Spot Capacity" for Glue FLEX. The correct exam language is: Glue FLEX uses **spare capacity / flexible execution class**, not EC2 Spot Instances. Glue runs on DPUs, never on EC2 — this remains an absolute exam rule.

**All checkpoints and quiz results**

| Session | Result |
|---|---|
| Checkpoint 1 | 12/12 ✅ |
| Checkpoint 2 | 15/15 ✅ |
| StackLessions Final Walkthrough | 18/20 ✅ |
| Standard Quiz today | 15/15 ✅ |
| Trap Quiz today | 13/15 ✅ |

**Personal note**

Seattle has turned out to be the ideal study location. Pocket Beach and the Museum of
History & Industry as spontaneous study spots — relaxed, focused, productive. After the
unbearable heat in Germany, Seattle is exactly the right climate and mental environment
for the final days before the exam.

>**What I understood**
>- The GSI independent capacity rule is now fully anchored: hot partition on GSI propagates to base table, fix is always key redesign
>- Glue FLEX = spare capacity / flexible execution class — the phrase "Spot" in that context is the trap

---

## June 24, 2026

**AWS DEA-C01 Retake Prep 2.0 — Day 9: Flight to Seattle: 9 Hours of Airborne Active Recall**

Today was the flight from Germany to Seattle. Nine hours in the air — used every minute
of it. The Master Battle Plan PDF was read three times cover to cover, with the third
pass using the same active recall method from the last few days.

**What was done at 30,000 feet**

- Full PDF reviewed three complete passes
- Third pass: keyword tables covered, answers spoken aloud before revealing
- Every domain covered: D1, D2, D3, D4
- Any remaining gaps are so marginal they will be locked in before June 29

The method is working. Every domain sits. If there are still any errors, they are at
the precision edge — not structural gaps, not service confusion, not category mistakes.

**The bigger picture**

This day deserves to be noted for more than just study hours. After an operation, a
transatlantic flight, and now landing in the city where the L5 Data & Feature
Infrastructure job at the end of this roadmap actually lives — everything is running
in parallel and everything is moving forward.

Seattle is not just a destination. It is the preview of the life this roadmap is
building toward. The energy here is different. The motivation is at a level that has
not been felt in years.

>**What I understood**
>- Nine hours of focused review on a long-haul flight with no distractions is more
  valuable prep time than most full days at home — no phone, no interruptions, just
  the material
>- The happiness is earned, not accidental: this is the result of months of consistent
  daily effort, an operation in between, and the refusal to stop

---

## June 23, 2026

**AWS DEA-C01 Retake Prep 2.0 — Day 8: Domain 3 + 4 Deep Study, Active Recall Complete**

The third day of the 8-day battle plan was completed successfully. Domain 3 and Domain 4
were studied with the same method as the previous days: three passes, active recall on the
third pass, answers hidden, spoken out loud.

**Why the method works**

This is not a new discovery — it has already proven effective in school for vocabulary
tests in French and English, and in history for memorizing events. Active recall is one
of the strongest long-term learning methods because the brain is forced to retrieve
information rather than just recognize it. Doing three repetitions and speaking the
answer aloud adds a second sensory channel, which strengthens retention even more.

**Topics covered today**

**Domain 3 — Data Operations & Support (22%)**

- Kinesis monitoring: `GetRecords.IteratorAgeMilliseconds` with Statistic `Maximum`, not Average
- IteratorAge fixes: shards, Parallelization Factor, Enhanced Fan-Out — never Lambda concurrency
- Glue Bookmark silent failure: disabled bookmark reprocesses everything without an error
- EMR node strategy: Master + Core On-Demand, Task nodes on Spot
- Orchestration: MWAA only when existing Airflow DAGs exist, Glue Workflows for Glue-only chains, Step Functions for multi-service pipelines
- Step Functions Standard vs Express: fixed at creation, not changeable later

**VPC Endpoints — now fully anchored**

- Gateway Endpoint = only S3 and DynamoDB, free, added in the route table
- Interface Endpoint = all other services (KMS, Glue, Kinesis, Athena, SSM, CloudWatch…)
- Key realization: the Gateway Endpoint belongs to the target service, not the caller. Glue, Athena, and Lambda are callers; S3 and DynamoDB are the only Gateway targets
- Metaphor: one lighthouse per island — every service needs its own lighthouse, Gateway only for S3 and DynamoDB

**Domain 4 — Data Security & Governance (18%)**

- Lake Formation setup order: register S3 location → revoke `IAMAllowedPrincipals` → set grants
- LF-Tags: scales from n×n to n+n, new resources inherit automatically
- Column Filter on Partition Key is not allowed — use Row Filter instead
- Macie + DataBrew: canonical PII pattern, Macie discovers and DataBrew masks
- EMR PHI encryption: SSE-KMS at rest + TLS in transit via EMR Security Configuration

**Personal note**

Tomorrow morning is the flight to Seattle. The heat in Germany has been a real productivity
killer in the last few days, so Seattle will at least be better from a climate perspective.
The flight will be used for a light review of the marked gaps.

>**What I understood**
>- Active recall is working because it converts passive familiarity into real retrieval, which is much closer to the exam than rereading
>- Domain 3 and Domain 4 are now being anchored through repeated recall rather than quick recognition
>- The VPC Endpoint rule is now locked in: Gateway only for S3 and DynamoDB, everything else Interface

---

## June 22, 2026

**AWS DEA-C01 Retake Prep 2.0 — Day 7: Domain 1 + 2 Deep Study: Active Recall with the Master Battle Plan**

Today was a strong focus day in the DEA-C01 retake prep. Domain 1 and Domain 2 were
worked through three times from the Master Battle Plan PDF, and on the third pass the
keyword tables were used actively by hiding the answer and saying it out loud before
revealing it.

**Study method of the day**

This was not passive reading. The approach was stepwise active recall:
- first pass for structure,
- second pass for reinforcement,
- third pass for hidden-answer retrieval.

That third pass is much closer to real exam conditions because it forces the brain to
retrieve the answer instead of just recognizing it.

**Topics covered today**

**Domain 1 — Data Ingestion & Transformation (34%)**

- Glue Data Catalog, crawler modes, DynamicFrames vs DataFrames
- ResolveChoice strategies, job bookmarks, Glue FLEX, and DQDL
- Glue Schema Registry, FindMatches, and VPC setup
- Kinesis Data Streams vs Firehose, Enhanced Fan-Out, IteratorAge, Firehose format conversion
- S3 event notifications, storage classes, Parquet, Snappy, and streaming-to-Parquet paths

**Domain 2 — Data Store Management (26%)**

- Redshift distribution styles, sort keys, VACUUM/ANALYZE, and COPY with manifest
- Spectrum and its Glacier limits
- Athena Partition Projection, CTAS, Workgroups, and Federated Query
- DynamoDB capacity modes and the classic GSI throttling trap

**Why this worked**

Three passes plus active self-questioning is a very effective mix of repetition and
retrieval training. Hiding the solution and answering out loud turns trigger phrases into
real recall patterns instead of simple recognition.

>**What I understood**
>- The third pass is the important one, because it forces active memory retrieval rather than passive familiarity
>- Domain 1 and Domain 2 are now being re-anchored through structure, not just through volume
>- This method is better aligned with the exam than another round of pure reading or passive review

---

## June 21, 2026

**AWS DEA-C01 Retake Prep 2.0 — Day 6: Full Content Analysis + 8-Day Battle Plan**

The most intensive planning day of the entire retake prep. The complete 484-page DEA-C01
compendium was analyzed and distilled across three iterations into a structured premium
study document. This was not reading — this was surgical content mapping.

**Domain 1 — Data Ingestion & Transformation (34%)**

**Glue**
- Glue Data Catalog: exactly 1 per account per region, metadata only, shared with Athena/EMR/Spectrum
- Crawler modes: `CRAWL_EVERYTHING` vs `CRAWL_EVENT_MODE` (S3 Events → standard SQS, no FIFO, hard cap 100k messages)
- `DynamicFrame` vs Spark `DataFrame`: DynamicFrame for messy/semi-structured data with choice types, no fixed schema required
- 5 ResolveChoice strategies: `cast` (silent data loss), `make_cols`, `make_struct`, `project`, `match_catalog`
- Bookmark modes: Enable / Disable (default) / Pause — state only saved after `job.commit()`
- FLEX: ~34% cheaper, not Spot (Glue = DPUs, never EC2)
- DQDL: declarative quality rules on DeeQu basis directly in the job
- Schema Registry: BACKWARD is default — for streaming payload schemas (≠ Data Catalog = table metadata)
- FindMatches: fuzzy deduplication without a common key
- VPC trick: S3 Gateway Endpoint in Route Table + Self-Referencing SG Inbound Rule

**DataBrew vs Lake Formation — the #1 Trap**
- DataBrew = **modifies data** (no-code, 250+ transforms, PII masking recipes) — "the Sous-Chef"
- Lake Formation = **controls access** (fine-grained, row/column/cell, cross-account) — "the Maître d'"

**Kinesis**
- Hard rule: Real-time → KDS; Near-real-time → Firehose. No exceptions.
- Latency: KDS + EFO ~70ms / KDS classic ~200ms / Firehose zero-buffering ~5s / standard ~60s
- Enhanced Fan-Out: Data Streams only, dedicated 2MB/s per consumer per shard
- `IteratorAge` fix: Shards + ParallelizationFactor (1→10) + EFO — never Lambda concurrency
- Firehose Format Conversion: in-flight JSON→Parquet, requires Glue Catalog schema, min buffer 64MB, set `CompressionFormat` to UNCOMPRESSED
- Kinesis Data Analytics = Managed Service for Apache Flink (renamed August 2023)

**S3 & VPC**
- Glacier Deep Archive: NO Expedited, min 180 days, Standard ≤12h
- Gateway VPC Endpoint = S3 + DynamoDB only, free
- Interface VPC Endpoint = everything else (KMS, Glue, Kinesis, SSM…)

**Data Formats**
- Priority order: Parquet FIRST (10x win) → Partition → Compress
- Cost per 1 TB: CSV $5 → GZip $1.50 → Parquet+Snappy $0.50 → Parquet+Partitions+Projection $0.10
- Three paths JSON→Parquet: CTAS (one-shot, serverless) / Glue ETL (recurring, transformations) / Firehose Conversion (streaming, zero ETL)

**Domain 2 — Data Store Management (26%)**

**Redshift**
- Distribution styles: KEY (high-cardinality joins), ALL (small dims <3M rows), EVEN (no good key), AUTO (default)
- Sort keys: COMPOUND = default, 90% of cases; INTERLEAVED = ad-hoc multi-column, `VACUUM REINDEX` is expensive
- `VACUUM REINDEX` = interleaved keys only; `ANALYZE` = statistics only; `VACUUM FULL` = space + re-sort
- `COPY` > `INSERT`: COPY is parallel across all slices; INSERT is a Leader Node bottleneck
- Manifest + `mandatory:true` = deterministic, fails if file is missing
- Spectrum reads: Standard/IA/One Zone-IA/Intelligent-Tiering/Glacier Instant — NOT Glacier Flexible/Deep Archive
- Kinesis→Redshift: Materialized View + `AUTO REFRESH YES` = lowest latency without S3 staging
- Data Sharing: RA3 or Serverless only, zero copy, matching encryption required

**Athena**
- Partition Projection = #1 exam concept, eliminates `MSCK REPAIR TABLE`
- CTAS max 100 partitions, then batched `INSERT INTO`
- Workgroups: `BytesScannedCutoffPerQuery` = cost guardrail, not a cost lever

**DynamoDB**
- GSI Throttling Trap: low-cardinality GSI partition key → hot partition → base table throttling
- Fix: redesign the GSI key — not increase base WCU, not switch to On-Demand, not add DAX

**Domain 3 — Data Operations (22%)**

- `IteratorAge` metric: `GetRecords.IteratorAgeMilliseconds` uses **Maximum**, not Average
- Glue Bookmark silent failure: Disabled/Reset Bookmark reprocesses everything without throwing an error
- EMR node strategy: Master + Core = On-Demand; Task = Spot (no HDFS)
- MWAA: most expensive option, only choose when "existing Airflow DAGs" or "no refactoring" appears in the stem
- Step Functions Standard: exactly-once, up to 1 year, `.sync` and `.waitForTaskToken`
- Step Functions Express: at-least-once, max 5 min, Request-Response only — **fixed at creation**

**Domain 4 — Data Security & Governance (18%)**

- Lake Formation is built **on top of** Glue Data Catalog — both IAM and LF checks must pass
- LF-Tags: scales from n×n to n+n, new resources inherit tags automatically
- Setup gotchas: register S3 location, revoke `IAMAllowedPrincipals`, Column Filter on Partition Key is not allowed → use Row Filter instead
- Canonical PII pattern: Macie finds, DataBrew masks
- EMR PHI: SSE-KMS at rest + TLS in transit via EMR Security Configuration

>**What I understood**
>- The 484-page compendium surfaced depth that no practice tool or YouTube video covered
  fully — especially the Glue Bookmark silent failure, the GSI hot partition trap, and the
  LF-Tag scaling math
>- The canonical patterns (Macie + DataBrew for PII, Gateway = S3 + DynamoDB only,
  IteratorAge → shards not Lambda) are now internalized as absolute rules, not exam tips
>- The goal for June 29 is not 720 — it is the maximum possible score

---

## June 20, 2026

**AWS DEA-C01 Retake Prep 2.0 — Day 5: 20-Question Final Walkthrough — StackLessions Course Complete**

Day 5 and the last day of the StackLessions course. The final video was a 20-question
walkthrough across all four domains. Score: **18/20 ✅**. Course complete.
The two wrong answers are the most valuable content of the entire walkthrough.

**Score: 18/20 — Q4 and Q19 wrong**

**Q4 — Athena CTAS vs Firehose Record Format Conversion:**
The mistake was logical: CTAS also converts JSON→Parquet, so it seemed valid. The
critical distinction: CTAS is a downstream step that creates a second job with extra
DPU costs, while Firehose Record Format Conversion happens **in-flight** — directly
at delivery, on a stream that is already running, with no extra pipeline.
Keyword: "cheapest ongoing conversion" + "stream" → always Firehose, never CTAS.

**Q19 — Gateway VPC Endpoint vs Interface VPC Endpoint for KMS:**
The classic VPC Endpoint trap. **Gateway Endpoints exist only for S3 and DynamoDB** —
these are the only two services, no exceptions. Everything else (KMS, Glue, Kinesis,
Secrets Manager, STS) uses **Interface Endpoints via PrivateLink**. The rule is absolute.

| Keyword | Answer |
|---|---|
| "cheapest ongoing conversion" + "stream" | Firehose in-flight conversion, not CTAS |
| VPC Endpoint for KMS / Glue / Kinesis | Interface Endpoint (PrivateLink), never Gateway |
| Gateway Endpoint = free + only two services | Gateway → S3 + DynamoDB only |

**Exam-Day Rules — Locked In**

1. Read the last sentence of the stem first — the qualifier lives there
2. Eliminate "manually" immediately — the automated option almost always wins
3. Prefer serverless and managed: Athena, Glue, Firehose, Lambda, Redshift Serverless unless ruled out
4. Match retrieval SLA to tier: minutes = Expedited, hours = Standard; Deep Archive has no Expedited
5. Flag and move on — never burn 5 minutes on one question

**Keyword → Answer Final Map**

- "Real-time" → KDS; "Near-real-time / deliver to S3" → Firehose
- "Row/column/cell-level access" → Lake Formation; "Unknown access pattern" → Intelligent-Tiering
- "Audit who decrypted" → SSE-KMS; "Cheapest private S3 access" → Gateway VPC Endpoint
- "Existing Kafka producers" → MSK; "Python DAG / Airflow" → MWAA; "CQL / Cassandra" → Keyspaces
- "Schema evolution / time travel on S3" → Apache Iceberg; "Exactly-once + streaming" → Managed Flink

**Full Checkpoint History**

| Checkpoint | Score |
|---|---|
| Checkpoint 1 | 12/12 ✅ |
| Checkpoint 2 | 15/15 ✅ |
| Final Walkthrough | 18/20 ✅ |

Zero wrong answers in the first two checkpoints. Two precision-level errors in the final
walkthrough — both identified, both understood, both closed. The shift is from "pass"
to maximum score.

>**What I understood**
>- 18/20 with both errors being precision-level distinctions is not a knowledge problem —
  it is the gap between 720 and 850+, and closing it requires this exact level of detail
>- The Firehose in-flight conversion vs CTAS and the Gateway vs Interface VPC Endpoint
  rules are now absolute; no exam question can land these as traps anymore
>- After 12/12, 15/15, and 18/20 in the final walkthrough, the target for June 29 is
  not 720 — it is the maximum possible score

---

## June 19, 2026

**AWS DEA-C01 Retake Prep 2.0 — Day 4: Exam Traps, Cost Optimization & Pipeline Architectures**

Broadest content day so far with the StackLession YouTube course: six videos plus a second
checkpoint video across all domains. Result: **15/15 ✅**. The pattern is now undeniable —
Checkpoint 1 was 12/12, Checkpoint 2 was 15/15, and the new learning format is clearly working.

**Exam Traps**

- Macie identifies PII, never modifies it; redaction belongs to Glue transformations or S3 Object Lambda
- Real-time means `KDS + Managed Flink`; eliminate Firehose, S3-as-a-hop, and SPICE from choices
- Spectrum cannot read any Glacier class; restore/copy to Standard first
- Known access pattern → S3 Lifecycle; unknown or changing → Intelligent-Tiering
- Glue runs on DPUs, not EC2; Glue FLEX is the cost lever, never Spot
- High `IteratorAge` = more shards + higher ParallelizationFactor; Lambda concurrency alone does nothing
- DataBrew prepares/transforms, Lake Formation governs access
- "Manually" is usually a trap when AWS offers a managed or event-driven alternative

**Cost Optimization**

- Glue: FLEX execution class for cheaper runs with spare capacity
- Redshift: Serverless for intermittent workloads, Reserved Nodes for steady baselines
- Athena: Parquet first, then partitioning, then compression; `BytesScannedCutoffPerQuery` is a guardrail, not a cost lever
- S3: Lifecycle for known access, Intelligent-Tiering for unknown; beware per-object monitoring on tiny files
- EMR: Spot only on Task nodes; Master/Core stay On-Demand
- Lambda: tune memory first, then ARM/Graviton2, then Provisioned Concurrency only when utilization is high
- DynamoDB: On-Demand for spiky/unpredictable, Provisioned + Auto Scaling for sustained load

**Reference Architectures**

1. Batch analytics: `S3 → Glue Crawler → Glue Catalog → Glue ETL → S3 Parquet → Athena/Redshift`
2. Real-time streaming: `KDS → Managed Flink → DynamoDB + Firehose → S3`
3. Near-real-time: `Firehose → S3 Parquet → Athena + Partition Projection`
4. Database migration: `SCT → DMS full-load + CDC → S3 staging → Glue ETL → Redshift`
5. Governed data lake: `S3 zones → Lake Formation + Glue Catalog → Athena/Spectrum/EMR/Glue ETL`

**Secondary Services + Apache on AWS**

| Apache / Open-Source | AWS Service |
|---|---|
| Spark | EMR / Glue |
| Flink | Managed Service for Apache Flink |
| Kafka | Amazon MSK |
| Hive Metastore | Glue Data Catalog |
| Airflow DAGs | Amazon MWAA |
| Cassandra / CQL | Amazon Keyspaces |
| Ranger | Apache Ranger on EMR |
| Iceberg | S3 / Athena / EMR / Glue |

Additional mappings: AppFlow for SaaS ingestion, Neptune for graph workloads, EventBridge Pipes for filtered point-to-point flows, OpenSearch UltraWarm for queryable older logs, and AWS Batch for highly parallel jobs.

**Checkpoint 2: 15/15 ✅**

Two checkpoints, zero wrong answers. Keyword-to-answer recognition is now fully locked in.

>**What I understood**
>- The shift is no longer just about knowing services — it is about instantly matching the problem to the right AWS pattern under time pressure
>- The traps are becoming easier to eliminate because the learning material now focuses on boundaries, not memorized answers
>- 15/15 on a six-domain checkpoint is the strongest proof yet that the new learning approach is deeply embedding the architecture logic

---

## June 18, 2026

**AWS DEA-C01 Retake Prep 2.0 — Day 3: DynamoDB, DMS, EMR, Security, CloudWatch & Glue Data Quality**

Third deep-dive day with the StackLession YouTube course. Six service blocks covered,
each with exam traps and keyword maps. The pattern is clear: every video reveals exactly
the kind of nuance the real exam tests that practice questions never explain.

**DynamoDB**

- **Partition Key** = must be unique; **Sort Key** = optional, enables range queries
- `GetItem` = single item by PK; `Query` = items by PK + optional SK filter; `Scan` = full table, expensive
- **GSI** (Global Secondary Index): new PK + SK, eventually consistent, separate RCU/WCU
- **LSI** (Local Secondary Index): same PK, different SK — must be defined at table creation, strongly consistent
- DAX = in-memory cache, microsecond latency, write-through; for read-heavy workloads
- **On-Demand** vs **Provisioned**: on-demand for unpredictable traffic, provisioned for consistent load + cost control
- DynamoDB Streams + Lambda = event-driven triggers for downstream processing
- Keyword map: "millisecond → microsecond latency" → DAX; "flexible schema + massive scale" → DynamoDB; "cross-region replication" → Global Tables

**DMS & EMR**

- DMS = ongoing replication + one-time migration; source stays live during migration
- SCT (Schema Conversion Tool) = converts schema when source ≠ target engine
- DMS replication instance = separate EC2 running the migration engine — size it appropriately
- **EMR**: master node manages jobs, core nodes store HDFS + run tasks, task nodes compute only (no HDFS, spot-friendly)
- EMR on EC2 vs EMR Serverless: serverless for variable workloads, EC2 for persistent clusters with tuning control
- Bootstrap actions = run scripts at cluster launch before applications start
- Keyword map: "heterogeneous migration" → DMS + SCT; "Hadoop / Spark managed cluster" → EMR; "cost-optimized compute workers" → EMR task nodes on Spot

**Data Security: IAM, KMS & Encryption**

- IAM Policy evaluation order: explicit Deny → Explicit Allow → implicit Deny
- **KMS CMK**: AWS-managed key vs Customer-managed key — customer-managed = full rotation + audit control
- SSE-S3 = AWS manages keys; SSE-KMS = customer controls key policy + CloudTrail audit; SSE-C = customer provides key per request
- VPC Endpoint for S3 = data never leaves AWS network; no NAT gateway needed
- Macie = detects PII and sensitive data in S3 automatically — not a firewall, not an encryptor
- Exam trap: **Macie identifies only** — it does not block or encrypt; action must come from separate policy/workflow

**CloudWatch, CloudTrail & Monitoring**

- CloudWatch Logs = collect; CloudWatch Metrics = measure; CloudWatch Alarms = react
- CloudWatch Logs Insights = ad-hoc query over log data with its own query syntax
- CloudTrail = API call audit log, every AWS action recorded; not real-time monitoring
- `IteratorAge` high → consumer is falling behind → increase shards or Parallelization Factor, not Lambda concurrency
- Exam trap: CloudTrail is **not** for real-time monitoring — use CloudWatch Events / EventBridge for that

**Glue Data Quality — DQDL Rules**

- DQDL = Data Quality Definition Language — declarative rules inside Glue ETL jobs
- Rule types: `Completeness`, `Uniqueness`, `ColumnValues`, `RowCount`, `Freshness`
- Results can be written to S3 or sent to CloudWatch for alerting
- Exam trap: DataBrew = visual profiling + transformation; Glue Data Quality = programmatic rule enforcement

**Amazon Macie — PII Detection**

- Macie scans S3 for PII: names, SSNs, credit card numbers, passport data
- Generates findings → EventBridge → SNS or Lambda for automated response
- Does **not** encrypt, does **not** block — detection only
- Keyword map: "find sensitive data in S3" → Macie; "automatically classify S3 objects" → Macie; "respond to PII finding" → EventBridge + Lambda

>**What I understood**
>- Every StackLession video delivers the exact layer that the real exam tests: not what a service does, but when it does it and what the trap is when you confuse it with the next closest option
>- The DynamoDB LSI vs GSI distinction, Macie's detection-only role, and CloudTrail vs CloudWatch confusion are all high-probability exam traps that were never explained at this depth in any practice tool

---

## June 17, 2026

**AWS DEA-C01 Retake Prep 2.0 — Day 2: Redshift, Athena, Lake Formation, S3 & Data Formats**

Second day with the StackLession YouTube course. Today covered the two heaviest blocks —
Redshift and Athena — not as a recap, but at real architectural exam depth.
First checkpoint result: **12/12 ✅** — every question correct.

**Redshift**

- `COPY` uses every slice in parallel — `INSERT` goes through the Leader Node row by row;
  never use `INSERT` for bulk loading
- Multiple concurrent `COPY`s into the same table force serialized load + `VACUUM` after —
  one `COPY` with many files is always the better pattern
- Manifest files are idempotent and deterministic; `mandatory: true` prevents silent partial loads
- `UNLOAD` with Parquet: every slice writes in parallel — Parquet is ~2x faster and ~6x smaller than CSV
- **Distribution styles**: KEY for large table joins on high-cardinality columns, ALL for
  small dimension tables (<3M rows), EVEN when no join key exists, AUTO as default
- `COMPOUND SORTKEY` = 90% of cases, cheap `VACUUM`; `INTERLEAVED` = brutally expensive
  `VACUUM REINDEX`, never for frequent loads
- Spectrum **cannot** read Glacier Flexible Retrieval or Deep Archive — restore to Standard first, then query
- **Streaming Ingestion**: `CREATE EXTERNAL SCHEMA FROM KINESIS` + Materialized View with
  `AUTO REFRESH YES` — no S3 staging, no Firehose, lowest latency path

**Athena**

- Price = $5 per TB scanned — every optimization saves money directly; Parquet + Snappy +
  Partition Projection brings the same query to ~$0.10
- `CTAS` = cheapest CSV→Parquet conversion without Glue or EMR, but max 100 partitions —
  larger jobs need batched `INSERT INTO`
- **Partition Projection** eliminates `MSCK REPAIR TABLE` entirely — Athena computes
  partitions at query time, zero Catalog lookups
- **Workgroups**: team isolation, `BytesScannedCutoffPerQuery` as cost guardrail, dedicated CloudWatch metrics
- **Federated Query**: Lambda Connectors connect Athena SQL to DynamoDB, RDS, Redshift — no ETL needed
- Keyword map:
  - "Avoid MSCK REPAIR TABLE" → Partition Projection
  - "Cheapest CSV→Parquet" → CTAS
  - "Isolate teams + cap cost" → Workgroups + `BytesScannedCutoffPerQuery`

**Lake Formation, S3 Storage Classes & Data Formats**

- Lake Formation = fine-grained access control at column and row level; not DataBrew (transforms), not Glue (catalogs)
- **Readable by Spectrum**: Standard, Standard-IA, One Zone-IA, Intelligent-Tiering, Glacier Instant Retrieval
- **NOT readable without restore**: Glacier Flexible Retrieval, Deep Archive
- Parquet vs CSV: columnar → reads only needed columns + predicate pushdown;
  strong compression (Snappy/GZIP); 1 TB CSV = $5, 1 TB Parquet ≈ $0.50

**Checkpoint 1: 12/12 ✅**

First objective proof that the new learning approach is working. The StackLession format
translates service mechanics directly into exam-ready decision patterns.

>**What I understood**
>- The difference between the current approach and all previous prep is architectural
  framing — understanding why Redshift distributes data a certain way makes the exam
  questions lose their trick entirely
>- Partition Projection and `CTAS` are the kind of specifics that third-party practice
  tools rarely explain at this depth — this is exactly the layer that the real exam tests
>- 12/12 on day 2 is an early signal, not a guarantee, but it confirms the direction is right

---

## June 16, 2026

**AWS DEA-C01 Retake Prep 2.0 — Day 1**

Today marked the shift from question volume to deep service understanding. The official DEA-C01
results made the real gap obvious: Domain 2 and Domain 3 still need the most work, while
Domain 1 and Domain 4 are already at **Meets Competencies**. That changed the prep strategy
immediately.

**What changed today**

The focus moved away from endless practice exams and into a structured YouTube deep-dive
series with service explanations, keyword mapping, and real exam traps. That format is
finally making the AWS services click in the right context instead of just being recognized
from memorized answer patterns.

Key DEA-C01 traps reinforced today:

- Crawler modes: `CRAWL_EVERYTHING` vs. `CRAWL_EVENT_MODE`
- `DynamicFrame` vs. `DataFrame`
- `DataBrew` vs. `Lake Formation`
- `Kinesis Data Streams` vs. `Firehose` vs. `MSK`

>**What I understood**
>- The main problem was not lack of volume — it was not enough deep knowledge in the
  deciding topics
>- Lambda is only the right fit for short, event-driven Glue work; longer jobs belong to
  Glue or EMR
>- Step Functions Standard and Express are fundamentally different and fixed at creation
>- High `IteratorAge` needs more throughput, not more Lambda concurrency
>- For sub-second or millisecond latency, Data Streams is the correct choice, not Firehose

**New exam date**

The next exam is already booked: **June 29, 2026 at 20:15 Seattle local time**.
The trip to Seattle from **June 24 to July 5** also makes the timing useful for getting a
first real look at the future relocation area.

---

## June 15, 2026

**AWS DEA-C01 Retake: Attempt 2 — Result**

Second attempt. Score: **680/1000** — Fail. Passing threshold: 720.

**What the result says**

Two attempts, two fails. The scores tell a clear story: the preparation methods used so
far — TutorialsDojo, Nex Arc, Neal Davis — do not sufficiently reflect the current
DEA-C01 exam pool. Both times, high practice scores did not translate to the real exam.
That is not a knowledge problem. That is a preparation alignment problem.

The exam pool for DEA-C01 is still evolving and the third-party providers are lagging
behind. This is a documented pattern with newer AWS certifications, and it means the
strategy needs to change more fundamentally before the next attempt.

>**What I understood**
>- A drop from 708 to 680 on a second attempt with significantly more preparation is
  the clearest possible signal that the practice material does not mirror the real
  question pool closely enough — more practice volume on the same sources will not fix this
>- This result does not change the direction — it changes the method. The goal is the same.

**What comes next**

Retake after the mandatory 14-day waiting period. The next preparation cycle will be
built differently: less question volume, more conceptual depth from official AWS sources.

The conviction is still there. This gets solved.

---

## June 14, 2026

**AWS DEA-C01 Retake Prep — Day 12: Final Day Before the Retake**

The day before the exam was not a rest day — it was a clean, confident confirmation pass.
All six Neal Davis practice tests completed, TutorialsDojo missed questions cleared.
Tomorrow at 10:45 CET the cert gets earned.

**What happened today**

- **Neal Davis PT1–PT6** — all six sets completed — **~90% correct across the board**
- **TutorialsDojo missed questions** — 23 flagged questions reviewed and cleared
- No new material, no pressure — just activating the patterns that are already there

>**What I understood**
>- ~90% across all six Neal Davis sets in a single day is the cleanest possible signal
  that the knowledge is not tied to one provider's style — it is genuinely internalized
>- The 23 missed questions from TutorialsDojo were the last remaining rough edges;
  clearing them the day before the exam removes every identified weak spot
>- The feeling of being ready is not wishful thinking — it is backed by two weeks of
  consistent scores above 80% on the hardest available material

---

## June 13, 2026

**AWS DEA-C01 Retake Prep — Day 11: Final Mixed Practice Day**

Today was the last major practice day before the retake on Monday, and the numbers are
still strong across multiple providers. The real takeaway is bigger than the scores: the
amount of AWS knowledge absorbed in a very short time is now showing up as real decision
making, not just memorized patterns.

**Practice Results**

| Source | Questions | Result |
|---|---:|---:|
| Nex Arc PT2 | 65 | **82%** |
| Neal Davis PT3 | 25 | **92%** |
| Neal Davis PT4 | 25 | **86%** |
| Neal Davis PT5 | 25 | **95%** |

Rotating across providers again confirmed the same thing: the service distinctions are
staying sharp even when the question style changes.

**System Design Reflection**

Just for fun, a typical L5 Data & Feature Infrastructure ML Platform system design prompt was generated and discussed. 
The core requirement was a platform for high-volume feature ingestion, low-latency serving, training/serving consistency, and versioned feature access. 
The answer was a real L5-level hit: **Kinesis Data Streams** for streaming ingestion and replay, **Firehose** for managed delivery where no replay was needed, a clear **offline/online feature store split** for training and serving consistency, and **observability** through metrics like CTR, lineage, and skew detection. 
That structure matched the platform problem exactly and made the whole answer feel like a true end-to-end architecture, not a buzzword list.

>**What I understood**
>- DEA-C01 prep has done more than prepare for one certification — it has built the mental model needed to reason about real platform tradeoffs under uncertainty
>- The answer structure for the L5-style question was already strong because the AWS services were used as system components with explicit responsibilities, not just exam options
>- Claude Sonnet being impressed by the reasoning is a side effect of something more important: the conceptual foundation is now broad enough to transfer beyond the exam

---

## June 12, 2026

**AWS DEA-C01 Retake Prep — Day 10: Mixed Practice Day**

Today was about variety and deeper anchoring. Different practice sources, different wording,
same result: the core AWS service patterns are holding up under pressure.

**Practice Results**

| Source | Questions | Result |
|---|---:|---:|
| Nex Arc PT3 | 65 | **87%** |
| Neal Davis PT1 | 25 | **95%** |
| Neal Davis PT2 | 25 | **91%** |

Mixing sources before the June 15 retake is doing exactly what it should: reinforcing the
same concepts through different phrasing so the services are anchored more deeply, not
just recognized from one provider's style.

>**What I understood**
>- Rotating between Nex Arc, TutorialsDojo, and Neal Davis is strengthening the same service patterns from multiple angles, which makes recall more durable
>- 87% on Nex Arc PT3 confirms that the hardest-style questions are still being solved cleanly even without repeating the same provider format every day
>- 95% and 91% on Neal Davis sets show that the fundamentals are stable enough to survive a different question style entirely

---

## June 11, 2026

**AWS DEA-C01 Retake Prep — Day 9: Nex Arc PT4: 91% ✅ + Exam Booked for June 15**

**91% on 65 questions.** Three consecutive Nex Arc sets: 84% → 81% → 91%.
The retake is no longer booked for June 16 — it was moved up to **June 15, 2026 at 10:45 CET**.

**Nex Arc PT4 — 65 Questions — 91% Correct**

Nex Arc practice questions are known to be harder and more complex than the real DEA-C01
exam, and the material is not always fully up to date with the current question pool.
That context matters — and it makes 91% an even stronger signal. This was not a test of
memorized answers. This was a test of whether the underlying AWS reasoning is solid
enough to solve questions that go beyond the scope of the actual exam. It is.

>**What I understood**
>- Three Nex Arc sets in a row above 80%, peaking at 91%, confirms that the knowledge is
  not only stable but actively improving across the hardest available practice material
>- The Nex Arc pool is deliberately harder and partially outdated relative to the real
  DEA-C01 — 91% on that benchmark means the real exam has a comfortable margin built in
>- Booking the exam one day earlier than planned is the right call: the preparation is
  ready, and there is no benefit in waiting

**Exam Booking**

- **Retake exam: June 15, 2026 — 10:45 CET**
- Booked today — one day earlier than originally planned
- 4 days of focused final consolidation remain

---

## June 10, 2026

**AWS DEA-C01 Retake Prep — Day 8: Nex Arc PT5 81% + Full TD Bank 97%**

Two full practice sessions today — one hard, one familiar. Both passed with authority.
6 days until the retake.

**Session Results**

| Source | Questions | Correct |
|---|---:|---:|
| Nex Arc Practice Test 5/6 | 65 | 81% |
| TutorialsDojo full bank (all domains) | 204 | 97% |
| **Total today** | **269** | — |

**What these numbers mean**

- **Nex Arc PT5 at 81%** — back-to-back Nex Arc sets above 80% confirms that the
  hardest available practice material is now consistently solvable, not just occasionally
- **TutorialsDojo 204 questions at 97%** — every single question in the TD catalog
  answered in one session with near-perfect correctness; the domain knowledge is fully
  saturated at this provider's level
- The two sources together cover both extremes: the hardest available questions and the
  broadest available coverage — both at a high level on the same day

>**What I understood**
>- Two consecutive Nex Arc exams above 80% (PT6: 84%, PT5: 81%) is a stronger signal
  than any single result — it proves the performance is stable, not fluked
>- 97% on 204 TD questions in one session means the entire provider question pool is
  now effectively solved; no question in that bank can surprise the retake
>- At this point the retake on June 16 is not a gamble — it is the confirmation of work
  that has already been done

---

## June 9, 2026

**AWS DEA-C01 Retake Prep — Day 7: Nex Arc Practice Exam 6/6: 84%**

The final Nex Arc practice exam — the one that was too complex to finish before the first
real attempt back in May — completed today with **84% correct on 65 questions**.
This is the clearest confidence signal of the entire retake prep phase.

**Context that makes this result significant**

Before the first exam attempt, Nex Arc PT6 was briefly attempted and immediately
abandoned. The question complexity was the direct trigger for purchasing the TutorialsDojo
subscription at the time. Coming back to that exact same test today and scoring 84%
is a measurable, concrete demonstration of how much the knowledge has grown.

**Why Nex Arc PT6 is a strong signal**

- Nex Arc practice questions are noticeably harder and more complex than the real DEA-C01 exam
- The actual exam uses more straightforward framing — Nex Arc deliberately tests deeper
  edge cases and service nuances that go beyond what the exam typically covers
- Scoring **84% on a harder-than-real-exam set** means the buffer going into the retake
  is substantial

>**What I understood**
>- The test that was too difficult to finish before the first attempt is now a clean 84% —
  that is not a coincidence, that is 3 weeks of targeted retake preparation making itself visible
>- If the hardest available practice set yields 84%, the real exam — which is less complex —
  should be well within passing range with comfortable margin
>- The retake on June 16 is no longer a question of whether the knowledge is there;
  it is a formality that the preparation has earned

---

## June 8, 2026

**AWS DEA-C01 Retake Prep — Day 6: Full TutorialsDojo Question Bank Completed**

Today was a milestone session: **204 questions** completed in one push, covering the
remaining TutorialsDojo catalog until every single question in the bank had been answered
at least once. It took nearly **3 hours**, but the result says everything — **85% correct**
on the day, with the latest weighted score now at **81%**.

**What happened today**

- Completed **204 TutorialsDojo questions**
- Finished the **entire TutorialsDojo question catalog**
- Study activity since the reset now shows **5 active study days** and **554 total questions**
- Score trend remains above the pass line and is currently marked **stable**

This is the strongest possible confirmation that the preparation is no longer partial.
There are no more untouched TutorialsDojo questions left.

**Why this matters**

- Finishing every TD question means the full provider coverage has now been exhausted
- **85%** across 204 questions is not a lucky burst — it reflects real endurance,
  pattern recognition, and domain familiarity over a large sample
- From tomorrow onward, a **Nex Arc 65-question practice exam** will be the cleanest
  external benchmark to measure the most honest current level before the retake

>**What I understood**
>- Completing the full TutorialsDojo bank removes uncertainty about whether any major
  practice topic was still untouched — now the focus shifts from coverage to validation
>- An 85% result over 204 questions shows that the knowledge holds not only in short
  bursts, but across long-form fatigue and repetition
>- Tomorrow's Nex Arc set matters because it is the best way to test how much of this
  progress transfers outside the question style that has now become fully familiar

---

## June 7, 2026

**AWS DEA-C01 Retake Prep — Day 5: Domain 3 & 4 Deep Review Day**

54 questions across the two remaining domains — 94% correct. Score trend now at **81%**.
All four domains have now completed at least one dedicated deep review pass since the retake strategy began.

**Session Breakdown**

| Domain | Status |
|---|---|
| D3 Data Operations & Support | ✅ Deep review complete |
| D4 Data Security & Governance | ✅ Deep review complete |
| **Total questions** | **54** |
| **Correctness** | **94%** |

**Score Trend**

- Latest weighted score (last 30d): **81%** — comfortably above the pass threshold
- 4 active study days since reset · all 4 domains reviewed

>**What I understood**
>- D3 and D4 completing at 94% confirms that the weakest domains from the first exam
  attempt are no longer the weak spots — the targeted review strategy closed exactly
  the gaps that cost points on June 1
>- With all four domains deep-reviewed, the remaining prep work before June 16 is
  consolidation and full-exam simulation, not gap-filling
>- 81% trend score on domain-specific practice — where questions are harder to game by
  context — is the strongest signal yet that the retake will land above 720

---

## June 6, 2026

**AWS DEA-C01 Retake Prep — Day 4: Domain 2 Deep Dive Day**

60 questions today across two domains — 97% correct. The score trend is now clearly
above the pass line and marked as **improving**. Latest weighted score: **79%**.

**Session Breakdown**

| Domain | Questions | Result |
|---|---:|---|
| D1 Data Ingestion & Transformation | 7 | ✅ Reinforcement pass |
| D2 Data Store Management | 53 | ✅ Deep dive complete |
| **Total** | **60** | **97% correct** |

**Score Trend**

- Latest weighted score (last 30d): **79%** — above the pass threshold
- Trend: **Improving**
- 3 active study days tracked since reseting TutorialDojos statistics · 250 total questions completed

>**What I understood**
>- 97% on 60 domain questions is not a fluked result — it reflects that core AWS service
  decision patterns are now genuinely stable and accessible under realistic practice conditions
>- Domain 2 at 53 questions deep means data store concepts — Redshift node types, DynamoDB
  partitioning, S3 lifecycle, Aurora vs RDS — are now in a reliable state for exam day
>- The retake on June 16 is not a gamble; the preparation is doing exactly what it was
  designed to do

---

## June 5, 2026

**AWS DEA-C01 Retake Prep — Day 3: Domain 1 Reinforcement Day**

Today was a pure reinforcement session. All **Domain 1** TutorialsDojo practice questions
were answered again, and every wrong answer was reviewed by reading the explanation
**twice**. The result was clear: the knowledge is getting sharper, the patterns are
sticking faster, and the retake is moving in the right direction.

**What I did today**

- Reworked all **Domain 1** TutorialsDojo practice questions
- Reviewed every missed answer with **two full explanation passes**
- Focused on locking in the decision patterns instead of just checking whether an answer was right

The strongest signal from today is not just repetition volume, but the feeling of
progressive clarity. More and more questions are being solved from understanding,
not hesitation.

**Performance signal**

- **95 questions** completed today
- **80% correct** on the day
- Score trend now marked as **improving**
- Latest weighted score shown in the app: **74%**

>**What I understood**
>- Domain 1 is becoming more stable because repeated exposure plus deep explanation review is reducing confusion between similar AWS service patterns
>- Reading the wrong-answer explanations twice forces the difference between “almost knew it” and “actually know it”
>- The retake attempt no longer feels like a hope-based try — it feels like the version that will earn the DEA-C01 certification

---

## June 4, 2026

**AWS DEA-C01 Retake Prep — Day 2: Deep Practice Day**

Today was about refinement, not fundamentals. Two TutorialsDojo 65-question runs and one official Skill Builder practice exam confirmed that the core AWS knowledge is already in place — the remaining gap is sharper decision-making between very similar service patterns.

**Practice Results**

- TutorialsDojo first cold run: **83%**
- TutorialsDojo second cold run: **87%**
- AWS Skill Builder practice exam: **100%** on 20 questions

The wrong answers were not random guesses. In most cases, the correct option had already been considered, which shows that the knowledge is stable but still needs tighter elimination under exam pressure.

>**What I understood**
>- The exam is now less about learning new basics and more about distinguishing close AWS service patterns quickly and accurately
>- The knowledge is already strong enough to eliminate distractors consistently; the remaining work is polishing the small differences that matter most
>- Taking the exam later in Month 4, as planned in my roadmap, would likely have made me even more comfortable. But today already showed that the foundation is in place

---

## June 3, 2026

**AWS DEA-C01 Retake Prep — Day 1: Domain 1 Review + Official Practice + TutorialsDojo Sprint**

First full retake prep day. Three sources, three signals. AWS Skill Builder confirmed
as the right primary strategy — the official framing alone improved answer precision
measurably by end of day.

**AWS Skill Builder | Domain 1 Review — Videos**

- **Lesson 1 — Perform data ingestion**: Kinesis Data Streams vs. Firehose, batch vs.
  streaming patterns, ingestion source types
- **Lesson 2 — Transform and process data**: Glue ETL, Lambda transformations, data
  format conversion (Parquet, ORC)

Key distinction locked in:
- **Firehose** = managed delivery + optional Lambda transformation, least operational
  overhead for streaming-to-S3 pipelines, no replay
- **Kinesis Data Streams** = control + replay + fan-out — use when consumers need
  independent read positions or reprocessing

**AWS Skill Builder | Official Practice Question Set DEA-C01 v2 — 20 Questions**

Score: **13/20 — 65%** · Duration: 18:08 min · Avg. answer time: 54s

| Topic | Mistake | Correct Pattern |
|---|---|---|
| IAM Policy S3 + Lambda | Chose to append `s3:GetObject` to existing policy | Replace `s3:*` with `s3:GetObject` + specific ARN `/prefix/*` — Least Privilege |
| Metadata store + fine-grained permissions | Chose Glue Data Catalog | Lake Formation = DB / Table / Column / Row / Cell level control |
| Redshift + S3 cost-effective 3/12-month retention | Chose wrong tiering | Redshift Spectrum for yearly analysis on S3 + Glacier Deep Archive for >12 months |
| Kinesis + Lambda IteratorAgeMilliseconds high | Chose provisioned concurrency | Reshard + Parallelization Factor + Enhanced Fan-Out — all three together |
| Streaming logs → Parquet → S3 least overhead | Chose Kinesis + EC2 + KCL | Firehose + Lambda transformation → S3 directly |
| Scanned documents full-text search performance | Chose Athena + Parquet | OpenSearch Service = most performance-optimized for full-text search |
| SQS downtime data loss (Select TWO) | Chose delay queue | Increase retention period (up to 14 days) + attach DLQ |

**3 core error patterns from this session:**

- **IAM Least Privilege**: `s3:*` is always wrong when only `s3:GetObject` is needed
- **Service differentiation**: Lake Formation vs. Glue Catalog, OpenSearch vs. Athena, Firehose vs. Kinesis+EC2
- **Multi-select questions**: first instinct lands one correct answer but misses the second

**TutorialsDojo | 40 Questions Sprint**

Score: **86%** — strongest single-session result since retake prep began.

>**What I understood**
>- Official Skill Builder questions use AWS-native framing that is less forgiving;
  TD questions are more pattern-recognizable — both are needed
>- 86% on TD after a focused Skill Builder session confirms the official material
  transfers directly to question performance — strategy working on Day 1
>- Skill Builder doesn't just teach facts; it trains the specific reasoning language
  AWS uses when writing exam questions — that is the retake edge

---

## June 2, 2026

Received the DEA-C01 exam result today: **708/1000** — 12 points below the passing threshold of 720. The retake was booked at the earliest possible date: **June 16, 2026**.

**AWS | DEA-C01 Exam Result Analysis**

- Score: **708** — fail by 12 points.
- Context: the exam was attempted **one full month ahead of roadmap schedule**, originally planned for Month 4 and taken in Month 3.
- The biggest lesson from this result is that the major third-party prep tools used so far — TutorialsDojo, NexArc, and Neal Davis — did not reflect the current exam pool well enough.
- Conclusion: for a newer AWS certification, **official AWS Skill Builder** is now the primary and most trustworthy prep source.

**Domain Breakdown**

| Domain | Weight | Status |
|---|---:|---|
| D1: Data Ingestion & Transformation | 34% | Needs Improvement |
| D2: Data Store Management | 26% | Needs Improvement |
| D3: Data Operations & Support | 22% | Needs Improvement |
| D4: Data Security & Governance | 18% | Needs Improvement |

**Retake Strategy — started immediately**

No cooldown, no self-pity, no wasted day. The recovery plan started the same day the score arrived.

- Switched fully to **AWS Skill Builder** as the only prep source for the retake.
- Started **Domain 1 Review: AWS Certified Data Engineer – Associate (DEA-C01)**.
- Covered:
  - Lesson 1: **Perform data ingestion**
  - Lesson 2: **Transform and process data**

The signal is clear: the miss was small, the gap is fixable, and the strategy is now cleaner than before.

**SQL | Exam Reflection — Copy Schema Without Data**

A useful SQL concept from today's exam reflection:

```sql
CREATE TABLE new_table AS
SELECT *
FROM old_table
WHERE 1 = 0;
```

---

## June 1, 2026

**AWS DEA-C01 | Exam Day ✓ — Result Pending**

The exam is done. 65 questions, real conditions, no practice buffer.
Result pending — AWS takes up to 5 business days for scoring.
Everything that could be prepared has been prepared. Now it waits.

**Pre-Exam Last-Minute Review**

Final session focused on the highest-density differentiation patterns:

- **Kinesis Streams** = control + replay; **Firehose** = managed delivery, no replay
- Sliding and tumbling windows → always **Managed Flink (Kinesis Data Analytics)**,
  never Firehose or Lambda alone
- **MSK** when the requirement mentions Kafka compatibility or existing Kafka ecosystem
- **Glue Schema Registry** for schema management and evolution in streaming pipelines
- Passing threshold: approximately 36 of 50 scored questions correct
  (15 of 65 are unscored pilot questions and do not count toward the result)

**What the Real Exam Taught**

Several questions covered topics that appeared in none of the major practice tools —
not NexArc, not TutorialsDojo, not Neal Davis.

>**What I understood**
>- DEA-C01 has been live since 2024 and its question pool evolves faster than
  third-party prep providers can update their material — recycled questions with
  surface-level rewording are not a reliable map of the current exam
>- Providers have no insider access to the active pool; their coverage reflects
  what was documented at launch, not what AWS has added since
>- The questions that were unfamiliar today were solvable through first-principles
  reasoning — knowing *why* a service exists and what problem it solves beats
  memorizing which answer a practice tool marks correct

The clearest takeaway from today: conceptual understanding is the only prep strategy
that transfers to questions no practice set has ever seen. Memorization has a ceiling.
Understanding does not.

**Sprint — Complete**

| Date | Activity | Result |
|---|---|---|
| May 22 | Set 1 Cold | 51% |
| May 24 | Set 2 Cold | 63% |
| May 27 | Set 3 | 75% ✅ |
| May 29 | Set 4 Cold | 68% |
| May 30–31 | 52 + 52 Missed Questions | Consolidated |
| **Jun 1** | **DEA-C01 Real Exam** | **⏳ Result pending** |

---

## May 31, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 21: Final Repetition — Exam Tomorrow**

Last day before the real exam. Same 52 missed questions from yesterday — repeated in
full. Exam booked for June 1, 2026. Tomorrow it counts.

**What today was about**

This was not a new session — it was a deliberate second pass over the exact same 52
questions reviewed yesterday. The goal was not discovery but consolidation: confirm that
the patterns locked in yesterday are still accessible today without re-reading
explanations. Repetition on the day before an exam is not cramming — it is making sure
the right mental pathways are the last ones activated before the real thing.

>**What I understood**
>- Repeating the same missed questions on back-to-back days targets active recall
  over passive re-reading — the brain retrieves the reasoning rather than just
  recognizing it, which is exactly the mechanism the real exam will test
>- The 90-minute TutorialsDojo timer created artificial pressure that does not exist
  in the real 130-minute DEA-C01 — tomorrow's exam gives roughly 2 minutes per
  question, enough space to apply the elimination patterns that were compressed in
  the practice sets
>- The sprint covered every major domain gap across 21 days: D1 Ingestion stabilized,
  D3 Operations reviewed twice, D4 Security consolidated — the knowledge is there

**Full Sprint — Final State**

| Date | Activity | Result |
|---|---|---|
| May 22 | Set 1 Cold | 51% |
| May 24 | Set 2 Cold | 63% |
| May 27 | Set 3 | 75% ✅ |
| May 29 | Set 4 Cold | 68% |
| May 30 | 52 Missed Questions review | Exam booked |
| **May 31** | **52 Missed Questions — second pass** | **Ready ✅** |

---

## May 30, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 20: Final Review Before Exam**

Last prep day before the real thing. 52 missed questions reviewed in full.
Confidence reading: at least 70% of the errors were recognized as known material on
re-attempt — not knowledge gaps, but pressure-induced mistakes. Exam booking confirmed
for tomorrow.

**What the missed questions review revealed**

The majority of errors across the TutorialsDojo sets trace back to one structural factor:
TutorialsDojo's default timer runs at 90 minutes for 65 questions — significantly tighter
than the real DEA-C01 format of 130 minutes for 65 questions. That gap created artificial
time pressure that pushed snap decisions on questions where the correct reasoning was
already available. Under real exam conditions, that pressure disappears.

>**What I understood**
>- 70%+ of the reviewed missed questions were answered correctly on re-attempt without
  hints — the knowledge is there; the errors were execution errors under compressed time,
  not conceptual gaps
>- The real DEA-C01 gives 2 minutes per question vs. roughly 83 seconds on TutorialsDojo
  default — that extra time per question is the exact buffer needed for the Kinesis stack
  and IAM role elimination patterns that caused the most damage
>- D3 Operations, D2 Store, and D4 Security were today's focus; after this review cycle
  all four domains have now had at least one dedicated consolidation pass

**Full Sprint — Final Snapshot**

| Date | Activity | Result |
|---|---|---|
| May 22 | Set 1 Cold | 51% |
| May 24 | Set 2 Cold | 63% |
| May 27 | Set 3 | 75% ✅ |
| May 29 | Set 4 Cold | 68% |
| **May 30** | **52 Missed Questions — final review** | **Exam booked ✅** |

---

## May 29, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 19: TutorialsDojo Set 4 Cold — 68%**

Fourth complete TutorialsDojo exam — no pauses, full exam conditions, 90 minutes timer.
68% — two points below the passing threshold. D1 Ingestion is now stable.
D3 Operations is the remaining main blocker. Exam moved to May 31, review day tomorrow.

**TutorialsDojo Set 4 — Cold Results**

| Domain | Score |
|---|---|
| D1 Data Ingestion & Transformation | 77% ✅ |
| D4 Data Security & Governance | 67% ⚠️ |
| D2 Data Store Management | 65% ⚠️ |
| D3 Data Operations & Support | 57% |
| **Total** | **68%** |

**Full TutorialsDojo Progression**

| Date | Set | Score |
|---|---|---|
| May 22 | Set 1 Cold | 51% |
| May 24 | Set 2 Cold | 63% |
| May 27 | Set 3 | 75% ✅ |
| May 29 | Set 4 Cold | 68% |

>**What I understood**
>- D1 Ingestion at 77% is now stable — the Kinesis stack, Glue triggers, and DMS
  nuances that were the main gap two days ago have consolidated after the missed
  questions review
>- D3 Operations at 57% is the clearest remaining blocker — CloudWatch alarms,
  Glue job monitoring, EMR troubleshooting, and Redshift WLM are the specific gaps
>- The 68% under full uninterrupted exam conditions is a more honest signal than
  any previous result — two points from threshold with one targeted review day
  remaining is a realistic and closeable gap

**Decision: Exam on May 31**

68% under real conditions is close but not stable enough to walk in without one more
consolidation pass. A review-only day tomorrow targeting D3, D2, and D4 specifically
is the right call. One extra day of precision review is better than an unnecessary
risk on an exam that is clearly within reach.

Tomorrow: D3 Operations (CloudWatch, Glue Monitoring, EMR), D2 Store (RA3 vs DC2,
DynamoDB partitioning, Aurora vs RDS vs Redshift), D4 Security (KMS key types,
Lake Formation, Glue IAM roles).

---

## May 28, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 18: TutorialsDojo Missed Questions Review**

No new exam today — targeted work on the weakest points only.
TutorialsDojo automatically tracks all questions answered incorrectly across every set.
54 of those flagged questions re-attempted and reviewed in full today.

**Why this review format is the most precise available**

After three complete sets, TutorialsDojo has automatically built a profile of the real
gaps — not a random sample, but the exact questions where the decision pattern is not
yet stable. Reviewing only these 54 questions removes all noise from already-known
material and targets exclusively the concepts that have cost points across multiple sessions.

This is the difference between practicing a lot and practicing correctly.

>**What I understood**
>- Missed Questions mode is more efficient than a full set at this stage —
  every minute goes toward a confirmed gap, not toward questions that are already solid
>- The questions that appear in the missed list across multiple sets are the ones the
  real exam will most likely test in a similar form; locking them in now removes
  the highest-probability error sources before Set 4
>- Tomorrow's Set 4 cold (130 minutes uninterrupted) will be the cleanest readiness
  signal of the entire sprint — no review boost, no warm-up, just stable knowledge

**Full TutorialsDojo Progression**

| Date | Activity | Result |
|---|---|---|
| May 22 | Set 1 Cold | 51% |
| May 22–23 | Set 1 deep review (32 questions) | Error pattern analysis |
| May 24 | Set 2 Cold | 63% |
| May 25 | 71 questions deep review | Framework depth built |
| May 26 | 96 questions mixed | 73% ✅ |
| May 27 | 47 warm-up + Set 3 | 75% ✅ PASSED |
| **May 28** | **54 Missed Questions deep review** | **Gaps targeted directly** |

---

## May 27, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 17: TutorialsDojo Set 3 — 75% PASSED ✅**

Third complete TutorialsDojo exam — 75% on 65 questions. Three domains above 79%.
+24 percentage points across three cold sets in 5 days of active work.

**TutorialsDojo Set 3 — Results**

| Domain | Score |
|---|---|
| D4 Data Security & Governance | 83% ✅ |
| D2 Data Store Management | 82% ✅ |
| D3 Data Operations & Support | 79% ✅ |
| D1 Data Ingestion & Transformation | 64% ⚠️ |
| **Total** | **75% ✅** |

**Full TutorialsDojo Progression**

| Date | Set | Score | Delta |
|---|---|---|---|
| May 22 | Set 1 Cold | 51% | — |
| May 24 | Set 2 Cold | 63% | +12% |
| May 27 | Set 3 | 75% ✅ | +12% |

>**What I understood**
>- D4 Security at 83% and D2 Store at 82% are the clearest proof that deep review
  works — both were the main blockers in Set 1 and are now the two strongest domains
>- D1 Ingestion dropped back to 64% after briefly hitting 73% in Set 2 — it is not
  stably anchored yet; Kinesis Streams vs. Firehose vs. Analytics, Glue trigger logic,
  and DMS nuances need one more targeted review cycle
>- The warm-up session of 47 questions before the exam activated the keyword
  elimination pattern — mental operating temperature matters under exam conditions

**Booking Status**

Booking trigger was: 2 consecutive cold sets above 70%.
Set 2 came in at 63% — below threshold. Set 3 at 75% — passed.
That is not yet two consecutive results above 70%. Set 4 is the decision point:
70%+ on Set 4 with D1 above 65% means the exam gets booked immediately.

---

## May 26, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 16: TutorialsDojo 73% — First Pass Threshold Hit**

96 questions from the TutorialsDojo pool completed today — 73% correct.
First time crossing the 70% TutorialsDojo pass threshold. 51% → 63% → 73% in 5 days.

**TutorialsDojo Full Progression**

| Date | Questions | Result | Type |
|---|---|---|---|
| May 22 | 65 | 51% | Set 1 Cold |
| May 23 | — | Deep Review | Set 1 full explanation review |
| May 24 | 65 | 63% | Set 2 Cold |
| May 25 | 71 | Deep Review | TD question bank full review |
| **May 26** | **96** | **73% ✅** | **Mixed / review-supported** |
| **Cumulative** | **204** | | |

>**What I understood**
>- 73% is meaningful but not yet the booking signal — it is review-supported,
  meaning previous exposure to overlapping questions contributed to the score;
  Set 3 cold (130 minutes, no pause, no prior exposure) is the real readiness test
>- The 51% → 63% → 73% progression in 5 days is a direct result of three things:
  understanding every mistake instead of just marking it, reading TD explanations
  in full for why wrong options fail, and applying keyword elimination before clicking
>- If Set 3 cold holds above 70% with no domain below 65%, the exam gets booked

Two consecutive cold sets above 70%, no domain below 65%.
Set 3 cold is the next test — 130 minutes uninterrupted, no review boost.
That result will determine whether the exam gets booked immediately.

---

## May 25, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 15: TutorialsDojo Deep Review — 71 Questions**

No new cold exam today — 71 questions from the TutorialsDojo DEA-C01 question bank
reviewed in full. Every explanation read completely. No question skipped.

**What today built**

- 71 questions across the full TutorialsDojo DEA-C01 pool (204 total questions)
  re-attempted and reviewed end to end
- Every explanation read in full — not just what is correct, but why every
  other option fails to meet all constraints simultaneously
- Tangible improvement in decision-framework clarity across all four domains

>**What I understood**
>- TutorialsDojo explanations are more valuable than the score — they explain the
  single constraint that makes exactly one answer correct and all others almost-correct;
  that is the precise skill DEA-C01 tests on every question
>- Active error context builds faster retention than passive reading — reviewing a
  question where the wrong choice was made creates a stronger memory trace than
  reading the same material without a concrete mistake attached to it
>- The review-to-cold-transfer proved on Set 2 (51% → 63%); working through the
  broader question bank today should produce the same lift on the next cold attempt

**TutorialsDojo Sprint — Full Progression**

| Date | Activity | Result |
|---|---|---|
| May 22 | Set 1 Cold | 51% |
| May 22 | Set 1 wrong-answer review | 32 questions analyzed |
| May 23 | Set 1 re-attempt + flashcards | Keyword pattern confirmed |
| May 24 | Set 2 Cold | 63% (+12%) |
| May 25 | 71 questions from TD question bank — full review | Framework depth built |

---

## May 24, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 14: TutorialsDojo Set 2 Cold 63%**

Second cold TutorialsDojo exam completed — 63% on 65 fresh questions.
+12 points from Set 1. D1 Ingestion is now stable above threshold.
D2, D3, and D4 are the remaining three domains to close before booking.

**TutorialsDojo Set 2 — Cold Results**

| Domain | Score |
|---|---|
| D1 Data Ingestion & Transformation | 73% ✅ |
| D2 Data Store Management | 59% |
| D3 Data Operations & Support | 57% |
| D4 Data Security & Governance | 58% |
| **Total** | **63%** |

**TutorialsDojo Progression**

| Set | Score | Delta |
|---|---|---|
| Set 1 — Cold | 51% | — |
| Set 2 — Cold | 63% | +12% |

>**What I understood**
>- The +12 point jump is a direct result of the Set 1 deep review and flashcards —
  D1 went from 55% cold to 73% cold; the review-to-next-set transfer is working
>- D2, D3, and D4 all sitting between 57–59% means no single domain is collapsing —
  they are all close to threshold and all need the same precision work, not new topics
>- The 30-minute pause during today's session means the 63% is solid but not yet
  tested under full 130-minute continuous exam pressure; that stamina needs to be
  built before exam day

**Honest Assessment**

63% is not "almost there" and not "a problem" — it is an honest intermediate level.
The last 7 points to the 70% TutorialsDojo pass threshold are not a volume problem.
They are a precision problem: D2, D3, and D4 each have specific sub-topics where its 
crucial to have deep knowledge about the DEA-C01 relevant topics.

DEA-C01 is a 65-question, 130-minute exam with a passing score of 720/1000.
TutorialsDojo is generally considered exam-representative at a 70–72% pass level.
The gap between today's result and exam readiness is measurable and closeable —
Set 2 deep review, flashcards for D2/D3/D4, and one full uninterrupted 130-minute
session are the three concrete next steps.

>**What I understood**
>- Fine tuning is the correct description of where the prep is right now — the
  foundation is solid across all four domains, the variance is in edge cases and
  multi-service decision points that require one more deliberate review pass
>- Continuous 130-minute exam stamina is a separate skill from knowing the content —
  it needs to be practiced explicitly before sitting the real exam

---

## May 23, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 13: TutorialsDojo Set 1 Deep Review**

No new exam today — full deep review instead. All 32 wrong answers from Set 1
re-attempted, fully read with explanations, and the hardest ones converted to flashcards.
Not moving forward until the mistakes are understood.

**Two Concepts Locked In Today**

**EKS + EC2 Nodes — Lowest Latency Storage**
- Question: containers on EKS EC2 nodes process distinct datasets, no sharing between
  containers — which storage has the LOWEST latency?
- Answer: ephemeral RAM-backed volume (emptyDir with `medium: Memory` in Kubernetes)
- Why: RAM storage has zero network hops and no disk I/O — it is physically on the node;
  since no sharing or persistence is required, EBS, EFS, and S3 are all overkill
- Keyword map: `"LOWEST latency"` + `"no sharing"` + `"EKS on EC2"` → ephemeral RAM volume;
  as soon as "sharing" or "persistence" appears, the answer changes

**SageMaker Canvas — Least Operational Overhead for Churn Prediction**
- Question: video streaming company has a dataset with nulls, duplicates, and irrelevant
  data; needs binary classification model with LEAST operational overhead
- Answer: SageMaker Canvas automated data cleansing + built-in binary classification
- Why: Canvas is a no-code ML tool — no infrastructure management, no Glue jobs,
  no custom training pipelines; it handles data prep and model training in one place
- Keyword map: `"LEAST overhead"` + `"data cleaning"` + `"binary classification"` → SageMaker Canvas;
  as soon as "custom model" or "existing pipeline" appears, Canvas is wrong

>**What I understood**
>- Both questions follow the exact same pattern identified yesterday — one keyword in
  the question eliminates every other answer if actively searched for before clicking
>- The keyword-elimination step is not automatic yet — it has to be a deliberate conscious
  check before locking in any answer; today's review confirmed this is the last gap

**Keyword Elimination Pattern — Confirmed Across 4 Questions**

| Question | Decisive Keyword | Eliminates |
|---|---|---|
| EKS Storage | `"LOWEST latency"` + `"no sharing"` | EBS, EFS, S3 — all have network latency |
| SageMaker Churn | `"LEAST operational overhead"` | Glue jobs, custom training, manual pipelines |
| Redshift KMS | `"compliance"` | Default and AWS-managed keys |
| DynamoDB Hot Partition | `"hot partition"` | LSI and GSI — indexes do not fix partition distribution |

>**What I understood**
>- DEA-C01 is a different exam category from CLF-C02 — CLF tests concepts,
  DEA tests architecture decisions under constraints; every question has 2–4 plausible
  answers and the exam systematically penalizes half-knowledge
>- Flashcards for hard questions are the correct next step — multi-service AWS
  combinations are not retained through reading, only through repeated retrieval with context
>- The method is confirmed: deep review before moving to the next set produces more
  durable knowledge than stacking new cold attempts on unreviewed mistakes

---

## May 22, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 12: TutorialsDojo Set 1 Cold 51%**

First complete TutorialsDojo exam — 65 questions cold. Closest source to the real
DEA-C01 difficulty and style. 51% is the honest baseline on the most representative
material available.

**TutorialsDojo Set 1 — Cold Results**

| Domain | Score |
|---|---|
| D3 Data Operations & Support | 71% ✅ |
| D1 Data Ingestion & Transformation | 55% |
| D4 Data Security & Governance | 42% |
| D2 Data Store Management | 35% |
| **Total** | **51%** |

**Exam Source Comparison**

| Source | Score | Exam Proximity |
|---|---|---|
| Nex Arc (best) | 81% | Harder than real exam |
| Nikolai Schuler V1 | 70% cold | Easier than real exam |
| TutorialsDojo Set 1 | 51% cold | Closest to real exam |
| Booking target | 75%+ (twice in a row) | — |

>**What I understood**
>- 51% cold on the most exam-representative source available is a clear and honest
  signal — the foundation is there but precision on AWS-specific decision points is not
>- The pattern across 32 wrong answers is not a knowledge gap — it is a reading
  precision problem; the right area gets identified, then the most similar answer gets
  chosen instead of the most precise one
>- D2 and D4 are the same consistent blockers: DynamoDB partitioning logic, Redshift
  node types, KMS key type distinctions, and Lake Formation governance

**Two Concepts Locked In From Review**

**Redshift KMS Encryption — Compliance always means CMK**
- Chose: default Redshift KMS key with periodic rotation via console
- Correct: Customer-managed KMS key (CMK)
- Why: the default Redshift key is AWS-managed — the customer does not control rotation;
  compliance requirements always require CMK because only CMK puts control in the customer's hands
- Keyword map: `"compliance"` + `"encryption"` → always CMK, never AWS-managed or default key

**DynamoDB Hot Partitions — Index is not the fix**
- Chose: LSI on PRODUCT_ID
- Correct: new DynamoDB table with PRODUCT_ID as partition key
- Why: LSI and GSI provide alternative query paths but use the same physical partitions —
  hot partitions stay hot regardless of indexes; only a higher-cardinality partition key
  distributes load across more physical partitions
- Keyword map: `"hot partition"` → better partition key, not an index

>**What I understood**
>- Both mistakes follow the same pattern: the correct domain was identified but the
  most similar answer was chosen instead of the most precise one — one extra step
  is needed before clicking: *"which keyword in the question eliminates my answer?"*
>- Compliance + encryption is a one-answer scenario on DEA-C01 — CMK every time;
  AWS-managed keys exist for convenience, not compliance control
>- DynamoDB indexes solve query patterns, not partition pressure — this distinction
  is tested in multiple forms across every practice set

---

## May 21, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 11: Nikolai Schuler 70% + TutorialsDojo 57%**

Two new exam sources added to the prep stack today. First contact with TutorialsDojo —
widely considered the closest match to the real DEA-C01 difficulty and question style.

**Nikolai Schuler Course-Internal Exam + TutorialsDojo — Results**

| Source | Score | D1 Ingestion | D2 Store | D3 Ops | D4 Security |
|---|---|---|---|---|---|
| Nikolai Schuler — Cold | 70% | 58% | 86% | 76% | 71% |
| TutorialsDojo — First 23 questions | 57% | — | — | — | — |

**What Each Source Is Worth**

| Source | Difficulty | Exam Proximity | Current Score |
|---|---|---|---|
| Nex Arc | Very Hard | Harder than real exam | 73–81% best |
| Nikolai Schuler (course-internal) | Medium | Easier than real exam | 70% cold |
| TutorialsDojo | Hard | Closest to real exam | 57% Day 1 |
| Real DEA-C01 | Hard | — | — |

>**What I understood**
>- 70% cold on Nikolai Schuler is a positive signal but not a reliable readiness
  indicator — the community consensus is that it is more conceptual and less
  trap-heavy than the actual exam; it confirms foundation, not exam readiness
>- 57% on first TutorialsDojo contact is not a step back — it is the most honest
  calibration available; new question style, new trap patterns, first attempt
>- TutorialsDojo + Nex Arc in parallel is the correct final stack — one mirrors
  real exam style, the other builds depth beyond what the exam requires

**Plan to Exam Day**

Both stacks run in parallel from here — TutorialsDojo for realistic calibration,
Nex Arc for depth. The review loop stays the same: cold attempt, full wrong-answer
review, repeat. At this pace the exam gets booked by end of May.

---

## May 20, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 10: PT5 V1 + PT4 V2**

Two tests today — a cold start on a brand new set and a review attempt on PT4.
D2 Data Store hit 82% cold for the first time. D4 Security remains the consistent gap.

**Nex Arc PT5 V1 — Cold + PT4 V2 Results**

| Test | Score | D1 Ingestion | D2 Store | D3 Ops | D4 Security |
|---|---|---|---|---|---|
| PT5 V1 — Cold | 60% | 59% | 82% | 43% | 50% |
| PT4 V2 — Review | 70% | 73% | 71% | 73% | 64% |

>**What I understood**
>- D2 Data Store at 82% cold is the clearest sign yet that Redshift, DynamoDB,
  and OpenSearch concepts have consolidated — this domain was at 29% cold on Day 4
>- D3 Operations dropping to 43% on a fresh set is a reminder that Nex Arc rotates
  traps across domains — stability on one set does not guarantee stability on another
>- D4 Security at 50% cold and 64% after review is the last consistent blocker;
  Lake Formation, KMS, and S3 Object Lock edge cases are the remaining precision gaps

---

## May 19, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 9: PT4 Cold Start 64%**

One full attempt today — PT4 V1 cold. Strongest cold baseline of the entire sprint.
The review day on May 18 had a direct and measurable impact on today's score.

**Nex Arc PT4 V1 — Cold Results**

| Domain | Score |
|---|---|
| D1 Data Ingestion and Transformation (22 questions) | 68% |
| D2 Data Store Management (17 questions) | 59% |
| D3 Data Operations and Support (15 questions) | 73% |
| D4 Data Security and Governance (11 questions) | 55% |
| **Total** | **64%** |

**Cold-Start Progression — All Practice Tests**

| Test | V1 Cold | Delta vs PT1 |
|---|---|---|
| PT1 | 43% | — |
| PT2 | 41% | -2% |
| PT3 | 47% | +4% |
| PT4 | **64%** | **+21%** |

>**What I understood**
>- The +21% jump from PT1 to PT4 cold is entirely attributable to the May 18 review day —
  systematic wrong-answer review across three test sets transferred directly into
  a stronger cold baseline on brand new questions
>- D2 Data Store Management went from 29–35% cold across PT1–PT3 to 59% cold today —
  that is the review system working exactly as intended
>- D4 Security at 55% is the only domain consistently below the pass line;
  Lake Formation fine-grained access, KMS key policies, and S3 Object Lock modes
  are the three remaining concrete gaps

**Reading Precision — Confirmed as the Highest Lever**

Slowing down and reading every keyword before selecting an answer made a bigger
difference today than any single new topic studied. The cold score improvement from
41% to 64% is not only new knowledge — it is more deliberate reading under exam conditions.

At 64% cold, the pass threshold of 75% is 11 points away — and PT4 V2 after full
wrong-answer review should push into the 78–82% range based on the consistent
V1 → V2 delta across every previous test.

>**What I understood**
>- Precision reading is now the single highest-leverage skill remaining before exam day —
  the knowledge is largely there, the margin is in reading every question completely
  before locking in an answer
>- 64% cold on the hardest available practice material is not a gap — it is a confirmation
  that the exam is ready to be booked; PT4 V2 will confirm the final readiness level

---

## May 18, 2026

**AWS DEA-C01 | PT1 / PT2 / PT3 Review Day**

Today focused on reviewing all wrong answers from the V2 attempts of PT1, PT2,
and PT3. The main takeaway was not a new topic gap, but a pattern of small,
exam-critical details getting overlooked.

**What I learned**

- **Keyword sensitivity:** a single word in the question or answer choice can change
  the correct answer completely
- **Reading speed vs. precision:** rushing through practice exams is costing points;
  slower reading is producing better results than trying to finish fast
- **Decision words:** terms like maximum, least cost, best, most efficient,
  within a partition, across all partitions, incremental, and full crawl
  are often the entire exam signal

>**What I understood**
>- The biggest mistakes right now are not knowledge gaps — they are careless reading
  errors caused by moving too quickly through the question
>- DEA-C01 is deliberately designed around small wording differences, so the real skill
  is not just knowing AWS services but spotting the exact phrase that changes the answer
>- Slowing down is currently more productive than pushing for speed;
  precision is the highest-leverage improvement for the next practice exams

**Current focus**

- Read every question twice!
- Check every answer option for the one decisive word
- Do not jump to the first plausible answer under time pressure
- Prioritize precision over speed

---

## May 17, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 7: Nex Arc PT3 47% → 73%**

Two full attempts on Nex Arc PT3 today — completely fresh questions again.
Cold baseline at 47% (already above PT1 and PT2 cold starts), closed at 73% after review.
The review loop is compressing faster with every new test set.

**Nex Arc PT3 — Full Progression**

| Attempt | Score | D1 Ingestion | D2 Store | D3 Ops | D4 Security |
|---|---|---|---|---|---|
| V1 — Cold | 47% | 64% | 35% | 47% | 36% |
| V2 — After Review | 73% | 77% | 65% | 87% | 64% |

**Delta V1 → V2**

| Domain | V1 | V2 | Delta |
|---|---|---|---|
| D1 Ingestion | 64% | 77% | +13% |
| D2 Store | 35% | 65% | +30% |
| D3 Ops | 47% | 87% | +40% |
| D4 Security | 36% | 64% | +28% |

>**What I understood**
>- PT3 cold start at 47% is already above PT1 and PT2 (both 41% cold) — knowledge
  from previous sets is transferring across test sessions, not just within them
>- D3 Operations & Support from 47% to 87% in one day confirms the review loop works
  even on domains that felt unstable — the pattern is consistent across every session
>- D2 and D4 are the only remaining blockers before the 75% passing threshold;
  both need one more targeted review session before PT3 V3 tomorrow

**Concept Locked In — DynamoDB GSI vs. LSI**

Scenario from PT3: gaming company stores session data with SessionID as Partition Key
and Timestamp as Sort Key. Only 2% of sessions are ACTIVE. How to query active sessions
efficiently with minimal cost?

- **Wrong answer:** LSI with SessionID as Partition Key and Status as Sort Key
- **Why wrong:** an LSI uses the same Partition Key as the base table — querying by
  Status across all SessionIDs is impossible because you would need to know the
  SessionID first; LSI only enables alternative sort key patterns *within* a partition
- **Correct answer:** GSI with Status as Partition Key — enables a direct table-wide
  query for Status = 'ACTIVE' without knowing SessionID; since only 2% of items have
  Status = 'ACTIVE', the index is small and cost-efficient (Sparse Index principle)

Keyword map:
- `"query across all partitions"` → always **GSI**
- `"alternative sort key within a partition"` → **LSI**
- `"sparse index on low-cardinality attribute"` → **GSI** (cost-efficient when few items carry the attribute)

>**What I understood**
>- The GSI vs. LSI trap is one of the most common wrong answers on DEA-C01 —
  the key is whether the query needs to cross partition boundaries; if yes, LSI cannot help
>- Sparse Index is a cost optimization pattern, not just a data model concept —
  when only a small percentage of items carry an attribute, a GSI on that attribute
  stays small and cheap; this is the correct framing for the exam scenario
>- This concept maps directly to Feature Store design: sparse feature attributes
  should use selective indexes, not full-table scans

**Full Sprint Progression — All Practice Tests**

| Test | V1 Cold | V2 | V3 | Best |
|---|---|---|---|---|
| PT1 | 43% | 64% | 81% | **81%** |
| PT2 | 41% | 73% | — | 73% |
| PT3 | 47% | 73% | — | **73%** |

The plateau at 73% across PT2 V2 and PT3 V2 is a clear signal — D2 Store Management
and D4 Security are the two remaining gaps before consistent 80%+ on fresh material.

---

## May 16, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 6: Nex Arc PT2 41% → 73%**

Two full attempts on Nex Arc PT2 today — completely new questions, no overlap with PT1.
Cold start at 41%, closed the day at 73% after the review loop. +32 percentage points
in one day, consistent with every previous session in this sprint.

**Nex Arc PT2 — Full Progression**

| Attempt | Score | D1 Ingestion | D2 Store | D3 Ops | D4 Security |
|---|---|---|---|---|---|
| V1 — Cold | 41% | 55% | 29% | 27% | 55% |
| V2 — After Review | 73% | 95% | 65% | 60% | 64% |

**Delta V1 → V2**

| Domain | V1 | V2 | Delta |
|---|---|---|---|
| D1 Ingestion | 55% | 95% | +40% |
| D2 Store | 29% | 65% | +36% |
| D3 Ops | 27% | 60% | +33% |
| D4 Security | 55% | 64% | +9% |

>**What I understood**
>- PT2 is a completely different question set from PT1 — knowledge does not transfer
  automatically between sets; it has to be locked in domain by domain through review
>- D3 Operations & Support at 27% in V1 was the new weak point — different from PT1
  where D3 was the most stable domain; Nex Arc deliberately rotates the hard questions
  across domains to prevent pattern-matching without real understanding
>- D1 Ingestion hitting 95% in V2 confirms that Kinesis, Glue ETL, and DMS are now
  fully solid — the same domain that caused the most variance in the early sprint days

**Concept Locked In — CloudTrail Log File Integrity Validation**

Scenario from PT2: a financial company must cryptographically verify that CloudTrail
logs stored in S3 have not been tampered with.

- **Solution:** enable Log File Integrity Validation on the trail
- **Mechanism:** SHA-256 hashing + SHA-256 with RSA digital signature on every log file
- AWS delivers hourly **Digest Files** to the S3 bucket — each digest covers all
  log files delivered in that hour
- **Verification:** `aws cloudtrail validate-logs --trail-arn <arn> --start-time <time>`
- Detects deletion, modification, and forgery of log files

Keyword map:
- `"cryptographically verify logs"` → CloudTrail log file integrity validation
- `"tamper-evident logs"` → CloudTrail log file integrity validation
- `"regulatory audit log files"` → `validate-logs` CLI command

>**What I understood**
>- Log File Integrity Validation is a D4 scenario question that appears in multiple
  forms — the keyword triggers are always around cryptographic proof and tamper detection
>- The digest file mechanism is what makes this different from simply storing logs in S3 —
  S3 alone does not prove the logs were not modified; the digest + signature chain does
>- This concept maps directly to data integrity requirements in production ML pipelines —
  audit trails for feature data transformations follow the same tamper-evidence principle

---

## May 15, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 5: Nex Arc 41% → 81% in One Day**

Three attempts on Nex Arc PT1 today with full wrong-answer review between each session.
+40 percentage points in a single day on material harder than the real exam.

**Nex Arc PT1 — Full Progression**

| Attempt | Score | D1 Ingestion | D2 Store | D3 Ops | D4 Security |
|---|---|---|---|---|---|
| V1 (May 14) | 43% | 59% | 29% | 40% | 27% |
| V2 (May 15) | 64% | 77% | 41% | 73% | 64% |
| V3 (May 15) | 81% | 95% | 71% | 73% | 82% |

>**What I understood**
>- D4 Security went from 27% to 82% in two review sessions — the review loop
  converts wrong answers into locked knowledge faster than any passive study method
>- D1 Ingestion hit 95% by V3 — Kinesis, Glue Streaming ETL, and MSK decision logic
  are now fully internalized after being the main gap on Day 1
>- D2 Data Store at 71% on Nex Arc translates to well above passing on the real exam —
  it is the last remaining variable but no longer a blocker

**Key Concepts Locked In Today**

- **OpenSearch Serverless collection types:** search = full-text, millisecond response;
  time-series = log analytics, metrics, time-based data; vector-search = ML embeddings,
  semantic similarity, RAG pipelines — got this wrong in V1, will never miss it again
- **Lake Formation fine-grained access:** column-level security via LF-Tags applied
  at the table level; row-level filtering through data filters — operates at the catalog
  layer, not at S3 storage; entirely different from bucket policies
- **DynamoDB Streams vs. Kinesis Data Streams for CDC:** DynamoDB Streams = 24h
  retention, native integration, free; Kinesis = up to 365 days, more consumers,
  costs extra — choose Kinesis when downstream consumers need extended replay or fan-out
- **Step Functions error handling:** Retry = same state, exponential backoff with
  MaxAttempts and IntervalSeconds; Catch = different fallback state when all retries
  exhausted; ResultPath preserves error info — never confuse the two
- **Redshift WLM:** Automatic WLM lets Redshift allocate memory dynamically;
  Manual WLM uses fixed queues and concurrency slots; Short Query Acceleration (SQA)
  separates short queries from long-running ones to prevent queue blocking

>**What I understood**
>- Every concept above was a wrong answer in V1 — the review loop does not just
  improve the score on the next attempt, it builds the exact knowledge the real exam tests
>- OpenSearch collection types and Lake Formation access control are the two most
  commonly missed DEA-C01 topics; locking them in now means zero hesitation on exam day
>- Redshift WLM is a scenario question on almost every DEA-C01 practice set —
  the distinction between Automatic, Manual, and SQA is now fully clear

**Score Translation — Nex Arc vs. Real Exam**

81% on Nex Arc after full review loop. 88% on Neal Davis. Real exam passing score: 72%.
Nex Arc is harder by design — people scoring 65–75% on Nex Arc routinely pass the real
DEA-C01 comfortably. At 81% on the hardest available material, exam readiness is high.

>**What I understood**
>- The review loop is the method: take cold, accept the score, review every wrong answer,
  repeat — it compounds fast and it works on any difficulty level of material

---

## May 14, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 4: The Real Challenge**

First Nex Arc practice exam completed. 43% on the surface — best learning signal
of the entire sprint underneath. Nex Arc is intentionally harder than the real DEA-C01
and the gaps it exposes are exactly what L5-level preparation requires.

**Nex Arc PT1 Results**

| Domain | Score | Questions |
|---|---|---|
| D1 Data Ingestion & Transformation | 59% | 22 |
| D2 Data Store Management | 29% | 17 |
| D3 Data Operations & Support | 40% | 15 |
| D4 Data Security & Governance | 27% | 11 |

**Overall: 43%** — on the hardest available DEA-C01 prep material.
Multiple people report 40–50% on Nex Arc then passing the real exam comfortably.

>**What I understood**
>- 43% on Nex Arc is not a score — it is a precise gap map; every wrong answer is
  a concept the real exam will test at a shallower level, which means understanding
  it deeply removes any hesitation on exam day
>- D2 and D4 are the priority targets: OpenSearch Serverless collection types,
  Redshift Spectrum vs. Athena decision logic, Lake Formation fine-grained access,
  and KMS key policies are the concrete gaps
>- The OpenSearch collection type question (search vs. time-series vs. vector-search)
  is a perfect example of Nex Arc's depth — got it wrong once, will never get it wrong again

**Practice Provider Comparison — Final Assessment**

| Provider | Questions | Difficulty | Best Use |
|---|---|---|---|
| Neal Davis | 25/test, 6 sets | Moderate | Early sprint, daily gap-finding |
| Maarek | 65/test, 4 sets | Exam-realistic | Full-length simulation before booking |
| Nex Arc | 65/test, 6 sets | Harder than real exam | Deep knowledge building, L5-level prep |

>**What I understood**
>- Neal Davis builds momentum, Maarek measures readiness, Nex Arc builds depth —
  all three serve a different purpose in the sprint
>- This is not about passing the DEA-C01 at minimum score — the target is to think like a L5 Data
  and Feature Infrastructure Engineer; Nex Arc aligns with that ambition better
  than any other available prep material

---

## May 13, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 3: 88% Stable**

Three tests today — all above 84%. Last day of Neal Davis practice sets.
Tomorrow: Maarek practice exams (65 questions, closest format to the real exam).

**Practice Test Results — May 13**

| Test | Score | D1 Ingestion | D2 Store | D3 Ops | D4 Security |
|---|---|---|---|---|---|
| PT 4 — Attempt 2 | 88% ✅ | 100% | 86% | 83% | 83% |
| PT 5 — Attempt 2 | 84% ✅ | 100% | 100% | 83% | 50% |
| PT 6 — Attempt 1 | 88% ✅ | 67% | 100% | 100% | 100% |

**Full Sprint Progression — May 11–13**

| Day | Tests | Score Range | Highlight |
|---|---|---|---|
| Day 1 — May 11 | 4 tests | 48% → 56% | Cold start, first orientation |
| Day 2 — May 12 | 4 tests | 64% → 72% | Passing score hit for the first time |
| Day 3 — May 13 | 3 tests | 84% → 88% | Stable above 84%, three times in a row |

48% → 88% in 3 days. Identical pattern to CLF-C02 (42% → 88%).

>**What I understood**
>- Domain 2 was the most persistent blocker of the entire sprint — today it hit 100%
  twice; Redshift, S3 Storage Classes, DynamoDB, and Glue Catalog are now solid
>- Domain 3 and Domain 4 each hit 100% once today — CloudWatch, IAM Policies,
  and Lake Formation are reproducible at a high level
>- Domain 1 remains the only variable: at larger sample sizes (9 questions) it dropped
  to 67% — Kinesis Sharding, Glue Streaming ETL, and MSK vs. Kinesis decision logic
  need one targeted review before Maarek PT1 tomorrow

**Neal Davis vs. Maarek | What Changes Tomorrow**

Neal Davis runs 25 questions per test across 6 sets — useful for daily gap-finding
but statistically limited; a domain with 2 questions can read 0% or 100% without
saying much. Maarek runs 65 questions per test across 4 sets — that is the actual
exam length, and the first real picture of exam readiness.

The Neal Davis scores are genuine, but Maarek is the honest benchmark.
Tomorrow is the first real readiness check.

>**What I understood**
>- Small sample sizes in domain scoring are noise — the 67% in D1 today with 3 questions
  is less meaningful than a 67% across 15 questions; Maarek removes that ambiguity
>- The sprint method is confirmed for the second time: same feedback loop, same progression
  curve as CLF-C02 — the only difference is the DEA-C01 took 3 days to reach 88%
  where CLF-C02 took 11; the material is harder, the method is faster

---

## May 12, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 2: Passing Score Hit**

2 cold tests today and 2 second trys, 8 tests across 2 days. First cold test hit 72%! —
the official DEA-C01 passing score.
That is 1.5 days from a 48% cold start to passing-level performance.

**Practice Test Results — May 12**

| Test | Score | D1 Ingestion | D2 Store | D3 Ops | D4 Security |
|---|---|---|---|---|---|
| PT 2 — Attempt 2 | 68% | 83% | 88% | 40% | 50% |
| PT 3 — Attempt 2 | 64% | 60% | 45% | 86% | 100% |
| PT 4 — Attempt 1 | **72% ✅** | 67% | 71% | 67% | 83% |
| PT 5 — Attempt 1 | 68% | — * | 64% | 100% | 67% |

*D1 had only 2 questions in PT5, i had both false.

>**What I understood**
>- 72% on PT4 is not a warm-up score — it is a fresh set, first attempt, no prior
  exposure to those questions; passing-level performance confirmed in 1.5 days
>- Domain 3 went from 40% to 86% and 100% in one day — targeted gap study after
  a practice test converts directly and immediately into correct answers;
  CloudWatch, Glue Job Monitoring, and Step Functions Error Handling are now solid
>- Domain 4 hit 100% once today — IAM, KMS, and Lake Formation clicked after
  deliberate review yesterday evening; the method works exactly as expected

**Why This Exam Is Actually Enjoyable**

The DEA-C01 is genuinely interesting in a way CLF-C02 was not. Every question is a real
data engineering decision. These are not abstract
AWS service facts. They are the concepts that will be applied directly on this roadmap
when building Feature Stores and streaming pipelines. Learning them under exam pressure
makes them stick faster than any course could.

The prediction: this exam gets passed significantly faster than CLF-C02 did —
and CLF-C02 only took 11 days from zero to certified.

>**What I understood**
>- Depth of interest accelerates learning — when the material connects directly to what
  comes next on the roadmap, every correct answer feels like progress on two fronts at once
>- The practice-first method is confirmed again: 48% → 72% in 1.5 days with no linear
  course consumption; the same loop that worked for CLF-C02 is working faster here
>- Passing the DEA-C01 without 2 to 3 years of Data Engineering experience is the point —
  the right method compresses that timeline; this is the second proof after CLF-C02

---

## May 11, 2026

**AWS DEA-C01 | Practice Exam Sprint — Day 1**

First real DEA-C01 practice exam day. 3 cold full tests back to back!
No prior exam-format preparation. Same method as CLF-C02: practice first, analyze errors,
study gaps. That approach took CLF-C02 from 42% to 88% in 11 days.

**Practice Test Results**

| Test | Score | Domain 1 | Domain 2 | Domain 3 | Domain 4 |
|---|---|---|---|---|---|
| PT 1 | 48% | 40% | 60% | 50% | 33% |
| PT 2 | 56% | 67% | 75% | 40% | 33% |
| PT 3 | 48% | 40% | 45% | 57% | 50% |
| PT 1 V2 | 64% | 80% | 40% | 75% | 83% |

>**What I understood**
>- Three cold tests in one day gives the most honest baseline possible — no score
  inflation from familiarity, just raw knowledge mapped against the actual exam domains
>- Domain 4 (Security & Governance) is the consistent weak point across all three tests —
  IAM Policies, KMS, Lake Formation, Macie, and CloudTrail need targeted study
>- Domain 3 (Data Operations & Support) is the most stable domain — monitoring and
  troubleshooting questions land consistently across all three tests

**DEA-C01 vs. CLF-C02 — First Impressions**

The DEA-C01 is a different class of exam. Questions are longer, scenario-based, and
frequently present multiple technically correct answers that must be weighed against
each other. There is no obviously wrong option to eliminate — these are real data
engineering decisions in concrete AWS scenarios.

What helped immediately: the CLF-C02 foundation means S3, IAM, Kinesis, Glue, and
Redshift are not unknown — the baseline is already there. SQL-based questions were
clearly the strongest area across all three tests: JOINs, aggregations, Window Functions,
GROUP BY, PARTITION BY — all handled confidently. PostgreSQL study from Month 1
is paying off directly right now.

>**What I understood**
>- Practically oriented questions sit better than purely theoretical ones 
>- The DEA-C01 does not test service recognition the way CLF-C02 did —
  it tests the ability to choose the right architecture for a specific engineering constraint
>- SQL fluency is a direct advantage on this exam — every hour spent on LeetCode SQL 50
  and PostgreSQL is converting into correct answers today

**Gap Analysis — 3 Tests**

- **Domain 4 — Security & Governance (33–50%):** IAM Policies, KMS encryption,
  Lake Formation permissions, S3 Bucket Policies, Macie, CloudTrail — Priority #1
- **Domain 1 — Ingestion & Transformation (40–67%):** Glue, Kinesis, DMS,
  Step Functions need more depth — high variance across tests
- **Domain 2 — Data Store Management (45–75%):** S3 Storage Classes and
  Redshift concepts still inconsistent depending on question framing
- **Domain 3 — Data Operations & Support (40–57%):** most stable domain —
  monitoring and troubleshooting questions are the strongest area

**Mindset | Course vs. Practice-First**

Paused the Maarek video course deliberately today. Consuming hours of video content
passively is not the most efficient way to learn — being confronted with real exam
questions, failing, and understanding why is. The feedback loop from a practice test
is faster and more honest than any lecture.

The DEA-C01 officially assumes 2 to 3 years of Data Engineering experience to pass.
That is the expectation. The goal here is to prove the opposite — that the right
learning method, applied with enough intensity, beats years of passive experience.
CLF-C02 in 11 days was the first proof. DEA-C01 is next.

>**What I understood**
>- Video courses are a reference, not a method — they work best when used to fill
  specific gaps identified by practice tests, not as a linear watch-through
>- The DEA-C01 experience requirement exists because most people learn slowly and
  passively; deliberate practice with immediate feedback compresses that timeline significantly
>- This is the same approach that worked for CLF-C02 — there is no reason to change
  a method that already delivered results

---

## May 10, 2026

**AWS DEA-C01 | Month 3 Started — Data Engineering Fundamentals + S3 Storage**

Month 3 of the L5 Roadmap begins! 
Sections 1 and 2 of the AWS DEA-C01 course completed in full, Section 3 (Storage) started
and worked through to S3 Event Notifications.

**AWS DEA-C01 | Section 2 — Data Engineering Fundamentals**

- **Types of Data:** Structured (tables, schemas), Semi-Structured (JSON, XML),
  Unstructured (images, logs, free text) — each requires a different storage and
  processing strategy
- **The 3 Vs:** Volume (how much), Velocity (how fast), Variety (what shape) —
  the three dimensions that define every data engineering problem
- **Data Warehouse vs. Data Lake vs. Lakehouse:** Warehouse = structured, query-optimized;
  Lake = raw, schema-on-read, cheap storage; Lakehouse = combines both — structured
  query performance on top of raw lake storage
- **Data Mesh:** decentralized data ownership at scale — domain teams own and serve
  their own data products instead of a central team owning everything
- **ETL Pipeline Orchestration:** managing dependencies, scheduling, and failure
  recovery across multi-step data pipelines — foundational for Airflow and Spark
- **Schema Evolution:** how pipelines handle changes to data structure over time
  without breaking downstream consumers — critical for long-running ML feature pipelines
- **Data Skew:** imbalanced data distribution across partitions in distributed systems —
  directly relevant to Spark optimization later in Month 3
- **SQL Review:** aggregations, GROUP BY, sorting, pivoting, all JOIN types,
  and intro to Regular Expressions — reinforced across multiple lectures and Quiz 1 ✅

>**What I understood**
>- The Lakehouse pattern is the architecture that Feature Stores are built on —
  raw feature data lives in a lake, served features are query-optimized like a warehouse;
  understanding this distinction now makes Feature Store design intuitive later
>- Schema Evolution and Data Skew are not theoretical topics — they are the two most
  common sources of silent failures in production ML pipelines at scale
>- Data Mesh directly maps to how Netflix organizes data ownership — domain teams
  owning their feature pipelines is the L5 organizational pattern, not a central data team

**AWS DEA-C01 | Section 3 — Storage (S3)**

- **Amazon S3 fundamentals:** object storage with buckets and keys — the backbone
  of every ML data storage pattern on AWS; everything in the stack eventually touches S3
- **S3 Security:** Bucket Policies control access at the bucket and object level —
  the primary mechanism for cross-account and public access management
- **S3 Versioning:** every object write creates a new version — delete markers preserve
  history; maps directly to feature data immutability concepts from Month 2
- **S3 Replication:** Cross-Region Replication (CRR) for disaster recovery and
  compliance; Same-Region Replication (SRR) for log aggregation and environment sync
- **S3 Storage Classes:** Standard for frequent access; Intelligent-Tiering for
  unpredictable access patterns; Glacier for archival — choosing the right class
  is direct cost optimization for large feature datasets
- **S3 Lifecycle Rules:** automatically transition objects between storage classes
  or expire them after a defined period — essential for managing feature dataset costs at scale
- **S3 Express One Zone:** single-AZ, ultra-low latency storage class for
  performance-sensitive workloads — trades availability for speed
- **S3 Event Notifications:** trigger Lambda, SQS, or SNS on object-level events
  (PUT, DELETE) — the entry point for event-driven pipeline architectures on AWS

>**What I understood**
>- S3 Versioning is feature immutability at the infrastructure level — every write
  creates a new version, nothing is ever silently overwritten; this is the storage
  primitive that makes reproducible ML pipelines possible
>- S3 Event Notifications + Lambda is the serverless trigger pattern for data pipelines —
  a file lands in S3 and the entire downstream processing chain starts automatically
>- Storage class selection is not a detail — for petabyte-scale feature datasets,
  the difference between Standard and Intelligent-Tiering is a significant cost lever

---

## May 9, 2026

**Python | Functional Programming + Pure Functions + Jikan Refactor + ML Platform Concept**

Two Python concepts locked in today! Functional programming patterns and pure functions,
both applied directly to the Jikan pipeline, plus the first real ML Platform concept:
feature immutability.

**Python | Functional Programming — map(), filter(), reduce()**

- **`map()`:** applies a function to every element of an iterable — replaces a manual
  `for` loop with a single declarative expression
- **`filter()`:** selects elements matching a condition — replaces `for` + `if` + `.append()`
  with a clean one-liner
- **`reduce()`:** from `functools` — accumulates an iterable down to a single value;
  useful for aggregations like total episode count across a dataset
- **Imperative vs. Functional:** imperative code describes *how* to do something
  (loop, append, mutate); functional code describes *what* to produce — less state,
  less room for bugs

>**What I understood**
>- `map()` + `filter()` together replace the most common data transformation pattern
  in pipelines — iterate, validate, transform — in a single composable line
>- `reduce()` is less common in day-to-day pipeline code but explains the mental model
  behind aggregations that collapse a collection into one value
>- Functional style does not mean lambdas everywhere — complex branching logic belongs
  in named functions; `map()` and `filter()` just call them more cleanly

**Python | Pure Functions**

- A Pure Function follows exactly two rules:
  1. Same input → always same output (deterministic, no randomness, no external dependency)
  2. No side effects — nothing outside the function is changed (no global mutation,
     no file writes, no API calls, no `print()`)
- `build_feature_record()` and `validate_anime()` in the Jikan pipeline are already
  Pure Functions — same anime dict in, always same feature record out, nothing mutated outside
- Pure Functions are trivial to unit test because the output is always predictable —
  no mocks, no setup, just input → assert output
- Netflix requires this explicitly: ML pipelines must guarantee that the same input data
  always produces the same features — determinism at scale is non-negotiable

>**What I understood**
>- Purity is not an abstract concept — it is what makes a function safe to run in parallel,
  safe to cache, and safe to test without infrastructure
>- The Jikan pipeline functions were already pure without explicitly naming them as such —
  understanding the principle now means it gets applied consciously going forward
>- Side effects do not disappear — they get pushed to the edges of the pipeline
  (API fetch, file write) so the transformation core stays pure and predictable

**Python | Jikan Feature Pipeline — Functional Refactor**

Refactored the validation and feature engineering loops in the Jikan pipeline
from imperative to functional style:

*Before:*
```python
validated_anime = []
for anime in anime_list:
    cleaned = validate_anime(anime)
    if cleaned is not None:
        validated_anime.append(cleaned)

feature_records = []
for anime in validated_anime:
    feature_records.append(build_feature_record(anime))
```

*After:*
```python
validated_anime = list(filter(lambda x: x is not None, map(validate_anime, anime_list)))
feature_records = list(map(build_feature_record, validated_anime))
```

>**What I understood**
>- The refactor does not change what the pipeline does — it changes how it reads;
  declarative code communicates intent immediately without tracing loop state
>- `validate_anime()` and `build_feature_record()` stay as named functions —
  they contain branching logic that does not belong in a lambda
>- This is exactly the kind of incremental improvement that separates a script
  from production-quality pipeline code

**ML Platform Concept | Immutable Features**

- The pipeline has two distinct phases:
  - **Phase 1 — Mutable:** raw data → cleaning → validation → feature engineering
  - **Phase 2 — Immutable:** validated features are written to storage and never overwritten
- Immutability enables full reproducibility — a model trained on Feature Version 42
  can always be debugged with the exact same data, even months later
- This is a core principle of production ML Platform engineering at L5 —
  without immutable features, model debugging and experiment tracking become unreliable

>**What I understood**
>- The boundary between mutable and immutable is the moment features enter the Feature Store —
  everything before that point can be changed; everything after is frozen by design
>- Immutability is not a constraint, it is a guarantee — it is what makes
  large-scale ML systems reproducible and trustworthy at any point in time
>- The Jikan pipeline output is already immutable by design via timestamped filenames —
  understanding the principle now makes that an intentional architectural decision

---

## May 8, 2026

**Docker | Sections 3–6 completed — Volumes, Networking, Multi-Container & Compose**

Four sections completed in a single day!!
Volumes, networking, multi-container architecture, and Docker Compose are all done.
Applied Docker Compose immediately to the Jikan pipeline.

**Section 3 — Volumes & Bind Mounts**

- **Bind Mounts:** mount a local folder directly into a container (`-v $(pwd):/app`) —
  code changes reflect instantly without rebuilding the image; development standard
- **Read-Only Volumes:** `:ro` flag prevents the container from writing back to the
  mounted folder — used when the container should only read, never modify
- **COPY vs. Bind Mount:** COPY bakes files into the image at build time for production;
  Bind Mount shares live files from the host at runtime for development
- **Environment Variables & ARG:** `ENV` sets runtime variables; `ARG` is build-time only —
  both can be injected via `--env-file .env` or `--build-arg` without hardcoding values
- **.dockerignore:** prevents `node_modules`, `.git`, and other noise from being
  copied into the image — as essential as `.gitignore` is for Git

>**What I understood**
>- The mental model is simple: Bind Mount = live connection to host; Named Volume =
  persistent storage managed by Docker; Anonymous Volume = temporary, never rely on it
>- `.dockerignore` is not optional in real projects — copying `node_modules` into an
  image bloats it and overrides the clean dependency install from `RUN npm install`
>- ARG vs. ENV is a build-time vs. runtime distinction — ARG values do not exist
  inside the running container, only during `docker build`

**Section 4 — Networking & Container Communication**

- **Three communication cases:** Container → WWW works out of the box;
  Container → Host requires `host.docker.internal` instead of `localhost`;
  Container → Container requires a Docker Network
- **Docker Networks:** containers on the same network communicate using container
  names as hostnames — Docker resolves them automatically, no IP management needed
- **Network Drivers:** `bridge` is the default for single-host setups; `host` removes
  network isolation; `overlay` connects containers across multiple hosts

>**What I understood**
>- `host.docker.internal` is the fix for every "connection refused" error when a
  containerized app tries to reach a service running on the host machine
>- Container names as DNS hostnames inside a network is one of Docker's most
  practically useful features — it makes service discovery in multi-container
  setups completely configuration-free

**Section 5 — Multi-Container Apps**

- Separated MongoDB, Node API, and React frontend into three independent containers —
  each with a single responsibility, each replaceable without touching the others
- Named Volume for MongoDB ensures data persists across container restarts
- Bind Mounts on Node and React containers enable live code updates during development
- All three containers connected via a shared Docker Network — communication by name

>**What I understood**
>- Single Responsibility per container is not just good practice — it is what makes
  the entire architecture testable, scalable, and replaceable piece by piece
>- A multi-container setup without a shared network is just isolated processes;
  the network is what makes them a system

**Section 6 — Docker Compose**

- **Docker Compose** orchestrates multiple containers from a single `docker-compose.yml` —
  one file replaces every long `docker run` command with flags
- `docker compose up` builds images if needed and starts all services;
  `docker compose down` stops and removes containers and networks cleanly
- Compose automatically creates a shared network for all defined services —
  container names become DNS hostnames within that network by default

>**What I understood**
>- Docker Compose is the production-ready way to manage multi-container setups —
  everything that was done manually with `docker run` flags is now declarative and
  version-controlled in a single file
>- `docker compose up --build` is the correct habit during development — it ensures
  the image always reflects the latest code, not a cached layer from hours ago

**Project | Jikan Pipeline — Docker Compose Applied**

Converted the Jikan Feature Pipeline from a manual `docker run` command to a
full Docker Compose setup:

```yaml
version: "3.8"
services:
  jikan-pipeline:
    build: .
    volumes:
      - ./output:/app/output
```

```bash
docker compose up
```

One command replaces the full `docker run -v ...` string — cleaner, reproducible,
and ready to extend with additional services when the pipeline grows.

---

## May 7, 2026

**Docker | Sections 2 & 3 + Jikan Feature Pipeline Containerized**

First day of Docker. Completed Section 2 fully and worked
through Section 3 up to Lecture 50, then immediately applied everything by containerizing
the Jikan Feature Pipeline built in Month 1.

**Section 2 — Docker Images & Containers**

- **Images vs. Containers:** images are read-only blueprints; containers are running
  instances — you never modify an image, you run a container from it
- **Dockerfile basics:** `FROM`, `WORKDIR`, `COPY`, `RUN`, `CMD` — the full build
  sequence to go from source code to a runnable image
- **Image Layers & Caching:** each Dockerfile instruction creates a layer — Docker
  caches unchanged layers on rebuild; layer order directly impacts build speed
- **Container lifecycle:** pull, run, stop, restart, delete — plus attached vs.
  detached mode (`-d`) and interactive mode (`-it`) for debugging
- **Naming & Tagging:** `--name` for containers, `-t name:tag` for images —
  essential for managing multiple versions and pushing to DockerHub
- **DockerHub:** `docker push` and `docker pull` — images are portable artifacts
  that run identically on any machine with Docker installed

>**What I understood**
>- Layer caching is the first real performance concept in Docker — putting `COPY requirements.txt`
  before `COPY . .` means dependency installation only reruns when requirements change,
  not on every code change
>- The difference between attached and detached mode matters immediately in practice —
  detached is the production default, attached is for debugging
>- DockerHub is to Docker images what GitHub is to code — same mental model, same workflow

**Section 3 — Managing Data & Volumes**

- **Data categories:** Application Data (baked into the image), Temporary Data
  (in-memory during runtime), Persistent Data (must survive container stop)
- **The core problem:** by default all container data is lost when the container stops —
  volumes are the solution
- **Anonymous vs. Named Volumes:** anonymous volumes are managed by Docker and not
  reusable; named volumes (`-v name:/app/path`) persist across container runs
  and are the standard approach for persistent data

>**What I understood**
>- Every container is ephemeral by default — this is a feature, not a bug;
  volumes are the explicit decision to make specific data persistent
>- Named volumes are the correct default for any data that needs to outlive a container —
  anonymous volumes exist but should not be relied on for anything important

**Project | Jikan Feature Pipeline — Containerized**

Took the Jikan REST API pipeline from Month 1 and fully containerized it today —
applying every concept from Sections 2 & 3 immediately in a real project.

- **Dockerfile:** Python 3.11-slim base, `requirements.txt` install, pipeline execution via `CMD`
- **Pipeline:** fetches Top Anime from the Jikan API, validates all records with
  None-protection, applies feature engineering — score flags, popularity buckets, genre metrics
- **Timestamped output:** every `docker run` produces a new JSON file
  (`feature_records_20260507_140022.json`) — no overwrites, full run history
- **Volume mount:** output lands directly in the local `output/` folder via `-v`

```bash
docker build -t jikan-feature-pipeline .
docker run -v "/Users/nicoostermann/Documents/Professional Projekte/jikan-feature-pipeline/output":/app/output jikan-feature-pipeline
```

>**What I understood**
>- `requirements.txt` contains only external libraries (`requests`) — `json`, `pathlib`,
  and `datetime` are Python built-ins and do not belong there
>- Colons in folder names break `$(pwd)` in Docker volume mounts — absolute path
  is the reliable workaround when `$(pwd)` fails
>- Building and running a real pipeline in Docker on Day 1 of the Docker section
  is exactly the right pace — theory without immediate application does not stick

**GitHub**

Pipeline pushed to `github.com/oster-dev/projects` under `jikan-feature-pipeline/`.
README updated with full Docker documentation. `.gitignore` configured for
`output/`, `.venv/`, and `__pycache__/`.

---

## May 6, 2026

**AWS | CLF-C02 — PASSED ✅ Certified Cloud Practitioner**

Unexpected end to the sprint... in the best possible way!!
After 11 days from course start to exam day, the AWS Certified Cloud Practitioner
CLF-C02 exam was passed today at 08:56pm, completed in 40 minutes.

**The Final Day**

After days of Maarek practice simulations, today brought a different move:
discovered **[EXAM REVIEWER] AWS Certified Cloud Practitioner CLF-C02 by Neal Davis on Udemy**
— a focused, clean review of all CLF-C02 topics that covered the full scope in exactly
the right format for a final pass before exam day, spontainiously booked the exam today. Full shoutout to Neal Davis —
it was the perfect capstone review resource.

The decision to stop simulating and just sit the exam came from a simple feeling:
the material was solid, the patterns were locked in, and another practice test
would not have added anything. That instinct was right.

>**What I understood**
>- Knowing when to stop preparing and just go is a skill — more repetition past a
  certain point adds anxiety, not knowledge; today was the right moment to pull the trigger
>- The CLF-C02 is broad by design — it maps the entire AWS ecosystem at a conceptual level;
  it was the necessary foundation for everything that comes next on this roadmap,
  even if the content felt general rather than deeply technical
>- 40 minutes for a 65-question exam that allows 90 minutes confirms what the
  practice tests showed all along — time was never the variable, knowledge depth was

**Roadmap Context**

April 25 → May 6: **11 days from zero to AWS Certified.**
The roadmap originally allocated significantly more time for this certification.
That buffer now flows forward into the Data & Feature Infrastructure work —
the parts of the roadmap that actually matter for the L5 target.

The pace on this roadmap is insane. That does not change... it accelerates!


---

## May 5, 2026

**AWS | CLF-C02 — Exam Sprint Day 5: Peak Scores + Language Decision**

Three practice tests completed today — all three above 80%, with Test 1 hitting 93%.
Two days out from the exam, scores are at their highest point of the entire sprint.
The exam will be taken in English.

**Practice Simulations — Maarek**

| Test | Score | Note |
|---|---|---|
| Practice Test 1 — Attempt 4 | **93%** ✅ | Highest score of the sprint |
| Practice Test 2 — Attempt 4 | **88%** ✅ | Consistent above threshold |
| Practice Test 3 — Attempt 3 | **84%** ✅ | Third run, solidly above passing |

>**What I understood**
>- 93%, 88%, and 84% across three tests in a single day is the clearest signal yet
  that the patterns are locked in — not just for familiar questions but for the
  reasoning process behind them
>- All three scores are well above the 70% passing threshold; the keyword-to-service
  mapping that was built on Day 1 of the sprint is now running automatically
>- Two days of consistent practice remain before May 7 — the goal is maintenance,
  not cramming

**Language Decision: Exam in English**

The German exam slots are only available between 00:00 and 05:00 — not a viable option.
The exam will be taken in English, and all remaining practice tests will be done in English.

Beyond scheduling, English is genuinely the better choice for this exam:
German AWS terminology makes questions visually dense and harder to parse quickly —
long compound words create the same readability problem as deeply nested SQL subqueries.
English keeps each question clean and scannable, which matters under exam conditions.

>**What I understood**
>- Language affects cognitive load under time pressure — even as a native German speaker,
  English reads faster for technical content because the terms are shorter and unambiguous
>- Switching practice tests to English two days before the exam is the right call;
  consistency between practice format and exam format removes one variable on test day
```
Progress | AWS CLF-C02
- Course: René Fürst AWS CLF-C02 — COMPLETED ✅
- Practice Test 1 (Maarek, Attempt 2): 64% (42/65)
- Practice Test 2 (Maarek, Attempt 1): 40% (26/65)
- Practice Test 3 (Maarek, Attempt 1): 67%
- Practice Test 4 (Maarek, Attempt 1): 62%
- Practice Test 5 (Maarek, Attempt 1): 59%
- Practice Test 6 (Maarek, Attempt 1): 71%
- Practice Test 1 (Maarek, Attempt 3): 88% ✅
- Practice Test 2 (Maarek, Attempt 3): 85% ✅
- Practice Test 1 (Maarek, Attempt 4): 93% ✅
- Practice Test 2 (Maarek, Attempt 4): 88% ✅
- Practice Test 3 (Maarek, Attempt 3): 84% ✅
- Exam language: English
- Next: final practice runs May 6 → sit the exam May 7
- Target exam date: May 7, 2026 (free retake secured)
```
---


---

## May 4, 2026

**AWS | CLF-C02 — Exam Sprint Day 4: Tests 1 & 2 Repeated**

Three full runs each through Practice Tests 1 and 2 today — focusing on locking in
the patterns behind previously missed questions before exam day on May 7.

**Practice Simulations — Maarek (Tests 1 & 2 Repeated)**

| Test | Best Score | Note |
|---|---|---|
| Practice Test 1 — Attempt 3 | **88%** | Best result to date |
| Practice Test 2 — Attempt 3 | **85%** | Consistent above passing threshold |

>**What I understood**
>- Repeating tests with known questions is a valid method when the goal is pattern
  reinforcement — the CLF-C02 draws from a finite service set and similar scenario
  framings appear on the real exam; recognizing the pattern matters as much as
  knowing the answer cold
>- 88% and 85% on repeated runs show that the keyword-to-service mapping is now
  instinctive for the majority of question types — the remaining gaps are edge cases,
  not foundational blind spots
>- The Udemy CLF-C02 exam purchase includes a 'free' second attempt — that removes pressure completely;
  the goal is to pass on May 7, but if not, the backup attempt is already paid for

**Mindset Going Into May 7**

The priority is simple: pass the exam and move forward on the roadmap.
With a free retake already secured, May 7 is an attempt with no downside.
Passing CLF-C02 ahead of schedule extends the roadmap buffer significantly —
more time for the AWS Solutions Architect Associate and the production-grade
projects that matter most for the L5 target.
```
Progress | AWS CLF-C02
- Course: René Fürst AWS CLF-C02 — COMPLETED ✅
- Practice Test 1 (Maarek, Attempt 2): 64% (42/65)
- Practice Test 2 (Maarek, Attempt 1): 40% (26/65)
- Practice Test 3 (Maarek, Attempt 1): 67%
- Practice Test 4 (Maarek, Attempt 1): 62%
- Practice Test 5 (Maarek, Attempt 1): 59%
- Practice Test 6 (Maarek, Attempt 1): 71%
- Practice Test 1 (Maarek, Attempt 3): 88% ✅
- Practice Test 2 (Maarek, Attempt 3): 85% ✅
- Next: final practice runs May 5–6 → sit the exam May 7
- Target exam date: May 7, 2026 (free retake secured)
```

---

## May 3, 2026

**AWS | CLF-C02 — Exam Sprint Day 3: Practice Tests 5 & 6**

Two more Maarek practice tests completed today. Test 6 crossed the 70% passing
threshold for the first time — a meaningful milestone four days out from the exam.

**Practice Simulations — Maarek (Practice Test 5 & 6)**

| Test | Score | Note |
|---|---|---|
| Practice Test 5 — Attempt 1 | **59%** | Dip — new service clusters exposed |
| Practice Test 6 — Attempt 1 | **71%** ✅ | First test above passing threshold |

>**What I understood**
>- Test 5 dipping to 59% before Test 6 hitting 71% is not a contradiction —
  each test covers a different service distribution; the overall trend is upward
>- With each additional test it becomes clearer how outdated the René Fürst course was —
  a fair estimate is that it covers roughly 60% of what the current CLF-C02 actually tests;
  it cannot be recommended as a standalone prep resource for this exam
>- 71% above the 70% passing line confirms the keyword strategy and feedback-driven
  approach are working — the remaining days are about consistency, not new methods

**Plan to May 7**

Continuous practice tests from here until exam day — no new theory, no re-reading notes.
Every remaining session is a full 65-question simulation, reviewed for gaps immediately after.
The exam gets attempted when the pattern feels consistent, not when every service is memorized.
```
Progress | AWS CLF-C02
- Course: René Fürst AWS CLF-C02 — COMPLETED ✅
- Practice Test 1 (Maarek, Attempt 2): 64% (42/65)
- Practice Test 2 (Maarek, Attempt 1): 40% (26/65)
- Practice Test 3 (Maarek, Attempt 1): 67% 
- Practice Test 4 (Maarek, Attempt 1): 62%
- Practice Test 5 (Maarek, Attempt 1): 59%
- Practice Test 6 (Maarek, Attempt 1): 71%
- Next: continuous practice tests until May 7 → consistent 80%+ → sit the exam
- Target exam date: May 7, 2026
```


---

## May 2, 2026

**AWS | CLF-C02 — Exam Sprint Day 2: Practice Tests 3 & 4**

Two more full Maarek practice tests completed today — both first attempts on fresh
question sets. Scores are trending in the right direction heading into exam week.

**Practice Simulations — Maarek (Practice Test 3 & 4)**

| Test | Score | Note |
|---|---|---|
| Practice Test 3 — Attempt 1 | **67%** | First attempt, new questions |
| Practice Test 4 — Attempt 1 | **62%** | First attempt, new questions |

>**What I understood**
>- The upward trend from 40% → 62–67% on first attempts reflects real consolidation,
  not score inflation from re-exposure to the same questions
>- Being confronted with unseen questions and immediate feedback is deliberately harder
  than re-reading material — but it is the only method that gives an honest picture
  of what is actually retained; from experience with theory exams, this approach
  consistently outperforms passive review
>- The AWS service scope is broader than expected — but each test surfaces fewer
  new unknowns than the one before, which means the gaps are closing

**Exam Date: Personal Challenge**

The roadmap allows until May 30 for CLF-C02. The exam is booked for May 7 —
under two weeks from course start to certified. May 30 is the backup, May 7 is the target.
```
Progress | AWS CLF-C02
- Course: René Fürst AWS CLF-C02 — COMPLETED ✅
- Practice Test 1 (Maarek, Attempt 2): 64% (42/65)
- Practice Test 2 (Maarek, Attempt 1): 40% (26/65)
- Practice Test 3 (Maarek, Attempt 1): 67% 
- Practice Test 4 (Maarek, Attempt 1): 62%
- Strategy: keyword mapping + unseen question exposure for honest feedback
- Next: Practice Tests 5 & 6 → close remaining gaps → exam May 7
- Target exam date: May 7, 2026
```

---

## May 1, 2026

**AWS | CLF-C02 — Exam Sprint Day 1: Patterns Recognized, Strategy Built**

Today was Day 1 of the final 7-day sprint toward the AWS Certified Cloud Practitioner
exam on May 7, 2026. The day consisted of two full practice exams from the Stéphane Maarek
course format and the development of a concrete study strategy for the remaining days.

**Practice Simulations — Maarek (Practice Test 1 & 2)**

Both tests were completed in full. Time was not a limiting factor on either simulation —
both were finished well under the 90-minute limit.

| Test | Score | Note |
|---|---|---|
| Practice Test 1 — Attempt 2 | **64%** (42/65) | Previously seen questions — time no issue |
| Practice Test 2 — Attempt 1 | **40%** (26/65) | New services not covered in the René Fürst course |

The weak result on the second test had a concrete cause: the René Fürst course used to start
CLF-C02 prep is outdated and does not cover many of the services currently tested on the exam.
It consumed more time than it delivered knowledge. Switching to Maarek simulations and
an independent keyword-based strategy was the right call.

>**What I understood**
>- The René Fürst course built a foundation but left real gaps — the exam tests services
  the course never mentioned; Maarek simulations are the correct benchmark
>- 64% on a repeat test and 40% on a fresh test is not a contradiction — it shows
  that familiarity with questions inflates scores, and new material exposes real gaps
>- Time is not the problem at this exam — finishing well under 90 minutes on both tests
  confirms that the only variable left to optimize is knowledge depth

**Core Problem Identified: CLF-C02 Is Not Rushable**

The exam does not test memorized facts. Nearly all answer options are technically correct —
what is evaluated is which answer best fits the described use case.

- Almost every question has multiple plausible answers — the difference is always in the detail
- Example: SQS, SNS, and EventBridge all do "messaging" — but for entirely different scenarios
- Classical memorization is ineffective; the exam demands conceptual understanding
  of where each service ends and the next one begins

>**What I understood**
>- "Which service does X?" is never the real question — "Which service fits this specific
  scenario best?" is; that distinction changes how every study session should be structured
>- Recognizing the boundary conditions between similar services is the skill the exam tests —
  not the ability to list what a service does in isolation
>- This is actually good news: once the keyword patterns click, the same logic applies
  to every question category across the entire exam

**Strategy Built: Keyword Mapping**

The most effective pattern identified today: every exam question contains one or more
**keywords** that point directly to the correct service.

| Keyword in the question | Service |
|---|---|
| `"decouple"`, `"queue"`, `"async"`, `"one receiver"` | **Amazon SQS** |
| `"broadcast"`, `"many receivers"`, `"push"`, `"fan-out"` | **Amazon SNS** |
| `"event-driven"`, `"trigger between services"` | **Amazon EventBridge** |
| `"who did what?"`, `"audit log"`, `"API calls"` | **AWS CloudTrail** |
| `"suspicious activity"`, `"threat detection"` | **Amazon GuardDuty** |
| `"scan vulnerabilities"`, `"CVE"`, `"vulnerability"` | **Amazon Inspector** |
| `"PII in S3"`, `"GDPR"`, `"detect sensitive data"` | **Amazon Macie** |
| `"TAM"`, `"Technical Account Manager"` | **Enterprise Support only** |
| `"budget alert"`, `"cost limit"` | **AWS Budgets** (not Cost Explorer) |
| `"analyze costs"`, `"spending over last months"` | **AWS Cost Explorer** |

The same keyword logic resolves the most common service confusions:

- **CloudTrail vs. CloudWatch vs. Config:** "Who did what?" → CloudTrail · "Metrics/performance?" → CloudWatch · "Configuration changes?" → Config
- **Shield vs. WAF:** "DDoS Layer 3/4?" → Shield · "SQL injection / XSS / HTTP attacks?" → WAF
- **Direct Connect vs. VPN:** "Dedicated physical line?" → Direct Connect · "Secure internet tunnel?" → VPN

>**What I understood**
>- Keyword mapping converts ambiguous multi-choice questions into pattern recognition —
  once the signal words are internalized, the correct answer surfaces before finishing the question
>- Managed services are patched by AWS (RDS); EC2 OS is patched by the customer —
  this is the Shared Responsibility Model applied to the most common exam scenario
>- TAM exists exclusively in the Enterprise Support Plan — Business Support has no TAM;
  this distinction appears on almost every Support Plan question

**Roadmap Context**

The CLF-C02 exam was originally planned for the end of May.
The course started April 25, 2026 — the exam is May 7, 2026.
That is under two weeks from zero to certified.

This puts me significantly ahead of the original roadmap schedule. With the right
approach — keyword signals, use-case differentiation, error review over blind clicking —
passing on May 7 is realistic, and the next step toward AWS Certified Data Engineer – Associate
can begin earlier than planned.
```
Progress | AWS CLF-C02
- Course: René Fürst AWS CLF-C02 — COMPLETED ✅
- Practice Test 1 (Maarek, Attempt 2): 64% (42/65)
- Practice Test 2 (Maarek, Attempt 1): 40% (26/65) — new services, gaps mapped
- Strategy: keyword mapping per service category — cheat sheet built
- Next: Practice Test 3 + Billing & Pricing deep dive
- Target exam date: May 7, 2026
```

---

## April 30, 2026

**AWS | CLF-C02 — Day 6: Storage, Databases & First Full Mock Test**

Continued CLF-C02 exam prep with category quizzes on Storage and Databases,
then completed the first official 65-question AWS practice test — not to pass,
but to map exactly where the gaps are before the exam on May 7.

**CLF-C02 | Exam Prep — Storage & Database Category**

- **Storage & Database Basics quiz:** 85% on first attempt — same threshold as Compute,
  confirming that core storage and database concepts from the course have landed
- **S3 Object Lock (WORM):** Write Once Read Many — objects cannot be modified or deleted
  for a defined retention period; used for compliance and regulatory data protection
- **CRR vs. SRR:** Cross-Region Replication copies S3 objects to a bucket in a different
  AWS region; Same-Region Replication copies within the same region — different use cases,
  same mechanism
- **Aurora vs. RDS:** both are relational managed databases — Aurora is re-architected
  for higher performance and availability; RDS is the standard managed SQL option
- **Snow Family:** Snowcone = small, portable, 8TB; Snowball Edge = mid-range with
  compute capability; Snowmobile = truck-scale, exabyte migration — size determines choice
- **ElastiCache vs. RDS Read Replicas:** ElastiCache is an in-memory cache for
  microsecond reads; RDS Read Replicas distribute SQL query load — different layers,
  both reduce primary database pressure

>**What I understood**
>- The Snow Family size ladder is a guaranteed exam question — memorize Snowcone →
  Snowball Edge → Snowmobile as small → medium → massive
>- ElastiCache and RDS Read Replicas both improve read performance but at different
  layers — ElastiCache bypasses the database entirely, Read Replicas scale within it
>- Quiz tool errors on the Details round exposed a real lesson: when the tool contradicts
  known correct answers, switch methods — manual chat-based Q&A is more reliable

**CLF-C02 | First Full Mock Test — 65 Questions**

- **Result: 35% (23/65) in 50 minutes** — first full official practice test,
  taken before all topic areas were reviewed; the score is a diagnostic, not a grade
- **Billing & Pricing: 9%** — Priority #1; almost no correct answers,
  this is the biggest lever before the exam
- **Security & Compliance: 33%** — Priority #2; IAM fundamentals are solid
  but compliance-specific services need targeted study
- **Technology: 41%** — mid-range; service-level details need sharpening
- **Cloud Concepts: 47%** — closest to passing threshold; foundational knowledge
  from the course is present, execution needs refinement under exam conditions

>**What I understood**
>- 35% on a full mock test is not a failure — it is the most honest map of exactly
  where time should go between now and May 7; without this test, those gaps stay hidden
>- Billing & Pricing at 9% means it needs to be treated as a standalone subject, not an afterthought
>- 50 minutes for 65 questions is on pace — time management is not the issue;
  knowledge gaps in Billing and Compliance are the only thing to fix
```
Progress | AWS CLF-C02
- Course: René Fürst AWS CLF-C02 — COMPLETED ✅
- Compute quiz: 85% ✅
- Storage & Database Basics quiz: 85% ✅
- First full mock test: 35% (23/65) — gaps mapped
- Priority #1: Billing & Pricing (9%) → tonight: 40 questions
- Priority #2: Security & Compliance (33%) → tomorrow: 40  questions → next mock test 65 questions for new gap mapping. 
- Target exam date: May 7, 2026
```

---

## April 29, 2026

**AWS | CLF-C02 — Day 5: Sections 15–17 completed — René Fürst Course Finished ✅**

Completed the final three sections of the René Fürst AWS CLF-C02 course today.
Sections 15–17 covered Developer Tools, AWS Migration, and AWS Analytics —
the last building blocks before moving into full exam practice mode.
The full course curriculum is now done. Next step: practice tests to 80%+ and book the exam.

**Developer Tools**

- **AWS CodeStar:** unified interface to manage software development activities —
  brings together CodeCommit, CodeBuild, CodeDeploy, and CodePipeline in one place
- **AWS CodeCommit:** managed Git-based source control repository on AWS —
  the AWS-native alternative to GitHub for storing and versioning code
- **AWS CodeBuild:** fully managed build service — compiles source code, runs tests,
  and produces deployable artifacts without managing build servers
- **AWS CodeDeploy:** automated deployment service — deploys applications to EC2,
  Lambda, or on-premise servers with rollback support on failure
- **AWS CodePipeline:** continuous delivery service — orchestrates the full
  build → test → deploy pipeline end to end, triggered on every code change
- **AWS X-Ray:** distributed tracing service — visualizes requests as they travel
  through microservices and identifies performance bottlenecks and errors

>**What I understood**
>- CodePipeline is the orchestrator — it calls CodeBuild to build and CodeDeploy to deploy;
  understanding which tool does what in the CI/CD chain is a guaranteed exam question
>- X-Ray is the answer whenever an exam question asks about tracing, debugging, or
  performance visibility across microservices — it maps the full request path
>- These tools together form the complete AWS-native DevOps stack — relevant not just
  for the exam but directly applicable to pipeline work on this roadmap

**AWS Migration**

- **AWS Application Discovery Service:** scans on-premise infrastructure to collect
  server configurations, usage data, and dependencies — maps what needs to migrate
  before the migration starts
- **AWS Server Migration Service (SMS):** automates the migration of on-premise
  virtual machines to AWS — incremental replication minimizes downtime
- **AWS Snowball:** physical data transfer device — ships petabyte-scale data
  to AWS when network transfer is too slow or expensive; Snowball Edge adds
  compute capability on the device itself
- **AWS Migration Hub:** central dashboard to track the status of all migrations
  across multiple AWS and partner tools in one place

>**What I understood**
>- Snowball is the answer whenever an exam question mentions large data volumes and
  slow or limited network connectivity — physical transfer beats network transfer at scale
>- Application Discovery Service comes before migration; SMS and Migration Hub manage
  the migration itself — the sequence matters for exam questions
>- Migration is a real roadmap topic: understanding how data moves into AWS is
  foundational for any data engineering pipeline that starts from on-premise sources

**AWS Analytics**

- **Amazon Athena:** serverless interactive query service — runs SQL queries directly
  on data stored in S3; no infrastructure to manage, pay per query
- **Amazon EMR (Elastic MapReduce):** managed big data platform — runs Apache Spark,
  Hadoop, and Hive on scalable EC2 clusters for large-scale data processing
- **Amazon Elasticsearch (OpenSearch):** managed search and analytics engine —
  indexes and queries log data, metrics, and full-text content in near real time
- **Amazon Kinesis:** real-time data streaming — ingest, process, and analyze
  streaming data from IoT devices, application logs, and clickstreams at scale
- **Amazon QuickSight:** cloud-native business intelligence tool — creates
  interactive dashboards and visualizations from data stored in AWS services
- **AWS Data Pipeline:** managed ETL service — moves and transforms data between
  AWS compute and storage services on a defined schedule
- **AWS Kinesis — Deep Dive:** Kinesis Data Streams for custom real-time processing,
  Kinesis Data Firehose for zero-code delivery to S3, Redshift, or Elasticsearch

>**What I understood**
>- Athena + S3 is the serverless analytics pattern — no cluster to spin up,
  no database to load; query raw files in S3 directly with SQL; critical for this roadmap
>- Kinesis is the AWS answer for real-time streaming data — Kinesis Data Streams
  for processing, Kinesis Firehose for delivery; know the difference for the exam
>- EMR is for large-scale batch processing with Spark or Hadoop — heavier than Athena
  but necessary when transformation logic is complex or data volumes are extreme

**CLF-C02 | Exam Prep — Compute Category**

- Built a complete CLF-C02 cheat sheet covering 85 AWS services — one memory sentence
  per service, organized by category; the single reference doc for final exam review
- Scored 85% on the Compute category quiz first attempt! — 3 rounds of 20 questions each,
  generated with Perplexity Pro (Claude)
- Rule established: 80%+ = category cleared, move on; below 70% = repeat the category
- Exam target date locked in: May 7–10, 2026

>**What I understood**
>- 85% on Compute on the first quiz attempt confirms the course content landed —
  EC2 pricing models, instance types, and the serverless distinctions are solid
>- The cheat sheet with one sentence per service is the most efficient recall tool —
  it forces distillation of each service down to its single defining characteristic
>- Tomorrow: Storage + Database category — 3 x 20 questions to validate or expose gaps
```
Progress | AWS CLF-C02
- Course: René Fürst AWS CLF-C02 — COMPLETED ✅ (All 17 Sections, Lectures 1–83)
- Course completed: April 29, 2026
- Next: Full practice exam sessions → push to 80%+ consistently → book exam
- Target exam date: May 7–10, 2026
```

---

## April 28, 2026

**AWS | CLF-C02 — Day 4: Sections 12–14 completed**

Continued the René Fürst AWS CLF-C02 course. Worked through Sections 12–14 covering
Amazon Databases, Networking & Content Delivery, and Application Integration —
three sections that are heavily represented in real-world cloud architectures
and on the CLF-C02 exam.

**AWS Databases**

- **Amazon RDS:** managed relational database service — supports MySQL, PostgreSQL,
  MariaDB, Oracle, and SQL Server; AWS handles backups, patching, and failover
- **Amazon Aurora:** AWS-native relational database — up to 5x faster than MySQL,
  fully managed, with automatic replication across multiple AZs
- **SQL vs. NoSQL:** SQL = structured, schema-based, relational (RDS, Aurora);
  NoSQL = flexible, schema-less, for high-scale unstructured data (DynamoDB)
- **Backup & Replicas:** automated backups and read replicas in RDS reduce
  recovery time and distribute read traffic across instances
- **Amazon DynamoDB:** fully managed NoSQL key-value and document database —
  single-digit millisecond latency at any scale, serverless and auto-scaling
- **Amazon Redshift & Redshift Spectrum:** columnar data warehouse for analytics
  at petabyte scale; Spectrum extends queries directly to S3 data
- **Amazon Neptune:** managed graph database — optimized for highly connected
  datasets like social networks, fraud detection, and knowledge graphs
- **ElastiCache:** in-memory caching layer — Redis or Memcached; used to reduce
  database load and serve frequently accessed data in microseconds
- **AWS Database Migration Service (DMS):** migrates databases to AWS with minimal
  downtime — supports homogeneous and heterogeneous migrations

>**What I understood**
>- RDS vs. DynamoDB is a guaranteed CLF-C02 question: RDS = relational, structured,
  SQL; DynamoDB = NoSQL, flexible schema, built for massive scale
>- Aurora is not just RDS with a different name — it is a re-architected database
  with higher performance and availability, still using SQL
>- Redshift is not a transactional database — it is an analytical warehouse;
  knowing when to use Redshift vs. RDS separates exam-ready answers from guesses

**Networking & Content Delivery**

- **Amazon VPC (Virtual Private Cloud):** logically isolated network within AWS —
  you define subnets, routing tables, internet gateways, and security groups;
  every AWS deployment lives inside a VPC
- **Amazon VPC — Deep Dive:** public vs. private subnets, NAT Gateway for outbound
  internet access from private subnets, VPC Peering for cross-VPC communication
- **Amazon Route 53:** managed DNS service — translates domain names to IP addresses;
  supports routing policies like latency-based, failover, and geolocation
- **Route 53 Hands-On — Domain Registration:** practical walkthrough of registering
  a domain and configuring DNS records — connects networking theory to real deployment
- **Amazon CloudFront:** global Content Delivery Network — caches content at Edge
  Locations worldwide to reduce latency and serve users from the nearest location

>**What I understood**
>- VPC is not optional — every resource on AWS lives inside one; understanding
  public vs. private subnets and security groups is foundational for any architecture
>- Route 53 + CloudFront is the standard pattern for globally distributed, low-latency
  web applications — know this pairing for both the exam and real projects
>- CloudFront is the answer whenever an exam question mentions "reduce latency for
  global users" — it pulls content to the edge, not from the origin on every request

**Application Integration**

- **Amazon SNS (Simple Notification Service):** managed pub/sub messaging —
  one message published to a topic fans out to multiple subscribers
  (email, Lambda, SQS, HTTP endpoints) simultaneously
- **Amazon SQS (Simple Queue Service):** managed message queue — decouples
  producers and consumers; messages wait in the queue until a consumer processes them;
  guarantees at-least-once delivery

>**What I understood**
>- SNS vs. SQS is a classic exam question: SNS = fan-out to many subscribers at once;
  SQS = one consumer processes each message from a queue — different patterns,
  different use cases
>- SNS + SQS together is the standard decoupled architecture pattern on AWS —
  SNS fans out, SQS buffers; combining both enables resilient, scalable pipelines
```
Progress | AWS CLF-C02
- Course: René Fürst AWS CLF-C02
- Today: Sections 12–14 completed (Lectures 69–79)
- Sections remaining: 15–17
- Next: Sections 15–17 → final practice tests → book exam
- Target exam date: mid-May 2026
```

---

## April 27, 2026

**AWS | CLF-C02 — Day 3: Sections 8–11 completed + LeetCode SQL 50**

Continued the René Fürst AWS CLF-C02 course. Worked through Sections 8–11 covering
Serverless, Security & IAM, Management & Governance, and Storage Services — the most
exam-heavy blocks of the entire CLF-C02 curriculum. Supplemented the day with LeetCode
SQL 50 problems to keep SQL fundamentals sharp alongside AWS prep.

**AWS Serverless**

- **AWS Lambda:** event-driven compute — functions trigger on events (API calls, S3 uploads,
  DynamoDB streams) and you pay only per execution, not for idle time
- **AWS Fargate (Serverless):** serverless compute engine for containers — revisited here
  to confirm that Fargate bridges the container and serverless worlds
- **Serverless Pricing Model:** Lambda is billed per number of requests and execution
  duration in milliseconds — scales to zero when idle, no minimum fee

>**What I understood**
>- Lambda is the core of AWS serverless — almost every event-driven architecture on AWS
  connects through Lambda at some point
>- Serverless does not mean "no servers" — it means AWS manages them invisibly;
  the CLF-C02 exam tests whether you understand this distinction
>- Lambda + Fargate cover two serverless paths: code-only (Lambda) vs. container-based (Fargate)

**AWS Security, Identität & Compliance**

- **Amazon IAM:** users, groups, roles, and policies — the permission model that enforces
  least privilege across every AWS service; the most important security layer on AWS
- **IAM Inspector, Certificate Manager, Directory Service:** Inspector scans for
  vulnerabilities; ACM manages SSL/TLS certificates; Directory Service connects AWS
  to Microsoft Active Directory
- **IAM Setup & User Creation (Hands-On):** practical walkthrough of creating users,
  assigning policies, and enabling MFA — the operational side of access control
- **IAM Cost Control:** using IAM policies and AWS Budgets to prevent unauthorized
  or accidental spending — cost governance starts with access governance

>**What I understood**
>- IAM is the single most important security service on AWS — every exam question about
  "who can access what" routes through IAM
>- The difference between IAM users (humans), roles (services), and groups (permission sets)
  is a guaranteed multi-question topic on CLF-C02
>- Inspector and Certificate Manager are supporting security services — know what they do
  but do not confuse them with IAM's core access control function

**Management & Governance**

- **CloudWatch:** monitoring and observability — metrics, logs, and alarms for every
  AWS resource; the operational dashboard of an AWS account
- **CloudTrail:** records every API call made in an AWS account — the forensic audit log
  for "who did what and when"
- **CloudFormation:** infrastructure as code on AWS — define resources in templates,
  deploy repeatable and version-controlled environments automatically
- **Trusted Advisor:** automated recommendations across cost, security, performance,
  and fault tolerance — the built-in best practice checker
- **API Gateway + Step Functions:** API Gateway exposes Lambda and backends as REST or
  WebSocket APIs; Step Functions orchestrates multi-step serverless workflows

>**What I understood**
>- CloudTrail is the standard CLF-C02 answer for any question about auditing or accountability —
  it logs every action taken in the account
>- CloudFormation is the AWS-native equivalent of Terraform — infrastructure as code is
  foundational for every production-grade deployment on this roadmap
>- API Gateway + Lambda is the default serverless API pattern — understanding this pairing
  matters for both the exam and real-world architecture

**Amazon Storage Services**

- **Amazon S3:** object storage — the default storage layer for files, backups, static
  websites, and data lakes; globally accessible via HTTP
- **Amazon Glacier:** long-term archival storage — significantly cheaper than S3 Standard
  but with retrieval times ranging from minutes to hours
- **Amazon EBS:** persistent block storage attached to a single EC2 instance — survives
  instance stops, scoped to one Availability Zone
- **Amazon EFS:** shared file storage accessible across multiple EC2 instances —
  scales automatically, no capacity planning required
- **Storage Gateway:** connects on-premise infrastructure to AWS storage — the bridge
  for hybrid cloud storage without full cloud migration
- **S3 Storage Classes & Cost Awareness:** Standard, Intelligent-Tiering, Glacier —
  choosing the right class based on access frequency is a direct cost optimization lever

>**What I understood**
>- S3 is the backbone of AWS storage — almost every AWS service reads from or writes
  to S3 at some point; storage classes and pricing are guaranteed exam topics
>- EBS vs. EFS vs. S3 is a classic CLF-C02 question: EBS = one EC2 one AZ;
  EFS = shared across many EC2; S3 = object storage globally accessible via HTTP
>- Storage Gateway is the answer whenever an exam question mentions "hybrid cloud + storage" —
  it is the dedicated bridge between on-premise and AWS

**LeetCode | SQL 50 — Continued SQL Practice**

- Completed a set of LeetCode SQL 50 problems to maintain SQL fluency alongside AWS prep
- Problem types: SELECT with filtering, GROUP BY with aggregation, JOIN across multiple
  tables, subqueries and nested SELECT
- Goal: keep query writing instinctive so SQL feels automatic when data engineering
  projects begin in Month 2
```
Progress | AWS CLF-C02
- Course: René Fürst AWS CLF-C02
- Today: Sections 8–11 completed (Lectures 52–68)
- Sections remaining: 12–17
- Next: Section 12 onwards → push practice tests to 80%+ → book exam
- Target exam date: mid-May 2026
```

---

## April 26, 2026

**AWS | CLF-C02 — Day 2: Sections 6–7 completed**

Continued the René Fürst AWS CLF-C02 course. Finished Section 6 (AWS Products & Services)
and Section 7 (AWS Container) — covering Compute services in depth, all major EC2 concepts,
and the full container ecosystem on AWS.

**AWS Products & Services**

- **EC2 deep dive:** instance types, pricing models (On-Demand, Reserved, Spot, Dedicated),
  and core EC2 features — the foundation of IaaS on AWS
- **AWS Elastic Beanstalk:** PaaS layer on top of EC2 — you deploy code, AWS manages
  the underlying infrastructure automatically
- **AWS Outposts:** AWS infrastructure physically installed in your own data center —
  extends AWS into on-premise environments
- **AWS Lightsail:** simplified compute for small workloads and beginners —
  fixed pricing, no need to configure VPCs or EC2 manually
- **AWS Batch:** fully managed batch processing — runs large-scale compute jobs
  without managing the underlying servers
- **AWS Wavelength:** brings AWS compute and storage to the edge of 5G networks —
  ultra-low latency for mobile and IoT use cases

>**What I understood**
>- EC2 pricing models are a guaranteed exam topic — On-Demand is flexible but expensive,
  Reserved saves up to 72% for predictable workloads, Spot is cheapest but can be interrupted
>- Elastic Beanstalk, Lightsail, and Outposts all sit on top of EC2 — understanding EC2 first
  makes every other compute service easier to classify
>- Wavelength and Outposts exist for edge and hybrid scenarios — not standard cloud deployments,
  but the exam tests whether you can distinguish them

**AWS Container**

- **Container vs. VM:** containers share the host OS kernel and are lightweight;
  VMs include a full OS per instance — containers start faster and consume fewer resources
- **Amazon ECS (Elastic Container Service):** AWS-native container orchestration —
  run and manage Docker containers at scale without managing the control plane
- **Amazon EKS (Elastic Kubernetes Service):** managed Kubernetes on AWS —
  for teams already using Kubernetes who want AWS to manage the control plane
- **AWS Fargate:** serverless compute engine for containers — no EC2 instances to manage,
  you only define CPU and memory and AWS runs the container
- **Amazon ECR (Elastic Container Registry):** private Docker image registry on AWS —
  stores, versions, and secures container images before deployment
- **AWS App2Container (A2C):** tool to containerize existing Java and .NET applications
  without rewriting code — lift-and-shift to containers
- **AWS Copilot:** CLI tool to deploy and operate containerized apps on ECS and Fargate —
  simplifies the full container deployment workflow

>**What I understood**
>- ECS vs. EKS is a classic exam question: ECS = AWS-native, EKS = Kubernetes — choose EKS
  when the team already has Kubernetes expertise, ECS when starting fresh on AWS
>- Fargate removes the need to think about EC2 at all for containers — it is the serverless
  path for container workloads and pairs with both ECS and EKS
>- ECR is to container images what S3 is to files — a managed, secure storage layer
  that sits before every container deployment
```
Progress | AWS CLF-C02
- Course: René Fürst AWS CLF-C02
- Today: Sections 6–7 of 17 completed (Lectures 34–51)
- Sections remaining: 8–17
- Next: Section 8 — AWS Serverless
- Target exam date: mid-May 2026
```

---

## April 25, 2026

**AWS | CLF-C02 Course Started**


Today I officially started AWS CLF-C02 preparation. Worked through Sections 1–6 of 17
in a single day and completed a first practice exam with 65% — a solid baseline for
Day 1 with no prior AWS knowledge.

**What I covered today**
- Core cloud concepts: what Cloud Computing is, why companies migrate, and the advantages over On-Premise
- The three cloud deployment models: Public Cloud, Private Cloud, and Hybrid Cloud
- The three service models: IaaS, PaaS, and SaaS — the difference and when each is used
- AWS Global Infrastructure: Regions, Availability Zones, and Edge Locations and why they matter for resilience and latency
- Shared Responsibility Model: what AWS owns (Security **of** the Cloud) vs. what the customer owns (Security **in** the Cloud)
- First overview of core AWS services: EC2, S3, RDS, Lambda, VPC, IAM, CloudWatch

>**What I understood**
>- The Shared Responsibility Model is the single most important concept for the exam — almost every security question revolves around it
>- Regions and AZs are not academic concepts — they are the foundation of every cloud architecture I will build later on this roadmap
>- IaaS = EC2 (you manage everything), PaaS = Elastic Beanstalk (AWS manages infrastructure), SaaS = ready-to-use software — this model helps classify every AWS service instantly

**First AWS CLF-C02 Practice Exam | Result: 65%**
- Completed a full CLF-C02 practice simulation with 20 questions on Day 1
- Result: 65% with no prior AWS study — shows that existing cloud and API context already provides a baseline
- Gaps are expected in service-specific details, pricing logic, and compliance topics
- Target: consistently 80%+ on practice tests before booking the real exam

>**What I understood**
>- 65% on Day 1 means the foundation is there — the details come with the remaining course sections
>- Pricing and Billing is its own learning block and the most common source of errors for beginners
>- The exam tests recognition and classification, not deep engineering knowledge
```
Progress so far | AWS CLF-C02
- Course: René Fürst AWS CLF-C02
- Today: Sections 1–6 of 17 completed
- Practice exam baseline: 65%
- Next: Complete Sections 7–17 → push practice tests to 80%+ → book the exam
- Target exam date: mid-May 2026
```

---

## April 24, 2026

**Advanced SQL | Theory + Final Project Complete: Personal Reflection**

At the start, Subqueries and CTEs felt genuinely complicated. The syntax is long,
the nesting can be hard to read at first, and the mental model takes time to build.
After completing the full theory section however, the concepts feel significantly
more logical and approachable than initially expected.

The theoretical foundation is now set. What remains is consolidating this through
practice — LeetCode SQL problems and other hands-on platforms will be the focus
going forward to make SQL truly job-ready. Query performance decisions, meaning
choosing the most efficient method for a given problem to avoid unnecessary
computation, is also an area I plan to deepen over time.

I also completed the final course project, which helped connect all the covered
concepts in one practical exercise. Some gaps remain, particularly around writing
complex queries independently from scratch, but I am confident these will close
over the coming weeks through consistent practice and real-world application.

The [Maven Advanced SQL](https://www.udemy.com/course/sql-advanced-queries/) course was an excellent choice for building this foundation.
It covered every relevant concept clearly, and I would fully recommend it for anyone
targeting a similar career path. The practical side will continue to develop throughout the roadmap, especially once
Spark SQL begins in Month 3.



**SQL | Data Analysis Applications — Section 8 Completed ✓**

Today I started and fully completed Section 8 — the final practical chapter of the
Maven Advanced SQL course. These are real data analysis workflow patterns, directly
transferable to Feature Engineering and data infrastructure work.

**Duplicate Detection & Removal**
- Identified and removed duplicates using `ROW_NUMBER()` — the cleanest pattern in production
- `GROUP BY` + `HAVING COUNT(*) > 1` surfaces duplicates — `ROW_NUMBER()` removes them
- Completed the assignment and solution

```sql
WITH ranked AS (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id, order_date, product_id
      ORDER BY transaction_id
    ) AS rn
  FROM orders
)
SELECT * FROM ranked WHERE rn = 1;
```

**Min / Max Value Filtering**
- Filtered rows containing the min or max value per group using both subquery and window function patterns
- Window Functions are more efficient when filtering across multiple groups simultaneously
- Completed the assignment and solution

**Pivoting**
- Learned how to reshape data from rows to columns using pivot logic in SQL
- Completed the assignment and solution

**Rolling Calculations**
- Covered rolling calculations including running totals and moving averages using window frames
- Completed the full demo, assignment, and solution

**Imputing NULL Values**
- Learned how to handle and replace NULL values in analytical datasets using `COALESCE` and window-based imputation
- Completed the full demo

---

## April 23, 2026

**SQL | Functions by Data Type — Section 7 Completed ✓**
- Covered Function Basics and Numeric Functions
- Learned `CAST()` and `CONVERT()` for explicit type conversion
- Completed the Numeric Functions assignment and solution
- Covered DateTime Functions and completed the assignment and solution
- Covered String Functions and completed the assignment and solution
- Covered Pattern Matching, including a full demo, assignment, and solution
- Covered NULL Functions and completed the assignment and solution
- Reviewed Key Takeaways and passed Quiz 5: Functions by Data Type

>Numeric, DateTime, String, Pattern Matching, and NULL handling are all directly
relevant for cleaning and transforming real-world structured data in pipelines.
These are the functions that appear daily in Feature Engineering and ETL work.

**SQL | Window Functions — Section 6 Completed ✓**

**FIRST_VALUE, LAST_VALUE & NTH_VALUE**
- `FIRST_VALUE()` grabs the first value within a partition — e.g. cheapest product per category
- `LAST_VALUE()` requires an explicit frame — without it, the window only runs up to the current row, not the end of the partition
- Correct frame definition for `LAST_VALUE()`:

```sql
LAST_VALUE(unit_price) OVER (
  PARTITION BY factory
  ORDER BY unit_price
  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
```

- `NTH_VALUE(column, n)` retrieves the nth value within a partition — e.g. second most expensive item
- Key distinction: `FIRST_VALUE` is frame-safe, `LAST_VALUE` and `NTH_VALUE` always require an explicit frame

**What I understood**
- `LAST_VALUE()` without a frame almost always returns wrong results — one of the most common Window Function bugs in production
- `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` means the entire partition, regardless of current row position

**LEAD & LAG — L5 Critical**
- `LAG(column, offset, default)` accesses the value n rows before the current row
- `LEAD(column, offset, default)` accesses the value n rows after the current row
- Default offset is 1 if not specified — `LAG(revenue)` = previous row
- The optional default value prevents `NULL` for the first or last row:

```sql
LAG(revenue, 1, 0) OVER (ORDER BY order_date)
```

- Period-over-Period is the most used Feature Engineering pattern with `LAG`:

```sql
SELECT
  user_id,
  event_date,
  session_count,
  LAG(session_count) OVER (PARTITION BY user_id ORDER BY event_date) AS prev_session_count
FROM sessions;
```

>Today I completed Section 6 in full. All Window Function types are done — from Value Functions to LEAD & LAG, NTILE, Statistical Functions, and Moving Averages. Passed Quiz 4.


**Why this matters for my L5 path**
- `LAG` and `LEAD` are Non-Negotiable for time-based Feature Engineering in production ML pipelines
- Period-over-Period comparisons are used daily in Feature Stores, event tracking, and user behaviour analysis - more Data Analysis on Section 8!
- Correct frame definitions are critical — `LAST_VALUE` without a frame is a silent bug that produces wrong output

---

## April 22, 2026

**SQL | Subqueries & CTEs — Completed ✓**
- Completed Recursive CTEs — anchor member defines the start, recursive part loops until the stop condition is met
- Completed Subqueries vs CTEs vs Temp Tables vs Views, understood when each pattern is appropriate
- Reviewed Key Takeaways and passed Quiz 3: Subqueries & CTEs

>By the end of today, the concepts behind Subqueries and CTEs are fully understood and clear.
>Writing them from scratch is still developing, but the logic and structure already make complete sense.

**SQL | Window Functions — Section 6 started**
- Covered Window Function Basics — the core rule: rows are never collapsed, every row is preserved
- Broke down the full anatomy of a Window Function with `OVER (PARTITION BY ... ORDER BY ...)`
- Completed the Window Functions assignment
- Covered the available function types for Window Functions
- Learned `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` — and the key difference between them
- Completed the Row Numbering assignment
- Learned `FIRST_VALUE()` to capture the first value in a partition
- Learned `LAST_VALUE()` to capture the last value in a partition, with careful attention to the window frame
- Learned `NTH_VALUE()` to retrieve the nth row value from within a partition

>Window Functions felt immediately more intuitive at first glance.
>`PARTITION BY` is just grouping without losing rows. `ORDER BY` inside `OVER` controls rank, not output order.

---

## April 21, 2026

**SQL | Hands-on Practice — Subqueries, CTEs & LeetCode**
- Practiced writing SQL queries independently for the first time without following a course
- Applied CTEs hands-on to solve real query problems — not just watching, but writing
- Practiced LeetCode SQL problems to validate understanding outside of course material
- Understood the concepts behind Subqueries and CTEs clearly — the logic makes sense
- Noticed that translating the concept into actual written SQL is still challenging at this stage — which is completely normal for day 2 of advanced SQL

> Writing SQL from scratch is harder than understanding it — and that gap closes only through repetition and use. This is exactly where I am right now, and it is the right place to be.

**SQL | Subqueries & CTEs**

**Subqueries**
- A subquery is a query inside another query — the inner query always runs first
- Scalar Subquery in `SELECT`: returns exactly one value, used to compare each row against a global aggregate
- Derived Table in `FROM`: treats a subquery result as a temporary table — always needs an alias
- Subquery in `WHERE` / `HAVING`: filters rows based on a dynamically calculated value
- `HAVING` cannot reference `SELECT` aliases — always repeat the aggregate function directly
- `ANY`: condition must be true for at least one value — equivalent to `> MIN()`
- `ALL`: condition must be true for every value — equivalent to `< MIN()`
- `EXISTS`: returns `TRUE` or `FALSE` based on whether a subquery returns any rows at all
- Correlated Subquery: inner query references the outer row — runs once per row, important for performance awareness

**CTEs**
- A CTE is a named temporary result set defined with `WITH name AS (...)` before the main query
- The CTE block is evaluated first — the outer query reads from it like a real table
- Direct equivalent of a Derived Table but far more readable and modular
- A single CTE can be referenced multiple times in the same query — avoids repeating logic
- Multiple CTEs can be chained with commas — each one can build on the previous

```sql
WITH step_one AS (
  SELECT factory, COUNT(*) AS product_count
  FROM products
  GROUP BY factory
),
step_two AS (
  SELECT factory, product_count
  FROM step_one
  WHERE product_count > 2
)
SELECT * FROM step_two
ORDER BY product_count DESC;
```

**Subqueries vs CTEs**
- Subqueries: inline, one-time use, harder to read when nested
- CTEs: named, reusable, easy to debug step by step
- Performance is similar — CTEs win on readability and maintainability
- Rule of thumb: subquery for simple one-liners, CTE for anything more complex

**Why this matters for my L5 path**
- CTEs are the standard query pattern in Spark SQL, dbt, Feast Feature Store, and every modern data pipeline
- Multiple CTEs map directly to pipeline stages — each CTE is one transformation step
- This is not just SQL theory — this is how production Feature Engineering queries are written

>Today I completed the most conceptually challenging chapter of the Maven Advanced SQL course. Subqueries and CTEs are the foundation of every real-world data engineering query pattern.

---

## April 20, 2026

**Advanced SQL | Month 2 Started**
- Officially started Month 2 of the L5 roadmap today
- Chose Maven Advanced SQL Querying as the main SQL course for this month
- Confirmed that Maven is the better fit for my L5 goal because it focuses on advanced querying, data analysis, subqueries, CTEs, window functions, and core SQL syntax
- The course being in MySQL is not a problem, because the concepts transfer directly to PostgreSQL and other relational databases

**SQL | Multi-Table Analysis — Review**
- Reviewed all JOIN types as a knowledge refresh: `INNER JOIN` and `LEFT JOIN` are the most practical, while `RIGHT JOIN` and `FULL JOIN` are less common in day-to-day work
- Revisited joining on multiple columns and joining more than two tables in a single query
- Revisited Self Joins and Cross Joins
- Reviewed `UNION` vs `UNION ALL`
- Completed Quiz 2 to validate the full section
>This was a deliberate review session, not new material.
Tomorrow: Subqueries and CTEs — which I already applied today to fix the `HAVING` alias issue.

**Why this matters for my L5 path**
- JOINs are Non-Negotiable for Spark SQL, Feature Stores, ETL pipelines, and any real multi-table data work
- `UNION` and `UNION ALL` are used constantly when merging data sources in pipelines
- Multi-table joining is a daily pattern in production data infrastructure

**SQL Environment Setup**
- Set up my main SQL environment with TablePlus + Docker + PostgreSQL
- Started a local PostgreSQL container successfully
- Connected to the container through TablePlus
- Imported the Maven course SQL setup into the containerized PostgreSQL database
- Verified that the `create_statements.sql` import worked correctly
- Confirmed that the `students` table exists and contains data

**SQL Learning Insight**
- Ran into an issue when trying to use a `SELECT` alias inside `HAVING`
- Learned that aliases from the `SELECT` clause are not available directly inside `HAVING`
- Fixed the problem by rewriting the query with a CTE
- Understood that the content inside the CTE parentheses is evaluated first
- That made my alias `avg_gpa` available outside the CTE for later filtering
- This made the execution order of SQL much clearer to me

**Why This Matters for My L5 Path**
- PostgreSQL, Docker, TablePlus, advanced querying, and CTEs are all part of building a strong data engineering foundation
- The goal is not just to learn SQL syntax, but to build the ability to query and work with real structured data in a clean, reproducible environment
- This setup is for now more directly aligned with my path toward Data & Feature Infrastructure / ML Platform Engineering

**Next Steps**
- Continue the Maven Advanced SQL course
- Use the Docker + PostgreSQL + TablePlus setup for all exercises
- Import course datasets step by step, only when needed for the current lesson

---

>## Month 2 TIL ⬆️

>*Entries above this line belong to Month 2 starting April 20, 2026.*

---

>## Month 1 TIL ⬇️

>*All entries below this line belong to Month 1.*

---

## April 19, 2026

**Milestone · Month 1 of the L5 Roadmap: COMPLETED ✓**
Month 1 (April 2026) — Python · SQL Basics · Git · Linux — is done.
Month 2 starts tomorrow: SQL Window Functions · Docker · AWS CLF-C02.

---

**Python | Angela Yu Bootcamp — Day 100 · Final Project**

>The final notebook of Angela Yu's 100-Days-of-Code Python Bootcamp was a complete data analysis project using real US education statistics datasets.

**Data Loading & Exploration**
- Loaded multiple CSV files with `pd.read_csv()`
- Inspected structure using `.head()`, `.shape`, `.dtypes`, and `.describe()`
- Renamed columns and reviewed them for relevance

**Data Cleaning — L5 Core Skill**
- Detected missing values with `isna().sum()`
- Applied `fillna()` and `dropna()` based on context rather than blindly
- Checked for and removed duplicates using `duplicated()` and `drop_duplicates()`
- Converted dirty numeric columns using the standard pipeline:
  `astype(str)` → `.str.replace()` → `pd.to_numeric(errors='coerce')`

**Aggregation & Ranking**
- Aggregated data by state with `groupby()` and `.mean()`
- Used `sort_values()` ascending and descending to produce rankings
- Retrieved best and worst performing states using `iloc[0]` and `iloc[-1]`
- Identified the highest and lowest high school completion rates per state

**What I understood**
- The `astype(str)` → `.str.replace()` → `pd.to_numeric()` pattern is the standard approach for messy numeric columns in real-world datasets
- `groupby().mean().sort_values()` is a clean aggregation pipeline for group-level insights
- These patterns apply directly to feature engineering in data pipelines

>**What I deliberately skipped**
>- All visualisations including barplots, scatterplots, and heatmaps — not relevant for Data & Feature Infrastructure work at L5. Time better spent on pipeline logic.

**SQL | Basics Completed ✓**
Today I finished the remaining SQL sections and closed out the full SQL basics chapter.

- Completed Views: `CREATE VIEW`, `ALTER VIEW`, `DROP VIEW`
- Completed `CASE WHEN` practice and challenge exercises
- Completed `COALESCE` — essential for NULL handling in feature pipelines
- Completed `NULLIF` — useful for safe division and NULL conversion patterns
- Completed `SUB-SELECT` / Subqueries — important for complex queries in Spark SQL
- Completed `SELF-JOIN`, `CROSS JOIN`, and `NATURAL JOIN` — full JOIN coverage done
- Reviewed SQL Best Practices section in full
  
>The gap that remains — `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`,
`PARTITION BY`, `LAG/LEAD`, and CTEs — is exactly what Month 2 is for.
That work begins tomorrow with MODE SQL.

## April 18, 2026

**Python | Auto Feature Pipeline Project Prototype**

Today I built and completed the first working version of my `auto-feature-pipeline` project. It now runs end-to-end: it fetches product data from an external API, validates selected fields against a config-driven schema, generates derived features, and exports the final output to both JSON and CSV. For now, the pipeline uses a fallback feature generation path, which still allowed me to test the full architecture successfully even without a live OpenAI API key.

The project is structured in a modular way with separate files for fetching, validation, feature engineering, registry/export, and orchestration. What matters most to me is that this is not just a random script — it is my first small step toward thinking like a Data & Feature Infrastructure / ML Platform Engineer: modular code, reproducible flow, configuration-driven logic, and outputs that can later evolve into a real feature pipeline.

At this stage, the project is intentionally simple. I am still in Month 1 of my roadmap, so my current focus remains on Python, SQL, Git, and Linux fundamentals. The more advanced infrastructure and ML platform topics are planned for later stages, not ignored. This project is simply the foundation.

>**Disclaimer:**  
>The inspiration for this project came from yesterday’s `jikan-feature-pipeline`, which effectively marked the starting point for this direction. That project has already been published on GitHub. If this `auto-feature-pipeline` continues to meet my expectations as I refine it, it will also be released publicly.

>**What this project currently demonstrates**
>- API data ingestion
>- Config-driven validation
>- Modular Python project structure
>- Fallback-based feature generation
>- JSON and CSV export
>- Basic debugging of environment, import, and request issues

>**What I want to improve next**
>- Expand the pipeline to use more input fields such as `title`, `description`, `brand`, `rating`, and `stock`
>- Improve feature generation quality so derived features reflect more than only price and category patterns
>- Add stronger logging and better exception handling
>- Add unit tests for each module
>- Improve documentation and architecture notes
>- Make the pipeline easier to extend for other APIs and schemas

>**Roadmap-aligned future improvements**
>- Docker for reproducible local execution and environment consistency
>- AWS for storage, deployment, and infrastructure thinking
>- Spark for scaling feature computation beyond small local datasets
>- Kafka and Airflow for streaming and orchestration patterns
>- Feast for feature store concepts
>- MLflow and Metaflow for experiment tracking and ML workflow management
>- CI/CD for automated testing and deployment
>- Better data quality checks, schema evolution handling, and performance optimization

**Python | Data Cleaning, Feature Engineering & Time-Series Analysis**
- Cleaned and explored a real dataset with Pandas by handling missing values, duplicates, datetime parsing, and feature creation
- Built a robust `Year` feature from mixed-format date strings and used it for time-based analysis
- Learned that mixed date formats often require more careful parsing than a single strict datetime format
- Used `groupby()`, `value_counts()`, and `size()` to answer practical analysis questions from the dataset
- Created descriptive statistics for both numeric and categorical columns to understand the data structure before visualisation
- Analysed launch activity by organisation, country, year, month, and mission status
- Compared total launches, successful launches, and failed launches over time
- Calculated failure rates over time and added a rolling average to smooth short-term fluctuations
- Built a country-level feature from messy location data and normalised historical or non-standard place names
- Converted country data into ISO-3 codes to enable choropleth mapping
- Created Plotly visualisations including line charts, bar charts, histograms, sunburst charts, pie charts, and choropleth maps
- Explored seasonal patterns in launch activity by month and identified the most and least active months
- Compared launch dominance across organisations over time, including the leading organisation in each year
- Analysed Cold War-era launch competition between the USA and the USSR
- Included former Soviet launch locations like Kazakhstan when analysing historical USSR totals
    
    **Most Relevant for My L5 Data & Feature Infrastructure ML Path**
    - Data cleaning and feature engineering from messy raw data
    - Building reliable time-based features from inconsistent date formats
    - Aggregation logic with `groupby()` for metrics, trends, and comparisons
    - Country normalisation and categorical mapping for analytical consistency
    - Time-series analysis with rolling averages and year-on-year trends
    - Interpreting data quality issues before building metrics or dashboards
    - Using domain context to define correct grouping logic, especially for historical datasets
    - Creating reusable analysis patterns that can be applied in future data and ML infrastructure work
    
    **Less Relevant for My L5 Data & Feature Infrastructure ML Path**
    - Exact Plotly syntax for every chart type
    - The specific styling details of each visualisation
    - Building a choropleth map mainly as a visualisation exercise
    - Memorising all country-code mappings by hand
    - Notebook-specific formatting details that are useful now but less important long term
    
    **What This Taught Me**
    - Data engineering thinking matters more than just plotting
    - Correct feature definitions are often more important than the chart itself
    - Historical datasets need domain-aware cleaning, not blind automation
    - Reusable analysis patterns are more valuable than one-off notebook tricks

**SQL | DROP TABLE, CHECK Constraints & Views**
- Learned how to use `DROP TABLE` to remove database tables when they are no longer needed
- Understood how `CHECK` constraints enforce data rules directly inside the database
- Practiced `CREATE TABLE` in an exercise to define structure and constraints from the start
- Reviewed import and export workflows for moving data in and out of SQL systems
- Explored the concept of views as reusable virtual tables for simplified querying
- Learned how to create a view with `CREATE VIEW`
- Learned how to update a view with `ALTER VIEW` or `REPLACE VIEW`
- Learned how to remove a view with `DROP VIEW`


---

## April 17, 2026

**Python | Requests & APIs / Jikan Feature Pipeline**
- Built a full Python data pipeline split into three clear layers
- Used `requests.get()`, `raise_for_status()`, and `response.json()` to ingest data from the Jikan REST API, which provides top anime data from MyAnimeList
- Validated the structure of the API response and cleaned individual anime records
- Normalised missing or incorrectly typed fields by converting them to `None` or fallback values
- Built derived features from the cleaned raw data:
  - `is_high_score` → boolean flag for score >= 8
  - `popularity_bucket` → categorisation into `very_popular`, `popular`, `mid_popular`, or `niche` using `if/elif/else`
  - `is_long_running` → boolean flag for episode counts greater than 24
  - `genre_count` → number of genres using `len()`
  - `has_multiple_genres` → boolean flag for more than one genre
- Added `None` protection across all feature logic because external APIs do not always return complete data
- Saved the transformed feature records as `feature_records.json` in the project folder using `Path(__file__).resolve().parent`

**First Public GitHub Project**
- Published a first public project on GitHub for the first time
- Combined API fundamentals, feature engineering, JSON output, and GitHub publishing in one day
- Finished the day with a strong sense of progress and a concrete public artifact

**Python | HTTP Requests, APIs & Environment Management**
- Built a PDF-to-Audio pipeline using `PyPDF2` to extract text and the
  ElevenLabs SDK to convert it to speech
- Managed API keys securely using `python-dotenv` and a `.env` file —
  keeping secrets out of source code
- Used `Path(__file__).parent` to load `.env` relative to the script location,
  making the path independent of the working directory
- Debugged a series of 401 API errors by distinguishing between
  `needs_authorization` (key not received) and `invalid_api_key` (wrong key)
- Resolved a mismatched virtual environment by identifying the active `.venv`
  path via `which python`
- Installed `ffmpeg` via Homebrew to enable audio playback through the
  ElevenLabs `play()` function
- Generated test PDFs programmatically using `fpdf` to validate the full
  pipeline end-to-end
- Configured VS Code workspace settings to correctly inject environment
  variables into the terminal

**NOTE | Relevance to Core Goal:**
> Proper environment isolation (`venv`), secret management (`.env`) and API
> error handling are foundational skills for any cloud or data engineering workflow.
> Understanding the difference between authentication and authorisation errors
> is directly applicable to working with cloud APIs, AWS SDKs and
> infrastructure tooling at scale.

**Python | Image Processing & Color Extraction**
- Loaded images as NumPy arrays using PIL and inspected array dimensions to understand height, width, and RGB channels
- Reshaped 3D image arrays `(height, width, 3)` into 2D pixel tables `(total_pixels, 3)` using `reshape(-1, 3)`
- Applied KMeans clustering with `n_clusters=10` to extract the top 10 dominant colors from pixel data
- Converted RGB cluster centers into HEX color codes using string formatting for web-friendly output
- Treated images as feature matrices where each pixel is a data point with 3 features: R, G, and B

**NOTE | Relevance to Core Goal:**
> This follows a useful feature engineering pattern: raw image → pixel feature extraction → clustering → dominant features.
> That pattern is directly relevant to production ML pipelines for dimensionality reduction and feature compression.

---

## April 16, 2026

**Python | Statistical Analysis & Hypothesis Testing**
- Visualised data distributions using histograms and superimposed multiple histograms
  with different sample sizes on the same chart
- Smoothed histogram data with a Kernel Density Estimate (KDE) to reveal the
  underlying distribution shape
- Improved KDE accuracy by specifying boundary constraints on the estimate
- Tested for statistical significance using `scipy` and interpreted p-values to
  determine whether an observed effect is real or due to chance
- Highlighted specific time periods on a Matplotlib time series chart to draw
  attention to key events in the data
- Added and configured a legend in Matplotlib for multi-series charts
- Used `np.where()` to conditionally process array elements without loops

**Python | Multivariable Regression**
- Used `sns.pairplot()` to quickly scan for relationships and correlations
  across all variable pairs in a dataset
- Split data into training and testing sets to evaluate model performance
  on unseen data
- Ran a multivariable regression to model a target variable using
  multiple input features simultaneously
- Evaluated the model by inspecting the sign and magnitude of its coefficients
  to confirm they match real-world logic
- Analysed residuals to detect patterns that indicate model weaknesses
  or violated assumptions
- Applied a log transformation to skewed data to improve regression
  model performance and linearity
- Made predictions by feeding custom input values into the trained model

**NOTE | Data Visualisation (lower priority for core goal):**
> Plotly, Matplotlib and Seaborn visualisation techniques 
> are not a primary focus for the path toward Data & Feature Infrastructure
> and ML Platform Engineering. Charts and visual tooling play a minor role
> in production data systems compared to pipeline engineering, distributed processing,
> and feature store architecture. Covered out of completeness and general data literacy.

**SQL | Tables & Columns**
- Reviewed the role of primary keys and foreign keys in relational database design
- Defined column-level constraints to enforce data integrity rules at the schema level
- Created new tables from scratch using `CREATE TABLE` with defined columns and types
- Practiced table creation with real examples in `CREATE TABLE — Praxis`
- Inserted new records into tables using `INSERT INTO`
- Updated existing records conditionally using `UPDATE ... SET ... WHERE`
- Deleted specific rows from a table using `DELETE FROM ... WHERE`
- Modified an existing table structure with `ALTER TABLE` — adding, renaming and dropping columns
- Practiced `ALTER TABLE` operations hands-on in `ALTER TABLE — Praxis` ✅

---

## April 15, 2026

**Python | NumPy & N-Dimensional Arrays**
- Created arrays manually with `np.array()`, as a number sequence with `np.arange()`,
  with random values using `np.random()`, and with evenly spaced values using `np.linspace()`
- Read array dimensions with `.shape` to understand structure before operations
- Sliced and subsetted multi-dimensional arrays with `arr[0:2, 1:]`
- Applied broadcasting to perform operations between arrays of different shapes
- Ran scalar operations directly on arrays — `arr * 2`, `arr + 10` — without loops
- Performed matrix multiplication with `np.matmul(a1, b1)` and the shorthand `a1 @ b1`
- Located non-zero elements by index using `np.nonzero()`
- Loaded an image with `Image.open()` from PIL and confirmed every image is a
  3D array with shape `(height, width, 3)` representing RGB channels

**Python | Seaborn & Linear Regression**
- Cleaned currency strings across multiple columns using nested loops with
  `.astype(str)` before `.str.replace()` to avoid `AttributeError`
- Ran quick data quality checks with `data.duplicated().sum()` and `data.dtypes`
- Filtered rows with `.loc[]` using multiple conditions combined with `&` and `|`
- Used `.query()` as a cleaner, more readable alternative to `.loc[]` for filtering
- Derived decade values from years using floor division — `year // 10 * 10`
- Plotted a scatter chart with an automatic regression line using `sns.regplot()`
- Applied a pre-built dark grid style with `sns.axes_style('darkgrid')`
- Added a third dimension to a scatter chart by mapping a numeric column to bubble `size`
- Fitted a linear regression model with `LinearRegression().fit(X, y)` from scikit-learn
- Interpreted the R² score as a measure of model fit — 0 means no explanatory power, 1 means perfect fit
- Read regression coefficients to quantify how much revenue increases per dollar of budget
- Concluded that higher budgets correlate with higher revenue but with significant spread

**Python | Capstone Data Analysis Project**
- Investigated and handled NaN values across a large real-world dataset
- Converted object and string columns to numeric types for analysis
- Applied `.value_counts()`, `.groupby()`, `.merge()`, `.sort_values()` and `.agg()` in a full analysis pipeline
- Created a rolling average to smooth time-series data and surface underlying trends
- Used `sns.lmplot()` with `row`, `hue`, and `lowess` parameters to show best-fit lines across multiple categories
- Visualised data distribution and descriptive statistics using a Seaborn histogram
- Compared box plots vs time-series analysis to understand how the same data tells different stories depending on the view

**NOTE | Data Visualisation (lower priority for core goal):**
> Plotly, Matplotlib and Seaborn visualisation techniques 
> are not a primary focus for the path toward Data & Feature Infrastructure
> and ML Platform Engineering. Charts and visual tooling play a minor role
> in production data systems compared to pipeline engineering, distributed processing,
> and feature store architecture. Covered out of completeness and general data literacy.

**SQL | Date & Time Functions**
- Got an overview of all available date and time functions in SQL

**SQL | Data Types**
- Learned how SQL categorises data into types: numeric, text, date/time, boolean
- Understood why correct data types matter for storage efficiency and query performance

---

## April 14, 2026

**Python | Pandas & Matplotlib**
- Used `.describe()` to get a quick statistical summary of a DataFrame
- Converted string columns to proper datetime with `pd.to_datetime()`
- Resampled two DataFrames to the same monthly frequency with `.resample('ME').sum()` and `.asfreq('ME')` to align them for comparison
- Smoothed noisy time series data with `.rolling(window=6).mean()` to reveal underlying trends
- Checked data quality with `.isna().values.sum()` before analysis
- Formatted a date-based x-axis using `YearLocator` and `DateFormatter` from `matplotlib.dates`
- Built a dual y-axis chart with `ax.twinx()` to overlay two datasets with different scales on one figure
- Differentiated two lines visually using `linestyle='--'` and `linestyle='-.'` combined with `'o'` and `'^'` markers
- Set `dpi=200` to export high-resolution charts and fine-tuned layout with `xlim`, `ylim`, `linewidth` and HEX colour codes
- Added `.grid()` to make seasonal patterns easier to read across a long time range

**Key Insight | Google Trends as a Leading Indicator**
- Noticed that search interest in *"unemployment benefits"* spiked weeks **before** the official unemployment rate appeared in FRED data
- Confirmed by the COVID spike in April 2020 where UNRATE hit ~14.7% — web searches predicted it earlier
- Concluded that search volume can be used as a real-time economic signal ahead of official reporting

**SQL | String Functions**
- Combined first and last name columns into a single field using `CONCATENATE`
- Located the position of a character inside a string with `POSITION` to split or validate data
- Extracted specific parts of a string by position and length using `SUBSTRING`
- Measured string length with `LENGTH` for data validation and cleaning
- Standardised casing across inconsistent text data with `LOWER` and `UPPER`

**SQL | Mathematical Functions & Operators**
- Applied `+`, `-`, `*`, `/` directly inside `SELECT` to compute derived columns on the fly
- Rounded query results to two decimal places with `ROUND()` for cleaner output
- Used `CEILING()` and `FLOOR()` to round values up or down to the nearest integer
- Retrieved absolute values from negative numbers with `ABS()` for distance and deviation calculations
- Calculated remainders with `MOD()` to filter even/odd rows and implement modulo-based logic

---

## April 13, 2026

**Python | Data Exploration with Pandas**
- Exploring DataFrames with `.head()`, `.tail()`, `.shape`, `.columns`, `.dtypes`
- Finding NaN values with `.isnull().sum()` and cleaning with `.dropna()` / `.fillna(0)`
- Selecting columns with `df['col']` and `df[['col1', 'col2']]`
- Accessing individual cells with `df['col'][index]` and `.loc[index]`
- Finding min/max values and their positions with `.min()`, `.max()`, `.idxmin()`, `.idxmax()`
- Sorting data with `.sort_values()` and adding new columns with `.insert()`
- Renaming columns with `.rename(columns={...})`
- Aggregating and grouping with `.groupby()` + `.sum()` / `.mean()` / `.count()`

**Python | Data Visualisation with Matplotlib**
- Cleaning date formats with `pd.to_datetime()`
- Reshaping DataFrames with `.pivot(index=, columns=, values=)`
- Replacing missing values after pivot with `.fillna(0, inplace=True)`
- Checking for NaN across entire DataFrame with `.isna().values.any()`
- Creating line charts with `plt.plot(x, y)`
- Styling charts with `.figure(figsize=)`, `.xlabel()`, `.ylabel()`, `.ylim()`, `.legend()`
- Plotting multiple lines in one chart
- Smoothing time series data with `.rolling(window=6).mean()`

**SQL | Comments & UNION**
- Inline comments with `--` and block comments with `/* */`
- `UNION` removes duplicates · `UNION ALL` keeps all rows
- Practice Test 2 completed ✅

