# Myntra Internship Deep Dive: Ratings & Reviews Microservice

> **Source:** the internship report (Dec 2025), 29 pages, signed off by the team's Senior Manager. Every fact, number and name below comes from that report. Sections marked **"Interview angle"** are prep added on top: likely questions, trade-offs and improvements.
> **Role:** Software Engineer Intern, **Product-SizeFit team, Product Detail Page (PDP) org**, Myntra.
> **Dates:** 14 July to 5 December 2025.
> **Service:** the **Ratings & Reviews microservice, written in Go**.

---

## 0. Quick reference: numbers and names to remember

| Item | Value |
|---|---|
| Live product styles covered | **5.7M** (5,710,991 exactly) |
| DB queries for the aggregated-ratings object | **2 → 1** (a 50% reduction in DB round trips/connections) |
| Batch size for bulk calls to Accio | **20 styles per batch** (Accio team recommended 20–25) |
| Aerospike TTL for related-style sets and aggregated ratings | **7 days** |
| Rate limiter | **Token bucket**, configurable **per pod**; started at **180 RPM per pod** |
| Cache warmup | **5 runs** to reach **100%** of 5,710,991 styles |
| Warmup fix that worked | Adding a **4 ms delay** between event publications, plus raising the rate limit |
| Error-queue retry delay | **Instant**: Go services had no delay function available |
| Aggregation key trick | `key = (mainID << 32) \| rating` in a single Go map |
| Tests run before release | **Memory leak test** and **load test** |

**Internal systems named in the report** (say the names confidently, and explain each in one line):

| Name | What it is |
|---|---|
| **Accio** | Internal service that is the **source of truth for relationships between styles** (which styles are variants of the same product) |
| **Aerospike** | In-memory/SSD **NoSQL key-value store**, used as the cache |
| **Kafka** | Event streaming for data communication between services, including the consumer queue and the **error queue** |
| **Airbus** | Myntra's **event communication** layer between services. You created its credentials for the stage environment. |
| **Bolt** | Where `IndexRatingAndPublish` publishes the computed aggregated ratings. *Treat it as a downstream publish target; if asked, describe it only at that level.* |
| **Vault** | Secrets manager where you stored the DB and Airbus credentials |
| **Style service** | Calls the ratings service with a list of style IDs to get ratings for the PDP |
| **MySQL** | The ratings and reviews database: aggregate-rating, review, image and video tables |

**Glossary from the report:**
- **Style ID:** a unique identifier for one specific version of a product, e.g. one colour.
- **Related styles:** style IDs that are variations of the same base product.
- **Cluster:** the full set of related style IDs for one base product.
- **Aggregated rating:** one combined rating across the whole cluster.
- **RPM:** requests per minute.

---

## 1. Pitches

**30 seconds**
> "At Myntra I was on the Product Detail Page org, working on the Ratings & Reviews microservice in Go. My main project fixed **aggregated ratings**. Variants of the same product, like different colours, showed separate ratings, so a new colour could show 'No ratings' while the base product had thousands. I built the flow that fetches related styles from Accio in **bulk batches of 20**, caches the full related set in **Aerospike**, and recomputes and writes the aggregated rating for **every style in the cluster**. I protected Accio with a **token-bucket rate limiter**, sent rate-limited events to a **Kafka error queue** for eventual consistency, and fixed a SQL bug caused by **Go's random map iteration order**. I merged **two queries into one** for 50% fewer DB round trips, and ran a **cache warmup for 5.7 million styles**. Before that, I onboarded the service onto a production-like **stage environment**."

**10 seconds:** "I fixed inconsistent product ratings across 5.7 million Myntra product variants, end to end: Go, MySQL, Aerospike, Kafka, rate limiting and a full cache warmup."

---

## 2. Project 1: Ratings stage onboarding

### 2.1 Problem
The Ratings & Reviews service existed only in **production and dev**. There was no production-like environment to validate changes before production deployment.

### 2.2 What you did (5 tasks)
1. **Configurations:**
   - Worked out which services the ratings service talks to.
   - Added their **stage endpoints**.
   - Configured connection credentials and **timeouts**.
2. **Database:**
   - Created the **MySQL schema** on stage, mirroring production.
   - Set up the ratings DB.
   - **Populated it with mock data** (no production data).
