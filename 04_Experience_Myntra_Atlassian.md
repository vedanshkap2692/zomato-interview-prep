# Internships: Myntra and Atlassian

> **Reading time:** about 25 minutes. Written for someone seeing this work for the first time; every technical word is explained where it first appears.
> **Sources:** Myntra comes from Vaideeka's internship report, so the facts are verified; `05_Myntra_Deep_Dive.md` has the full detail. **Atlassian is reconstructed from the three resume bullets plus standard practice for this kind of system.** Anything marked *(check)* is a likely design, not a known fact — replace it with what was actually built, or say "that part was owned by another team".

---

# Part 1: Myntra — Software Engineer Intern, Product Detail Page (14 Jul – 5 Dec 2025)

## 1.1 Where you were

Myntra is a fashion e-commerce site. The **Product Detail Page (PDP)** is the page you see when you open one product. You were on the Product-SizeFit team, working on the **Ratings & Reviews microservice**, written in **Go**.
- A **microservice** is a small, separate program that owns one job (here, ratings and reviews) and talks to other services over the network, instead of everything being one big application.
- **Go** is a language from Google built for network services. It handles thousands of simultaneous tasks cheaply using *goroutines* (very lightweight threads).

## 1.2 Project 1: a stage environment

Software usually runs in several **environments**: *dev* (for developers, often with fake dependencies), *stage* (a copy of production used for final testing) and *production* (what real users hit). The ratings service had only dev and production, so there was nowhere realistic to test a change before releasing it.

