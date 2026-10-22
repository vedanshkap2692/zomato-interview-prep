# Real-world engineering facts for classic design questions (sourced)

Compiled 2026-10-23 for a Zomato SDE-1 system design guide.

**How to read the tags**
- **[P]** I read the primary source myself (company blog, official docs, or paper) and the fact comes from it.
- **[P-snip]** The fact is from the primary source, but I only saw a search excerpt because the page blocked fetching (mostly Medium-hosted blogs).
- **[S]** A secondary source (a newsletter, news report, or summary of the original). Use it as supporting colour only.

Every fact below carries its source date and URL. Anything I could not verify is listed under "Not verified" at the end of each section, so nothing here is guessed.

---

## 0. Zomato's own numbers (start every answer here)

1. **[P]** Zomato Engineering, "A Tale of Scale: Behind the Scenes at Zomato Tech for NYE 2023", **29 Feb 2024**. These are the New Year's Eve 2023 peaks.
   - Orders peaked at **8,400 orders/min** and the system peaked at **80.92 million requests/min**. The day crossed the **3 million orders** mark.
   - **Kafka** handled **450+ million messages/min**, with a total cluster peak of **5.6 GBps**.
   - **ElastiCache** was scaled to **35 TB** of memory and carried **20 Gbps**, with **1 billion GETs/min** and about **100 million SETs/min**.
   - **DynamoDB** peaked at **12.3M RCUs and 5.2M WCUs per minute**. Aurora RDS handled **610k selects/s and 16,000 write transactions/s**. MongoDB peaked at **470k read ops/s and 180k write ops/s**.
   - Infrastructure was pre-scaled to **11,000 EC2 instances** running about **40,000 ECS tasks**, across **300+ microservices**.
   - Before the day: **14,000 minutes** of benchmarking across 100+ services, peaking at **50M requests/min**, with **30B+ requests** fired and no service outage.
   - Practices: kill switches and degraded modes so low-tier dependencies cannot hurt critical services; chaos tests (latency injection through the service mesh, cache-miss injection); gradual load ramps so autoscaling can keep up.
   - **60%** of orders were delivered within 30 minutes. The observability platform held **2.2 billion** active time series, at a peak of 17.8M/s.
   - URL: https://blog.zomato.com/a-tale-of-scale-behind-the-scenes-at-zomato-tech-for-nye-2023
2. **[P-snip]** Swiggy Bytes, "A brief introduction to Engineering challenges at Swiggy" (date not seen).
   - Traffic runs at about **100K to 300K rpm at lunch** and about **500K rpm at dinner**. Swiggy autoscales instead of provisioning for the peak.
   - With an average **fan-out of 3 to 5x**, several services have a latency budget of about **50 ms at P99**, which the post uses to justify a caching layer.
   - URL: https://bytes.swiggy.com/engineering-challenges-at-swiggy-430dea6c86a3

**Interview use:** Quote one real number to size the problem. For example: "Zomato peaked at about 8.4k orders/min on NYE 2023, roughly 140 orders/s. Order writes are small, but reads around them hit about 80M rpm, roughly 1.3M rps. So read paths need caching, and writes need idempotency and async fan-out through Kafka." Kill switches and degraded modes are a strong, Zomato-specific answer to "what if a dependency is slow?"

---

## 1. Payments: idempotency, ledgers, webhooks, reconciliation

### Idempotency keys
3. **[P]** Stripe blog, "Designing robust and predictable APIs with idempotency", Brandur Leach, **22 Feb 2017**.
   - The client generates a unique ID for each operation and sends it in an `Idempotency-Key` header on mutating (POST) requests. It retries with the same key.
   - If the first attempt already succeeded, the server returns the cached result.
   - Retries use exponential backoff (wait proportional to 2^n) plus random **jitter**, to avoid a thundering herd of synchronized retries. Stripe's Ruby library does this automatically.
   - URL: https://stripe.com/blog/idempotency
4. **[P]** Stripe API docs, "Idempotent requests" (current, read Oct 2026).
   - Keys can be up to **255 characters**, and Stripe suggests V4 UUIDs.
   - For API v1, keys may be pruned once they are **at least 24 hours old**. A key reused after pruning creates a new request.
   - Stripe saves the status code and body of the first request once endpoint execution begins, **including 500s**.
   - If the parameters on a retry differ from the original, Stripe returns an error.
   - Nothing is saved if validation fails or the request **conflicts with a concurrently executing request**, so those can be retried.
   - `GET` and `DELETE` are idempotent by definition and need no key.
   - URL: https://docs.stripe.com/api/idempotent_requests
5. **[P]** Brandur Leach, "Implementing Stripe-like Idempotency Keys in Postgres", **27 Oct 2017**.
   - Table `idempotency_keys` holds `idempotency_key`, `locked_at`, `request_params` (JSONB), `response_code`, `response_body` and `recovery_point`, with a **unique index on (user_id, idempotency_key)**.
   - The request is split into **atomic phases**, each committed in its own SERIALIZABLE transaction and each updating a **recovery point** (`started → ride_created → charge_created → finished`).
   - A **foreign state mutation** (a Stripe charge, an email, or a Kafka publish) happens between phases and must be treated as fallible.
   - A **completer** process finishes abandoned requests. A **reaper** deletes keys after about **72 hours** (the intro suggests about 24h).
   - URL: https://brandur.org/idempotency-keys