3. **Caching layer:**
   - **Sized and provisioned the storage and compute** for **Aerospike** on stage.
   - Set it up to reduce DB load and speed up PDP responses.
4. **Credentials:**
   - Created **read and write DB credentials**.
   - Created **Airbus credentials** for events to and from the service.
   - Stored all of them in **Vault**.
5. **Deploy and test:**
   - Deployed to stage.
   - Validated the DB and cache connections by calling APIs and checking responses.
   - Verified the deployment logs; there were no deployment errors.

### 2.3 Impact
- Production parity for **integration testing** of service-to-service calls.
- Developers can test without production deploys.
- A safe place to debug and to test cache strategies and DB queries.

### 2.4 Interview angle

| Likely question | Answer outline |
|---|---|
| Why does a stage environment matter if you have dev? | Dev is usually partial: mocked dependencies, different configs. Stage mirrors production's topology (real DB engine, cache, event bus, downstream stage endpoints), so integration, config and timeout bugs show up before production. |
| Why mock data, not a production copy? | Privacy and compliance (user reviews are user data), plus size and cost. Mock data must still cover the edge cases: styles with no ratings, large clusters, images and videos. |
| Why are timeouts important in service config? | Without timeouts, a slow dependency ties up goroutines and connections, and the failure cascades. Set timeouts based on the dependency's p99 latency; pair them with retries and circuit breakers. |
| Why Vault? | Secrets never live in code or config repos. You get access control, auditing and rotation. *(Contrast: an API key hard-coded in a deploy script is exactly what Vault prevents.)* |
| How did you size the Aerospike cluster? | From the expected key count × average record size × replication factor, plus headroom; compute from the expected QPS. *Give the real numbers only if you remember them; otherwise explain the method.* |
| What's Aerospike vs Redis? | Both are key-value stores. Aerospike is built for **SSD/flash plus RAM** (index in RAM, data on SSD), so it holds very large datasets cheaply with sub-millisecond latency, and it's distributed with automatic sharding. Redis keeps everything in RAM and has richer data structures. Myntra uses Aerospike for large catalogue caches. |

---

## 3. Project 2: Aggregated ratings (the main story)

### 3.1 The problem
- Users saw **per-style ratings** instead of one rating for the product. For example, two colour variants of the same item showed **4.4★ (1.6k ratings)** and **4.3★ (3.5k ratings)**, and a new colour might show **"No Ratings"**.
- **Root cause:** related styles were **not automatically inserted or updated in the cache**. After a new rating, the newly computed aggregated rating was **not written to the other related styles** in the cache.

### 3.2 The four issues you identified

**Issue 1: Read flow (cache refresh of existing ratings).**
1. The style service calls the ratings service with style IDs.
2. The ratings service reads Aerospike.
3. On a cache miss, it publishes an event to refresh those style IDs.
4. **Bug:** the refresh writes the style's **individual** rating, which may not be the aggregate.

**Issue 2: Write flow (adding a new rating).**
1. A user rates style 1.
2. `SetRating` updates the DB.
3. `IndexRatingAndPublish` runs **asynchronously**: it computes the aggregate for style 1, publishes to Bolt, and refreshes **style 1's** cache.
4. **Bug:** the **related styles of style 1 are never updated**.

**Issue 3: The SQL CASE bug (Go map ordering).** *This is a great debugging story.*
- The query mapped each style ID to its group's main ID with a `CASE WHEN style_id IN (...) THEN mainID ...` built in Go.
- SQL `CASE` evaluates `WHEN` clauses **in order, and the first match wins**.
- Some style IDs were both **main IDs themselves** and **members of another main ID's related list**. In the report's example, 34476499 and 33435241 are main IDs and also appear in 34220967's related list.
- The CASE clauses were generated by **iterating over a Go map, and Go map iteration order is deliberately random**. Sometimes 34220967's clause came first, so 34476499 and 33435241 were absorbed into it and **didn't appear as their own groups**.
- **Result:** aggregated ratings weren't correctly updated for every related style. The bug was **non-deterministic**: it came and went between runs.

**Issue 4: Stale cache after deployment.**
- Existing cache entries held possibly-individual ratings.
- Even after the fix shipped, related styles would disagree **until their TTL expired**, up to 7 days.
- That's why a **cache warmup** was needed.

### 3.3 Solution architecture (6 components)

