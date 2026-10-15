# Internship Experience Prep: Myntra & Atlassian

> **Important.** The **Atlassian** section is reconstructed from the three resume bullets plus standard industry practice for SQS / Spring Boot systems. **Anything marked _(inferred)_ is a plausible design, not a known fact.** Before the interview, replace it with what you actually built, or be ready to say "I worked on X part; Y was owned by another team".
> The **Myntra** section is summarised here; the full version, verified against the internship report, is in `05_Myntra_Deep_Dive.md`.

---

## Part A: Atlassian, Software Engineer Intern, Atlassian Marketplace (May–Jul 2025)

### A.1 Context: what is Atlassian Marketplace?
- Atlassian Marketplace is the store where third-party vendors (partners) publish **apps/plugins** for Jira, Confluence, Bitbucket and other Atlassian products. Customers discover, try, buy and install them there.
- **Publishing an app** means creating the app listing plus versions, metadata, compatibility, pricing and so on, so that it goes live on the Marketplace.
- **The problem your work solved:** the release/publishing process had **manual steps**. A request came in and someone had to check it and create the app. That was slow and error-prone, and it didn't scale with request volume.

### A.2 Resume bullets → plain English

| Resume bullet | Plain-English version |
|---|---|
| Event-driven automation system using Kotlin, Spring Boot, Java and AWS to publish applications on the Marketplace | Instead of humans processing publish requests, a request becomes an **event**. A service reacts to it and does the publishing automatically. |
| Scalable AWS SQS-based event processing pipeline with worker pools; high volumes, fault tolerance, horizontal scalability | Events go into an **SQS queue**. A **pool of workers** pulls and processes them in parallel. If a worker fails, the message is retried. More instances can be added under load. |
| Automated request validation and real-time application creation workflows using Spring Boot | Each request is **validated** (fields, permissions, duplicates) before work starts, and the app is **created immediately** once valid. |

### A.3 Architecture _(flow is inferred from the bullets; adjust to reality)_

```
 Vendor / internal tool
        │  POST /publish-requests  (JSON payload)
        ▼
 ┌─────────────────────────────┐
 │ Spring Boot API (Kotlin)    │
 │  1. Auth check              │
 │  2. Request validation      │──── invalid ──► 400 + error details
 │  3. Persist request status  │
 │     = RECEIVED              │
 │  4. Send message to SQS     │
 └──────────────┬──────────────┘
                │ 202 Accepted (requestId)
                ▼
        ┌───────────────┐   after N failed receives   ┌──────────────┐
        │  AWS SQS queue│ ───────────────────────────► │  Dead-letter │
        └───────┬───────┘        (redrive policy)      │  queue (DLQ) │
                │ long-poll receive (batch ≤10)         └──────────────┘
                ▼
 ┌─────────────────────────────────────┐
 │ Worker service (N instances)        │
 │  each instance = pool of K workers  │
 │  - idempotency check (requestId)    │
 │  - call Marketplace APIs to create  │
 │    app / version / listing          │
 │  - update status = PUBLISHED/FAILED │
 │  - delete message from SQS on success│
 └─────────────────────────────────────┘
```

**Walk-through to say out loud:**
1. A publish request hits the Spring Boot API.
2. It is validated synchronously: required fields, formats, vendor permission, duplicate app key, version compatibility. Bad requests fail fast with a 400 and never enter the queue.
3. Valid requests are recorded and published as a message to SQS. The API returns **202 Accepted** with a request ID, so the client doesn't wait for the slow work.
4. Worker instances long-poll SQS and pull up to 10 messages per call. Each message goes to a thread or coroutine in a bounded pool.
5. The worker performs the app creation. On success it **deletes** the message; on failure it does nothing, and the message reappears after the **visibility timeout** for a retry.
6. After N failed attempts, SQS moves the message to a **DLQ**, where it can be alerted on and inspected.

### A.4 Key concepts the interviewer will probe

**Why event-driven / async instead of a synchronous API?**
- Publishing involves multiple slow downstream calls. Synchronous handling ties up request threads and fails the user's request on any transient error.
- A queue **decouples** producer and consumer, **absorbs spikes** (a release day with many vendors), and gives **free retries**.
- Trade-offs: eventual completion instead of immediate, more moving parts, and a need for status tracking and idempotency.

**Why SQS (and not Kafka)?**
- SQS is fully managed and a natural **work queue**: each message is processed by one worker, then deleted. There's no cluster to run.
- Kafka is better when many independent consumers need the same event stream, when you need replay, or when you need very high throughput with ordering per key.
- At Myntra you used Kafka. Contrasting the two from experience is a strong answer.

**SQS facts to know cold**

