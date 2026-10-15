# Resume & Skills Brief: Zomato SDE-1, Round 1

> Candidate: **Vaideeka Agrawal**, B.Tech CSE, IIT Bhilai (2022–2026), CGPA 8.4
> Goal of this file: know **every line** on the resume well enough to answer two levels of "why?" and "how?", with priority on what a Zomato backend interviewer is likely to probe.

---

## 0. How to use this file

| Priority | Meaning | Action |
|---|---|---|
| **P0** | Almost certain to be asked in Round 1 | Practise answering out loud, 2–3 times |
| **P1** | Likely if the interviewer has time or digs | Know the concept and one concrete example |
| **P2** | Unlikely for Zomato; listed on resume | Know a 2-line definition so you aren't caught blank |

**Golden rule:** anything on the resume is fair game. If you can't defend a skill for 2 minutes, you should be able to say *"I've used it at a basic level in X, mainly for Y"*. That is better than bluffing.

---

## 1. Self-introduction

### 1.1 Sixty-second version (default)

> "Hi, I'm Vaideeka, a final-year Computer Science student at IIT Bhilai with an 8.4 CGPA.
> I've done two backend internships. At **Myntra**, on the Product Detail Page team, I built an **aggregated ratings pipeline in Go and MySQL** that keeps ratings consistent across related product styles for **over 5.7 million products**. I also optimized query and cache access patterns, which **cut database connections by 50%**.
> Before that, at **Atlassian Marketplace**, I built an **event-driven app-publishing system** in Kotlin and Spring Boot on **AWS SQS with worker pools**.
> Outside work, I build full-stack ML products. **TraceX** is a graph-based anti-money-laundering investigation platform, and **SupplySenseAI** predicts shipment delays from live weather and disaster data.
> I enjoy problem solving. I'm excited about Zomato because real-time food delivery at India's scale is a hard systems problem, and the work I did on caching, queues and high-traffic services maps directly onto it."

### 1.2 Two-minute version (when asked to "walk me through your resume")

1. **Education.** IIT Bhilai, CSE, 8.4 CGPA. Coursework: DSA, DBMS, OS, Cloud Computing, Information Security, ML.
2. **Myntra (Jul–Dec 2025), Product Detail Page / Ratings & Reviews.**
   - *Problem:* related styles of one product (for example, the same t-shirt in 5 colours) each had their own ratings, so customers saw inconsistent or sparse ratings.
   - *What I built:* a pipeline that groups related styles, computes an aggregated rating, and syncs it across them. It uses bulk APIs, cache warm-up, rate limiting and an error queue for retries.
   - *Impact:* consistent ratings for 5.7M+ products and a 50% drop in DB connections after the query and cache optimization.
   - Also built a production-parity **stage environment** (MySQL, Aerospike, Kafka, downstream services) for safe pre-deployment testing.
3. **Atlassian (May–Jul 2025), Marketplace.** Publishing apps used to need manual steps. I built an event-driven flow: request → validation → SQS → worker pool → app created in real time. It is fault-tolerant and scales horizontally.
4. **Projects.** TraceX (AML graph investigation; FastAPI, Next.js, NetworkX, XGBoost, Isolation Forest, LLM copilot). SupplySenseAI (shipment-delay risk; XGBoost, SHAP, NetworkX, Docker, Cloud Run).
5. **Achievements.** Amazon ML Challenge 2024 top 0.5% (rank 334 of 75,000+). iDEA Hackathon 2026 top 10 of 27,000+ teams. Hack-A-Sol 2024 AI/ML winner and 1st runner-up overall. GSSoC 2026 contributor.
6. **Close.** "I like owning a system end to end, from the data model to caching to how it behaves under load. That's what I'd want to do at Zomato."

### 1.3 Delivery tips
- Say numbers slowly and confidently: **5.7 million**, **50%**, **top 0.5%**.
- Don't list every skill. The interviewer has the resume; your job is to **steer** them toward topics you're strongest in (Myntra pipeline, caching, queues).
- End on a hook ("...the ratings pipeline was the most interesting problem I've worked on") so the first follow-up lands where you're prepared.

---

## 2. Line-by-line resume walkthrough with expected questions

### 2.1 Education: IIT Bhilai, B.Tech CSE, CGPA 8.4

| Likely question | How to answer |
|---|---|
| Favourite subject? | Pick **DBMS** or **OS**, because both connect to your Myntra work (indexes, connection pools, caching). Avoid naming a subject you can't defend in depth. |
| Why CSE / why software? | One honest line + one concrete moment (e.g., first time a system you built handled real users). |
| Why Zomato and not an ML role, given all the ML on your resume? | "My internships were backend, and that's where I want to grow. ML is a tool I use in projects. At Zomato's scale, backend problems like dispatch, caching, ETA and search are where the impact is, and many of them use ML too." |

