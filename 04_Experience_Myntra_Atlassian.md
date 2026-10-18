# Internships: Myntra and Atlassian

> About 25 minutes. Read it top to bottom; each part builds on the one before. Myntra facts come from Vaideeka's internship report (`05` has more detail). The Atlassian part is reconstructed from the resume bullets — anything marked *(check)* should be confirmed against what was actually built.

---

## Myntra (Jul – Dec 2025): Ratings & Reviews service

### What the service is

When you open a product on Myntra, that page (the Product Detail Page, PDP) is assembled from many small programs, each owning one job. One of them owns ratings and reviews. These small, independent programs are called **microservices**: each runs on its own and other services call it over the network — "give me the ratings for these product IDs". Vaideeka worked on the Ratings & Reviews microservice, written in **Go**, a language built for exactly this kind of network service.

The service stores ratings in **MySQL**, a normal relational database (tables, rows, SQL). But databases are slowish and every product page needs ratings, so the service also keeps copies of ready-made answers in a **cache** — fast storage you check first. The cache here is **Aerospike**, a key-value store: you give it a key (a style ID), it gives back the value (that style's rating) in under a millisecond.

Two cache words you'll use constantly:
- **Cache hit / miss.** If the answer is in the cache, it's a hit and you return it. If not, it's a miss: you compute it from the database and store it in the cache for next time.
- **TTL (time to live).** How long an entry stays before it's thrown away. Here it's 7 days. A TTL is a safety net — even if something goes wrong, a wrong value can't survive forever.

### The problem

On Myntra, each colour (or size) of a product is a separate **style** with its own ID. A red shirt and the same shirt in blue are two styles. Styles of the same product form a group — call it a **cluster**.

Ratings were stored and shown *per style*. So the red shirt showed 4.4★ from 1.6k ratings, the blue one 4.3★ from 3.5k, and a newly added green one showed "No Ratings" — even though it's the same shirt with 5,000+ ratings. The goal: every style in a cluster shows **one combined rating**.

Why was it broken? Two places:
- **When a page loaded and the cache missed**, the service filled the cache with that one style's own rating, not the cluster's.
- **When someone submitted a rating**, the database was updated and then only *that* style's cache entry was refreshed. Its siblings kept their old numbers.

### The fix, step by step

**1. Find out who the siblings are.** The service doesn't know which styles belong together — another internal service, **Accio**, does. So whenever the rating needs recomputing, you ask Accio "what are the related styles of X?". Many teams depend on Accio, so you mustn't flood it. Instead of one call per style, you send **20 styles per call** (Accio's team recommended 20–25). That alone cuts the calls about 20×.

**2. Cache the sibling list.** Which styles belong together rarely changes, so the list itself goes into Aerospike for 7 days. Next time anyone in that cluster needs a recompute, the list is already there — no Accio call.

**3. Compute the combined rating correctly.** Add up how many 1★, 2★, 3★, 4★ and 5★ ratings exist across *all* styles in the cluster, then take the average of that. Don't average each style's average — that gives a style with 10 ratings the same weight as one with 1,000. Quick example: style A is 4.0★ from 1,000 ratings, style B is 5.0★ from 10. Average of averages says 4.5★. Correct answer: (4×1000 + 5×10) / 1010 ≈ **4.01★**.

**4. Write the result to every style in the cluster**, not just the one that triggered it. Now all colours show the same number.

**5. Protect Accio with a rate limiter.** Even with batching, a burst of cache misses (say right after a deploy) could hammer Accio. A **rate limiter** puts a cap on how fast you call it. The one used is a **token bucket**: imagine a bucket that holds up to, say, 10 tokens and gets refilled at a steady 3 tokens per second. Each call takes one token. If the bucket is empty, the call is refused. So a short burst of 10 goes straight through, but over time you can't exceed 3 per second. The limit was set **per pod** — a pod is one running copy of the service — starting at **180 requests per minute per pod** (which is exactly 3 per second).

One catch worth mentioning: since each pod has its own bucket, the real total is *per-pod limit × number of pods*, and that changes whenever the service scales up or down. For a strict overall cap you'd keep a shared counter (e.g. in Redis) or let Accio enforce a quota.

**6. Don't lose refused work — the error queue.** What happens to a call the limiter refuses? Dropping it would leave that cluster wrong. Instead, the event is moved to an **error queue** on **Kafka** and processed again later.

Kafka in one paragraph: it's a durable log of events. One service writes events into a named stream (a *topic*); other services read from it at their own pace and remember how far they've read. Messages stay for days, so nothing is lost if a reader is slow or restarts. The cache-refresh events and the error queue both live on Kafka.

The result is **eventual consistency**: for a short while a cluster might show a slightly old number, but it always catches up. For ratings that's perfectly fine.

One honest weakness: retries from the error queue were **instant**, because the Go services had no delay function available. Instant retries can hit the same empty bucket and fail again. The better design is to wait before retrying and wait longer each time (1 s, 2 s, 4 s… — **exponential backoff**, with a little randomness called *jitter* so retries don't all land together), give up after a few attempts, and move hopeless messages to a **dead-letter queue** for someone to inspect. Say this yourself — interviewers love it.

### The SQL bug (your best debugging story)

Before the fix, the database work was done with two SQL queries. To group styles into clusters, the code built a SQL `CASE` expression on the fly — basically "if the style is in this list, its group is A; else if it's in that list, its group is B; …". The code built those lines by looping over a Go **map** (a key → value lookup table) of group → styles.

Two facts collided:
1. SQL `CASE` checks the lines **in order and stops at the first match**.
2. Go **deliberately shuffles the order** you get when you loop over a map — every run is different.

Some style IDs appeared in two groups' lists (for example, 34476499 was a group of its own *and* in 34220967's list). Whichever line happened to come first won. So **the same input gave different groupings on different runs** — the bug appeared and disappeared, which is what made it hard to catch.

The fix: one simpler query that just returns ratings and review/image counts **per style**, and the grouping done in Go code where the logic is explicit, predictable and testable. The new query is **parameterised** — values are passed separately with `?` placeholders rather than glued into the SQL text, which also blocks **SQL injection** (attackers smuggling SQL in through input).

That's where the resume's **"50% fewer database connections"** comes from: two queries became one, so one database round trip instead of two **for each rating computation**. If they ask "50% of what?", say exactly that. It's not a service-wide number.

Two small details you might be asked:
- The query uses a `LEFT JOIN` so styles with no reviews still appear, `COALESCE(x, 0)` to turn the resulting blanks into 0, and `COUNT(DISTINCT …)` because joining a review to several images multiplies rows and would over-count.
- In Go, counts were added up in one flat map whose key packs two numbers into one: `(mainID << 32) | rating` — group ID in the top 32 bits, star rating in the bottom 32. That avoids creating a separate inner map per group (the report estimates about 50% less memory). It works because style IDs (~8 digits) and ratings (1–5) both fit in 32 bits.

### The 5.7-million-style cache warmup

Here's the last problem. When the fix went live, the cache had no sibling lists at all. Every product page would miss the cache and call Accio at the same time. So before launch the cache had to be **warmed up** — pre-filled for all **5,710,991** live styles.

Vaideeka built an endpoint that published refresh events in batches of 20 styles (about 285,000 events), run overnight when traffic is low, through the same code path as normal traffic. Each run retried only what failed last time.

**Run 1 refreshed only about 22%** (1.28M). Two reasons:
- Events were fired as fast as possible. A token bucket allows a burst and then only its refill rate, so the bucket emptied almost instantly and nearly everything after that was refused.
- One refused Accio call failed its whole batch of 20.

Runs 2 and 3 crept up. In **run 4** she added a **4 ms pause between events** and raised the limit. Now events arrived at roughly the rate the bucket refills, so they stopped being refused: **99.99%** success. Run 5 cleared the last 120. 100% after 5 runs.

The lesson, in her words: *"Work out the throughput budget first — pods × per-pod limit — and pace the sender to match it. Don't expect the rate limiter to absorb an unlimited burst."* What she'd do differently: calculate the pace up front, and retry failed styles one by one instead of failing a whole batch.

Before release, a **load test** (does it hold up under heavy traffic?) and a **memory-leak test** (does memory keep growing over time?) both passed.

### The stage environment (her first project)

Code usually goes through several environments: **dev** (for developers, with lots of faked parts), **stage** (a realistic copy of production for final testing) and **production** (real users). The ratings service had no stage, so changes went from half-fake dev straight to customers.

She built it: pointed the service at the stage versions of everything it talks to, with **timeouts** (how long to wait for an answer before giving up — without them, one slow dependency can freeze your whole service); created the MySQL database with the production structure and made-up data (real reviews are user data, so not copied); set up Aerospike; created database and event credentials and stored them in **Vault**, a secrets manager, so passwords never live in code; then deployed and checked the connections and logs.

### Smaller work
- **Ratings mismatch:** the reviews section of the page showed an old rating while the ratings section showed the new one — two separate caches, and only one was being refreshed. She made both refresh on the same events. *Every cached copy must update on the same triggers.*
- **Hacker Ramp** (Myntra's Gen AI hackathon): a tool that maps which internal services call which APIs. Her part parsed API definitions from GitHub links.
- **Big Fashion Festival** (Myntra's biggest sale): monitored the service and helped debug live issues. What you watch during a peak: error rate, slowest response times (p99 latency), cache hit rate, queue length.

### Myntra questions
- **"Walk me through it."** Problem (4.4 vs 4.3 vs "No Ratings") → why it broke → Accio in batches of 20 → cache the sibling list → combined rating written to every style → rate limiter and error queue → SQL bug and 2→1 queries → warmup. Two minutes, then stop.
- **"Why move grouping out of SQL into Go?"** The data per request is small; the SQL version was built from strings, buggy and untestable; Go is predictable, testable and saves a round trip.
- **"How does the cache stay in sync with the database?"** Every new rating recomputes from the database and overwrites every style in the cluster; a miss triggers a full recompute; the 7-day TTL is a safety net.
- **"What if two people rate two colours at the same moment?"** Each recompute reads the latest database state and overwrites the whole cluster, so whichever finishes last writes a correct, current value.
- **"How would this apply at Zomato?"** Ratings across outlets of one restaurant brand, or across variants of a dish — same pattern.

---

## Atlassian (May – Jul 2025): Marketplace app publishing

### What the job was

Atlassian makes Jira, Confluence and Bitbucket. **Atlassian Marketplace** is their app store: outside companies (vendors) publish apps there and customers install them. Publishing an app means creating its listing, version and details so it goes live.

That process had manual steps — someone looked at each request and created the app by hand. Slow, error-prone, and it couldn't keep up as requests grew. Vaideeka built a system that does it automatically.

The tools: **Kotlin**, a modern language that runs on the Java platform and mixes freely with Java code (its headline feature is *null safety* — a variable can't be empty unless you say it may be, which prevents a whole class of crashes); **Spring Boot**, the standard framework for building Java/Kotlin web services; and **AWS SQS**, Amazon's managed message queue.

### The core idea: don't make the caller wait

The simple way to build this: vendor sends a request, the server does all the work — checks, creates the app, sets up the listing — and replies when done. Problem: that work is slow and touches several other systems. The vendor waits, and if any step hiccups, their whole request fails.

The better way is **event-driven**. The server does only the quick part (checks the request is valid), drops the request as a message onto a **queue**, and immediately replies "got it". Separate programs called **workers** pick messages off the queue and do the slow work in the background.

What you gain: the API is always fast; a sudden flood of requests just waits in the queue instead of crashing anything; a failed step can be retried without the vendor knowing; and you can add more workers without touching the API. What it costs: the result isn't instant, there are more pieces, and you have to track status and guard against duplicates.

### The flow *(check details)*

```
Vendor ─► Spring Boot API
            check permissions
            validate the request ── invalid ──► 400 Bad Request (with reasons)
            save it as RECEIVED
            put a message on SQS
            reply 202 Accepted + request ID
                     │
                     ▼
                 SQS queue ── keeps failing ──► dead-letter queue
                     │
                     ▼
            Worker pool
            already done this request ID? → skip
            create the app via Marketplace APIs
            success → mark PUBLISHED, delete the message
            failure → do nothing; message comes back and is retried
```

`202 Accepted` is the honest reply for async work: "received and queued, not finished yet" (vs `201 Created`, which means done).

### How SQS makes this reliable

The clever part of SQS is the **visibility timeout**. When a worker takes a message, SQS doesn't delete it — it just *hides* it for a while (30 seconds by default). If the worker finishes and deletes it, it's gone. If the worker crashes halfway, it never deletes it, so after the timeout the message **reappears** and another worker picks it up. You get crash recovery without writing any code for it. Set the timeout longer than your slowest job, or a slow-but-healthy worker's message will reappear and get done twice.

If a message keeps failing — say the request is broken in a way validation didn't catch — after a set number of attempts SQS moves it to a **dead-letter queue (DLQ)**, so it can't clog the pipeline forever. You set an alert on the DLQ and inspect what lands there.

Other SQS facts: workers use **long polling** (wait up to 20 s for a message rather than asking repeatedly and getting empty answers); a receive returns up to 10 messages; messages can be up to 256 KB. A standard queue delivers **at least once** — occasionally the same message twice. A **FIFO** queue keeps strict order and removes duplicates, at lower throughput.

When something downstream is down, retry with **exponential backoff and jitter** — wait 1 s, then 2 s, then 4 s, plus a little randomness so all workers don't retry at the same instant and knock the recovering service over again.

### Worker pools

A **worker pool** is a fixed number of workers (threads, or Kotlin *coroutines*, which are much lighter) pulling messages and processing them in parallel. Why a fixed number? With unlimited workers you'd run out of memory and flood the systems you're calling. A bounded pool is a built-in speed limit. And when every worker is busy, they simply stop taking new messages — the rest wait safely in the queue. That's **backpressure**: the queue is your buffer.

To handle more load you add more worker *machines* on the same queue — **horizontal scaling** (more machines, rather than a bigger machine). This works because workers keep nothing of their own; all state is in the queue and the database. SQS gives each message to only one worker at a time. You can autoscale on queue length.

### Duplicates and idempotency (the follow-up they'll definitely ask)

Remember: the message comes back if it isn't deleted. Now picture this — a worker creates the app successfully, then crashes *just before* deleting the message. The message reappears, another worker picks it up, and you've published the app twice.

So processing must be **idempotent**: doing it twice has the same effect as doing it once. Three layers of protection:
- Every request has a unique ID. Before doing anything, check "have I already processed this ID?".
- A **unique constraint** in the database (e.g. on vendor + app + version), so a duplicate insert simply fails.
- Status only moves forward: RECEIVED → IN_PROGRESS → PUBLISHED. A finished request can't be restarted.

### Validation

Validation happens in the API, *before* queuing, in two layers. Format checks — required fields present, right types, sensible lengths — are annotations on the request class in Spring (`@NotBlank`, `@Size`, switched on with `@Valid`). Business checks — does this vendor own this app? is the version newer than the last one? — are code. Doing it before the queue means the vendor gets a clear error immediately, and workers never waste time on requests that were always going to fail.

### Spring Boot in a nutshell
You write classes; Spring creates them and hands each one the others it needs — **dependency injection**. Your code doesn't build its own dependencies, so it's loosely coupled and easy to test with fakes. The usual layers: `@RestController` (handles HTTP) → `@Service` (business logic) → `@Repository` (database). `@Transactional` on a method runs it inside a database transaction — all of it commits or none of it does.

### Atlassian questions
- **"What if a worker dies halfway?"** The message isn't deleted, so it reappears after the visibility timeout and another worker retries. The idempotency check stops a duplicate.
- **"Why SQS and not Kafka?"** SQS is a simple **job queue**: each message is done once by one worker, then deleted. Kafka keeps messages for days so many different services can read the same stream and replay it. "SQS at Atlassian for jobs, Kafka at Myntra for events — I've used both."
- **"Why not just run a thread pool inside the API server?"** Work held in memory is lost if the server crashes, and you can't scale the API and the workers separately. A queue is durable and gives retries for free.
- **"10× traffic?"** The queue absorbs it; add workers based on queue length; cap concurrency so the systems you call aren't overwhelmed.
- **"How would you monitor it?"** Queue length, age of the oldest message, DLQ size, success/failure rate, processing time.
- **"What would you improve?"** The **outbox pattern**. Saving the request to the database and putting it on SQS are two separate steps — crash between them and the message is lost. Instead, save the request *and* an "outbox" row in the same database transaction (both or neither), and let a small background process move outbox rows onto SQS.
- **"How did you test it?"** *(check)* Unit tests with a fake SQS client, integration tests against a local AWS emulator such as LocalStack, and deliberately failing a worker to confirm the retry and DLQ behaviour.

**STAR version:** app publishing had manual steps, so releases were slow and error-prone (**S**). The task was to automate it reliably at high volume (**T**). She built a Spring Boot API that validates and queues requests on SQS, worker pools that create the apps, retries and a DLQ, and idempotent processing (**A**). Result: no manual work in the publishing path, faster and more reliable releases, and capacity that grows by adding workers (**R** — add a real number if there is one).

---

## Which story for which behavioural question

| Question | Story |
|---|---|
| A challenging problem | Myntra aggregated ratings across 5.7M styles |
| Improving performance | Myntra 2 → 1 queries; TraceX N+1 fix (325 → 96 ms) |
| A hard bug | The Go map-order bug |
| A failure | Warmup run 1 at 22%, and what fixed it |
| Learning fast | Go at Myntra, Kotlin at Atlassian |
| Teamwork / leadership | Hack-A-Sol win, iDEA top 10 |
| Why Zomato | Scale, real-time systems, same stack (Go, Kafka, MySQL, caching) |

Keep each answer to 60–90 seconds, say "I" for your work and "we" for the team's, end with a number, and have one "what I'd do differently" ready.