| Concept | Detail |
|---|---|
| Standard queue | At-least-once delivery, best-effort ordering, near-unlimited throughput |
| FIFO queue | Ordered within a *message group ID*, exactly-once processing (5-minute dedup window), lower throughput (300 TPS, or 3,000 with batching; higher in high-throughput mode) |
| Visibility timeout | After a receive, the message is invisible for T seconds (default 30 s). Not deleted in time → visible again → reprocessed. Set it longer than worst-case processing time, or extend it with `ChangeMessageVisibility`. |
| Long polling | `WaitTimeSeconds` up to 20 s; fewer empty responses, lower cost |
| DLQ | `maxReceiveCount` in the redrive policy; failed messages are parked for debugging |
| Batch | Receive and delete up to 10 messages per call |
| Retention | Default 4 days, max 14 days |
| Message size | Up to 256 KB (larger payloads go to S3 with a pointer in the message) |

**Worker pool design**
- **Why bounded?** Unlimited threads would exhaust memory and overwhelm the downstream Marketplace APIs and DB. A bounded pool means bounded concurrency, which is effectively a rate limit.
- **Sizing:** I/O-bound work allows more threads than cores. A rule of thumb is `threads ≈ cores × (1 + wait_time / compute_time)`. Tune it with load tests.
- **Backpressure:** if all workers are busy, stop polling. Messages simply wait in SQS, so the queue *is* the buffer.
- In Kotlin this can be coroutines with a `Semaphore` or `Dispatchers.IO.limitedParallelism(k)`. In Java it's an `ExecutorService` / `ThreadPoolExecutor`. _(Say what you actually used.)_

**Fault tolerance**
- Worker crashes mid-processing → the message is never deleted → it reappears after the visibility timeout → another worker retries it.
- Poison message (always fails) → goes to the DLQ after `maxReceiveCount`, so it can't block the pipeline forever.
- Downstream outage → retries with **exponential backoff + jitter** (avoid hammering a recovering service).

**Idempotency (very likely follow-up)**
- At-least-once delivery means **the same message can be processed twice** (for example, a worker finished the work but crashed before deleting the message).
- Fix: an idempotency key (the request ID). Before creating, check "already processed?" in the DB, or use a unique constraint on (vendor, appKey, version), so a duplicate insert fails harmlessly. Status transitions are also guarded: `RECEIVED → IN_PROGRESS → PUBLISHED` only moves forward.

**Horizontal scalability**
- Workers are stateless, and all state lives in the DB or queue, so you can run N instances behind the same queue. SQS hands each message to only one receiver at a time.
- Auto-scale on the `ApproximateNumberOfMessagesVisible` CloudWatch metric (queue depth).

**Validation**
- Syntactic: Bean Validation annotations on DTOs (`@NotBlank`, `@Pattern`, `@Size`) with `@Valid` on the controller.
- Semantic: does the vendor own this app key, is the version newer than the last one, is the product compatibility valid.
- Why validate before enqueueing? To fail fast with a useful error to the caller, and to avoid wasting worker capacity on requests that are guaranteed to fail.

**Spring Boot basics they may ask**
- IoC/DI: Spring creates and wires beans. `@Component`, `@Service`, `@Repository`, `@RestController`. Constructor injection is preferred.
- `@Transactional`: wraps a method in a DB transaction. Note that self-invocation bypasses the proxy.
- Layering: Controller → Service → Repository. DTOs vs entities.
- Kotlin vs Java: null safety, data classes, coroutines, concise syntax, full Java interop (hence "Kotlin, Java" on the resume).

### A.5 Likely interview questions with answer outlines

1. **"Explain your Atlassian project end to end."** Use the A.3 walkthrough, about 90 seconds, then stop.
2. **"What happens if a worker dies halfway?"** Visibility timeout → redelivery → idempotency check prevents duplicates.
3. **"How do you guarantee an app isn't published twice?"** Idempotency key plus a DB unique constraint; also the FIFO dedup ID option.
4. **"How would you handle 10× traffic?"** Queue absorbs the burst. Autoscale workers on queue depth. Make sure downstream can take it (rate-limit workers). Batch receives.
5. **"Why not just use a thread pool inside the API server?"** In-memory work is lost on a crash and can't scale independently. The queue gives durability, retries, and separate scaling of API vs workers.
6. **"How did you test it?"** Unit tests with mocked SQS client, integration tests with LocalStack _(inferred, use what you did)_, and failure injection (throw in the worker and confirm retry/DLQ).
7. **"How did you monitor it?"** Queue depth, age of oldest message, DLQ count alarms, success/failure rates, processing latency.
8. **"What was the hardest part?"** Pick a real one. Common candidates: choosing the visibility timeout vs long-running work, making creation idempotent, or handling partial failures across several downstream calls.
9. **"What would you improve?"** A status/notification webhook back to vendors, an outbox pattern so the DB write and the enqueue are atomic, and finer-grained retries per step.

> **Outbox pattern** (good "improvement" answer): writing to the DB and sending to SQS are two systems, so a crash between them can lose the event. Instead, write the request **and** an "outbox" row in the same DB transaction, and have a relay publish outbox rows to SQS.

### A.6 STAR story: Atlassian