### 2.2 Myntra: Software Engineer Intern, Product-SizeFit team, PDP (14 Jul – 5 Dec 2025) **[P0]**

The full deep dive, verified against the internship report, is in `05_Myntra_Deep_Dive.md`. This is the short version of each bullet.

**Bullet 1: Aggregated ratings pipeline (Go + MySQL, bulk APIs, cache warmup, rate limiting, error queues, 5.7M products)**
- **Style:** a style ID is one version of a product, such as one colour. Related styles form a **cluster**. **Accio** is the internal source of truth for which styles are related.
- **Problem:** the cache held per-style ratings, and a new rating updated only that one style.
- **Fix:**
  1. Fetch related styles from Accio in **batches of 20**.
  2. Cache the related set in **Aerospike** (7-day TTL).
  3. Recompute the cluster's aggregate and **write it to every related style**.
- **Aggregate:** a histogram merge, summing per-star counts across the cluster, then a weighted average. **Never average the averages.**
- **Rate limiting:** a **token bucket**, configurable per pod (started at 180 RPM per pod), on calls to Accio.
- **Error queue:** a Kafka queue for events rejected by the limiter, giving eventual consistency. Retries were instant; backoff is a good "what I'd improve" point.
- **Cache warmup:** **5,710,991 styles over 5 runs**. Run 1 refreshed only 1.28M. Adding a 4 ms pacing delay between publishes and adjusting the rate limit took it to 100%.

**Bullet 2: Optimized queries and cache access, 50% fewer DB connections**
- **What actually changed:** two SQL queries with hard-coded `CASE` style-to-group maps (one for ratings, one for review and image counts) became **one parameterised query**, with the aggregation moved into Go. That is **2 connections/round trips down to 1** for each aggregated-ratings computation.
- **A bug fixed along the way:** the CASE clauses were generated from a **Go map, whose iteration order is random**. Because SQL `CASE` takes the first match, styles were non-deterministically grouped under the wrong main ID.
- **The Go trick:** a single map keyed by `(mainID << 32) | rating`, instead of a map of maps.
- **"50% of what?"** It's 50% fewer DB round trips for the aggregated-ratings computation. It is not a service-wide number, and it did not come from connection-pool tuning.

**Bullet 3: Production-parity stage environment (Go, MySQL, Aerospike, Kafka, downstream services)**
- **Before:** the service existed only in dev and production.
- **What you did:**
  1. Configured stage endpoints and timeouts for its dependencies.
  2. Created the MySQL schema with mock data.
  3. Sized and provisioned Aerospike.
  4. Created DB read/write credentials and **Airbus** (event) credentials, stored in **Vault**.
  5. Deployed, then validated via API calls and logs.
- **Know these:**
  - **Aerospike:** a distributed key-value store with the index in RAM and data on SSD.
  - **Kafka:** a partitioned log, with consumer groups and offsets.

### 2.3 Atlassian: Software Engineer Intern, Marketplace (May–Jul 2025) **[P0]**

**Bullet 1: Event-driven automation to publish apps (Kotlin, Spring Boot, Java, AWS)**
- *Event-driven vs request-response:* the publisher submits a request and gets an immediate acknowledgement. The heavy work happens asynchronously when a consumer processes the event. This decouples components, absorbs spikes and allows retries.

**Bullet 2: SQS pipeline with worker pools (high volume, fault tolerance, horizontal scaling)**
- Know SQS: **standard queue** (at-least-once delivery, best-effort ordering) vs **FIFO queue** (exactly-once processing, ordered within a message group). **Visibility timeout**: a message is hidden while a worker processes it and reappears if the worker doesn't delete it in time. **DLQ** via a redrive policy after N receives. **Long polling** reduces empty receives.
- *Worker pool:* a fixed number of threads/coroutines pulling messages. Bounded concurrency protects downstream services.
- *Fault tolerance:* if a worker crashes, the message becomes visible again and another worker picks it up. That is why **processing must be idempotent**, because at-least-once delivery means duplicates can happen.
- *Horizontal scaling:* add more consumer instances. SQS distributes messages across them with no coordination needed.

**Bullet 3: Automated request validation + real-time application creation (Spring Boot)**
- Validation: schema/field checks (Bean Validation `@Valid`, `@NotNull`), business rules (duplicate app key, vendor permissions). Fail fast with clear errors before queuing.