6. **[S]** Airbnb Engineering, "Avoiding Double Payments in a Distributed Payments System" (Medium; blocked for fetch). Summary from Biweekly Engineering Ep. 32, 14 Jul 2024.
   - The **Orpheus** library splits a request into three phases: **Pre-RPC** (check or record the idempotency key in the DB), **RPC** (call the payment provider), and **Post-RPC** (record the response).
   - Rules: no service calls in the pre- and post-RPC phases, and **no DB work during the RPC**.
   - All idempotency reads go to the **master, not replicas**, because replica lag can cause duplicate charges.
   - **5xx errors are retryable; 4xx are not.**
   - Original: https://medium.com/airbnb-engineering/avoiding-double-payments-in-a-distributed-payments-system-2981f6b070bb
   - Summary: https://biweekly-engineering.beehiiv.com/p/handling-double-payments-airbnb-biweekly-engineering-episode-32

### Ledgers and double-entry bookkeeping
7. **[P]** Stripe, "Ledger: Stripe's system for tracking and validating money movement", Ilya Ganelin, **16 Feb 2024**.
   - Ledger is an **immutable, append-only log** built on double-entry bookkeeping. **About 5 billion events/day.**
   - **99.99% of dollar volume** is fully ingested and verified **within 4 days**. Together with the data-quality platform, more than **99.9999%** of money movement is explained.
   - Each producer system is modelled as a **state machine**: events move funds between accounts. At steady state, **clearing accounts are empty**, so any **clearing account with a nonzero balance** means a missing, late, or wrong event.
   - Corrections are made by reverting and reprocessing, never by editing records.
   - URL: https://stripe.dev/blog/ledger-stripe-system-for-tracking-and-validating-money-movement
8. **[P]** Modern Treasury, "Accounting for Developers, Part I", Lucas Rocha, **17 Aug 2022** (updated Aug 2025).
   - Every transaction has **at least two entries**, and total debit-normal balances must equal total credit-normal balances.
   - Assets and expenses are debit-normal; liabilities, equity and revenue are credit-normal.
   - Store **immutable entries and derive balances** from them, rather than mutating a balance column.
   - URL: https://www.moderntreasury.com/journal/accounting-for-developers-part-i
9. **[P]** Uber, "How LedgerStore Supports Trillions of Indexes at Uber", Kaushik Devarajaiah, **4 Apr 2024**.
   - LedgerStore is immutable storage for money-movement records: **over 2 trillion unique indexes**, hundreds of billions of ledgers, and a petabyte-scale index footprint.
   - **Strongly consistent indexes** use a 2-phase commit (index intent, then record write, then intent commit). They are used for card authorization holds.
   - **Eventually consistent indexes** use materialized views. They are used for the payment-history page.
   - Old ledgers are tiered to cold storage. The move off DynamoDB saved an estimated **over $6M/year**.
   - URL: https://www.uber.com/blog/how-ledgerstore-supports-trillions-of-indexes/

### Webhooks and payment-provider behaviour (Razorpay)
10. **[P]** Razorpay docs, "Webhooks Best Practices" (current).
    - Your endpoint must respond within **5 seconds**; otherwise Razorpay treats the delivery as a timeout and resends. **Any non-2xx counts as a failure.**
    - Retries use **exponential backoff for 24 hours** from event creation. After 24 hours of failures, the **webhook is disabled** and an alert email is sent.
    - Delivery is **at-least-once**, so deduplicate on the **`x-razorpay-event-id`** header.
    - **Event order is not guaranteed.**
    - TLS 1.2 or later is required. For synchronous needs, poll the Payments, Orders or Refunds APIs.
    - URL: https://razorpay.com/docs/webhooks/best-practices/
11. **[P]** Razorpay docs, "About Webhooks".
    - If a webhook has not arrived within your business's time window, call the fetch APIs to check status.
    - `callback_url` (the browser redirect) is **not** a substitute for webhooks.
    - URL: https://razorpay.com/docs/webhooks/
12. **[P-snip]** Razorpay Engineering, "How Razorpay's Notification Service Handles Increasing Load" (Medium; date not seen).
    - The service carries SMS, email and webhooks. The old system could handle **up to 2K TPS**; peak was usually about **1K TPS**, yet **p99 latency rose from about 2s to about 4s**.
    - Original design: API pods, then SQS, then workers, then a DB, plus a scheduler for retries.
    - Bottlenecks: **database inserts and connections**, and a **single queue for all request types**, which meant no prioritization.
    - **[S]** Per secondary write-ups, the fix added a rate limiter that routes by request type and customer into **P0/P1 queues**, a separate **retry SQS queue**, and **Kinesis** for results (so workers no longer write to the DB directly).
    - URL: https://engineering.razorpay.com/how-razorpays-notification-service-handles-increasing-load-f787623a490f
    - Secondary: https://arpit.substack.com/p/how-razorpay-scaled-their-notification

### UPI and Indian payments scale
13. **[S]** News reports citing NPCI data.
    - UPI first crossed **20 billion transactions in a month in Aug 2025** (20.01B). Sep 2025 was **19.63B**, worth Rs 24.90 lakh crore, at an average of about **654M/day**.
    - Dec 2025 hit **21.63B** (Rs 27.97 lakh crore). Calendar 2025 totalled more than **228B**.
    - Sources: https://entrackr.com/news/upi-transaction-volume-dips-slightly-in-september-10518184 and https://www.businesstoday.in/technology/news/story/upi-ends-2025-on-a-high-with-record-monthly-and-annual-transactions-with-plans-for-future-initiatives-509218-2026-01-02
