# Myntra Internship: Ratings & Reviews Service

> **Reading time:** about 30 minutes. **Source:** Vaideeka's 29-page internship report (Dec 2025). Every fact and number comes from the report; anything marked *(prep)* is interview advice added on top.
> **Role:** Software Engineer Intern, Product-SizeFit team, Product Detail Page (PDP) org. **Dates:** 14 Jul – 5 Dec 2025. **Service:** Ratings & Reviews microservice, written in **Go**.

---

## 1. The whole internship on one page

You did two projects and three smaller things.

1. **Stage onboarding.** The ratings service only existed in dev and production. You built it a production-like **stage** environment so changes could be tested safely.
2. **Aggregated ratings (the main project).** Colour and size variants of the same product showed *different* ratings, and a new colour often showed "No Ratings". You made every variant show **one combined rating**, kept in sync automatically, for **5.7 million** live products.
3. **Smaller work:** a ratings-mismatch bug fix, the Hacker Ramp hackathon, and on-call support during the Big Fashion Festival sale.

**Numbers to know cold**

| What | Value |
|---|---|
| Live product styles covered | **5,710,991** (say "5.7 million") |
| SQL queries per aggregated-rating computation | **2 → 1** (this is the "50% fewer DB connections") |
| Styles per bulk call to Accio | **20** (Accio's team recommended 20–25) |
| Cache TTL (related styles and aggregated rating) | **7 days** |
| Rate limiter | **Token bucket, per pod**, started at **180 requests/minute per pod** |
| Cache warmup | **5 runs** to reach 100%; run 1 succeeded for only **~22%** |
| What fixed the warmup | A **4 ms delay** between published events, plus a higher rate limit |
| Tests before release | **Load test** and **memory-leak test** |

**The 60-second pitch** (memorise this one; it is the only pitch you need):

> "At Myntra I worked on the Ratings & Reviews microservice in Go. Variants of one product, like different colours, showed separate ratings, so a new colour could show 'No Ratings' while the base product had thousands. I fixed both the read and the write paths. They now fetch the product's related styles from Accio in bulk batches of 20, cache that set in Aerospike, and write one aggregated rating to every style in the group. A token-bucket rate limiter protects Accio, and anything it rejects goes to a Kafka error queue so nothing is lost. Along the way I found a non-deterministic SQL bug caused by Go's random map ordering, and I merged two queries into one, which halved the DB connections for this computation. Finally I ran a cache warmup across 5.7 million styles."

---

## 2. Background you need first

Learn these five terms before anything else. The rest of the story uses them constantly.

**Style, related styles, cluster.** A *style* is one sellable version of a product, such as the red version of a T-shirt. It has a *style ID* (about 8 digits). *Related styles* are the other versions of the same base product (blue, black…). The full set is a *cluster*. One style in the cluster acts as the **main ID** that the others are grouped under.

**Aggregated rating.** One rating computed over the whole cluster, so every colour shows the same "4.3★ from 5,100 ratings".

**Accio.** Myntra's internal service that is the **source of truth for which styles are related**. Your service has to ask Accio "what are the related styles of X?". Accio is shared by many teams, so you must not flood it.

**Aerospike.** A distributed **key-value database used as a cache**. It keeps its index in RAM and data on SSD, so it can hold huge datasets cheaply with sub-millisecond reads. Compared with Redis: Redis keeps everything in RAM and has richer data types (lists, sorted sets); Aerospike is cheaper at very large sizes. In this service it stores two things per style: its related-styles set and its aggregated rating.

**Kafka.** A durable, partitioned **event log**. Producers append events to a *topic*; consumers read them at their own pace and remember their position (*offset*). Your service uses Kafka for cache-refresh events and for the **error queue**.

**Other names in the report** (one line each is enough):
- **MySQL** holds ratings, reviews, images and videos.
- **Vault** is the secrets manager where DB and event credentials live.
- **Airbus** is Myntra's inter-service event layer.
- **Bolt** is a downstream system that receives published aggregated ratings.
- The **style service** calls your service with a list of style IDs when a product page loads.

---

## 3. Project 1: Stage onboarding

**Problem.** The ratings service had only dev (partial, mocked) and production. There was nowhere realistic to test a change before it went live.

**What you did, in five steps**
1. **Configuration.** Listed every service the ratings service talks to, pointed each at its stage endpoint, and set credentials and **timeouts**.
2. **Database.** Created the MySQL schema on stage to mirror production and filled it with **mock data** (no production user data).
3. **Cache.** Sized and provisioned the Aerospike cluster (storage and compute).
4. **Credentials.** Created DB read/write credentials and Airbus credentials, and stored them all in **Vault**.
5. **Deploy and verify.** Deployed, called the APIs to confirm the DB and cache connections worked, and checked the logs for errors.

**Why it matters:** developers can now run real service-to-service integration tests, debug safely and try out cache and query changes without touching production.

**Questions you may get**
- *"Why stage if you already have dev?"* Dev usually mocks dependencies and uses different config. Stage copies production's shape: same DB engine, same cache, same event system, real (stage) downstream services. Config mistakes, timeout problems and integration bugs show up there instead of in production.
- *"Why mock data and not a copy of production?"* Reviews are user data, so copying them raises privacy and compliance issues. A full copy is also large and costly. The mock data still has to cover edge cases: styles with no ratings, big clusters, reviews with images and videos.
- *"Why do timeouts matter?"* Without a timeout, one slow dependency holds goroutines and DB connections open until the service runs out of them and fails too. That is a *cascading failure*. Set timeouts from the dependency's normal worst latency (p99), and combine them with retries and circuit breakers.
- *"Why Vault?"* Secrets never sit in code or config repos. Vault adds access control, an audit trail and rotation.
- *"How do you size a cache?"* Number of keys × average record size × replication factor, plus headroom for growth; compute nodes from expected requests per second. If you don't remember the actual numbers, explain the method.

---

## 4. Project 2: Aggregated ratings

### 4.1 The problem

A customer opening two colours of the same shirt saw **4.4★ (1.6k ratings)** on one and **4.3★ (3.5k ratings)** on the other. A newly added colour showed **"No Ratings"** even though the product had thousands.

**Root cause in one sentence:** the cache stored ratings **per style**, and when someone rated one style, **only that style's cache entry was updated**, never its related styles.

### 4.2 The four issues you found

1. **Read path wrote the wrong value.** When a product page loaded and the cache missed, the service published a refresh event. The refresh stored that style's **individual** rating, not the cluster's aggregate.
2. **Write path forgot the siblings.** A user rates style 1 → `SetRating` writes to MySQL → `IndexRatingAndPublish` runs asynchronously, computes the aggregate and refreshes **style 1 only**. Style 1's related styles keep their old values.
3. **A non-deterministic SQL bug** (your best debugging story, section 4.4).
4. **Stale cache after release.** Even after the fix shipped, old entries would keep wrong values for up to 7 days until their TTL expired. So the cache had to be **warmed up** in advance (section 4.6).

### 4.3 The design

```
READ PATH (product page loads)                 WRITE PATH (user submits a rating)

style service → ratings service                SetRating → MySQL
      │                                              │ (async)
 Aerospike hit? ── yes → return rating          IndexRatingAndPublish
      │ no                                           │
 publish refresh event (Kafka)                       │
      │                                              │
      └──────────────┬───────────────────────────────┘
                     ▼
     rate limiter (token bucket, per pod)
        │ allowed                  │ rejected
        ▼                          ▼
 Accio bulk call (20 styles)    Kafka ERROR QUEUE → retried later
        ▼
 store related-styles set in Aerospike (TTL 7 days)
        ▼
 ONE SQL query for the whole cluster → aggregate in Go
        ▼
 write the aggregated rating to EVERY style in the cluster (TTL 7 days)
 (+ publish to Bolt on the write path)
```

The six pieces, and **why each exists**:

1. **Bulk fetch from Accio (20 per call).** One call per style would mean millions of calls. Batching cuts the request rate on Accio about 20×. Accio's team chose 20–25 as the balance between latency and payload size.
2. **Cache the related-styles set first.** The set rarely changes and is needed on every recompute. It is written to Aerospike *before* computing the aggregate, on both paths, so the next request for any style in that cluster finds it without calling Accio.
3. **Token-bucket rate limiter on Accio calls.** Protects a shared service during traffic spikes, cache-miss storms after deploys, and the warmup. You used the team's existing limiter framework, configured per pod.
4. **SQL fix and merge.** One parameterised query instead of two string-built ones (section 4.4).
5. **Kafka error queue.** Events the limiter rejects are not dropped; they are moved to an error queue and processed again later. The data ends up correct eventually (*eventual consistency*) instead of being lost.
6. **Cache warmup before launch.** Fill the cache for all 5.7M styles so the release doesn't start with every lookup missing (section 4.6).

**How a token bucket works** (expect this question). A bucket holds up to *B* tokens and is refilled at *R* tokens per second. Each request takes one token; if the bucket is empty, the request is rejected. This allows short bursts (up to *B*) while holding the long-run average to *R*. It costs two numbers per limiter: the current token count and the last refill time.

**The catch with "per pod".** Every pod (running copy of the service) has its own bucket. Total traffic to Accio is therefore *per-pod limit × number of pods*, which changes when the service autoscales. For a strict global cap you would use a shared counter in Redis or Aerospike, or have Accio enforce a quota per caller.

### 4.4 The SQL bug and the query rewrite

**The old setup: two queries.**
- **Query 1 (ratings)** mapped each style ID to its main ID with a `CASE` expression built in Go, then grouped by main ID and counted ratings per star (`SUM(CASE WHEN rating = 1 THEN count END)`, and so on for 2–5).
- **Query 2 (counts)** joined reviews to images, grouped by the same `CASE`-derived main ID, and returned review and image counts.

The `CASE` looked like this, generated from a Go map of `mainID → [related style IDs]`:

```sql
CASE
  WHEN style_id IN (34220967, 34476499, 33435241, ...) THEN 34220967
  WHEN style_id IN (34476499, ...)                     THEN 34476499
  ...
END AS main_id
```

**The bug.** Two facts collided:
- SQL `CASE` checks `WHEN` clauses **in order and stops at the first match**.
- Go **deliberately randomises map iteration order**, so the clauses came out in a different order on each run.

Some IDs, such as 34476499 and 33435241, were main IDs of their own *and* also appeared in 34220967's related list. On runs where 34220967's clause came first, those two styles were swallowed into 34220967's group and their own groups disappeared. **The same input produced different results on different runs.** Bugs like this are hard to find because they appear and vanish.

**The fix: one parameterised query, grouping moved into Go.**

```sql
-- shape of the new query (reconstructed from the report)
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
      AND (r.status IN ('ACTIVE','IMAGE_REJECTED','IMAGE_ONLY_ACCEPTED')
           OR i.id IS NOT NULL OR v.id IS NOT NULL)
    GROUP BY r.style_id
) rc ON rc.style_id = ar.style_id
WHERE ar.style_id IN (?) AND ar.rating IN (1,2,3,4,5) AND ar.rating_type = 1;
```

The query now returns plain **per-style** rows. Go then applies the style → main-ID mapping (from Accio's sets) and adds up the numbers. Because the mapping is explicit code, a style that belongs to several groups is handled correctly, and the result is the same every time.

**Three SQL details to be able to explain:**
- **`LEFT JOIN`** keeps every rating row even when the style has no reviews; an inner join would drop it.
- **`COALESCE(x, 0)`** turns the NULLs that a LEFT JOIN produces into 0.
- **`COUNT(DISTINCT …)`** is needed because joining one review to several images and videos multiplies rows. A plain `COUNT` would over-count (the "join fan-out" problem).

**What the rewrite bought you:**
- 2 round trips → 1, which is **the 50%**.
- Parameterised (`?`) instead of string-built SQL, so no SQL injection risk and the DB can reuse the query plan.
- Grouping logic in Go is unit-testable and deterministic.
- Overlapping group membership is handled.

**The bit-packing trick in Go.** To sum counts per (main ID, star), you used **one flat map** instead of a map of maps:

```go
key := (uint64(mainID) << 32) | uint64(rating) // main ID in the high 32 bits, rating in the low 32
counts[key] += row.Count

mainID := key >> 32          // unpack
rating := key & 0xFFFFFFFF
```

- *Why:* a nested `map[mainID]map[rating]int` allocates a new inner map for every main ID. One flat map avoids that; the report estimates about **50% less memory**.
- *Why it's safe:* both values fit in 32 bits. Style IDs (~8 digits) are far below 2³² ≈ 4.29 billion, and ratings are 1–5.
- *Alternative to mention:* Go allows a struct as a map key, `map[struct{ main, rating uint32 }]int`. It's more readable with similar performance.

### 4.5 Computing the aggregate correctly

For each cluster, **add up the count for each star across all its styles**, then take the weighted average:

- `count_k` = total number of k-star ratings across the cluster (k = 1…5)
- `total = count_1 + … + count_5`
- `average = (1·count_1 + 2·count_2 + … + 5·count_5) / total`

This is a **histogram merge**, and it is exact. **Never average the per-style averages:** a style with 2 ratings would count as much as one with 2,000. Example: style A has 4.0★ from 1,000 ratings, style B has 5.0★ from 10. The average of averages is 4.5★; the correct answer is (4,000 + 50) / 1,010 ≈ **4.01★**.

### 4.6 The 5.7-million-style cache warmup

**Why.** Before launch, the cache had no related-style sets at all. Releasing without a warmup would make every product page miss the cache and call Accio at once.

**How.** You built an HTTP endpoint that published refresh events in **batches of 20 style IDs** for all 5,710,991 live styles (about **285,000 events**). They went through the same corrected code path as normal traffic. It was run **overnight**, away from peak traffic. Each run retried only the styles that had failed before.

| Run | Styles sent | Refreshed | Rejected by rate limit |
|---|---|---|---|
| 1 | 5,710,991 | 1,279,840 | 4,431,151 |
| 2 | 4,431,151 | 950,940 | 3,480,211 |
| 3 | 3,480,211 | 757,780 | 2,722,431 |
| 4 | 2,722,431 | 2,722,311 | 120 |
| 5 | 120 | 120 | 0 |

**What went wrong in runs 1–3**
- **180 RPM per pod was far too low** for this volume.
- **Events were published as fast as possible.** The token bucket allows a burst and then only its refill rate, so it emptied within moments and almost everything after that was rejected.
- **One rejection failed a whole batch.** Most batches of 20 had at least one style needing an Accio call, so one rejected call failed all 20.

**What fixed it in run 4:** a **4 ms pause between published events**, plus a raised rate limit. Pacing made events arrive at roughly the rate the bucket refills, so they stopped being rejected. Success jumped from ~22% to **99.99%**. Run 5 cleared the last 120.

**The lesson to say out loud:** "Work out the throughput budget first — pods × per-pod limit — and pace the producer to match it. Don't rely on the rate limiter to absorb an unlimited burst."

**What you'd do differently** *(prep)*
- Calculate the safe publish rate up front and pace from run 1.
- On a batch failure, retry the failed styles individually instead of failing all 20.
- Save progress checkpoints automatically rather than tracking runs by hand.

**Back-of-envelope** *(prep, illustrative pod count)*: 5.71M ÷ 20 ≈ 285k batches. With 10 pods at 180 RPM, Accio-bound throughput is 1,800 batches per minute, so about 160 minutes if every batch needs Accio. Calculating this first tells you how long the job takes and what pacing to use.

### 4.7 Testing and impact

**Testing:** before/after screenshots (related styles went from different ratings to the same one), cache inspection showing related sets present, and a passing **load test** and **memory-leak test** before rollout.

**Impact:**
- **Users:** one consistent rating across all variants; no more "No Ratings" on new colours.
- **Data:** 100% of 5.7M styles refreshed, fully automated; a new rating reaches all related styles straight away, without slowing the user's request because it runs asynchronously.
- **Database:** 2 → 1 queries per computation.
- **Accio:** bulk calls cut its request rate; the limiter protects it during spikes.
- **Reliability:** the error queue means rejected work is retried, not lost.

---

## 5. Smaller contributions

**Ratings mismatch bug.** The PDP's **reviews section** showed an old rating while the **ratings section** showed the new aggregate. They read from two different caches and only one was refreshed. You made the reviews-section cache refresh on the same two triggers (a cache miss on get-ratings and a new set-rating). *Lesson:* every cached copy of a value must be updated on the same events, or the copies drift apart.

**Hacker Ramp (Myntra's Gen AI hackathon).** The team built a tool that shows which internal services call which, and through which APIs. Your part parsed **API definitions from GitHub raw links** to build that dependency map automatically.

**Big Fashion Festival.** Myntra's biggest sale. You monitored the service and helped debug live issues. What to watch during a peak: error rate, latency (p99), cache hit ratio, queue depth. *Zomato link:* this is the same as New Year's Eve or IPL-final peak readiness.

---

## 6. The ten questions most likely to come up

**1. "Walk me through the design."** Problem with the 4.4 vs 4.3 example → read and write paths both broken → Accio bulk fetch → cache the related set → one query, aggregate in Go → write to every style → limiter, error queue, warmup. About two minutes, then stop and let them pick.

**2. "Why move aggregation from SQL into Go? Isn't the database faster?"** Each request's data is small (a cluster's styles × 5 star values). The SQL version needed `CASE` maps built as strings, which were buggy, untestable and couldn't express a style in two groups. The Go version is deterministic, testable, parameterised and saves a round trip. Trade-off: more rows travel over the network. Fine at this size; for very large clusters you'd reconsider.

**3. "What was the trickiest bug?"** The `CASE` ordering bug (section 4.4). Say why it was hard: it was non-deterministic. Say the general lesson: **never depend on Go map order**; if order matters, sort the keys first.

**4. "Why a token bucket, and why per pod?"** It allows short bursts but caps the average, it's cheap, and the team already had it. Per pod is simple but means the total limit scales with pod count; for a strict global limit use a shared counter or a quota at Accio.

**5. "What are the weaknesses of your error queue?"** Retries were **instant** because the Go services had no delay function available. Instant retries hit the same empty bucket and fail again (a *retry storm*). Improvements: delayed retry topics (retry-after-1-minute, retry-after-10-minutes), exponential backoff with jitter, a maximum attempt count followed by a dead-letter queue, and an alert on error-queue size. Raising this yourself shows maturity.

**6. "How do you keep the cache consistent with the database?"** On every write, an async recompute reads the current DB state and overwrites the entry for **every** style in the cluster. A cache miss triggers a full recompute. The 7-day TTL is a safety net.
- *Two people rate two styles of the same cluster at the same moment?* Each recompute reads the latest DB state and overwrites the whole cluster, so whichever finishes last writes a correct, up-to-date value.
- *A new colour is added?* It misses the cache, so the read path fetches its related set from Accio and computes the aggregate. Older siblings pick up the change on their next rating or TTL expiry. Improvement: listen to Accio's change events and invalidate immediately.

**7. "Why cache the related-styles set separately?"** It rarely changes and is needed for every recompute; caching it avoids calling Accio on every rating.

**8. "How did you run a 5.7M backfill safely in production?"** Overnight; same code path as live traffic; batches of 20; rate limited; idempotent, so re-running is safe; retried only failures; tuned with pacing. Then add what you'd do differently (section 4.6).

**9. "50% fewer DB connections — 50% of what?"** "For each aggregated-rating computation, two queries became one, so one DB round trip instead of two. It's not a service-wide figure." Say it exactly like this; don't let it sound bigger than it was.

**10. "How would this apply at Zomato?"** The same pattern fits **restaurant ratings across outlets of one brand** or **dish ratings across menu variants**: a source-of-truth service for the grouping, a cache of aggregates with TTLs, event-driven updates, a rate limit on shared services, and a warmup before launch.

**Possible extra: "Why Go?"** It was the service's language, and it suits I/O-heavy work: cheap goroutines for concurrent calls, a strong standard library (`database/sql` connection pooling, `context` for timeouts) and a single static binary to deploy. Two Go gotchas you hit: random map order, and no built-in delayed retry.

---

## 7. Three STAR stories

**A. Consistent ratings at scale** (*"a project you're proud of"*, *"a hard problem"*)
- **Situation:** variants of the same product showed different or missing ratings; related styles were never refreshed.
- **Task:** one correct rating across every variant, for 5.7M styles, without overloading shared services.
- **Action:** fixed both paths to fetch related sets from Accio in batches of 20 and cache them; wrote the aggregate to every style; added a token-bucket limiter and a Kafka error queue; merged two queries into one with aggregation in Go; ran a 5.7M warmup overnight.
- **Result:** 100% of 5,710,991 styles refreshed, consistent ratings, half the DB round trips for the computation, no manual work, load and memory tests passed.

**B. The bug that came and went** (*"a hard bug"*, *"debugging"*)
- **S:** in testing, some clusters had wrong aggregates, and *which* ones changed between runs.
- **T:** find the source of the randomness.
- **A:** traced it to the generated `CASE` clauses: first-match-wins plus Go's random map order put shared IDs into the wrong group. Replaced the SQL mapping with one parameterised query and deterministic grouping in Go.
- **R:** correct every time, one query instead of two, testable and injection-safe.

**C. The warmup that failed 78% at first** (*"a failure"*, *"learning from a mistake"*)
- **S:** warmup run 1 refreshed only 1.28M of 5.71M styles; the rest were rate-limited.
- **T:** finish without hurting Accio.
- **A:** found that publishing unpaced emptied the bucket and one rejection failed a whole batch; re-ran only failures; added a 4 ms pause between events and raised the limit.
- **R:** 99.99% in run 4, 100% after run 5. Lesson: compute the throughput budget first and pace the producer.

**Rules:** 60–90 seconds each. Say "I" for your own work and "we" for the team's. End with a number. Have one "what I'd do differently" ready.