### 2.4 TIH-IoT, IIT Bombay (Chanakya Fellowship): Research Intern (Mar–Dec 2024) **[P1]**
- **React Native** app for data visualization. ML to assess **soil suitability for UAV landing** at **75% precision**.
  - *Expect:* "Why precision and not accuracy?" → A false positive (calling unsafe soil safe) is the costly error for a drone landing, so precision on the "safe" class matters most. Know precision = TP/(TP+FP), recall = TP/(TP+FN), F1.
- **Wireless SSH data transfer pipeline** from Raspberry Pi to mobile, **99% transmission accuracy**.
  - *Expect:* "How did you measure 99%?" (checksums / matching received vs sent records). "What caused the 1%?" (dropped connections; retries / resume).
  - Know SSH basics: encrypted channel over TCP port 22, key-based auth, SCP/SFTP built on it.

### 2.5 RDSO, Indian Railways: Summer Intern (Jun–Jul 2024) **[P2]**
- **Flutter** Android app with dynamic input forms, popup notifications and interactive graphs driven by user input.
- Be ready for one line on Flutter: Dart-based cross-platform UI toolkit; widget tree; stateful vs stateless widgets.

### 2.6 Projects **[P0]**
Full deep dive in `03_Projects_TraceX_SupplySenseAI.md`. Minimum you must be able to do:
- Draw the architecture on paper in under 2 minutes.
- Explain **why** each technology was chosen (FastAPI, NetworkX, XGBoost, Isolation Forest, SHAP).
- Explain one **graph algorithm** used in each (likely BFS/DFS over transaction or supply graphs). This bridges nicely into DSA.

### 2.7 Achievements **[P1]**

| Achievement | 1-line story to prepare |
|---|---|
| Amazon ML Challenge 2024, top 0.5% (rank 334 / 75,000+) | What the problem was (2024 edition: extracting entity values like weight/dimensions from product images), your approach, and what moved your score most. |
| iDEA Hackathon 2026 (Union Bank of India), top 10 of 27,000+ teams | Likely linked to TraceX (AML fits a bank hackathon). Confirm and be ready to say so. |
| Hack-A-Sol 2024, IIIT Raipur, AI/ML track winner, 1st runner-up overall | What you built in 24–36 hrs and your role in the team. |
| GSSoC 2026 contributor | Name **one** specific PR (repo, what it fixed). Vague open-source claims get probed. |

---

## 3. Skills: what to know, priority for Zomato

Zomato's backend is mostly **Go, Java/Kotlin, some Python/PHP legacy**. It uses **MySQL/Aurora, Redis, Kafka, Elasticsearch, Kubernetes on AWS**. (Verify against `01_Zomato_Interview_Guide.md` research notes.) Your resume overlaps heavily, so lean into it.

### 3.1 Programming languages