14. **[S]** Indian Startup News, **4 Feb 2026**, tied to PhonePe's UDRHP (IPO filing).
    - PhonePe peaked at **22,369 TPS in H1 FY26**, on **1.04 million cores** and **30.95 PB** of storage.
    - Server cost is **Rs 0.06 per transaction**, and PhonePe runs its own data centres.
    - URL: https://indianstartupnews.com/news/how-phonepe-is-using-in-house-technology-stack-to-fix-the-profit-puzzle-as-it-prepares-for-ipo-11077019
15. **[S]** AWS case study on Juspay (vendor-published).
    - Over **175M daily transactions** and **50,000 TPS** capacity. Juspay's own about page later claims 300M+/day.
    - URL: https://aws.amazon.com/solutions/case-studies/juspay-case-study/

**Interview use (Zomato checkout or "design a payment system"):**
- **Order service to payment gateway.** Create the `payment_attempt` row first, with a unique `(order_id, attempt_no)` or a client idempotency key. This is Airbnb's pre-RPC phase, done on the primary DB.
- **Call the gateway** with an idempotency key, and don't hold a DB transaction open across the call.
- **Record the result** in the post-RPC phase.
- **Treat the webhook as the source of truth.** Razorpay delivers at-least-once and out of order, and needs a reply within 5 seconds. So dedupe on the event ID, ack fast, and process asynchronously.
- **Run a reconciliation job** against gateway settlement reports, and a poll fallback for PENDING payments.
- **Money goes into a double-entry ledger** with immutable entries. Stripe's trick works well here: a nonzero clearing account means something is broken.
- **Say the numbers.** Stripe keeps keys for at least 24h; Brandur suggests a 72h reaper; Stripe stores 500s as the final answer too.

**Not verified:** the full original Airbnb post (Medium blocked; I relied on a summary); Razorpay's own posts on UPI or ledgers (none found on the engineering blog); a Paytm engineering ledger post (none found); NPCI's own statistics page (it would not render).

---

## 2. Rate limiting

16. **[P]** Stripe blog, "Scaling your API with rate limiters", Paul Tarjan, **30 Mar 2017**. Stripe runs four limiters:
    - (1) **Request rate limiter**: N req/s per user, with a burst allowance. It is the most important one and rejected "millions" of requests that month.
    - (2) **Concurrent requests limiter**: caps in-flight requests, for CPU-heavy endpoints. It rejected 12,000 that month.
    - (3) **Fleet usage load shedder**: reserves capacity for critical methods. The example reserves **20% for critical and 80% for non-critical** traffic; excess non-critical requests get **503**.
    - (4) **Worker utilization load shedder**: sheds by priority (critical methods, then POSTs, then GETs, then **test-mode traffic shed first**), ramping slowly to avoid flapping. It rejected 100 that month.
    - Algorithm: **token bucket in Redis**.
    - Rollout advice: **fail open** if Redis is down; return clear 429 vs 503 errors; keep kill switches; **dark-launch** limiters first.
    - URL: https://stripe.com/blog/rate-limiters
17. **[P]** Cloudflare blog, "Counting things, a lot of different things", **7 Jun 2017**.
    - **Sliding window counter**: `rate = prev_count × ((window − elapsed)/window) + curr_count`. Example: 42 × (45/60) + 18 = **49.5** against a limit of 50.
    - Tested on **400M requests from 270K sources**: only **0.003%** were wrongly allowed or limited, with an average 6% gap from the real rate and no false positives.
    - Storage is memcache (Twemproxy + Twemcache) per PoP, needing only GET, SET and INCR, with asynchronous increments. Once a client is mitigated, servers cache that state locally.
    - The largest mitigated attack was about **400K rps to one domain**.
    - URL: https://blog.cloudflare.com/counting-things-a-lot-of-different-things/
18. **[P]** Figma blog, "An alternative approach to rate limiting", Nikrad Mahdi, **12 Apr 2017**.
    - Compares token bucket, fixed window and sliding window log, then picks **sliding window counters**: buckets each 1/60 of the window (per-minute counters for an hourly limit), stored in a Redis hash per user.
    - Memory for 500 req/day × 10K users: a sliding **log** needs about 5M values (**about 20 MB**); **counters** need about 600K values (**about 2.4 MB**).
    - Figma chose to shadow-ban rather than return 429.
    - URL: https://www.figma.com/blog/an-alternative-approach-to-rate-limiting/
19. **[P]** GitHub blog, "How we scaled the GitHub API with a sharded, replicated rate limiter in Redis", Robert Mosolgo, **5 Apr 2021**.
    - GitHub moved off a **shared Memcached** because cache-memory pressure **evicted live rate-limit counters**.
    - The new design uses Redis, **sharded by key in the application**, with **one primary for writes and replicas for reads**. **Lua scripts** keep each operation atomic.
    - Bugs they hit: the reset time "wobbled" when computed from a Redis TTL, fixed by storing the reset time explicitly with a TTL 1s later. Stale replicas showed "Remaining: 5000" on a rejection, fixed by ignoring expired windows read from replicas.
    - URL: https://github.blog/2021-04-05-how-we-scaled-github-api-sharded-replicated-rate-limiter-redis/