You built its stage environment:
1. Pointed it at the stage versions of every service it talks to, with credentials and **timeouts** (a limit on how long to wait for a reply, so one slow service can't freeze yours).
2. Created the **MySQL** database (a relational database — data in tables, queried with SQL) with the same structure as production, filled with **mock data** (made-up, since real reviews are user data).
3. Set up **Aerospike**, the cache. A **cache** is fast storage that keeps copies of frequently read data so you don't hit the slower database every time.
4. Created the database and event-system credentials and stored them in **Vault**, a secrets manager, so passwords never sit in code.
5. Deployed it and checked that the database and cache connections worked and the logs were clean.

**Why it matters:** bugs in configuration and in how services talk to each other now show up in stage instead of in front of customers.

## 1.3 Project 2: aggregated ratings — the main story

**The problem.** Each colour or size of a product is a separate **style** with its own ID. Ratings were stored and shown *per style*, so one colour showed 4.4★ from 1.6k ratings, another 4.3★ from 3.5k, and a new colour showed **"No Ratings"** even though the product had thousands. Related styles form a group called a **cluster**; the goal was one **aggregated rating** per cluster, shown on every style in it.

**Why it was broken.** When a page loaded and the cache didn't have the rating, the service filled the cache with that one style's own rating. When someone submitted a rating, only that style's cache entry was refreshed — its sibling styles were never updated.

**What you built — the flow:**

```
Page load misses the cache   or   a user submits a rating
                     \           /
                      ▼         ▼
           rate limiter (token bucket)
             │ allowed           │ rejected
             ▼                   ▼
   ask Accio for related      Kafka error queue → retried later
   styles (20 per call)
             ▼
   cache the related-styles set (7 days)
             ▼
   one SQL query for the whole cluster → add up in Go
             ▼
   write the combined rating to EVERY style in the cluster
```

Each piece, simply:
- **Accio** is Myntra's internal service that knows which styles belong together. You call it in **batches of 20** instead of once per style — about 20× fewer requests on a service many teams share.
- The **related-styles set is cached for 7 days**, because it rarely changes and is needed every time. A **TTL** (time to live) is how long a cached value is kept before it expires.
- A **rate limiter** caps how fast you call Accio, so traffic spikes don't overload it. You used a **token bucket**: a bucket holds up to B tokens and refills at a steady rate; each call uses one token; if the bucket is empty, the call is refused. That allows short bursts but caps the average. It was set **per pod** (a *pod* is one running copy of the service), starting at 180 requests per minute per pod.
- **Kafka** is a durable event log: one service writes events, others read them at their own pace. Calls the rate limiter refused were put on a Kafka **error queue** and processed again later, so the result becomes correct eventually instead of being lost. That's **eventual consistency**: data may be briefly out of date but always catches up.
- **The aggregate maths:** add up the number of 1★, 2★ … 5★ ratings across all styles in the cluster, then take the weighted average. *Never average the styles' averages* — a style with 2 ratings would count as much as one with 2,000.

**The SQL bug — your best debugging story.** The old code built a SQL `CASE` expression ("if the style is in this list, its group is X; else if it's in that list…") by looping over a Go **map** (a key-value lookup table). Two facts collided:
1. SQL `CASE` stops at the **first** matching rule.
2. Go deliberately returns map entries in a **random order** each time.

Some styles appeared in two groups' lists. Depending on which rule happened to come first, they landed in different groups — so **the same input gave different results on different runs**, which made it hard to find. You replaced the two old queries with **one parameterised query** (values passed separately with `?`, never pasted into the SQL text, which also blocks **SQL injection** — attackers sneaking SQL in through input) and did the grouping in Go, where it is predictable and testable.

**That's the "50% fewer DB connections":** two queries became one, so one database round trip instead of two **per rating computation**. If asked "50% of what?", say exactly that — it's not a service-wide number.

**The 5.7-million-style cache warmup.** Before launch, the cache had no related-style data at all; launching like that would send every page load to Accio at once. So you **warmed up** the cache — pre-filled it for all **5,710,991** live styles — by publishing refresh events in batches of 20, overnight, through the same code path as normal traffic.
- **Run 1 refreshed only about 22%.** Events were published as fast as possible, so the token bucket emptied almost instantly and most calls were refused; and one refused call failed its whole batch of 20.
- **Fix:** a **4 ms pause between events** plus a higher rate limit, so events arrived at about the rate the bucket refills. Run 4 reached 99.99%; run 5 cleared the last 120. 100% after 5 runs.
- **Lesson:** "Work out the throughput budget first — pods × per-pod limit — and pace the sender to match. Don't rely on the limiter to absorb an unlimited burst."

**Testing:** before/after checks that sibling styles show the same rating, plus a **load test** (does it hold up under heavy traffic?) and a **memory-leak test** (does memory keep growing over time?).

## 1.4 Smaller work
- **Ratings mismatch bug:** the reviews section of the page showed an old rating while the ratings section showed the new one — two caches, only one refreshed. You made both refresh on the same events. *Lesson: every cached copy must update on the same triggers.*
- **Hacker Ramp** (Myntra's Gen AI hackathon): a tool mapping which services call which APIs; you parsed API definitions from GitHub links.
- **Big Fashion Festival** (Myntra's biggest sale): you monitored the service and helped debug live issues. Watch error rate, slowest-response times (p99 latency), cache hit rate and queue length.

## 1.5 Myntra questions — short answers
- **"Why move the grouping from SQL into Go?"** The data per request is small; the SQL version was built from strings, buggy and untestable; the Go version is predictable, testable and saves a round trip.
- **"Weakness of your error queue?"** Retries were instant (the services had no delay function), so retries could hit the same empty bucket again. Better: delayed retry queues, **exponential backoff** (wait 1 s, 2 s, 4 s… with some randomness), a maximum number of attempts, then a **dead-letter queue** for messages that keep failing.
- **"How does the cache stay consistent with the database?"** Every new rating triggers a recompute from the database that overwrites every style in the cluster; a cache miss triggers a full recompute; the 7-day TTL is a safety net.
- **"Per-pod limits — any problem?"** The total limit grows with the number of pods. For a strict overall cap, use a shared counter (e.g. in Redis) or a quota enforced by Accio.
- **"How does this apply at Zomato?"** Restaurant ratings across outlets of one brand, or dish ratings across menu variants — same pattern.

---

# Part 2: Atlassian — Software Engineer Intern, Marketplace (May – Jul 2025)

## 2.1 Where you were

Atlassian makes Jira, Confluence and Bitbucket. **Atlassian Marketplace** is their app store: outside companies (*vendors*) publish apps and plugins there, and customers install them. *Publishing* an app means creating its listing, version, details and compatibility so it goes live.

**The problem:** publishing had manual steps — someone checked each request and created the app by hand. That was slow, error-prone, and couldn't keep up as requests grew.

**What you built (from the resume):** an **event-driven** system in **Kotlin** and **Spring Boot** on **AWS**, using an **SQS** queue and **worker pools**, that validates requests and creates apps automatically.

The tools, simply:
- **Kotlin** is a modern language that runs on the Java Virtual Machine and works seamlessly with Java code (hence "Kotlin, Java" on the resume). Its big feature is **null safety**: a variable can't be empty (null) unless you mark it as allowed to be, which prevents a whole class of crashes.
- **Spring Boot** is the standard Java/Kotlin framework for building web services. You write classes; Spring creates them and connects them to each other (**dependency injection** — a class receives what it needs instead of creating it, which makes code loosely coupled and easy to test).
- **AWS** is Amazon's cloud. **SQS (Simple Queue Service)** is its managed message queue.

## 2.2 Event-driven, in plain words

In a normal **request-response** system, the caller waits while the server does all the work. If the work is slow or fails halfway, the caller sees the failure.

In an **event-driven** system, the server records the request as a message (an *event*) on a **queue** and replies straight away. Separate **workers** pick messages off the queue and do the slow work.

Why this is better here:
- The API replies fast, even when publishing takes a while.
- A burst of requests (say, a release day) just waits in the queue instead of crashing the system.
- If a step fails, the message can be retried without the vendor noticing.
- The API and the workers can be scaled separately.

The cost: the result isn't instant, there are more moving parts, and you need to track status and protect against duplicates.

## 2.3 The flow *(check details)*

```
Vendor ──► Spring Boot API
             1. check login and permissions
             2. validate the request ──── invalid ──► reply "400 Bad Request" with reasons
             3. save it with status RECEIVED
             4. put a message on SQS
             5. reply "202 Accepted" + request ID
                        │
                        ▼
                    SQS queue ──── fails too many times ──► dead-letter queue (DLQ)
                        │
                        ▼
             Worker pool (several servers × several workers each)
             - already processed this request ID? skip
             - create the app through Marketplace APIs
             - success: mark PUBLISHED and delete the message
             - failure: leave it; it reappears and is retried
```

**HTTP status codes in there:** *400 Bad Request* = your request is invalid; *202 Accepted* = received and queued, not finished yet (the honest code for async work); for comparison, *201 Created* = done.

## 2.4 The concepts behind each resume bullet

**Validation (bullet 3).** Two kinds:
- *Format checks:* required fields present, correct types and lengths. In Spring these are annotations on the request class such as `@NotBlank` and `@Size`, switched on with `@Valid`.
- *Business checks:* does this vendor own this app? Is the version newer than the last one?

Checking *before* queuing gives the vendor a clear error straight away, and stops workers wasting time on requests that will fail anyway.

**How SQS works (bullet 2).**
- **Visibility timeout** — the key idea. When a worker takes a message, SQS doesn't delete it; it **hides** it for a set time (default 30 s). If the worker finishes and deletes it, it's gone. If the worker crashes, the message **reappears** after the timeout and another worker picks it up. That's how you get fault tolerance without writing extra code. Set the timeout longer than the slowest job.
- **Dead-letter queue (DLQ):** if a message fails a set number of times, SQS moves it to a separate queue so one broken message can't block everything. You alert on the DLQ and inspect it.
- **At-least-once delivery:** a standard SQS queue may occasionally deliver a message twice. (A *FIFO* queue keeps order and removes duplicates, with lower throughput.)
- **Long polling:** a worker waits up to 20 s for messages instead of asking repeatedly and getting empty answers — cheaper and less wasteful.
- Up to 10 messages per receive; up to 256 KB per message.

**Worker pools.** A **fixed** number of workers (threads or Kotlin coroutines — lightweight tasks) pull messages and process them in parallel. Why fixed? Unlimited workers would run out of memory and overwhelm the downstream APIs and database; a bounded pool is a natural speed limit. If all workers are busy, they simply stop taking messages, and the rest wait safely in the queue — the queue acts as a buffer. This is called **backpressure**.

**Fault tolerance.**
- Worker crashes → message reappears → retried.
- A message that always fails → DLQ.
- Downstream service down → retry with **exponential backoff and jitter** (wait 1 s, 2 s, 4 s… plus a small random amount so all retries don't hit the recovering service at the same instant).

**Idempotency — the most likely follow-up.** *Idempotent* means doing something twice has the same effect as doing it once. Because SQS can deliver a message twice — for example, a worker creates the app, then crashes before deleting the message — the same request may be processed again. Protections:
- Give every request a unique ID and check "already done?" before acting.
- Add a database **unique constraint** (e.g. on vendor + app + version) so a duplicate insert fails harmlessly.
- Only allow status to move forward: RECEIVED → IN_PROGRESS → PUBLISHED.

**Horizontal scaling.** *Horizontal* scaling = adding more machines (vs *vertical* = a bigger machine). Workers keep no data of their own — everything lives in the queue and the database — so you can just run more worker servers on the same queue; SQS hands each message to only one of them at a time. Scale automatically on queue length.

## 2.5 Atlassian questions — short answers

1. **"Explain your project end to end."** Use the 2.3 flow in about 90 seconds, then stop.
2. **"What if a worker dies halfway?"** The message isn't deleted, so it reappears after the visibility timeout and another worker retries; the idempotency check stops a duplicate app.
3. **"How do you make sure an app isn't published twice?"** Request-ID check plus a database unique constraint; forward-only status.
4. **"Why SQS and not Kafka?"** SQS is a simple managed **job queue**: each message is handled by one worker, then deleted. **Kafka** keeps messages for days so many different services can each read the same stream and replay it. "I've used both — SQS at Atlassian for jobs, Kafka at Myntra for events."
5. **"Why not just a thread pool inside the API server?"** Work in memory is lost if the server crashes, and the API and workers can't scale separately. A queue is durable and retries for free.
6. **"What happens at 10× traffic?"** The queue absorbs the burst; add workers based on queue length; cap concurrency so downstream services aren't overwhelmed.
7. **"How would you monitor it?"** Queue length, age of the oldest message, DLQ count, success and failure rates, processing time.
8. **"What would you improve?"** The **outbox pattern:** saving the request to the database and sending it to SQS are two separate steps, so a crash between them loses the message. Instead, save the request *and* an "outbox" row in the same database transaction (both or neither), and let a small background process send outbox rows to SQS. Also: notify vendors when publishing finishes.
9. **"How did you test it?"** *(check)* Unit tests with a fake SQS client, integration tests against a local AWS emulator such as LocalStack, and deliberately throwing errors in a worker to confirm retries and the DLQ work.

**STAR story.** **S:** app publishing had manual steps, so releases were slow and error-prone. **T:** automate it so requests are validated and published without a person, reliably at high volume. **A:** built a Spring Boot API with validation that queues requests on SQS, worker pools that create the apps, retries and a DLQ for failures, and idempotent processing. **R:** no manual work in the publishing path, faster and more reliable releases, and capacity that grows by adding workers. *(Add a real number if you have one — requests per day, time saved.)*

---

# Part 3: Behavioural questions — which story to use

| Question | Story | Key line |
|---|---|---|
| A challenging problem | Myntra aggregated ratings, 5.7M styles | "The hard part was doing it safely against shared production services." |
| Improving performance | Myntra: 2 queries → 1; or TraceX N+1 fix, 325 → 96 ms | Before → change → after |
| A hard bug | Myntra: the random Go map order bug | Why it was hard (it came and went), how you found it, the fix |
| A failure | Myntra warmup run 1: only 22% | What went wrong, the fix, what you'd do first next time |
| Learning fast | Go at Myntra, Kotlin at Atlassian | How you learned: reading the codebase, asking mentors |
| Teamwork or leadership | Hack-A-Sol win, iDEA top 10 | Your role, a disagreement and how it was settled |
| Why Zomato? | Scale, real-time systems, your stack (Go, Kafka, MySQL, caching) | Name a Zomato feature you use and an engineering problem in it |

**Rules for STAR answers:** 60–90 seconds. "I" for your work, "we" for the team's. End with a number or a concrete result. Have one "what I'd do differently" ready for every story.
