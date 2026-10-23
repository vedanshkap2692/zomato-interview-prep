# Zomato SDE-1 Prep: Vaideeka Agrawal

| # | File | What's inside | Study priority |
|---|---|---|---|
| 1 | `01_Zomato_Interview_Guide.md` | **Section 0 = Round 2 (new):** format, how it differs from R1, R2 DSA ranked from 81 reports, 52-problem tiered list, follow-ups, rejection reasons, 7-day plan. Then the original process guide, patterns, ~120 problems, sources | **Start here for Round 2** |
| 1b | `01b_DSA_Must_Code.md` | C++ for every must-know problem; **new section 12 = Round 2 code** (rate-limiter classes, percentile tracker, weighted Jump Game IV, LC 444, Ctrl+R trie, browser history, min/max stack, open-hours index, one-offs actually asked, hard DP, blocking queue) — compiled and tested | Daily: type from memory |
| 1c | `01c_CS_Fundamentals_SQL_Design.md` | DBMS (indexing, ACID, SQL vs NoSQL), 14 SQL queries, OS, CN, OOP/HashMap, HLD (rate limiter, notifications, food ordering, seat locking), LLD/schema, Kafka/Redis | Days 2–9 |
| 2 | `02_Resume_and_Skills_Brief.md` | 1.5–2 hour read: intros, every resume line explained with answers, every listed skill explained in plain words (languages, SQL, REST, Kafka, Docker/K8s, ML, system design, coursework), risky claims, checklist | Before every mock |
| 3 | `03_Projects_TraceX_SupplySenseAI.md` | 25-minute read, code-verified: what each project does, architecture, what you built, every term explained simply, weak spots, likely questions | Days 4–7 |
| 4 | `04_Experience_Myntra_Atlassian.md` | 25-minute read: Myntra and Atlassian explained simply (SQS, worker pools, idempotency, token bucket, caching), likely questions, STAR stories | Days 1–4 |
| 5 | `05_Myntra_Deep_Dive.md` | 30-minute read, verified from the internship report: background terms, stage onboarding, aggregated-ratings design, the SQL bug, the 5.7M warmup, top 10 questions, 3 STAR stories | Days 1–3 |
| 6 | `06_System_Design_Master.md` | **System design master (~55 pages, one doc):** what Zomato actually asks (from ~100 reports) and the 7-step framework; requests and scaling; databases; caching and Kafka/queues; APIs, real-time, reliability, rate limiting, concurrency; geo, search, dispatch, ETA, estimation, security, LLD; how Zomato is actually built; 50-question final exam. 53 diagrams, real-company sources | SD: read first (~3 h) |
| 7a | `07a_SD_QA_Core_Zomato_Designs.md` | **Solutions 1 (Q1–Q7):** food delivery end to end, live tracking, dispatch, nearby search, ETA, notifications, rate limiter with code — each with diagrams, failures, real-company callouts and follow-ups | ~55 min |
| 7b | `07b_SD_QA_Payments_Booking_and_More.md` | **Solutions 2 (Q8–Q20):** payments and ledger, seat booking and waiting rooms, Blinkit/Zepto inventory, coupons, ratings, delayed jobs, NYE peak, URL shortener, chat, streaming, uploads, feature flags, leaderboards | ~65 min |
| 7c | `07c_SD_QA_LLD_Projects_RapidFire.md` | **Solutions 3 (Q21–Q32):** distributed cache, **your own projects as HLD** (Myntra, Atlassian, TraceX, SupplySense), LLD in Java (menu, banner, cart, schema, open-now, parking lot, Splitwise, LRU/LFU, rate limiter, notifications), 40 rapid-fire traps | ~55 min |
| 8 | `08_SD_Revision_Sheet.md` | **SD revision sheet (~1 h):** framework, numbers to quote, concept cheat sheets, a design card for every question, top 40 follow-ups, 10-minute checklist | Night before / morning of |

**Round 1 for an off-campus fresher (2024–26 reports):**
1. A 20–40 minute deep dive into your resume and internships.
2. Database questions.
3. One or two medium-to-hard DSA problems, typed into a Google Doc. They must be fully working code.
4. Often a small high-level design or schema question.

**System design reading plan:** read `06_System_Design_Master.md` (~3 hours, can split over two days). Then practise `07a` → `07c`: sketch each design for 10 minutes before reading the solution. Night before and morning of: `08_SD_Revision_Sheet.md`. All diagrams are Mermaid and render on GitHub.


**Sources:** `sources/` holds the research notes behind 06–07, with a link for every fact: Zomato/Eternal engineering, Indian companies (Swiggy, Blinkit, Zepto, BookMyShow), global companies (Uber, DoorDash, Grab…), payments/rate limiting/caching patterns (Stripe, Razorpay, Cloudflare, Facebook…), and ~100 interview reports, plus `round2_reports.md` (Round 2+ evidence from 81 write-ups).