```
                         READ FLOW (PDP fetch)                                WRITE FLOW (new rating)
 Style service ──► Ratings svc: get ratings(styleIDs)              User rates style S
                     │                                                   │
              Aerospike hit? ── yes ─► return aggregated rating     SetRating → MySQL
                     │ no                                                │ (async)
                     ▼                                             IndexRatingAndPublish
         publish refresh event (Kafka)                                   │
                     ▼                                                   ▼
         consumer: rate limiter (token bucket, per pod)       fetch related styles of S
            │ allowed                    │ rejected           (Accio bulk, batches of 20, rate-limited)
            ▼                            ▼                               │
   Accio BULK fetch related styles   ERROR QUEUE (Kafka)          single SQL query: ratings +
   (20 styles / batch)               → re-processed later         review/image counts for cluster
            ▼                          (eventual consistency)            │
   write related-style set to Aerospike (TTL 7d)                aggregate in Go by mainID
            ▼                                                            │
   single combined SQL query → aggregate in Go                  write aggregated rating for ALL
            ▼                                                    related styles to Aerospike (TTL 7d)
   write aggregated rating for all related styles (TTL 7d)       + publish to Bolt
```

1. **Bulk fetch of related styles:**
   - Calls to **Accio** are batched, **20–25 styles per request**, instead of one call per style. The Accio team recommended this size to balance latency against bandwidth.
   - This lowers the RPM sent to Accio.
2. **Cache management:**
   - The **related-styles set is written to Aerospike before the aggregate is computed**, in **both** the read flow (on a miss) and the write flow.
   - So later requests always find the set.
   - **TTL is 7 days** for both the set and the aggregated rating.
3. **Rate limiting:**
   - A token bucket on calls to Accio protects it during traffic bursts.
   - It is configurable **per pod**, using the team's existing rate-limiter framework.
4. **SQL fix:** the query now accounts for the related-styles set correctly (section 3.5).
5. **Error queue:**
   - Events **rejected by the rate limiter** are redirected from the consumer queue to an **error queue** and processed later.
   - This gives **eventual consistency** instead of data loss.
6. **Cache pre-population (warmup):** fill Aerospike with aggregated ratings and related sets **before** the solution goes live, to avoid a 100% miss rate (section 3.6).

### 3.4 What you built (8 tasks)

| # | Task | Details |
|---|---|---|
| 1 | **Fixed the write flow** | Fetch related styles from Accio; publish the corrected aggregated rating to the cache for **all** related styles |
| 2 | **Related-styles bulk flow** | A new bulk path to Accio with batches of **20**; used to insert related sets into the cache |
| 3 | **Fixed the read flow** | On a miss for ratings or related styles, use the bulk flow to load the full related set **before** computing the aggregate |
| 4 | **Error queue setup** | Configured a Kafka error queue to hold messages rejected by the rate limit |
| 5 | **Rate limiting** | Token bucket where Accio is called; RPM configurable per pod |
| 6 | **Error flow** | Route over-limit messages to the error queue; retries are instant, because Go services had no delay function available |
| 7 | **SQL query fix + optimisation** | Fixed the CASE grouping bug; merged 2 queries into 1, so ratings, reviews and image counts come back over **1 DB connection instead of 2**: **50% fewer connections** |
| 8 | **Cache pre-population** | An HTTP endpoint that publishes cache-refresh events for **all 5.7M live styles**; **deployed overnight** to avoid peak traffic; the full solution went out before the warmup completed |

### 3.5 SQL optimisation in detail

**Old approach: 2 queries, both with hard-coded CASE maps.**
- **Query 1, ratings:**
  1. A subquery on the aggregate-rating table, filtered by `style_id IN (...)`, derives `main_id` through a `CASE` map.
  2. The outer query groups by `main_id` and **pivots** the counts per star with conditional aggregation: `SUM(CASE WHEN rating = 1 THEN count END)` and so on.
- **Query 2, review and image counts:**
  1. Joins the review table to the image table (`LEFT JOIN`), filtered by style IDs and status (`ACTIVE`, `IMAGE_REJECTED`).
  2. Groups by the CASE-derived ID.
  3. Returns `COUNT(*)` (reviews) and `COUNT(i.id)` (images; counts only non-null rows from the LEFT JOIN).