- **S:** Marketplace app publishing involved manual steps, causing slow, error-prone releases.
- **T:** As an intern, automate the flow so requests are validated and published without manual intervention, reliably at high volume.
- **A:** Designed an event-driven pipeline: Spring Boot API with validation, enqueue to SQS, worker pools that create apps, retries and DLQ for failures, and idempotent processing.
- **R:** Manual effort removed from the publishing path, faster and more reliable release cycles, and a pipeline that scales by adding workers. _(Add any real number: requests/day, time saved, number of apps.)_

---

## Part B: Myntra, Software Engineer Intern, Product-SizeFit team, PDP org (14 Jul – 5 Dec 2025)

> **Full deep dive, verified against the internship report: `05_Myntra_Deep_Dive.md`.** This is the summary.

### B.1 One-paragraph pitch
"On Myntra's Product Detail Page org, I worked on the **Ratings & Reviews microservice in Go**. Variants of the same product showed separate ratings; a new colour could show 'No Ratings' while the base product had thousands. I fixed both the read and write flows. They fetch related styles from **Accio** (the style-relationship service) in **bulk batches of 20**, cache the related set in **Aerospike** (7-day TTL), and write the aggregated rating to **every style in the cluster**. A **token-bucket rate limiter** (per pod) protects Accio, and rejected events go to a **Kafka error queue** for eventual consistency. I fixed a non-deterministic SQL bug caused by **Go's random map iteration order**. I merged **two queries into one** (50% fewer DB connections for the computation) and ran a **cache warmup for 5.7M styles** over 5 runs. Earlier, I onboarded the service onto a **production-like stage environment**: MySQL with mock data, Aerospike, Airbus and DB credentials in Vault."

### B.2 Corrections to earlier guesses (the report is the source of truth)

| Topic | What actually happened |
|---|---|
| **50% fewer DB connections** | Two CASE-based SQL queries (ratings, then review/image counts) merged into **one parameterised query**, with aggregation in Go. It was **not** connection-pool tuning. |
| Aggregation | Histogram merge per cluster in Go, using a single map keyed by `(mainID << 32) \| rating` |
| Related styles | Come from **Accio**, fetched in batches of 20 (recommended 20–25) |
| Rate limiter | Token bucket from the team's existing framework, configurable per pod; started at 180 RPM per pod |
| Error queue retries | Instant (no delay function was available in the Go services). A good "what I'd improve" point. |
| Warmup | 5,710,991 styles; run 1 refreshed only 1.28M; pacing (4 ms between publishes) plus a rate-limit adjustment fixed it; 100% after 5 runs |

### B.3 STAR summary
- **S:** Related product styles showed fragmented or missing ratings across 5.7M+ styles, and the cache never updated the related styles.
- **T:** Automate consistent aggregated ratings without overloading Accio or MySQL.
- **A:** Bulk Accio flow, related-set caching, a write-to-all-related-styles write flow, token-bucket rate limiting, a Kafka error queue, a SQL fix and merge (2 queries → 1), and an overnight 5.7M cache warmup.
- **R:** 100% of styles refreshed, consistent ratings across variants, 50% fewer DB round trips for the computation, no manual intervention, and passing load and memory-leak tests.

## Part C: Behavioral stories mapped to common questions

Zomato's Round 1 is mostly DSA plus resume, but a few behavioral questions often appear at the end. Prepare one story per row and reuse them.

| Question | Story to use | Key line |
|---|---|---|
| Tell me about a challenging technical problem | Myntra ratings pipeline at 5.7M scale | "The hard part wasn't the math, it was running it safely against production MySQL." |
| A time you improved performance | Myntra: 2 CASE-based queries merged into 1 (50% fewer DB round trips); 5.7M cache warmup | Show before → change → after |
| A time something failed / a bug | Myntra: warmup run 1 refreshed only 22% (rate limit), or the non-deterministic Go-map SQL bug (05, section 6) | What broke, how you found it, the fix, what you changed so it can't recur |
| Working with ambiguity | Atlassian: automating a manual process with unclear requirements | You asked stakeholders, wrote a small design doc, and iterated |
| Learning something fast | Go at Myntra / Kotlin at Atlassian | Productive within N weeks; how you learned (codebase reading, mentors) |
| Teamwork / leadership | Hackathons (Hack-A-Sol win, iDEA top 10) | Your role, a disagreement, how it was resolved |
| Disagreement with a mentor/teammate | Design choice (e.g., sync vs async) | Data over opinion; disagree and commit |
| Why Zomato? | Scale + real-time systems + overlap with your stack (Go, Kafka, MySQL, caching) | Mention a Zomato/Blinkit product you use and an engineering problem in it (e.g., delivery-time prediction, rider assignment) |
| Where do you see yourself in 2–3 years? | Strong backend engineer owning a service end to end | Keep it grounded |

**Rules for STAR answers:** 60–90 seconds. Say "I" for what you did and "we" for the team. End with a number or a concrete outcome. Have one "what I'd do differently" ready for every story.
