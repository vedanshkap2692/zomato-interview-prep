# Zomato SDE-1 Prep: Vaideeka Agrawal

| # | File | What's inside | Study priority |
|---|---|---|---|
| 1 | `01_Zomato_Interview_Guide.md` | Process and round format, what interviewers grade, DSA patterns ranked, ~120 reported/expected problems with approaches, 10-day plan, sources | Start here |
| 1b | `01b_DSA_Must_Code.md` | C++ code for every must-know problem (LRU, LFU, BST delete, graphs, sliding window, DP, intervals, binary search, linked lists, Skyline…) | Daily: type from memory |
| 1c | `01c_CS_Fundamentals_SQL_Design.md` | DBMS (indexing, ACID, SQL vs NoSQL), 14 SQL queries, OS, CN, OOP/HashMap, HLD (rate limiter, notifications, food ordering, seat locking), LLD/schema, Kafka/Redis | Days 2–9 |
| 2 | `02_Resume_and_Skills_Brief.md` | 1.5–2 hour read: intros, every resume line explained with answers, every listed skill explained in plain words (languages, SQL, REST, Kafka, Docker/K8s, ML, system design, coursework), risky claims, checklist | Before every mock |
| 3 | `03_Projects_TraceX_SupplySenseAI.md` | 25-minute read, code-verified: what each project does, architecture, what you built, every term explained simply, weak spots, likely questions | Days 4–7 |
| 4 | `04_Experience_Myntra_Atlassian.md` | 25-minute read: Myntra and Atlassian explained simply (SQS, worker pools, idempotency, token bucket, caching), likely questions, STAR stories | Days 1–4 |
| 5 | `05_Myntra_Deep_Dive.md` | 30-minute read, verified from the internship report: background terms, stage onboarding, aggregated-ratings design, the SQL bug, the 5.7M warmup, top 10 questions, 3 STAR stories | Days 1–3 |
| 6a | `06a_SD_Interview_Requests_Scaling.md` | **System design master, ch. 1:** what Zomato actually asks (from ~100 interview reports), how you're scored, the 7-step framework with a worked example; DNS, TCP/TLS, HTTP, MQTT, latency numbers, percentiles; scaling from 1 server to thousands, load balancers, gateways, service mesh, CDN, autoscaling, microservices | SD: start here |
| 6b | `06b_SD_Databases.md` | **Ch. 2:** SQL vs NoSQL with real company choices, B+ vs LSM trees, indexes, ACID, isolation anomalies with timelines, MVCC, deadlocks, normalisation, replication, sharding, consistent hashing, CAP/PACELC, 2PC vs sagas, IDs, OLTP vs OLAP, interview traps | SD |
| 6c | `06c_SD_Caching_and_Async.md` | **Ch. 3:** caching patterns and races, eviction, stampede/penetration/hot keys (Facebook leases), Redis deep dive; queues, Kafka and SQS deep dives, delivery semantics, idempotency, retries, DLQ, outbox, CDC, sagas, delayed jobs | SD |
| 6d | `06d_SD_APIs_Reliability_Concurrency.md` | **Ch. 4:** REST/gRPC, idempotency keys (Stripe), pagination, webhooks (Razorpay), polling vs SSE vs WebSocket vs MQTT, push limits; timeouts, circuit breakers, load shedding, rate limiting in depth, availability math, observability, chaos testing; race conditions, locking, Redlock and fencing | SD |
| 6e | `06e_SD_Geo_Search_Media_Estimation_LLD.md` | **Ch. 5:** geohash, quadtree, S2, H3, serviceability, ETA, dispatch and batching (Swiggy, DoorDash, Uber, Grab); search and ranking (Zomato's Solr); uploads and CDNs; estimation; security; LLD, SOLID and design patterns | SD |
| 6f | `06f_SD_How_Zomato_Is_Built_and_Final_Exam.md` | **Ch. 6:** Zomato's real architecture from its own engineering blog (NYE scale, databases, MQTT tracking, Time service, Solr, NYE playbook), one order end to end, 50-question final exam | SD: read last |
| 7a | `07a_SD_QA_Core_Zomato_Designs.md` | **Solutions 1 (Q1–Q7):** food delivery end to end, live tracking, dispatch, nearby search, ETA, notifications, rate limiter with code — each with diagrams, failures, real-company callouts and follow-ups | One design a day |
| 7b | `07b_SD_QA_Payments_Booking_and_More.md` | **Solutions 2 (Q8–Q20):** payments and ledger, seat booking and waiting rooms, Blinkit/Zepto inventory, coupons, ratings, delayed jobs, NYE peak, URL shortener, chat, streaming, uploads, feature flags, leaderboards | One design a day |
| 7c | `07c_SD_QA_LLD_Projects_RapidFire.md` | **Solutions 3 (Q21–Q32):** distributed cache, **your own projects as HLD** (Myntra, Atlassian, TraceX, SupplySense), LLD in Java (menu, banner, cart, schema, open-now, parking lot, Splitwise, LRU/LFU, rate limiter, notifications), 60 rapid-fire traps | Before the interview |

**Round 1 for an off-campus fresher (2024–26 reports):**
1. A 20–40 minute deep dive into your resume and internships.
2. Database questions.
3. One or two medium-to-hard DSA problems, typed into a Google Doc. They must be fully working code.
4. Often a small high-level design or schema question.

**System design reading plan:** 06a → 06f teach everything from zero (about 9 hours in total; split across several days). Then 07a → 07c: for each question, sketch your own answer for 10 minutes first, then read the solution and say it out loud. All diagrams are Mermaid and render on GitHub.


**Sources:** `sources/` holds the research notes behind 06–07, with a link for every fact: Zomato/Eternal engineering, Indian companies (Swiggy, Blinkit, Zepto, BookMyShow), global companies (Uber, DoorDash, Grab…), payments/rate limiting/caching patterns (Stripe, Razorpay, Cloudflare, Facebook…), and ~100 interview reports.