- **Problems:**
  - 2 round trips.
  - Hard-coded SQL built by string manipulation for every input.
  - The ordering bug.
  - Can't handle **overlapping** mappings, where one style belongs to several main IDs.

**New approach: 1 parameterised query, with aggregation in Go.**

```sql
-- Shape of the new query (reconstructed from the report's description)
SELECT ar.style_id, ar.rating, ar.count,
       COALESCE(rc.total_reviews, 0) AS total_reviews,
       COALESCE(rc.total_images, 0)  AS total_images
FROM aggregate_rating ar
LEFT JOIN (
    SELECT r.style_id,
           COUNT(DISTINCT r.id) AS total_reviews,
           COUNT(DISTINCT CASE WHEN i.status = 'APPROVED' THEN i.id END) AS total_images
    FROM review r
    LEFT JOIN image i ON i.review_id = r.id
    LEFT JOIN video v ON v.review_id = r.id
    WHERE r.style_id IN (?)
      AND ( r.status IN ('ACTIVE','IMAGE_REJECTED','IMAGE_ONLY_ACCEPTED')
            OR i.id IS NOT NULL      -- has an approved image
            OR v.id IS NOT NULL )    -- has an active video
    GROUP BY r.style_id
) rc ON rc.style_id = ar.style_id
WHERE ar.style_id IN (?) AND ar.rating IN (1,2,3,4,5) AND ar.rating_type = 1;
```