20. **[P]** Shopify REST Admin API rate limits (current docs).
    - Uses a **leaky bucket**: **40 requests per app per store, leaking 2 req/s** on standard plans; **400 bucket at 20 req/s** on Plus.
    - The `X-Shopify-Shop-Api-Call-Limit` header shows usage (e.g. `32/40`). Over-limit requests get **429 with `Retry-After`**.
    - URL: https://shopify.dev/docs/api/admin-rest/usage/rate-limits
21. **[P]** envoyproxy/ratelimit (GitHub README).
    - A **Go/gRPC service from Lyft**, in production there for more than 2 years. Decisions are made per **domain plus descriptors**, configured in YAML.
    - Uses **Redis** (Memcache is experimental) and **fixed windows** of second, minute, hour or day.
    - An optional **local cache of over-limit keys** (freecache) skips Redis for clients already over the limit. A per-rule `shadow_mode` allows the request but records stats. Redis pipelining is available.
    - URL: https://github.com/envoyproxy/ratelimit
22. **[P]** Brandur Leach, "Rate limiting, Cells, and GCRA", **18 Sep 2015**.
    - **GCRA** tracks a single **theoretical arrival time (TAT)** per key.
    - A request is allowed if `TAT − burst_tolerance ≤ now`. When allowed, the TAT advances by the emission interval T.
    - There is **no background "drip" process** (unlike a leaky bucket), so it is more stable. Use the store's clock (Redis `TIME`) to avoid clock skew.
    - URL: https://brandur.org/rate-limiting