| Skill | Priority | What you must know |
|---|---|---|
| **C++** | **P0** (DSA language) | STL: `vector`, `unordered_map` vs `map` (hash vs red-black tree, O(1) avg vs O(log n)), `set`, `priority_queue` (max-heap by default; min-heap via `greater<>`), `deque`, `stack`, `queue`, `pair`, `sort` with comparator, `lower_bound`/`upper_bound`. Pass by reference, `auto`, range-for. Iterator invalidation in `vector` after `push_back`. |
| **Go** | **P0** (used at Myntra; Zomato uses Go) | Goroutines vs OS threads (cheap, M:N scheduling), channels (buffered vs unbuffered), `select`, `sync.WaitGroup`, `sync.Mutex`, `context` for cancellation/timeouts, worker-pool pattern, error handling (`if err != nil`, wrapping with `%w`), interfaces (implicit implementation), slices vs arrays (slice = pointer+len+cap), maps aren't safe for concurrent writes, `defer`. |
| **Java** | P1 | OOP pillars, `HashMap` internals (array of buckets, hashing, collisions → linked list → red-black tree at 8 entries in Java 8+), `equals`/`hashCode` contract, `ArrayList` vs `LinkedList`, `ConcurrentHashMap`, checked vs unchecked exceptions, `final`/`finally`/`finalize`, JVM memory (heap/stack), GC basics, `String` immutability. |
| **Kotlin** | P1 | Null safety (`?`, `?.`, `?:`, `!!`), data classes, coroutines (vs threads), `val` vs `var`, extension functions, interop with Java. |
| **Python** | P1 | Lists/dicts/sets, comprehensions, generators, GIL (why threads don't parallelize CPU work), decorators, `async`/`await` (FastAPI). |
| **C** | P2 | Pointers, memory (malloc/free), stack vs heap. |
| **JavaScript / TypeScript** | P2 | Event loop, promises, async/await, `let`/`const`/`var`, closures; TS adds static types. |

> **Go worker pool, worth being able to write from memory** (it ties Myntra + Atlassian together):
> ```go
> func workerPool(jobs <-chan Job, results chan<- Result, n int) {
>     var wg sync.WaitGroup
>     for i := 0; i < n; i++ {
>         wg.Add(1)
>         go func() {
>             defer wg.Done()
>             for j := range jobs {
>                 results <- process(j)
>             }
>         }()
>     }
>     wg.Wait()
>     close(results)
> }
> ```

### 3.2 Web & app development

| Skill | Priority | What you must know |
|---|---|---|
| **SQL** | **P0** | Joins (inner/left/right/full/self), `GROUP BY` + `HAVING`, window functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`), subqueries vs joins, indexes (B+ tree, composite index + leftmost-prefix rule, covering index), `EXPLAIN`, transactions + ACID + isolation levels, normalization (1NF→BCNF). Classic queries: Nth highest salary, duplicates, top-K per group, customers with no orders. See the Zomato guide. |
| **Spring Boot** | P1 | Dependency injection / IoC, `@RestController`, `@Service`, `@Repository`, `@Autowired`, bean scopes, `@Transactional`, layered architecture (controller → service → repository). |
| **FastAPI** | P1 | Async Python web framework, Pydantic validation, auto OpenAPI docs, dependency injection with `Depends`. Used in both projects. |
| **React / Next.js** | P1 | Components, props vs state, hooks (`useState`, `useEffect`), virtual DOM + reconciliation; Next.js = React + routing + SSR/SSG. |
| **REST** | **P0** | HTTP methods + idempotency (GET/PUT/DELETE idempotent, POST not), status codes (200, 201, 204, 400, 401, 403, 404, 409, 429, 500, 503), statelessness, pagination (offset vs cursor), versioning. |
| TypeScript, PHP, HTML/CSS, React Native, Flutter | P2 | One-liners. Mention PHP only if asked; Zomato's early stack was PHP, which is a nice conversational fact. |

### 3.3 AI/ML **(P1 for project questions; P2 otherwise)**
Zomato Round 1 for SDE won't test ML theory unless you steer there. But your projects use ML, so know these well enough for project questions:

| Concept | 2–3 line answer |
|---|---|
| **XGBoost** | Gradient-boosted decision trees: trees added sequentially, each fitting the residual errors (gradients) of the previous ensemble. Has regularization, handles missing values, strong on tabular data. |
| **Isolation Forest** | Unsupervised anomaly detection. Random trees split features randomly; anomalies are isolated in **fewer splits** (shorter average path length). Good for fraud where labels are scarce. |
| **SHAP** | Shapley values from game theory: each feature's contribution to a single prediction relative to the baseline. Gives per-prediction explanations ("delay risk high because rainfall +0.3, port congestion +0.2"). |
| **Precision / Recall / F1 / ROC-AUC** | Know formulas and when each matters (fraud → recall; drone landing "safe" → precision). |
| **Class imbalance** | Fraud is rare. Use `scale_pos_weight`, resampling (SMOTE), PR-AUC instead of accuracy. |
| **LLMs / RAG / LangChain** | LLM = transformer trained for next-token prediction. RAG = retrieve relevant docs → put them in the prompt → generate grounded answers. LangChain = orchestration library. |
| **Transformers** | Self-attention: each token attends to all others; Q/K/V; positional encoding. |
| Adversarial ML, CV, NLP, PyTorch, Streamlit | P2: one line each. |

### 3.4 Additional tools

| Skill | Priority | What you must know |
|---|---|---|
| **Kafka** | **P0** (Myntra + Zomato use it) | Topic → partitions → ordered offsets. Producer chooses the partition by key (same key → same partition → ordering per key). Consumer group: each partition is consumed by exactly one consumer in the group. Offsets committed for at-least-once delivery. Retention-based, not delete-on-read (vs SQS). Replication factor, leader/follower, ISR. |
| **Kafka vs SQS** | **P0** (you've used both!) | Kafka: log, replayable, high throughput, ordering per partition, consumer pulls with offsets, you manage the cluster (or MSK). SQS: managed queue, message deleted after processing, visibility timeout, no replay, simpler. *"I used SQS at Atlassian because it's fully managed and we needed a work queue; Myntra uses Kafka for streaming events to many consumers."* |
| **Docker** | P1 | Image vs container, layers + caching, Dockerfile basics, container vs VM (shared kernel vs full guest OS). |
| **Kubernetes** | P1 | Pod, Deployment, Service, ReplicaSet, HPA (horizontal pod autoscaler), ConfigMap/Secret, liveness vs readiness probes. |
| **Git** | P1 | merge vs rebase, cherry-pick, resolving conflicts, `git stash`, branching strategy. |
| **Linux / Shell** | P2 | `grep`, `awk`, `ps`, `top`, `kill`, file permissions (`chmod 755`), pipes. |
| **GCP** | P2 | Cloud Run (serverless containers; used for SupplySenseAI). |
| **System Design** | **P1** | You wrote it on the resume, so expect at least one design question. See below. |

### 3.5 System design: minimum knowledge because it's on the resume

Know these concepts with one example each, ideally from your own internships:

- **Caching:** cache-aside (Myntra), TTL, eviction (LRU/LFU), invalidation, cache stampede / thundering herd (fix: warm-up, request coalescing, jittered TTLs). **Implement LRU cache** is a frequent DSA question; see the Zomato guide.
- **Rate limiting:** token bucket (refills at rate r, capacity b, allows bursts), leaky bucket (smooth constant outflow), fixed window (simple, but boundary bursts), sliding window log/counter. Distributed rate limiting with Redis (`INCR` + `EXPIRE`, or Lua scripts for atomicity).
- **Message queues:** decoupling, buffering spikes, retries, DLQ, at-least-once + idempotency (Atlassian).
- **Database scaling:** indexing, read replicas, sharding (by user_id / city), connection pooling (Myntra).
- **Consistency:** CAP theorem, eventual consistency (aggregated ratings tolerate a few minutes of staleness, which is a good trade-off example).
- **Load balancing:** round robin, least connections, consistent hashing.
- **Observability:** logs, metrics, traces; alerting on error-queue depth.

---

## 4. Coursework talking points

| Course | 1 link to your work |
|---|---|
| Data Structures & Algorithms | Graph traversal in TraceX (fund-flow tracing = BFS/DFS on a directed graph). |
| Database Systems | Myntra query optimization, indexes, connection pooling. |
| Operating Systems | Worker pools/threads (Atlassian), goroutines (Go), concurrency control. |
| Cloud Computing | AWS SQS (Atlassian), GCP Cloud Run (SupplySenseAI), Docker/K8s. |
| Information Security | SSH pipeline (IIT Bombay), AML/fraud detection (TraceX). |
| Machine Learning | XGBoost, Isolation Forest, SHAP in projects; Amazon ML Challenge. |

---

## 5. Risky claims: prepare before the interview

These lines invite probing. Have a crisp, honest answer for each.

1. **"Kubernetes", "Kafka", "System Design" in Skills.** Expect "What's a consumer group?" or "What's a pod?". Revise section 3.4.
2. **"Adversarial ML", "Computer Vision", "Transformers", "Generative AI".** Long ML list for an SDE role. If asked, tie each to a concrete project or course. If you can't, say "coursework-level exposure".
3. **"5.7 million products" and "50% fewer DB connections".** 5.7M is the count of live styles refreshed in the cache warmup (5,710,991). The 50% is 2 queries/connections becoming 1 for each aggregated-ratings computation, not a service-wide figure.
4. **"75% precision" (UAV landing).** Expect "Is 75% good enough for a drone?" Answer: it was a research prototype. In production you'd want a human-in-the-loop or a conservative threshold that trades recall for higher precision.
5. **"99% transmission accuracy".** Expect "How measured?" (checksums / record counts).
6. **Many languages (C, C++, Python, Java, Go, JS, Kotlin, PHP, TS, Dart).** Expect "Which is your strongest?" → **C++ for DSA, Go for backend**. Say this clearly at the start of coding rounds.
7. **Overlapping dates.** Atlassian May–Jul 2025 and Myntra Jul–Dec 2025 meet in July; TIH-IoT Mar–Dec 2024 overlaps RDSO Jun–Jul 2024. Expect "How did you manage both?" (e.g., TIH-IoT was a part-time/remote fellowship). Have a one-line answer.
8. **"Leadership, Management" in soft skills.** Have one STAR story (hackathon team lead?). See `04_Experience_Myntra_Atlassian.md`, Behavioral section.
9. **Projects with "LLMs" and "copilot".** Expect "What happens if the LLM hallucinates?" (grounding in case data, showing sources, keeping the human investigator in the loop).