Then, in Go:
1. Build the **style_id → main_id** mapping (from Accio's related sets).
2. Run the one query.
3. Aggregate per main ID with **a single map keyed by `(mainID << 32) | rating`**.
4. Compute the averages and return.

**The bit-packing trick** (they may ask you to explain or code it):

```go
key := (uint64(mainID) << 32) | uint64(rating)   // pack two 32-bit values into one uint64
counts[key] += row.Count                           // one flat map instead of map[mainID]map[rating]int

mainID := key >> 32                                // unpack
rating := key & 0xFFFFFFFF
```

- **Why:** one flat map avoids allocating an inner map per main ID. The report estimates about **50% less memory** than nested maps, and lookups are O(1).
- **Precondition:** both values fit in 32 bits. Style IDs are about 8 digits, well under 2³² ≈ 4.29 billion, and ratings are 1–5.
- **An alternative to mention:** a struct key, `map[struct{main, rating uint32}]int`. Go allows comparable structs as map keys. It's more readable, with similar performance.

**How the aggregate is computed** (the classic follow-up):
- For each main ID, **sum the per-star counts across every style in the cluster**: `count_k = Σ count(style, rating = k)`.
- Then `avg = Σ k·count_k / Σ count_k`, and the user count is `Σ count_k`.
- This is a **histogram merge**. It's exact, and unlike *averaging the averages* it weights each style by its number of ratings.
- **Why averaging the averages is wrong:** a style with 2 ratings would weigh as much as one with 2,000.

**Improvements to cite:**
- 50% fewer DB round trips.
- Parameterised queries, so no SQL injection.
- Logic in Go is testable.
- Handles **overlapping** mappings.
- Partial results can be processed.

### 3.6 The cache warmup: 5.7M styles

**Why:** before the fix, Aerospike had **no related-style data**. Deploying would have caused a **100% cache miss** on related styles, which means a flood of Accio calls.

**What you did:**
- An endpoint that **publishes refresh events in batches of 20 style IDs** for all 5.7M styles: about **285k events**.
- The events were consumed by the corrected flow.

**Execution: 5 runs.** Each run re-processed only the styles that had failed before.

| Run | Style IDs in | Refreshed | Rate-limit errors |
|---|---|---|---|
| 1 | 5,710,991 | 1,279,840 | 4,431,151 |
| 2 | 4,431,151 | 950,940 | 3,480,211 |
| 3 | 3,480,211 | 757,780 | 2,722,431 |
| 4 | 2,722,431 | 2,722,311 | 120 |
| 5 | 120 | 120 | 0 |
| **Total** | **5,710,991** | **5,710,991 (100%)** | **0** |

**What you learned (straight from the report; this is the story interviewers love):**
- **The initial limit of 180 RPM per pod was too low** for 5.7M styles. Run 1 succeeded for only about 22%.
- **Batch granularity hurt:** most batches contained at least one style needing an Accio call, so **one rate-limit rejection failed the whole batch**.
- **The breakthrough in run 4:** adding **4 ms delays between event publications**, plus adjusting the rate limit, raised that run's success rate from about 22% to **99.99%**.
- **Incremental processing:** re-running only the failed IDs is idempotent and cheap.

**Interview angle: why did adding a delay help so much?**
- A token bucket lets through a burst up to its capacity, then refills at a steady rate. Publishing 285k events as fast as possible **drained the bucket instantly**, and most events after that were rejected.
- A 4 ms gap between publishes **smooths the arrival rate** to roughly the refill rate. That's client-side pacing, the same idea as a leaky bucket.
- **Rule of thumb:** match the producer's rate to the consumer's sustained capacity, and don't rely on the limiter to absorb unbounded bursts.

### 3.7 Testing and verification
- **Before/after screenshots:** related styles showed different ratings before the fix and the same aggregated rating after.
- Cache inspection: related-style sets present for both styles after running the fix.
- A **memory leak test** and a **load test** both passed before rollout.

### 3.8 Impact

| Type | Impact |
|---|---|
| User | One consistent rating across all colour and size variants; no more "No Ratings" on new variants; better purchase decisions |
| Data | 100% of 5.7M styles refreshed; fully automated (no manual intervention); an update reaches **all related styles instantly** after a new rating, with no significant latency cost because processing is async |
| DB | **2 → 1 queries / connections** per aggregated-ratings computation (**50%**) |
| Downstream | Bulk calls (20 per batch) cut the RPM to Accio; the rate limiter protects it during spikes |
| Reliability | The error queue gives eventual consistency instead of data loss |

---

## 4. Additional contributions

**4.1 Bug fix: ratings mismatch between PDP sections.**
- **The bug:** the **reviews section** of the PDP showed a non-refreshed rating that differed from the aggregated rating in the **ratings section**. They were two caches out of sync.
- **The fix:** also refresh the reviews-section cache on the **get-ratings cache miss** path and the **set-rating** path.
- **The lesson:** in distributed systems, *every* cached copy of a value must be updated on the same triggers, or the copies drift apart.

**4.2 Hacker Ramp (Myntra's Gen AI hackathon).**
- **The team's project:** a platform that helps engineers understand **which services talk to which, through which APIs**.
- **Your part:** a feature that **parses API definitions from GitHub raw links**, to build the service-dependency view automatically.

**4.3 Big Fashion Festival sale support.**
- Myntra's biggest traffic event.
- You **monitored the system** and **helped debug live issues** during the sale.
- **Zomato tie-in:** this is the same as NYE or IPL peak readiness. Talk about watching dashboards for error rates, latency and cache hit ratio during peak.

---

## 5. Deep-dive questions with answers

**Q1. Walk me through the aggregated ratings design.**
Use section 3.3, in this order:
1. The problem, with the 4.4 vs 4.3 example.
2. The two flows: read and write.
3. Accio bulk fetch.
4. Related set cached for 7 days.
5. The single query, with aggregation in Go.
6. Write the aggregate to *all* related styles.
7. Rate limiter, error queue, warmup.

Aim for 2 minutes.

**Q2. How do you compute the aggregate correctly?**
A histogram merge across the cluster: sum the per-star counts, then compute a weighted average (section 3.5). Never average the averages.

**Q3. What was the trickiest bug?**
The **CASE ordering bug caused by Go's randomised map iteration** (section 3.2, issue 3).
- **Why it was hard:** it was non-deterministic; the same input gave different groupings on different runs.
- **The fixes:**
  - Removed the CASE mapping from SQL entirely.
  - Fetch per-style rows with one parameterised query.
  - Do the mapping in Go, where overlapping memberships are handled explicitly.
- **The general lesson:** never depend on Go map order. If order matters, sort the keys (`sort.Slice` over the keys) first.

**Q4. Why move aggregation from SQL into Go? Isn't the DB faster at aggregating?**
- The data per request is small: a cluster's styles × 5 ratings.
- SQL needed **dynamic, string-built CASE maps** that were buggy, untestable and couldn't express overlapping memberships.
- Go code is **unit-testable**, takes a parameterised query (no SQL-injection risk), and saves a round trip.
- **Trade-off:** more rows cross the network (per style instead of per group). That's fine at this scale. For huge clusters you'd reconsider.

**Q5. Why a rate limiter on Accio, and why a token bucket?**
- **Why limit at all:** Accio is a shared internal dependency. Cache misses (especially after deploys) or a warmup could flood it and hurt every other caller.
- **Why a token bucket:**
  - It allows short bursts up to the bucket size while enforcing an average rate.
  - It's cheap: 2 numbers per limiter.
  - The team **already had the framework**.
- **Per pod vs global:**
  - **The catch:** each pod has its own limit, so the total RPM is the per-pod limit × the number of pods. It changes with autoscaling.
  - **For a strict global cap:** use a distributed limiter (Redis/Aerospike counters) or have Accio enforce a quota.

**Q6. Why an error queue, and what are its weaknesses?**
- **Why:** rate-limited events would otherwise be dropped. Parking them gives **eventual consistency**.
- **Weakness:** the **retries were instant**, because Go services had no delay function available. That risks a **retry storm**: retries hit the same exhausted limiter and fail again.
- **Improvements:**
  - **Delayed retry topics** (e.g., retry-1m, retry-10m), consumed only after a delay.
  - **Exponential backoff with jitter.**
  - A **max-attempts** limit followed by a dead-letter queue.
  - Alerting on error-queue depth.
- Saying this unprompted shows maturity.

**Q7. How do you keep the cache consistent with the DB?**
- **Write flow:** the DB update is followed by an async recompute that **overwrites the cache entry for every related style**.
- **Read miss:** a full recompute.
- **TTL of 7 days** as a safety net.
- **Edge case: concurrent ratings on two styles in the same cluster.** Each recompute reads the **current DB state** and overwrites the whole cluster, so the last writer leaves a correct, recent value. It's eventually consistent and self-healing.
- **Edge case: a new variant added to a cluster.**
  - Accio is the source of truth. The new style misses in the cache, and the read flow fetches its related set and recomputes.
  - Existing members pick up the change on their next write or TTL expiry.
  - **Improvement:** subscribe to Accio's change events for immediate invalidation.

**Q8. Why cache the related-styles set separately?**
- It changes rarely, and it's needed on every recompute.
- Caching it (7-day TTL) avoids calling Accio on every rating and lowers RPM.
- It's written *before* the aggregate is computed, so later requests in the same cluster hit the cache.

**Q9. How did you run a 5.7M-item backfill safely in production?**
1. **Deployed overnight**, outside peak traffic.
2. Event-driven batches of 20 through the **same production code path**, so no special-case logic.
3. Rate limited, to protect Accio.
4. **Idempotent** refreshes, so re-running is safe.
5. **Re-ran only the failures** each time.
6. Tuned it with **publish pacing (4 ms)** and an adjusted rate limit.
7. Tracked progress per run: 5 runs to reach 100%.

**What you'd do differently:**
- Compute the safe publish rate up front: total pods × per-pod RPM. Pacing from the start would have avoided runs 1–3.
- Use **smaller effective batches**, or retry the failed styles individually, so one rejection doesn't fail all 20.
- Checkpoint progress automatically.

**Q10. What's your back-of-envelope for the warmup?**
- 5.71M styles ÷ 20 per batch ≈ **285.5k batch events**.
- Accio's capacity is the limit. At, say, P pods × R RPM per pod (180 at first), Accio-bound throughput is **P × R batches per minute**.
- Example: with 10 pods (illustrative, not from the report), that's 1,800 batches/min, about 160 minutes for every batch that needs Accio.
- The point: compute the drain time and pace the producer to match, rather than discovering it through rejections.

**Q11. How would this design change at Zomato?**
The same pattern fits **restaurant ratings across outlets of the same brand**, or **dish ratings across menu variants**:
- A relationship service (like Accio) is the source of truth for the grouping.
- A cache holds aggregates with TTLs.
- Updates are event-driven, and calls to shared services are rate-limited.
- A warmup runs before launch.

At Zomato's scale, Redis or Aerospike for the cache, Kafka for events, and per-city partitioning.

**Q12. Why Go for this service?**
It was the service's existing language. Beyond that, Go suits this work well:
- Cheap goroutines for concurrent I/O (Accio calls, DB, cache).
- Simple deployment as a static binary.
- Strong standard library (`database/sql` with pooling, `context` timeouts).
- **Gotchas you hit:** randomised map iteration, and no built-in delayed retry in the queue client.

**Q13. What does `COALESCE` do, and why `LEFT JOIN`?**
- **`LEFT JOIN`** keeps every rating row even when the style has no reviews.
- **`COALESCE(x, 0)`** turns the resulting NULLs into 0.
- **`COUNT(DISTINCT ...)`** is needed because joining images and videos multiplies the rows. Without DISTINCT, the review counts would be inflated (the fan-out / join-multiplication problem).

**Q14. What did you learn?**
- Design for downstream protection: bulk calls, rate limiting, error queues.
- Warm caches before launches.
- Every cached copy must be updated on the same triggers (the ratings mismatch bug).
- Non-determinism in Go maps.
- Operating safely in production during a major sale.

---

## 6. STAR stories from this internship

**Story A: Fixing aggregated ratings at scale** (for "a challenging problem", "a project you're proud of")
- **S:** Variants of the same product showed different or missing ratings on Myntra's PDP. Related styles weren't refreshed in the cache when a rating changed.
- **T:** Make every style in a cluster show one correct aggregated rating automatically, across 5.7M live styles, without overloading shared services.
- **A:**
  1. Fixed both the read and write flows to fetch related sets from Accio in batches of 20 and cache them (7-day TTL).
  2. Recomputed and wrote the aggregate to every style in the cluster.
  3. Added a token-bucket limiter and a Kafka error queue.
  4. Rewrote two CASE-based queries into one parameterised query, with aggregation in Go.
  5. Ran a 5.7M-style cache warmup overnight.
- **R:**
  - 100% of 5,710,991 styles refreshed, with consistent ratings across variants.
  - 50% fewer DB connections per computation.
  - No manual intervention needed.
  - Load and memory-leak tests passed.

**Story B: The non-deterministic SQL bug** (for "a hard bug", "debugging")
- **S:** During testing, some clusters' aggregates were wrong, and *which* ones changed between runs.
- **T:** Find out why the grouping was non-deterministic.
- **A:**
  1. Traced it to the generated SQL `CASE` clauses.
  2. The first-match-wins semantics, combined with Go's randomised map iteration, meant main IDs that also appeared in another main ID's related list were sometimes absorbed into the wrong group.
  3. Replaced SQL-side mapping with one parameterised query and deterministic Go-side aggregation that handles overlaps.
- **R:** Correct grouping every time, 2 → 1 queries, and code that is testable and injection-safe.

**Story C: The warmup that failed 78% of the time at first** (for "a failure", "resilience", "learning from mistakes")
- **S:** Run 1 of the 5.7M warmup refreshed only 1.28M styles; 4.43M were rejected by the rate limit.
- **T:** Finish the warmup without hurting Accio.
- **A:**
  1. Analysed the cause: the bucket drained instantly, and one rejection failed a whole batch of 20.
  2. Re-ran only the failures.
  3. Added a 4 ms pacing delay between publishes and adjusted the per-pod limit.
- **R:**
  - Run 4 hit 99.99% success; run 5 cleared the last 120.
  - 100% after 5 runs.
  - **The lesson:** calculate the throughput budget up front and pace the producer.

**Story D: Production during the Big Fashion Festival sale** (for "working under pressure")
Monitored the service and helped debug live issues during Myntra's highest-traffic event. Learned what to watch (errors, latency, cache hit ratio) and how to respond fast.

---

## 7. Resume line → report mapping (so your answers match your resume)

| Resume bullet | What the report shows |
|---|---|
| "Scalable aggregated ratings pipeline using Go and MySQL, leveraging bulk APIs, cache warmup, rate limiting, and error queues … 5.7 million products" | Project 2, sections 3.3–3.6: Accio bulk (20/batch), 5.7M warmup in 5 runs, token bucket per pod, Kafka error queue |
| "Optimized database queries and cache access patterns, reducing database connections by 50%" | Section 3.5: **2 queries → 1** for ratings, reviews and image counts (1 connection instead of 2); cache-first related sets |
| "Production-parity stage environment … MySQL, Aerospike, Kafka, and downstream services" | Project 1, section 2: configs and timeouts, MySQL schema plus mock data, Aerospike provisioning, Airbus and DB credentials in Vault, deploy and test |

> **Be precise about the 50%:** it's "50% fewer DB round trips/connections **for the aggregated-ratings computation**", from merging two queries into one. It is **not** a service-wide 50% connection reduction. If asked "50% of what?", say exactly that.