**Interview use (rate-limit the Zomato API or OTP sends):**
- **Pick an algorithm on purpose.** Token bucket (Stripe) allows bursts. Sliding window counter (Cloudflare, Figma) gives about 99.997% accuracy in O(1) memory. Sliding log is exact but memory heavy (20MB vs 2.4MB in Figma's example). GCRA stores one timestamp.
- **Store counters in a dedicated Redis, not the shared cache.** Cite the GitHub eviction story.
- **Make check-and-increment atomic** with Lua.
- **Fail open** and dark-launch first (Stripe).
- **Return 429 with `Retry-After`** (Shopify).
- **Cache "already blocked" decisions locally** (Envoy, Cloudflare) to save a Redis round trip.
- **Separate rate limiting from load shedding.** Stripe's critical-first shedding maps directly to "place-order and payment APIs over browse APIs on NYE".

---

## 3. Notifications

23. **[P]** LinkedIn Engineering, "Air Traffic Controller: Member-First Notifications at LinkedIn", Changji Shi and Adriel Fuad, **1 Mar 2018**.
    - ATC is the single decision maker for every notification, built on **Samza with local RocksDB state** (keyed by member ID).
    - Its steps: **channel selection** (email, SMS, push, in-app, desktop), **aggregation** (delay and batch, e.g. a week of invitation reminders in one email), **delivery-time optimization** (no overnight alerts), and **dedupe**. Dedupe is needed because Samza replays un-checkpointed requests after a restart.
    - It also **rate-limits upstream apps** to prevent accidental spam.
    - Scale: **more than 1B requests/day**. P90 latency for messaging push fell from **about 12s to about 1.5s**. Local RocksDB reads take a couple of ms vs 10 to 100 ms for remote calls. **Member complaints were cut in half.**
    - URL: https://www.linkedin.com/blog/engineering/messaging-notifications/air-traffic-controller-member-first-notifications-at-linkedin
24. **[P]** Uber, "How Uber Optimizes the Timing of Push Notifications using ML and Linear Programming", Vinay Sharma et al., **3 Nov 2022**.
    - The **Consumer Communication Gateway (CCG)** has four parts:
      - A **Persistor** stores each push in a per-user inbox (sharded MySQL, **partitioned by user UUID**).
      - A **Schedule Generator** re-plans all of a user's buffered pushes whenever a new one arrives, using ML plus a linear-program solver.
      - A **Scheduler** built on Cadence, with Kafka topics per hour of the planning horizon, which is idempotent and handles **tens of thousands of triggers/s**.
      - **Push Delivery** does last-mile checks and can drop or expire a push.
    - Constraints include **frequency caps** (example: 2/day), **minimum gaps** (example: 8h), send windows, expiry and restaurant hours.
    - Problems this fixed: conflicting and duplicate pushes arriving minutes or hours apart.
    - URL: https://www.uber.com/en-CA/blog/how-uber-optimizes-push-notifications-using-ml
25. **[P]** Uber, "Uber's Real-Time Push Platform", Madan Thangavelu et al., **18 Dec 2020**.
    - Before the platform, **80% of API gateway requests were polling calls**. Uber replaced polling with server push: first SSE (the RAMEN system), then gRPC bidirectional streams.
    - Messages go into three **priority buckets: High, Medium, Low**. High gets server retries and cross-region replication.
    - Delivery is **at-least-once with per-session sequence numbers**: the client reconnects with the highest sequence it has seen, and the server resends from there. Message TTL ranges from seconds to 30 minutes. Heartbeats go every 4s, and the client declares the connection dead after 7s.
    - Scale grew from 70K msg/s with 600K connections to **more than 1.5M concurrent connections and 250K+ msg/s**, at 99.99% server-side reliability.
    - URL: https://www.uber.com/en/blog/real-time-push-platform/
26. **[P]** FCM docs, "Throttling and quotas".
    - Default **600k messages/min per project**; over-quota requests get 429. **1,000 concurrent fan-outs per project.** Topic subscription changes are capped at **3,000 QPS**.
    - **Collapsible messages** are limited to a **burst of 20 per app per device, refilling 1 every 3 minutes**.
    - On Android, a single device can receive up to **240 msgs/min and 5,000/hour**.
    - URL: https://firebase.google.com/docs/cloud-messaging/throttling-and-quotas
27. **[P]** FCM docs, collapsible vs non-collapsible messages and message lifespan.
    - FCM stores at most **4 distinct collapse keys per device**.
    - Android stores up to **100 non-collapsible messages**; past that limit, **all stored messages are discarded** and the app receives `onDeletedMessages()`.
    - TTL ranges from **0 to 28 days** (default 4 weeks). TTL=0 means "deliver now or drop".
    - **High priority** tries to wake devices in Doze. Data messages to iOS must use normal priority.
    - URLs: https://firebase.google.com/docs/cloud-messaging/customize-messages/collapsible-message-types and https://firebase.google.com/docs/cloud-messaging/customize-messages/setting-message-lifespan
28. **[P]** Apple, "Sending notification requests to APNs".
    - Requires **HTTP/2 and TLS 1.2+**. The payload limit is **4 KB** (VoIP: 5 KB).
    - `apns-priority` is **10** (immediate, the default), **5** (power-aware, required for background pushes), or **1**.
    - `apns-collapse-id` can be at most **64 bytes**. `apns-expiration=0` means try once and don't store.
    - APNs **stores only one notification per bundle ID** for an offline device (usually the latest), for **up to 30 days**, and **may reorder** notifications. Priority 5 and 1 pushes may be grouped and delivered in bursts.
    - Best practice is to reuse connections for hours to days.
    - URL: https://developer.apple.com/documentation/usernotifications/sending-notification-requests-to-apns
29. **[S]** InfoQ/QCon talk, "Duolingo high-scale notification" (Vitor Pellegrino, Zhen Zhou).
    - For a Super Bowl ad, Duolingo built a system to send about **4 million pushes within about 5 seconds**, dry-run on 4M users in January.
    - Lesson: each notification tap triggers several backend calls, so **the downstream click load multiplies**.
    - URL: https://www.infoq.com/presentations/duolingo-high-scale-notification/
30. **[P-snip]** DoorDash careers blog, "Part 3: DoorDash 2025 summer intern projects".
    - The Notification Hub's default dedupe only **hid duplicates in the UI**, so users still received identical pushes.
    - The fix filtered out users who got the same message in the **last 30 days**.
    - URL: https://careersatdoordash.com/blog/part-3-doordash-2025-summer-intern-projects/

**Interview use (Zomato order-status and promo notifications):**
- **Split by priority**, like Uber's High/Medium/Low and Razorpay's P0/P1 queues. Transactional messages ("rider arriving") must never queue behind a promo blast.
- **Put one gateway** (LinkedIn ATC, Uber CCG) in charge of dedupe, frequency caps, quiet hours or delivery time, aggregation and channel choice.
- **Partition the per-user inbox or state by user ID**, so you get local reads and per-user ordering.
- **Use collapse keys** (`apns-collapse-id`, FCM `collapse_key`) for "order status" updates, so only the latest status shows. FCM's 4-key limit and APNs' one-stored-notification behaviour make this the natural fit.
- **Give live tracking a short TTL**, since a stale ETA is useless.
- **Plan provider quotas.** FCM's 600k/min per project is the ceiling, so a 10M-user promo blast takes at least about 17 minutes. Stagger it to protect the backend from the click spike (the Duolingo lesson).

---

## 4. Ticketing and seat booking, waiting rooms, distributed locks

31. **[P]** Ticketmaster, "Taylor Swift | The Eras Tour Onsale Explained", **19 Nov 2022**.
    - **3.5M** people registered for Verified Fan; about **1.5M got codes** and 2M were waitlisted.
    - The site took **3.5 billion system requests, about 4x its previous peak**. More than **2M tickets sold on 15 Nov**, a single-day record for an artist.
    - About **15% of interactions had issues**, including passcode-validation errors that made fans lose tickets already in their carts.
    - Ticketmaster slowed some sales and pushed others back to stabilize the system.
    - URL: https://business.ticketmaster.com/press-release/taylor-swift-the-eras-tour-onsale-explained/
32. **[S]** Semafor, CFO Dive and ABC reporting on Live Nation's Joe Berchtold, Senate Judiciary Committee, **24 Jan 2023**.
    - Bot traffic was **3x anything seen before**, and bots targeted the **Verified Fan access-code servers** for the first time.
    - Bots didn't buy tickets, but the attack forced sales to be slowed and paused. Berchtold said sales should have been staggered over a longer period.
    - URL: https://www.semafor.com/article/01/24/2023/heres-how-ticketmaster-explained-the-taylor-swift-tour-debacle-to-congress
33. **[S]** Business Today interview with BookMyShow CEO Ashish Hemrajani, **29 Jan 2025**, on the Coldplay India onsale (22 Sep 2024).
    - "We were underprepared for **10 million instances** that hit our site", largely bots. The site was **down for about 9.5 minutes**.
    - The original queue was **linear (FIFO)**. Later shows used **randomised token IDs**, and the per-transaction limit was cut **from 8 to 4 tickets**.
    - Other reports say about **1.3 crore** (13M) people tried to access the site.
    - URL: https://www.businesstoday.in/amp/india/story/we-were-underprepared-for-the-bots-bookmyshow-ceo-on-coldplay-ticketing-row-462362-2025-01-29
34. **[P-snip]** Queue-it docs and blog.
    - The default waiting room is **FIFO**.
    - For scheduled sales, early arrivals wait in a **pre-queue with no position**. At start time they are **randomized**, and later arrivals join FIFO behind them. This removes the advantage of arriving early.
    - URLs: https://queue-it.com/blog/first-in-first-out-randomization and https://queue-it.com/developers/how-queue-it-works
35. **[P]** Cloudflare, "Building Waiting Room on Workers and Durable Objects", **16 Jun 2021**.
    - Each user gets an encrypted cookie holding `bucketId` (first visit rounded to the minute), `lastCheckInTime` and `acceptedAt`.
    - Each edge worker reports counts to a per-data-center Durable Object, which reports to a **global Durable Object**.
    - Free slots = Total Active Users limit − current active users, handed to the **oldest buckets first** (approximately FIFO).
    - Each worker gets slots/worker_count, so it can **decide locally without coordinating**.
    - Example estimate: 60 people ahead ÷ 30 admitted/min = 2 min wait.
    - URL: https://blog.cloudflare.com/building-waiting-room-on-workers-and-durable-objects
36. **[P]** Redis docs, "Distributed Locks with Redis".
    - Single-instance lock: `SET resource rand NX PX 30000`. Release with compare-and-delete (a Lua script, or `DELEX key IFEQ` in Redis 8.4+) so you never delete someone else's lock.
    - **Redlock**: N=5 independent masters; the lock counts only if acquired on a **majority (≥3)** within the validity time, and validity = TTL − elapsed − clock drift.
    - Failover-based setups are unsafe because **replication is async**: a master can crash before the lock key replicates, so two clients get the lock.
    - The docs themselves say: **implement fencing tokens**, and note that **Redis TTL doesn't use a monotonic clock**.
    - URL: https://redis.io/docs/latest/develop/clients/patterns/distributed-locks/
37. **[P]** Martin Kleppmann, "How to do distributed locking", **8 Feb 2016**.
    - Distinguish **efficiency locks** (a single Redis is fine) from **correctness locks**.
    - A **stop-the-world GC pause** (or page fault or network delay) can outlast the lease, so two clients both act as the lock holder.
    - The fix is **fencing tokens**: a monotonically increasing number issued with each lock, with the storage layer rejecting older tokens. Redlock's random value can't do this.
    - For correctness, use ZooKeeper (zxid or znode version as the token) or a transactional DB.
    - URL: https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html
38. **[P]** antirez (Salvatore Sanfilippo), "Is Redlock safe?" (Feb 2016).
    - Rebuttal: Redlock needs only bounded clock *rate* drift, not synchronized clocks. A system that can enforce fencing barely needs a strong lock anyway.
    - He **concedes** that Redis should use monotonic time.
    - URL: http://antirez.com/news/101

**Interview use (BookMyShow-style seat hold; also Zomato "limited-quantity deal" or flash sale):**
- **Put a waiting room in front** with a randomized pre-queue at sale start, then FIFO (Queue-it). BookMyShow changed from linear to randomized tokens after Coldplay.
- **Admit by available capacity** rather than raw arrival (Cloudflare's slots = limit − active).
- **Cap tickets per transaction** (8 → 4).
- **Hold a seat** with `SET seat:{show}:{seat} user NX PX 600000` (a 10-minute hold) for speed. The **source of truth is the DB**, with an `UPDATE seats SET status='HELD', hold_id=?, version=version+1 WHERE id=? AND status='AVAILABLE'` (a conditional write that works like a fencing token or optimistic lock). Then, even if a Redis lock is lost to a GC pause or failover, the DB rejects the stale writer. That is Kleppmann's point, and the Redis docs agree.
- **Expire holds** with TTL plus a sweeper.
- **Make payment confirmation idempotent** (Section 1).
- **Expect bots to be most of the traffic** (Ticketmaster 3x, BookMyShow 10M "instances") and **protect auth or code-validation services**, since that is where Ticketmaster broke.

**Not verified:** any official BookMyShow or Paytm Insider engineering post on seat-lock internals (only third-party write-ups exist, which I left out); exact Queue-it internals beyond the docs.

---

## 5. Caching at scale and cache stampedes

39. **[P]** Nishtala et al., "Scaling Memcache at Facebook", **NSDI 2013** (paper PDF read).
    - Handles **billions of requests per second** and **trillions of items**.
    - It is a **demand-filled look-aside cache**: on a write, update the DB, then **delete** the cache key rather than setting it, **because deletes are idempotent**.
    - **Leases**: on a miss, the client gets a 64-bit token, and its later set is accepted only if the token is still valid. A delete invalidates the token, which prevents **stale sets**.
    - For **thundering herds**, the server hands out a token **only once every 10 seconds per key**, and other clients wait and retry.
    - In a one-week test of keys prone to herds, the **peak DB query rate dropped from 17K/s to 1.3K/s**. Stale values can be served while refilling.
    - **Gutter pool** (about **1% of memcached servers**): failed gets are retried there with short TTLs instead of rehashing. Rehashing could overload a server, since one hot key can be **20% of a server's requests**. Gutter **cuts client-visible failures by 99%** and turns 10 to 25% of failures into hits.
    - **mcsqueal** tails the MySQL commit log and broadcasts deletes (invalidation via change data capture). **mcrouter** is the routing proxy.
    - URL: https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf
40. **[P]** Yang, Yue and Rashmi, "A large scale analysis of hundreds of in-memory cache clusters at Twitter", **OSDI 2020**.
    - Traces from **153 Twemcache clusters**: about **700B requests (80 TB)**, with clusters of 20 to thousands of instances.
    - **More than 35% of clusters are write-heavy** (write ratio above 30%). **Eight of the top 10 clusters have a miss ratio below 5%** (six about 1%).
    - TTL matters: **more than 25% of workloads have a mean TTL under 20 minutes**, and only 20% use a single TTL.
    - **FIFO performs as well as LRU** for many workloads.
    - URL: https://junchengyang.com/publication/osdi20-twemcache.pdf (also https://www.usenix.org/conference/osdi20/presentation/yang)
41. **[S]** InfoQ, "Netflix global cache" (Sriram Rangarajan and Prudhviraj Karumanchi, Netflix), **11 Oct 2024**.
    - **EVCache**: 4 regions, **200 Memcached clusters, 22,000 servers, 400M ops/s**, about **2 trillion items (14.3 PB)**, using extstore to spill to SSD.
    - **Cross-region replication**: the client writes locally and sends **metadata only (key, TTL, timestamp) to Kafka**. A reader service re-reads the latest value locally and ships it over REST to a writer in each remote region. Failures go to an SQS retry queue.
    - Zstd batching cut bandwidth by **about 35%**.
    - URL: https://infoq.com/articles/netflix-global-cache
42. **[P]** Discord, "How Discord Stores Trillions of Messages", Bo Ingram, **6 Mar 2023**.
    - Discord moved from **177 Cassandra nodes to 72 ScyllaDB nodes**.
    - p99 latency: historical reads went from **40 to 125 ms down to 15 ms**, and inserts from **5 to 70 ms down to 5 ms**.
    - Rust **data services** add **request coalescing** (many concurrent requests for the same row become one DB query) and **consistent-hash routing** by channel ID, so identical requests land on the same instance and coalesce.
    - URL: https://discord.com/blog/how-discord-stores-trillions-of-messages
43. **[P-snip]** Instagram Engineering, "Thundering Herds & Promises", Nick Cooper, **16 Apr 2019**.
    - When a new cluster comes up with an empty cache, Instagram caches a **promise (future) of the value** rather than the value itself, so concurrent misses wait on one in-flight fetch.
    - URL: https://instagram-engineering.com/thundering-herds-promises-82191c8af57d
44. **[P-snip]** Vattani, Chierichetti and Lowenstein, "Optimal Probabilistic Cache Stampede Prevention", **VLDB 2015** (vol 8, issue 8).
    - **XFetch**: each request recomputes early with a probability that rises as expiry nears, based on the stored recompute time `delta` and `beta` (default 1.0).
    - The rule is: recompute if `now − delta·beta·ln(rand()) ≥ expiry`. This is the well-known form of the rule; I confirmed `delta` and `beta` against the docs.rs xfetch crate (https://docs.rs/xfetch/), not against the paper text.
    - DOI: 10.14778/2757807.2757813

**Interview use (Zomato restaurant menu and listing cache):**
- **Use look-aside caching and delete on write** (Facebook: deletes are idempotent).
- **Stop the stampede on a hot key**, such as a popular restaurant's menu at 8pm:
  - Use **leases or single-flight**: Facebook hands one token per key per 10s and cut DB peak 17K → 1.3K/s; Instagram's promises and Discord's coalescing do the same job.
  - Use **probabilistic early refresh** (XFetch) or serve stale while revalidating.
  - Add **TTL jitter** so keys don't all expire together.
- **On a cache-node failure, use a gutter pool** rather than rehashing (Facebook: one key can be 20% of a server's load).
- **Invalidate from the DB commit log** (mcsqueal, which is basically CDC) rather than from application code.
- **Size it with Zomato's numbers:** 35 TB ElastiCache, 1B GET/min vs 100M SET/min, so about 10:1 read to write at peak.
- **Not every cache is read-heavy** (Twitter: over 35% write-heavy), so check the ratio before assuming a cache helps.

---

## 6. Kafka, transactional outbox, CDC

45. **[P-snip]** Kreps, Narkhede and Rao, "Kafka: a Distributed Messaging System for Log Processing", **NetDB 2011** (LinkedIn).
    - Kafka was built at LinkedIn for high-volume, low-latency log collection, for both offline and online consumers. At the time it carried hundreds of GB of new data per day.
    - Copy: https://classpages.cselabs.umn.edu/Spring-2018/csci8980/Papers/PublishSubscribe/Kafka.pdf
46. **[P]** LinkedIn Engineering, "How LinkedIn customizes Apache Kafka for 7 trillion messages per day", Jon Lee and Wesley Wu, **8 Oct 2019**.
    - **More than 7 trillion messages/day**, on **100+ clusters, 4,000+ brokers, 100,000+ topics and 7 million partitions**.
    - URL: https://www.linkedin.com/blog/engineering/open-source/apache-kafka-trillion-messages
47. **[P]** Uber, "Presto on Apache Kafka at Uber Scale", **14 Apr 2022**.
    - Uber runs one of the largest Kafka deployments, **processing trillions of messages and multiple petabytes daily**.
    - Uses: rider and driver app events, Flink streaming, **database changelogs to downstream subscribers**, and ingestion into the Hadoop data lake.
    - URL: https://www.uber.com/en-NL/blog/presto-on-apache-kafka-at-uber-scale/
48. **[P]** Uber, "Introducing uForwarder: The Consumer Proxy for Kafka Async Queuing", **5 Feb 2026**.
    - A **push-based consumer proxy** fetches from Kafka and pushes each message to services over gRPC. **More than 1,000 consumer services** are onboarded, and the fleet grew from two-digit to thousands of servers.
    - It solves partition-count parallelism limits and **head-of-line blocking** using **out-of-order commits** (a per-partition offset tracker).
    - A **DLQ** handles permanent poison pills. Messages above the **4 MB gRPC default** can't reach handlers, which is another way to get a poison pill.
    - URL: https://www.uber.com/en-AU/blog/introducing-ufowarder/
49. **[P]** Debezium blog, "Reliable Microservices Data Exchange With the Outbox Pattern", Gunnar Morling, **19 Feb 2019**.
    - **Dual writes** (DB plus Kafka) can't share one transaction, because Kafka doesn't support XA. One write can succeed while the other fails, leaving them inconsistent.
    - Outbox columns: `id` (a UUID, used by consumers to dedupe), `aggregatetype` (selects the topic, e.g. OrderEvents), `aggregateid` (becomes the Kafka key, so one aggregate's events stay in order), `type`, `payload`.
    - Debezium **tails the WAL/binlog** rather than polling.
    - The service can **insert then delete the outbox row in the same transaction**. The log keeps the insert, so the table never grows.
    - Delivery is **at-least-once**, so consumers keep a processed-ID table.
    - URL: https://debezium.io/blog/2019/02/19/reliable-microservices-data-exchange-with-the-outbox-pattern/
50. **[P]** Chris Richardson, microservices.io, "Pattern: Transactional outbox".
    - Write the message into an outbox table in the **same DB transaction** as the business change, and have a relay publish it. The relay is either **transaction log tailing** or a **polling publisher**.
    - This avoids 2PC and preserves order, but the relay can publish **duplicates**, so **consumers must be idempotent**.
    - URL: https://microservices.io/patterns/data/transactional-outbox.html
51. **[P]** Confluent, "Exactly-once Semantics Are Possible: Here's How Kafka Does It", Neha Narkhede and Guozhang Wang, **30 Jun 2017**.
    - Arrived in **Kafka 0.11**. The **idempotent producer** (`enable.idempotence=true`) uses per-batch sequence numbers, so brokers drop duplicates.
    - **Transactions** give atomic writes across partitions plus offset commits, and need a `transactional.id`. Consumers read with `isolation.level=read_committed`.
    - Overhead: about **3% lower throughput** vs `acks=all` with 1 in-flight request, and about 20% vs `acks=1` with 5 in-flight (1 KB messages, 100 ms transactions).
    - **Exactly-once covers only Kafka-to-Kafka processing.** Side effects in external systems (an RPC, or sending a push) still need idempotency.
    - URL: https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/
52. **[P]** Uber's real-time push platform (fact 25) and LinkedIn ATC (fact 23) both depend on **at-least-once delivery plus dedupe** (sequence numbers in one, a "seen request" filter after Samza replay in the other). This is the same pattern as the outbox consumer, in production.

**Interview use (Zomato order placed → restaurant, rider assignment, notifications, analytics):**
- **Write the order and an `order_events` outbox row in one transaction.** Use Debezium or CDC, or a poller, to publish to Kafka keyed by `order_id` (Debezium's aggregateid), so all of one order's events land in the same partition, in order.
- **Assume duplicates everywhere.** Consumers keep a processed-event-ID table or use idempotent upserts, because Kafka exactly-once doesn't cover external side effects (Confluent) and outbox relays can republish (Richardson).
- **Send poison messages to a DLQ** with out-of-order commits so one bad message doesn't block a partition (Uber uForwarder).
- **Cite the scale:** Zomato's Kafka ran 450M+ msg/min (about 7.5M msg/s) on NYE 2023. LinkedIn passed 7T/day in 2019. Uber runs trillions/day.

---

## Fact count

There are 52 numbered entries. Most contain several distinct sourced numbers, so the total is well over 100 individual sourced facts.

## Gaps and caveats
- Fetching was blocked for Medium-hosted blogs (Airbnb, Razorpay Engineering, Instagram, Swiggy Bytes), so those facts come from search excerpts or secondary summaries and are tagged [P-snip] or [S].
- NPCI's statistics page did not render, so UPI monthly volumes come from news reports citing NPCI.
- FCM's 4096-byte payload limit is often quoted but did not appear in the doc pages I could read, so I left it out. APNs' 4 KB limit is confirmed.
- The Swiggy, DoorDash and Slack notification internals and BookMyShow seat-lock internals have no primary engineering source, so I didn't invent any.
- The Netflix EVCache numbers come from an InfoQ article written by Netflix engineers, not from the Netflix tech blog (it returned 403).
