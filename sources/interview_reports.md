# Design-round questions from real interview reports: Zomato/Eternal group and Indian product companies (2023–2026)

## How this was collected (and how far to trust it)

- **Main source: LeetCode Discuss.** WebFetch gets a 403 there, so I read the posts through LeetCode's public GraphQL endpoint (`ugcArticleDiscussionArticle`) and searched with `ugcArticleDiscussionArticles`. Every LeetCode report below was read in full or nearly in full. Each URL has the form `https://leetcode.com/discuss/post/<id>/`.
- **Other sources.** Medium posts were read through the freedium mirror, because medium.com blocks direct fetches. I also read GeeksforGeeks, the roundz Substack and daily.dev. Glassdoor, Naukri Code360 and AmbitionBox either blocked fetches or gave only snippets, so they are barely used. I found no YouTube or Reddit threads with usable detail.
- **All reports are anonymous and written by the candidates themselves.** Dates are post dates unless the post gives the interview date. The question wording is the candidate's own, and is quoted or closely paraphrased.
- **[UNCERTAIN] marks** anything I saw only in a search snippet, or that I could not read in full.
- **Skipped:** about 40 compensation-only posts (salary numbers, no rounds) and "what should I expect?" question posts.
- **Count:** about 100 reports with design-round content, plus about 10 "no design round" data points. 34 are from the Zomato/Eternal group (Zomato, District, Nugget, Blinkit, Hyperpure).

---

## Part 1. Report-by-report log

Format for each report: company | role/level and experience | date | rounds that contain design work | outcome | URL.

### 1A. Zomato / Eternal (incl. District, Nugget)

**Z1. Zomato SDE-1. 1-year intern, tier-3, Aug 2026. Offer.** https://leetcode.com/discuss/post/8434690/
- **R1 (SDE-2, GenAI team, 1h45m):**
  - Discussion of the candidate's current company architecture.
  - "One ongoing problem Zomato is trying to solve, how would you approach it."
  - About 10 minutes on scaling and failures.
  - Databases: SQL vs NoSQL, connection pooling, indexing, "how DBs make apps slow".
- **R2 (SDE-2, Core team):**
  - Hard DP problem; working code expected.
  - Event-driven architecture: pub/sub with Redis.
  - Redis deep dive: why it is single-threaded, internals, cache population and eviction, where caching helps, hot key (celebrity problem), thundering herd and fixes.
  - Pseudocode for an LFU cache, plus how LRU works.
- **R3 (EM, 10+ yrs):** Internship design decisions, then one HLD problem (not named). DB design, SQL vs NoSQL, DB choice for AI agents.
- **R4:** VP, culture.
- **Focus:** thought process and practical trade-offs.

**Z2. Zomato/Eternal SDE-1. About 1-2 YOE (earlier rejected for "lack of experience"), Jul 2026. Ghosted, then rejected.** https://leetcode.com/discuss/post/8377019/
- **R1 (Senior SE), HLD + DB schema:** Schema for delivery partners and orders. Follow-up: an SQL query for the top 5 delivery partners in an area by count of a given order type.
- **R2 (Team Lead):**
  - "Basic system design discussion around rate limiting."
  - Implement token bucket and fixed window in code. The candidate could not finish sliding window.
- **R3 (Senior EM, in person):** "Design a resilient notification system at Zomato scale". Requirements: avoid duplicate notifications, crash resilience, retry guarantees, fault tolerance. Then 3-4 follow-ups.
- **Stated rejection reasons:**
  - Could not confidently justify **"why MySQL vs PostgreSQL"**. The interviewer said the candidate should have said "I don't have enough real-world exposure to make that trade-off" instead of over-explaining.
  - Needed 2-3 iterations to understand one follow-up, which was "considered a negative signal".
  - The interviewer said the overall design was good.

**Z3. Same candidate, Zomato Search team re-interview, Jul 2026. Outcome unknown.** https://leetcode.com/discuss/post/8385519/
- **R1, notification system again.** New constraints:
  - How other teams and services integrate with it.
  - Deduplication.
  - **Cooldowns at user level and team level.**
  - APIs and scalability.
- **Production debugging:** The interviewer showed a real DB spike graph from about 10 days earlier and asked: what caused it, how would you investigate, how would you flatten it *without raising latency or scaling up*.
- **Graph problem:** a mathematical proof of correctness was wanted, not code.

**Z4. Zomato SWE1. 2 months FTE + 1.6 yrs internships, Feb 2026. Rejected.** https://leetcode.com/discuss/post/7609011/
- **R1 (Senior EM):**
  - **Payments system design:** end-to-end payment orchestration, chargebacks, dispute lifecycle (won/lost), refunds (full, partial, proration), audit and compliance (India), HLD of taxation + billing.
  - Event-driven: Kafka vs SNS+SQS (ordering, replayability, scale).
  - LLD in the last 20 minutes: "Zomato Enterprise", food benefits for companies and their employees. Services, order lifecycle, payment/order consistency, restaurant and delivery-partner flow.
  - Handling a DDoS: rate limiting (frontend and backend), caching, queue buffering. The interviewer was not convinced.
- **Feedback:**
  - Handled happy paths well but needed more depth on **non-happy paths**: microservice orchestration failures, retries, cross-service consistency, and DB modelling of complex workflows.
  - "~80% close". The role wanted more production experience; for an entry-level role the candidate would have moved forward.

**Z5. Eternal (Nugget team) SDE Intern + FTE. Tier-3 final year, Dec 2025. Offer.** https://leetcode.com/discuss/post/7407389/
- **R1, online ordering flow:**
  - User side: login, discover restaurants by location, menu, place order. Restaurant-manager side: login, multiple restaurants (many-to-many), menu updates.
  - Every API with sample request/response payloads, backend steps and schema.
  - SQL vs NoSQL for ordering (the candidate picked SQL; joins vs denormalization).
  - **Nearby restaurants in SQL:** bounding box was not enough; the interviewer pushed for geo-indexing/PostGIS and was not satisfied.
  - Manager-restaurant mapping moved from 1:1 to an M:N join table.
  - Service/controller layering and error handling.
  - **Image upload:** the candidate proposed base64; "what about big images?". The interviewer wanted **pre-signed S3 URLs**, which the candidate did not know.
- **R2 (onsite):**
  - Redesign the candidate's internship system ("phase 2").
  - Monolith vs microservices; when to split.
  - RabbitMQ, Redis Streams, Kafka.
  - **Rate limiter:** what it is, implement a simple one, then sliding window in pseudocode.
  - **Real-time support chat:** 1-to-1 customer-agent, then groups, then message visibility and permissions. Why chat leans NoSQL. MongoDB indexing, compound index left-prefix rule, write a Mongo query.
  - DSA problem.
- **R3:** VP, culture.

**Z6. Zomato (Eternal) SDE-1. Aug 2025. Offer.** https://leetcode.com/discuss/post/7206483/
- **R1:** DSA (minimum swaps), then **schema + approach for "show a banner X times in Y hours for Z durations"**: track frequency, enforce limits, reset counters.
- **R2:**
  - Project grill: moving heavy audio files to the backend, LLM token limits on meetings over an hour, why MySQL, table schema and data types.
  - **LLD: restaurant menu with configurable options** (size, crust, toppings; the Domino's case), as a schema.
  - DSA: suggest a dish price using two heaps.
- **R3:** VP.

**Z7. Zomato SDE. Tier-1 final year, off-campus via cold mail, Mar 2025. Offer.** https://leetcode.com/discuss/post/6493240/
- **R1:** SQL vs NoSQL, thread vs process, IPC, TCP/UDP, "type zomato.com", indexing internals, multithreaded merge sort, JWT.
- **R2, two design questions:**
  - **Design a rate limiter for an API:** fixed and sliding window "implemented using Redis and log storage", plus a leaky vs token bucket trade-off discussion.
  - **Order-processing API:** accepts batches of timestamped orders and answers "number of orders in the last X minutes (X up to 30)". Solved with a ring of 30 one-minute buckets plus timestamps to reset stale buckets, using ReentrantLock for thread safety.

**Z8. Zomato SDE-1 backend. Off-campus, Sep 2023. Selected.** https://leetcode.com/discuss/post/4068294/
- **R1:** How would you scale your internship work; DynamoDB internals.
- **R2:** **"Design an image upload system"** (45 minutes):
  - What protocol.
  - **Write code for the CRUD APIs**: status codes, request/response payloads, headers, authentication.
  - SQL join queries for it, optimizing the queries, indexing (implementation and use).
- "They deep dived into System design in both rounds", on Google Docs.

**Z9. Zomato SDE Intern (6-month). Off-campus, Nov 2024. Offer.** https://leetcode.com/discuss/post/6023375/
- **R1 (SDE-III):**
  - Mongo vs MySQL.
  - **Separate indexes on A and B vs a composite (A,B) for `WHERE A=10 AND B=20`.**
  - **Design an attendance system:** roles teacher/dean/student with different views, courses, a "classes needed to reach 75%" calculation. Wanted DB structure, APIs, logic and a **login design**.
  - Hashing vs encryption.
  - "If someone steals your session id, how do you stop them?" Follow-up: "what if they share your IP?". The candidate ran out of answers.
- **R2:** EM, DSA and a puzzle.

**Z10. Zomato SDE Intern. Dec 2024. Offer. No design question.** https://leetcode.com/discuss/post/6164662/
- Two DSA-heavy rounds (DP, graphs).
- ACID with isolation examples, NoSQL vs RDBMS, deep resume probing.

**Z11. Zomato (District) SDE-1. 1 YOE, Apr 2025. Selected.** https://leetcode.com/discuss/post/6647834/
- **R2:**
  - **Restaurant timing system:** one in-memory structure answering both "all open/close slots for a restaurant" and "all restaurants open now"; data loaded daily.
  - SQL: restaurants with no cuisines, **JOINs only**.
  - BST vs red-black tree; index internals.
  - A deadlock scenario to analyse.
  - **HLD + LLD of a contract management system** (from the candidate's resume): FR/NFR, traffic estimation, microservices, inter-service communication, schema.

**Z12. Zomato SDE-2. Nov 2025. Rejected.** https://leetcode.com/discuss/post/7573378/
- **R1:**
  - Redis: why it is fast; how it scales while single-threaded.
  - Kafka: how to raise consumer throughput; what happens when consumption lags production.
  - **SQL on order state transitions** (Created, then Confirmed, Reached Merchant, Reached Customer, Delivered): latest state per order in the last hour, excluding delivered. How it behaves at scale, which indexes.
  - **LLD: Snake and Food game** with O(1) movement, growth, data-structure trade-offs, pseudocode.

**Z13. District SDE-1. 1.2 YOE, Mar 2026. Offer.** https://leetcode.com/discuss/post/7617100/
- **R1 (DSA + HLD):** **"Design a notification system for something like food delivery"**:
  - Push, SMS and email channels; some notifications go to one channel, some to several.
  - Must avoid duplicates; scalability.
  - "In-depth discussion on Kafka, scaling Kafka and trade-offs."
- **R2:** One-hour resume grill: why X over Y, trade-offs in libraries used. Then implement the algorithm behind a library the candidate had used.

**Z14. Nugget by Zomato SDE-1. 1y3m, Aug 2025. Offer.** https://leetcode.com/discuss/post/8540075/
- **R1:** DSU problem, then a light design question, **peer-to-peer chess matched by rating** (tournaments and leaderboards later). The interviewer jumped straight to data modelling.
- **R2, "Ad Pacing Service":**
  - Example: Domino's pays Rs 10,000 at Rs 10 CPC, which gives 1,000 tokens. Spend must last the whole day, not run out by noon, across peak and off-peak traffic.
  - Solution: hour-weighted token allocation, a form of rate limiting.
  - HLD discussion: Kafka with consumer groups, master-slave read replicas behind a load balancer, read caching, cache sharding, moving from plain hashing to consistent hashing.
- **R3 (Senior EM):** Culture stress test (10-14 hr days).
- **R4:** VP.

**Z15. District by Zomato SDE-1 backend. Fresher with 1 yr internships, Oct 2026. Rejected after R2.**
- URLs: R1 https://leetcode.com/discuss/post/8550229/ ; R2 https://leetcode.com/discuss/post/8559623/
- **R1:** Postgres indexes, EXPLAIN ANALYZE, batching, stateful vs stateless, large REST responses, Kafka vs SQS, partitions, ordering, replay, **consumer idempotency**, Prometheus high-cardinality labels.
- **R2, "API to fetch a user's District Money transaction history, paginated":**
  - Offset vs cursor; cursor = (createdAt, transactionId); composite index; API contract and SQL.
  - New constraint: the data comes from **3 independent tables**. JOIN vs UNION ALL vs query separately and merge in the application, and how pagination works across sources.
  - **The candidate had to write the backend code and SQL.**
  - Trap: "what other pagination besides offset and cursor?" (page-number).
- **Self-reported reason for rejection:** messy hand-written code, too used to AI-assisted coding.

**Z16. Zomato SDE-2 (React Native/full-stack). Nov 2025. Rejected.** https://leetcode.com/discuss/post/7371586/
- Design JWT auth across backend and frontend.
- **Design WhatsApp QR-code login**, then refine and look for issues, then **TOTP**: design the QR that changes every minute.
- Rejected after failing an O(1) min/max stack question in the last 15 minutes.

**Z17. Zomato SDE-2 R1. Jan 2026. Outcome unknown.** https://leetcode.com/discuss/post/7478818/
- Bipartite matching DSA.
- "Design a search algorithm returning products with the highest matching frequency" over name, tags and description. The expected answer was an inverted index / Elasticsearch.

**Z18. Zomato SDE-1. 2025 grad, off-campus, Oct 2024. Rejected after R1.** https://leetcode.com/discuss/post/5947992/
- 90-minute round: DP on strings, then a system design question (not named) that "I struggled with… 40 minutes", then OOP.
- No feedback given.

**Z19. Zomato SDE-1. 9 months experience, Apr 2023. Accepted.** https://leetcode.com/discuss/post/3384363/
- **R1:** LLD of 1M objects with 3-4 properties each: which data structure makes group-retrieval queries fast, and what factors drive the choice. Then flatten a JSON in code.
- **R2:**
  - Rate limiting (token and leaky bucket), consistent hashing vs normal hashing step by step, primary and secondary indexes.
  - Full request lifecycle (DNS, gateway internals, CDN, cache types), reducing latency, vertical vs horizontal scaling, geoDNS.
  - **Design a basic Spotify** (playlist, add song, search): DB choice, normalize vs denormalize, schema.

**Z20. Zomato SDE-1 new grad. Off-campus intern converted to FTE, Jul 2023. Offer.** https://leetcode.com/discuss/post/3741232/
- One 2-hour onsite round covering system design, DBs, CN and OS. "DSA was in shape of real world problems, not leetcode style."

**Z21. Zomato on-campus fresher. Jul 2023. Selected.** https://leetcode.com/discuss/post/3768540/
- Single round: 2 DSA problems, 5 SQL queries, and "System Design, music player related". In practice this was one function to optimize to O(N).
- OOP / polymorphism / virtual functions.

**Z22. Zomato on-campus 2024 batch. Aug 2023. Rejected.** https://leetcode.com/discuss/post/4473000/
- Line-by-line resume grill; multithreading emphasis.
- **"Design a queue which is independent of multithreading"**, i.e. a thread-safe queue.
- ACID and how MySQL provides isolation; REST; a shell script.

**Z23. Zomato SDE-1. Apr 2024. Rejected.** https://leetcode.com/discuss/post/5007288/
- Promised DSA and system design. In fact about 1 hour of MongoDB and then SQL (the interviewer's specialty), with DSA rushed at the end.
- Rejected on the SQL query.

**Z24. Zomato on-campus FTE (tier-1 NIT). Aug 2025. Rejected.** https://leetcode.com/discuss/post/7046636/
- About 30 minutes on backend concepts from the resume.
- B+ tree in depth: complexity, effect of node order, why not a sorted array.
- 3 DSA problems. No design question.

**Z25. Zomato SDE-1 UI. Jun 2025. Selected.** https://leetcode.com/discuss/post/8359475/
- R1: UI machine coding plus JS/React internals.
- R2: **Design Google Calendar** (architecture, data flow, event management, scalability, frontend decisions).

**Z26. Zomato SDE-1 frontend. 2024. Offer.** https://leetcode.com/discuss/post/5021709/
- React internals; implement useState/useEffect. No backend design.

**Z27. Zomato SDE-1. Post Feb 2026, 5 rounds. Offer.** https://igreaper.medium.com/my-zomato-sde-1-interview-experience-5-rounds-self-doubt-and-a-moment-ill-never-forget-eaa8a6fb49e4 (read via freedium)
- **R1:** HLD of a feature the candidate built at a previous employer.
- **R2:** Architecture and scaling of a past product; **design a dashboard system and an LLM-related feature**; ACID, OSI, TLS/mTLS.
- **R3 (1h40m):** ALB vs NLB vs Nginx; reverse proxies; **about 40 minutes on Kafka**; "a well-known Zomato scaling problem"; end-to-end HLD; LLD of two features; producer-consumer; concurrency vs parallelism.
- **R4 (onsite):** **Multi-tenant database architectures**: types, pros/cons, when to use each.
- **R5:** VP.

**Z28. Zomato SDE-1. Post Aug 2025. Outcome not stated.** https://medium.com/@abhishekworks787/zomato-sde-1-interview-experience-effb60e232fa (read via freedium)
- R1: Multithreading coding, pointers.
- R2: **"Design a payments backend"**:
  - Flow: user, then Zomato, then payment gateway, then confirmation.
  - Tables: orders, transactions, users.
  - **What if payment fails midway**, **idempotent payment APIs**, retries, scaling.
  - The approach mattered more than a perfect architecture.

**Z29. Zomato SDE-1. About 5 months experience, post Dec 2024. Selected.** https://www.geeksforgeeks.org/interview-experiences/zomato-interview-experience-for-sde1-1-year-experienced/ [UNCERTAIN: the GFG page title says "1 year experienced"; this content may come from shivambhadani's Medium post, https://medium.com/@shivambhadani_/zomato-interview-experience-for-sde-1-c0eed08e9d57; the fetched text matched the "5 months / New Year" story]
- R1: Connection pooling, SQL GROUP BY/HAVING, graph components.
- **R2 HLD (about 70 minutes):** "After New Year closures, when a restaurant reopens, notify users, **only if delivery partners are available in that area**, for millions of users."
- Then OOP code for a Ctrl+R reverse command search; a Trie optimization was expected.

**Z30. Zomato SDE Intern. Off-campus, post May 2026. Offer.** https://partik-malasi-05.medium.com/my-zomato-sde-intern-interview-experience-off-campus-a798d2c867f0 (via freedium)
- **R1 mini-design:** Route messages across SMS providers that differ in cost and success rate.
- **R2 HLD:** **Distributed key-value store** with get/set/addServer/removeServer and smooth redistribution. Consistent hashing, minimizing data movement, hotspots, hash map vs B+ tree, goroutines, gRPC vs HTTP/1.1 vs HTTP/2, autoscaling, tracing.

**Z31. District (Zomato) SDE-1. Off-campus, Amazon intern, post Jul 2026. Outcome not stated.** https://medium.com/@samarth.bhutani/district-zomato-sde-1-interview-experience-off-campus-56-lpa-ctc-7806143fa708 (via freedium; also on daily.dev)
- **R1 LLD:** **Rate limiter**: fixed-window boundary burst, why naive sliding-window counting is O(N), leaky bucket; complexity, edge cases, scale.
- **R2 HLD (90 min):** **Voice transcription platform**, about 10 MB per file and 100K uploads/day:
  - Pre-signed S3 uploads, a queue plus async workers.
  - Retries, worker failure, queue semantics, schema, API contracts, **job status tracking**.

**Z32. Zomato SDE-1 Android, fresher. Sep 2025. Rejected.** https://roundz.substack.com/p/interview-experience-184-zomato-sde1-fresher
- R2: **LLD of a card deck**: pop, shuffle, distribute, Card class with 4 suits and 2 colors.
- R3: Implement LiveData, LRU cache internals, XML for a Zomato card.

**Z33. Zomato SDE-1. 1 YOE, GFG (updated Jul 2025). Selected.** https://www.geeksforgeeks.org/interview-experiences/zomato-interview-experience-for-sde1-1-year-experienced/ [UNCERTAIN: may be the same page as Z29; the fetch returned a "pre-screening + live coding + HR" structure]
- Live coding round: symmetric tree / n-ary tree, plus **"Database design for a food delivery app"**: Users, Orders, PK/FK.

**Z34. [UNCERTAIN, snippet only] Eternal SDE-I on-campus.** https://medium.com/@sohanshanbhag15/eternal-interview-experience-for-sde-i-on-campus-d9dbb7fa573b (the full text was blocked)
- Per the search snippet: "design the shuffle button of Spotify" and implement randint; then "design a system to predict the time taken for food delivery in Zomato".

**Hyperpure:** Only two LeetCode posts (https://leetcode.com/discuss/post/7369370/, https://leetcode.com/discuss/post/7367617/, Nov 2025). Both ask about timelines; neither gives any question. **No Hyperpure design questions found.** Blinkit-group reports say off-campus Eternal hiring (Zomato, Blinkit, Hyperpure, District, Bistro) is almost entirely by referral (https://leetcode.com/discuss/post/7199947/).

### 1B. Blinkit (Eternal)

**B1. Blinkit SDE-1 backend. 2026 grad, 6-month intern, Sep 2026. Offer.** https://leetcode.com/discuss/post/8514070/
- **R1 (SDE-3):** Stack memory, HashMap internals and collisions, index internals, peak element, then **design an LFU cache** (`get`/`put`).
- **R2 (Team Lead):** **"Design a Notification System"** on draw.io with UML, as discussion rather than code. Covered notification flow, APIs/queries, DB design, indexing, **Kafka vs direct API calls**, scalability, and "why this approach and what alternatives".

**B2. Blinkit SDE-1 backend. 2026 grad, 6-month intern, Sep 2026. Offer (24 base + 5 bonus).** https://leetcode.com/discuss/post/8515131/
- **R1:** Clustered vs non-clustered indexes, when indexes hurt, composite index purpose; rotated array search; **LRU cache**.
- **R2 (Team Lead):** **"Design a Food Delivery Order Tracking System"** on draw.io:
  - Order lifecycle, APIs, schema, entity relationships, status updates.
  - Event-driven design, Kafka vs synchronous calls, high traffic, **idempotency**, failure scenarios, service boundaries.
  - Exact follow-ups: "Why asynchronous? What problem does it solve? What happens if the consumer is down? How would you handle duplicate events?"

**B3. Blinkit SDE-1. 1 YOE, Jun 2026. Offer.** https://leetcode.com/discuss/post/8331559/
- R1: House Robber I/II; SQL vs NoSQL; **"what happens if we use only Redis and no database"**.
- **R2:** DSA, then **BookMyShow**, probed on decisions rather than a full design:
  - UX for a high-traffic popular event.
  - **Two users selecting the same seat at the same time.**
  - Pessimistic vs optimistic locking, Redis.

**B4. Blinkit SDE-1. Dec 2024. Offer.** https://leetcode.com/discuss/post/6207331/
- **R2 (SDE-2, 75 min):** **"Rider onboarding application"**:
  - 15-20 minutes on requirements and scope; SQL vs NoSQL ("grilled thoroughly for every decision").
  - **DynamoDB internals, index types, partition key and sort key per table.**
  - Reducing HTTP server latency, status-code-based retries, long vs short polling.

**B5. Blinkit SDE-1. 1 YOE, Jul 2025. Offer.** https://leetcode.com/discuss/post/6916829/
- **R2 "System Design":** Microservices and event-driven questions, then **build a working API (curl-able) that fetches all transactions for a user_id, with Postgres**. Partially done; then DB queries.

**B6. Blinkit SDE-1. 2.3 YOE, Apr 2025. Offer.** https://leetcode.com/discuss/post/6702653/
- **R2 (SDE-3):** "Collect credentials from multiple financial apps, scrape data, process it, and generate analytics for users and admins". Admin analytics were added mid-way. Elasticsearch/inverted index, DB trade-offs, master-slave, caching.
- **R3 (HM):** Kafka deep dive on Blinkit's async order processing problems.

**B7. Blinkit SDE-1 backend, fresher. May 2024. Rejected in the SD round.** https://leetcode.com/discuss/post/5156809/
- **SD round:** **"Design WhatsApp"** plus follow-ups; nth max salary SQL; mutex vs semaphore. "Wasn't adequately prepared for the system design round."

**B8. Blinkit SDE-1. 1+ YOE, May 2024. Outcome not stated.** https://leetcode.com/discuss/post/5123971/ and https://leetcode.com/discuss/post/5123902/
- **R3:** Project questions, SQL joins/subqueries, **HLD of a grocery delivery application**.
- **R4:** **HLD then LLD of a Circuit Breaker service** between services: detect down/up and store user requests while the backend is down. PySpark; API questions.

**B9. Blinkit LLD round. Dec 2023. Level unknown.** https://leetcode.com/discuss/post/4470525/
- **"Design BookMyShow with how admin will add/delete a show in a hall in a theatre."**

**B10. Blinkit SDE Intern backend. Jan 2024. Selected.** https://leetcode.com/discuss/post/4630366/ and https://www.geeksforgeeks.org/interview-experiences/blinkit-interview-experience-for-sde-internship/
- **R1:**
  - Does your API hit DynamoDB or the cache first; TTL.
  - **"If a Zomato Order table is read for order and payment details, SQL or NoSQL and why?"**, then "why is NoSQL better for read-heavy? can't SQL scale?"
  - Quora schema, then 3 mandatory SQL queries.
- **R2:** LLD of **browser tab back/forward/new-tab history** as a class: public/private methods, test cases.

**B11. Blinkit SDE. Dec 2025. Outcome unknown.** https://leetcode.com/discuss/post/7416559/
- R1 was "DSA with system-design flavour": **"Design an Oncall System"** in sub-parts building on each other. Explain the model and data structures, then write complete code.

**B12. Blinkit SDE-2. Aug 2025. Rejected.** https://leetcode.com/discuss/post/7083108/
- R1 trap: **"Assume infinite cost. Why can't we just replace a database with a cache?"** The candidate could not answer.
- **R2:** **LLD of a movie ticketing system (BookMyShow)**. Not finished because too long was spent on the project discussion.

**B13. Blinkit SDE-2. Sep 2024. Offer.** https://leetcode.com/discuss/post/5869260/
- **R2 SD (60 min):** **BookMyShow**: HLD, API design, DB. Edge cases: "spurious load, prevention of double booking, quick search".

**B14. Blinkit SDE-2. Sep 2024. Offer.** https://leetcode.com/discuss/post/5895086/
- **R2:** "Build a vehicle rental service / ZoomCar". The interviewer said to focus on **low-level design**.

**B15. Blinkit SDE-2. About 2 YOE, Jun 2026. Rejected.** https://leetcode.com/discuss/post/8359730/
- No design question. A resume deep dive on DB/framework choices, backward compatibility and own vs team contribution, then Insert-Delete-GetRandom.
- Candidate's takeaway: "don't put a system on your resume for SDE-2 unless you can sketch its failure modes."

**B16. Zomato (Blinkit) SDE Intern. Dec 2025. Selected.** https://medium.com/@Ayush-Tibrewal/zomato-blinkit-interview-experience-sde-intern-selected-1b24989da5dd (via freedium)
- **R1:** **HLD/LLD of Spotify**, then runnable code for shuffle without repeats; Kafka architecture, consumer groups, partitions, replication.
- **R2:** **LLD of a feature flag service like LaunchDarkly** (the hardest part); scaling a short-URL service, partitioning/sharding, DynamoDB vs Postgres, thread vs process.

**B17. Blinkit SDE-1 fresher (Data Platform). Post Jun 2025. Selected.** https://medium.com/@utkarsho.dev/blinkit-sde-1-interview-experience-fresher-39ba436055f3 (via freedium)
- **R1:** HLD grill on the candidate's own project: RabbitMQ vs Kafka, why a cache, why Postgres, how it changes with more users; many-to-many in SQL.
- **R2:** **Coupon creation and validation system**: admin-defined rules, expiry, limited-user coupons, a **5-minute payment window** after applying. APIs, DB design, end-to-end logic.
- **R3 (HM):** Why Kafka for event streaming instead of a DB cluster.

### 1C. Swiggy (incl. Instamart)

**S1. Swiggy SDE-1. Oct 2025. Rejected.** https://leetcode.com/discuss/post/7278246/
- **LLD round (60 min):** **"Design and implement a Splitwise"**. About 10 minutes on requirements and objects, a UML diagram, then code in VS Code.
- "Interviewer heavily focused on coding… expected a fully working application." The candidate had partial functionality and was rejected.

**S2. Swiggy SDE-1. Mar 2026. Offer.** https://leetcode.com/discuss/post/7642548/
- R2 (third-party): DSA plus 20-25 Java questions (GC, collections, multithreading).
- **R3 LLD (1 hr):** 10-15 minutes of resume, then **"assign drivers to orders"** with VIP/non-VIP orders, driver ratings and matching conditions. Design the approach, choose data structures, **write code, dry run**. Data-structure choices were cross-questioned.

**S3. Swiggy SDE-1 backend. Dec 2025. Rejected.** https://leetcode.com/discuss/post/7389374/
- **LLD:** **"Design a Notification System"**, then write code. The candidate blanked.

**S4. Swiggy Instamart SDE-1. 1.5 YOE, Jan 2025. Outcome unknown.** https://leetcode.com/discuss/post/6260445/ (journey in https://leetcode.com/discuss/post/6234046/)
- **R3 (LLD & HLD):** **"Design your own Cron system"** (commands are unix commands) and **LLD of Snake and Ladder**.

**S5. Swiggy SDE-1 backend (Instamart, Go). 1+ YOE, May 2024. Rejected after the EM round.** https://medium.com/@mpooja9440/swiggy-sde-1-backend-interview-experience-1-years-of-experience-8b181e8962d2 (via freedium)
- LLD/machine coding (75 min) with an SDE-2: class fields, schema, Java code. Solved 3 of 5 parts. The problem was not named.

**S6. Swiggy SDE-2 BLR. 2025. Offer.** https://leetcode.com/discuss/post/7069404/
- **R2 LLD:** **Swiggy cart**: add/remove products, compute total, validate against inventory, discounts, payment, notifications.
- **R4 bar raiser:** **rate limiter (sliding window and leaky bucket), coded**, plus **vending machine** LLD.

**S7. Swiggy SDE-2 backend. 2+ YOE, Nov 2025. Rejected.** https://leetcode.com/discuss/post/7383296/
- R1: **Parking lot** (Vehicle, Slot, Ticket, Payment, allocation strategy, class diagram).
- **R2 LLD + HLD:** **Train ticket booking**: seat allocation, **concurrency control**, waitlist, basic HLD.
- Candidate's read: "They expect you to basically mug up all LLD/HLD patterns and solve instantly."

**S8. Swiggy SDE-2. Jan 2026. Rejected.** https://leetcode.com/discuss/post/7480610/
- **LLD:** **Multiple parking slots reservable by multiple companies**, and how to handle bookings.

**S9. Swiggy SDE-2. Nov 2024. Ghosted after HM.** https://leetcode.com/discuss/post/6051809/
- **LLD:** **Coupon-based discounting system**: class diagrams, APIs, DB schema, optimisations (Redis caching, async UI loading).

**S10. Swiggy SDE-2. 5+ YOE, late 2024. Outcome unknown.** https://leetcode.com/discuss/post/6421018/
- **SD:** **Warehouse management for Instamart**: APIs, schemas, HLD, "managing inventory and being consistent across different systems". The HM round asked about design patterns.

**S11. Swiggy SDE-2. Jan 2024. Rejected (DSA).** https://leetcode.com/discuss/post/4540407/
- **Machine coding:** **Splitwise** (workat.tech-style requirements), about 3 hours including explanation.

**S12. Swiggy SDE-1 frontend. Dec 2024. Rejected.** https://leetcode.com/discuss/post/6334442/
- OA machine coding: file explorer.
- HM: a generic "build an app from scratch" discussion (planning, components, data flow, patterns, performance, **testing, monitoring**), and requirements for an auto-suggest input.
- The candidate believes the gaps in testing and monitoring caused the rejection.

**S13. Swiggy SDE-1 Android. May 2025. Passed rounds.** https://leetcode.com/discuss/post/6783426/
- **Machine coding:** **Library management app in 45 minutes.**

**S14. Swiggy SDE Intern. 2025. Selected.** https://leetcode.com/discuss/post/6837213/
- No design question: DSA, Mongo vs SQL, ACID.

### 1D. Zepto (mostly BarRaiser / InterviewVector-run rounds, Aug–Oct 2024)

**P1. Zepto SDE-2 backend. 3.5 YOE, Sep 2024. Selected.** https://leetcode.com/discuss/post/5796138/
- **LLD:** **Orders and deliveries in a quick-commerce platform**: inventory, **order prioritization at peak**, payment methods (card, wallet, COD).
- Wanted class diagrams, API design, DB design, design patterns, and **SQL or NoSQL per component, with reasons**.

**P2. Zepto SDE-2. Aug–Sep 2024. Offer.** https://leetcode.com/discuss/post/5796357/
- **LLD:** Same quick-commerce question:
  - DB schema first.
  - Write the APIs for add-to-cart and place-order, **with edge cases and error handling**.
  - Authentication for these APIs (JWT, how it works).
  - Patterns: the interviewer dug into **Factory for payment processors**, and the candidate implemented it in Java.
- **HM:** **Google Calendar**, DB design only. The interviewer pushed for more entities to decouple.

**P3. Zepto SDE-2. 2+ YOE, Aug 2024. Rejected after LLD.** https://leetcode.com/discuss/post/5706420/
- Same quick-commerce LLD. Mentioned patterns but **"wasn't able to fully design the classes"**.

**P4. Zepto SDE-2. 2.2 YOE, Sep 2024. HM cancelled.** https://leetcode.com/discuss/post/5796308/
- **LLD:** **Insights tool like SurveyMonkey**. Expectations: entities, DB schema, API design.

**P5. Zepto SDE-2. Aug 2024.** https://leetcode.com/discuss/post/5703566/
- **LLD:** **DB schema for Uber**. The interviewer moved to HLD (components, caching, flow), then **concurrency at the driver's end**.
- Tip from the post: "focus on database schema modelling"; interviewers had 1-4 YOE.

**P6. Zepto SDE-2. Sep 2024.** https://leetcode.com/discuss/post/5724806/
- **LLD:** **IRCTC**: search trains by number and by source/destination/date, **concurrent booking handled fairly**, cancellation.
- More interested in DB design than implementation. The candidate introduced a "segment" entity so seats can be reused across route segments.

**P7. Zepto SDE-2. 3 YOE, Sep 2024. Positive.** https://leetcode.com/discuss/post/5821188/
- **LLD:** **Google Drive document sharing and permissions**, ending in some HLD.

**P8. Zepto SDE-1. 1y4m, Sep 2024. Positive (position then filled).** https://leetcode.com/discuss/post/5804284/ (mirrored at https://devbrainiac.com/blogs/122/zepto-sde-1-interview-experience-backend-developer-real-dsa-lld-chess-game-design/)
- **LLD + DB schema:** **Chess game**: classes, inheritance, patterns; tables for players and moves; APIs for start game, make move and get state, with payloads.

**P9. Zepto SDE-1. 14 months, Sep 2024. Selected, then put on hold.** https://leetcode.com/discuss/post/5776849/
- **LLD:** **E-commerce application**: schema, endpoints, patterns, OOP.
- HM: indexing in MongoDB vs MySQL, drawbacks of indexing, how you scaled your project.

**P10. Zepto SDE-1. Sep 2024. Rejected without feedback.** https://leetcode.com/discuss/post/5760479/
- **LLD:** **Online bookstore with borrow/return**: schema, APIs, patterns (singleton, abstract factory, strategy), implemented in Python.

**P11. Zepto SDE-1. 2.3 YOE, Sep 2024. Cleared all rounds.** https://leetcode.com/discuss/post/5836483/
- **LLD:** **DB schema, classes and entities for a quick-commerce app** from 3-4 functional requirements, then scaling follow-ups.
- **HM:** **HLD of the hardest project at your current company.**

**P12. Zepto SDE-1. 1.5 YOE, Jun 2026. Rejected at HM.** https://leetcode.com/discuss/post/8402132/
- **LLD:** **Vehicle rental**: multiple shops, bikes/scooters, inventory, reservations, checkout/return, rental times. Vehicle base class, status enums, **Strategy for pricing**. **No code required.**
- **HM:** Draw a past project's architecture on draw.io; on-call, alerts, incidents.

**P13–P15. Zepto SDE-3 contrast.**
- P13 (https://leetcode.com/discuss/post/5612526/): LLD Cricbuzz, LLD Google Calendar (schema + APIs), HLD ticketing/support system.
- P14 (https://leetcode.com/discuss/post/5806260/): LLD hotel booking (patterns, concurrency, locks), HLD URL shortener, HLD calendar.
- P15 (https://leetcode.com/discuss/post/7353272/): LLD meeting scheduler with recurring meetings; a two-phase-commit follow-up was missed.

### 1E. Flipkart

**F1. Flipkart SDE-1, 2023–24 grads. No design round.** https://leetcode.com/discuss/post/4919649/, https://leetcode.com/discuss/post/4999276/, https://leetcode.com/discuss/post/7691468/
- Process: OA (3 problems), 2 PSDS rounds, HM.
- Several 2025 SDE-1 posts ask about an **SDE-1 machine coding round (14 Aug 2025 drive)**: https://leetcode.com/discuss/post/7079616/, https://leetcode.com/discuss/post/7092450/. [UNCERTAIN: no question was shared]

**F2. Flipkart SDE-2. 3 YOE, 2024. Offer.** https://leetcode.com/discuss/post/5303379/
- **MC (1.5 hr, zip upload):** **Affiliate shopping system**: influencers earn a percentage or flat share, paid only after the return window.
- **Design round:** **Parking lot**: entities, interfaces, fee calculation, spot assignment, **extensibility**.

**F3. Flipkart SDE-2. 2.5 YOE, Dec 2024. Rejected (design round).** https://leetcode.com/discuss/post/6168500/
- **MC:** **Appointment booking** (https://leetcode.com/discuss/post/5377854/). The evaluation asked how to **avoid concurrency issues when booking**.
- **Design round (SDE-3 plus shadow):** **Movie ticket booking**: functionalities, actors, models and relationships, APIs, DB schema, then HLD questions.

**F4. Flipkart SDE-2. Apr 2025. Rejected.** https://leetcode.com/discuss/post/6667785/
- **MC:** **Food ordering system** (https://leetcode.com/discuss/post/6614846/).
- Tip: "The evaluator seemed to care more about whether the features were working correctly than about design, clean code, or patterns."

**F5. Flipkart SDE-2. 2025.** https://leetcode.com/discuss/post/6874431/
- **MC "FoodKart":** In-memory; add restaurants; update menu; order using a pluggable **restaurant selection strategy**; accept only if items and capacity allow; dispatch frees capacity; process commands sorted by timestamp.

**F6. Flipkart SDE-2. 2025.** https://leetcode.com/discuss/post/6933851/
- **MC:** **E-commerce billing**: bill, discount, points/levels. The evaluation covered separation of concerns and "what if a DB is added".
- **Design:** **Tournament racing game**: zero-lag leaderboard updates and push notifications during the race ("Totally flopped").

**F7. Flipkart SDE-2. Feb 2026. Positive through the design round.** https://leetcode.com/discuss/post/7591327/
- **MC:** **Distributed task scheduler with dynamic load management**. Evaluation: use a priority queue instead of sorting; "two tasks arrive at the same time?"
- **Design round (Google Doc):** **Airline ticket management**: entities, schema, APIs, then service implementations, all written into the doc.

**F8. Flipkart SDE-2. 4.3 YOE, Sep 2026. Offer.** https://leetcode.com/discuss/post/8567797/
- **MC:** **Buy-Now-Pay-Later**: inventory, credit limits, PREPAID vs BNPL, partial dues clearance, blacklisting after 3 defaults.
- **Evaluation:** **Two threads buying the last unit**, consistent credit limits under concurrent BNPL orders, locking strategy.
- **SD:** **Flash sale running about 15 minutes under massive traffic, no overselling or underselling.**

**F9. Flipkart SDE-2. May 2024.** https://leetcode.com/discuss/post/5221179/
- **MC:** **"FlipkartOverFlow"** (a Q&A platform).
- Format: 30 min explanation, 90 min code, 15 min upload, 30 min evaluation; proctored; internet for syntax only.

### 1F. PhonePe (machine coding on CodeSignal, 1.5–3 hours, then a code-discussion round)

- **Ph1. SDE-1 backend, 2025.** https://leetcode.com/discuss/post/6437867/
  - DSA plus POST vs PUT, ACID, monolith vs microservices, Kafka. No design problem.
- **Ph2. 1.7 YOE, Dec 2023.** https://leetcode.com/discuss/post/4472947/
  - **E-commerce MC** (auth, catalog, checkout, order management) in 2 hours.
  - The interviewer cared more about challenges around the scenario than about completion.
- **Ph3. 2024.** https://leetcode.com/discuss/post/4992773/
  - **PhonePe mini wallet MC**: register, top-up, balance, send money, pluggable offers (e.g. cashback when balances become equal).
  - "Clear separation between data and service layers."
- **Ph4. 2025.** https://leetcode.com/discuss/post/6437969/
  - **Hackathon platform MC**: problems with difficulty, tags and score; filter/sort; a pluggable scoring strategy; an extensible recommendation strategy.
- **Ph5. 2025.** https://leetcode.com/discuss/post/6661034/ and https://leetcode.com/discuss/post/7211822/
  - **App version management MC**: install, update, rollout to devices.
- **Ph6. SDE, 0–3 YOE, 2025.** https://leetcode.com/discuss/post/7218643/ and SDE-2 https://leetcode.com/discuss/post/6724101/
  - **Customer issue resolution system MC**: agents with expertise, an assignment strategy, a waitlist when all agents are busy, search, history.
  - **The 30-minute next-day discussion focused on concurrency and thread safety.**
- **Ph7. 2 YOE, 2025.** https://leetcode.com/discuss/post/6759550/
  - **Fitness class booking MC**: user tiers with booking limits, capacity, **waitlist**, cancellation.
- **Ph8. SDE-2, 2025.** https://leetcode.com/discuss/post/6888456/
  - **Ad engine MC**: campaigns, targeting, bids, budgets, constraints.
- **Ph9. SDE-2, 4 YOE, Nov 2024. Rejected (DSA).** https://leetcode.com/discuss/post/6093920/
  - **Flight management MC** in 3 hours. "Focus on readability, concurrency."
- **Ph10. SDE-2, 3–5 YOE, 2024.** https://leetcode.com/discuss/post/5241117/
  - **HLD:** Add a **subscription feature to an existing e-commerce site**: which services to reuse or add, DB schema, API contracts, failure cases.

### 1G. Razorpay, Meesho, Myntra

- **Razorpay MC, 2026.** https://leetcode.com/discuss/post/7600874/
  - **In-memory pub-sub queue with guaranteed, in-order delivery per consumer.** Bonus: a topic service and spilling to disk. Runnable code required.
- **Razorpay SDE-2 MC, Mar 2024. Rejected.** https://leetcode.com/discuss/post/4824582/
  - **Logger system** using chain of responsibility, facade and factory.
- **Meesho SDE-1 backend, Oct 2024.** https://leetcode.com/discuss/post/5925685/
  - **Order management MC (90 min, HackerRank)**: addProduct/createOrder/confirmOrder/getStock.
  - The trap: **block inventory at createOrder and decrement at confirmOrder.**
- **Meesho SDE-1, Apr 2025.** https://leetcode.com/discuss/post/6701623/
  - **Stock exchange / order book MC**: place, modify, cancel; match when sell price is at most the buy price, oldest first. "Maintain concurrency & thread safety, deal with exceptions." Bonus: order expiry.
- **Meesho SDE-1, Apr 2025.** https://leetcode.com/discuss/post/6702200/
  - **Clinic/doctor slot booking MC** (30-minute slots, specializations).
  - Follow-up: **two patients booking the same slot**, "like BookMyShow with 1000s of users".
- **Meesho SDE-1, Jun 2025. Rejected.** https://leetcode.com/discuss/post/6834461/
  - MC like the clinic booking above, on HackerRank, no own IDE.
  - **The HM asked "Design notification service"**, which went badly.
- **Myntra SDE-1 backend, 2024–25.** https://leetcode.com/discuss/post/6336413/ and https://leetcode.com/discuss/post/6422177/
  - **No design round.** DSA, then HM questions on Kafka architecture and ACID.
- **Myntra intern (Ramp Up), 2025–26.** https://leetcode.com/discuss/post/8242610/
  - DSA only.

---

## Part 2. (a) Frequency table of design questions

Counted from the reports above. A report counts once per theme. "Group" means the Zomato/Eternal group (Zomato, District, Nugget, Blinkit).

| # | Design theme (as asked) | Total | Group | Others | Report IDs | Typical level |
|---|---|---|---|---|---|---|
| 1 | **Your own past project / internship as an HLD** ("redesign it", "how would you scale it", "draw its architecture") | 13+ | 10 | 3 | Z1, Z4?, Z5, Z6, Z8, Z11, Z13, Z27, B6(HM), B17, P11, P12, F7(HM) | Every level; almost universal at Eternal |
| 2 | **Order / food-ordering / q-commerce order flow / order tracking** (incl. schema-only food delivery) | 12 | 6 | 6 | Z2(schema), Z4(enterprise), Z5, Z33, B2, B8(grocery HLD), P1, P2, P3, P11, F4, F5 | SDE-1 and SDE-2 |
| 3 | **Ticket / seat booking** (BookMyShow, movie, train/IRCTC, airline, flight, appointment/clinic) | 12 | 4 | 8 | B3, B9, B12, B13, S7, P6, F3(MC appt + movie), F7, Ph9, Meesho-clinic, Ph7(fitness) | SDE-1 (Blinkit) to SDE-2 |
| 4 | **Notification system** (dedupe, multi-channel, cooldowns, retries) | 8 | 6 | 2 | Z2, Z3, Z13, Z29, B1, S3, Meesho-HM, (Z14 not) | **SDE-1 at Eternal, the most repeated single HLD** |
| 5 | **Rate limiter** (design, then code token bucket / fixed / sliding window) | 7 | 6 | 1 | Z2, Z5, Z7, Z19, Z31, Z4(DDoS), S6 | Fresher to SDE-2 |
| 6 | **Payments / wallet / BNPL / billing / refunds** | 6 | 2 | 4 | Z4, Z28, Ph3, F8, F6, Ph10(subscription) | SDE-1 (Zomato) to SDE-2 |
| 7 | **Cache class** (LRU/LFU implementation or internals) + Redis-only trap | 6 | 6 | 0 | Z1(LFU), B1(LFU), B2(LRU), Z32(LRU), B3(Redis-only), B12(cache-replaces-DB) | SDE-1 |
| 8 | **Cart / e-commerce / inventory / order management / warehouse** | 6 | 0 | 6 | S6, S10, P9, Ph2, Meesho-OMS, F6 | SDE-1 and SDE-2 |
| 9 | **Calendar / meeting scheduler** | 5 | 1 | 4 | Z25(frontend), P2(HM), P13, P14, P15 | Mostly SDE-2/3 |
| 10 | **Spotify / music player (incl. shuffle)** | 4 | 4 | 0 | Z19, Z21, B16, Z34[UNCERTAIN] | Fresher / intern |
| 11 | **File / image / voice upload pipeline** (pre-signed URLs, async workers) | 3 | 3 | 0 | Z8, Z5, Z31 | SDE-1 |
| 12 | **Chat / WhatsApp / QR login** | 3 | 3 | 0 | Z5(support chat), B7(WhatsApp), Z16(WhatsApp QR + TOTP) | Fresher to SDE-2 |
| 13 | **Coupons / discounts** | 3 | 1 | 2 | B17, S9, S6(discount part) | SDE-1/2 |
| 14 | **Parking lot / slot reservation** | 3 | 0 | 3 | F2, S7, S8 | SDE-2 |
| 15 | **Consistent hashing / distributed KV / cache sharding** | 3 | 3 | 0 | Z30, Z14, Z19 | Intern to SDE-1 |
| 16 | **Splitwise / expense sharing** | 2 | 0 | 2 | S1, S11 | Swiggy SDE-1/2 |
| 17 | **Vehicle rental (ZoomCar)** | 2 | 1 | 1 | B14, P12 | SDE-1/2 |
| 18 | **Chess** | 2 | 1 | 1 | Z14, P8 | SDE-1 |
| 19 | **Driver/rider: assignment, onboarding, Uber schema** | 3 | 1 | 2 | B4, S2, P5 | SDE-1/2 |
| 20 | **Search / inverted index / Elasticsearch** | 2 | 2 | 0 | Z17, B6 | SDE-1/2 |
| 21 | **Ads: pacing / ad engine** | 2 | 1 | 1 | Z14, Ph8 | SDE-1/2 |
| 22 | **Games: snake, snake & ladder, card deck, racing leaderboard** | 4 | 2 | 2 | Z12, Z32, S4, F6 | SDE-1/2 |
| 23 | **Schema-only modelling / SQL design tasks** (banner frequency cap, attendance, menu configs, order state SQL, Quora, restaurant hours, delivery-partner top-5) | 9 | 9 | 0 | Z2, Z6(x2), Z9, Z11, Z12, B10, Z33, Z15 | **Very common at Zomato for SDE-1** |
| 24 | **Paginated API (cursor vs offset, multi-table)** / write a working API | 2 | 2 | 0 | Z15, B5 | SDE-1 |
| 25 | Other single occurrences | — | — | — | Circuit breaker (B8), feature flags (B16), cron (S4), on-call system (B11), pub-sub queue (Razorpay), logger (Razorpay), stock exchange (Meesho), hackathon (Ph4), app versioning (Ph5), issue resolution (Ph6), flash sale (F8), flight/airline (Ph9/F7), Google Drive permissions (P7), SurveyMonkey (P4), multi-tenant DB (Z27), dashboard + LLM feature (Z27), SMS routing (Z30), voice transcription (Z31), delivery-time prediction (Z34, uncertain), thread-safe queue (Z22), browser history (B10), library app (S13), affiliate system (F2), task scheduler (F7), 1M-object data structure (Z19), order count in last X minutes (Z7) | — |

**What the Zomato group asks most, in order:**
1. Your own project as an HLD.
2. Schema / SQL modelling of a Zomato-like entity.
3. Notification system.
4. Rate limiter, with code.
5. Order flow / order tracking.
6. Cache classes and Redis internals.
7. BookMyShow (Blinkit).
8. Payments.

Kafka comes up in about 12 group reports, nearly always as a follow-up on queues, consumer lag, idempotency or Kafka vs SQS/RabbitMQ.

---

## Part 3. (b) Common follow-ups and traps (verbatim or near-verbatim)

**Databases and SQL** (the most common source of rejections at Zomato)
- "Why SQL vs NoSQL for this?" Asked in almost every report.
  - Trap: "can't SQL scale?" (B10).
  - Trap: **"Why MySQL vs PostgreSQL?"** The Z2 rejection; the advice was to say honestly that you lack enough exposure rather than bluff.
- **Separate indexes on A and B vs composite (A,B)** for `A=10 AND B=20` (Z9).
- Composite-index left-prefix rule (Z5); clustered vs non-clustered; "when does an index hurt?"; "how does the DB decide to use an index?" (B2).
- Write the SQL live:
  - top-5 delivery partners per area by order type (Z2);
  - latest state per order in the last hour excluding delivered, plus which indexes (Z12);
  - restaurants with no cuisine using JOINs only (Z11);
  - nth max salary (B7).
- Connection pooling; "how can a DB make an app slow?"; EXPLAIN ANALYZE (Z1, Z15).
- DynamoDB partition/sort key design and index types (B4); MongoDB compound indexes and writing a Mongo query (Z5).
- Geo queries: "nearby restaurants in SQL". Bounding box is not enough; they want geo-indexing / PostGIS / geohash (Z5).
- Many-to-many modelling: manager to restaurants (Z5); restaurant to cuisine (Z11).

**Caching / Redis**
- "Assume infinite cost: why not replace the DB with a cache?" (B12, the candidate failed). "What if we use only Redis and no DB?" (B3). Expected: durability, persistence, querying, consistency.
- Redis single-threaded yet fast; hot keys / celebrity problem; thundering herd (Z1, Z12).
- Cache-first vs DB-first reads and TTL (B10); cache sharding with consistent hashing (Z14).

**Messaging / async**
- "Why asynchronous? What happens if the consumer is down? How do you handle duplicate events?" (B2).
- Kafka vs SNS+SQS / SQS / RabbitMQ: ordering, replayability (Z4, Z15, B17).
- Consumer lag and raising throughput with partitions and consumer groups (Z12); consumer idempotency (Z15).

**Notification-specific** (prepare this one fully)
- Dedupe (idempotency keys), multi-channel fan-out, user-level and **team-level cooldowns**, how other teams integrate (API contract), crash resilience, retry guarantees (Z2, Z3, Z13).
- Business-rule twist: **notify only if delivery partners are available in the area** (Z29).

**Concurrency** (the standard trap in every LLD or machine coding evaluation)
- Two users booking the same seat or slot: pessimistic vs optimistic locking, Redis locks (B3, B13, F3, Meesho-clinic, S7, P6).
- Two threads buying the last unit; keeping credit limits consistent (F8). "What if two tasks arrive at the same time?" (F7).
- Concurrency at the driver's end (P5); block inventory at createOrder (Meesho).
- Thread-safe structures: ReentrantLock buckets (Z7); a queue independent of multithreading (Z22).

**Failures / non-happy paths** (the Z4 rejection reason)
- Payment fails midway; idempotent payment APIs; retries (Z28).
- Chargebacks, refunds and disputes; cross-service consistency in microservice orchestration (Z4).
- Worker crash, retry logic, job status tracking (Z31). Circuit breakers (B8). DDoS (Z4). How you handle an external outage such as Cloudflare (Z5).

**APIs**
- Write every API with request/response payloads, status codes, headers and auth (Z5, Z8, P2, P8).
- Pagination: offset vs cursor, cursor design, pagination across multiple sources. Trap: "what other way?" (page-number) (Z15).
- Auth: JWT internals (Z7, P2, Z16). Session hijack: "what if the attacker has the same IP?" (Z9).
- Large uploads: **pre-signed URLs** (Z5, Z31); base64 is the wrong answer.

**Process traps**
- Too much time on the project/intro leaves the design unfinished, and that leads to rejection (B12, P14).
- In LLD rounds at Swiggy, Flipkart, Meesho and PhonePe, running code beats pretty UML (S1, F4, F7).
- Slow to understand a follow-up (2-3 iterations) was itself held against a candidate (Z2).
- Interviewers add requirements mid-design (B6, Z15, Z5). They test adaptability, not the first answer.
- Messy hand-written code after a good discussion leads to rejection (Z15).

---

## Part 4. (c) What freshers get vs SDE-2

**Freshers, interns and new grads (0 YOE or internship only) at Zomato/Eternal**
- Design **is** asked of freshers at Zomato. About 15 of 18 fresher/intern reports in the group had some design element. It is usually one of:
  1. **Schema + APIs for a Zomato-like feature**: online ordering (Z5), attendance (Z9), image upload with CRUD code (Z8), banner frequency cap (Z6), food-delivery DB (Z33), Quora schema (B10).
  2. **A classic small HLD**: rate limiter (Z5, Z7, Z31), notification system (Z29, B1), order tracking (B2), Spotify (B16, Z19), coupons (B17), voice transcription (Z31), KV store (Z30).
  3. **LLD as a class with data structures and code**: LRU/LFU (B1, B2), browser history (B10), card deck (Z32), Ctrl+R search (Z29), feature flags (B16), order counts in the last X minutes (Z7).
- **On-campus** rounds are much more DSA, CS-fundamentals and SQL heavy, with little or no HLD (Z21, Z22, Z24). Off-campus referral rounds look more like a backend-engineer interview.
- Depth expected of freshers: correct entities and relationships, sensible APIs with payloads, a justified SQL/NoSQL choice, indexes, a basic cache and queue, and graceful handling of added constraints. Interviewers accept "I don't know X, here's how I'd reason" (Z5, Z9); bluffing hurts (Z2).
- The **resume deep dive is the biggest single component.** Many freshers had their own project turned into the HLD round (Z5, B17, Z13). Every technology on the resume (Kafka, Redis, Mongo, RabbitMQ) gets probed down to internals.
- The final round is almost always VP/EM culture fit: pace, 10-14 hr days, 5-day office (Z14). Offers are often made in that round.

**SDE-1 with 1–2 YOE**
- Same question types, but with expectations of production depth: failure modes, retries, idempotency, monitoring.
- Z4 was rejected at "~80%" for lacking non-happy-path depth on a payments HLD; the interviewer said the bar was closer to an experienced hire. The production-debugging question (Z3) shows they like real-incident reasoning.

**SDE-2 (2–5 YOE)**
- **Zomato SDE-2:** Redis/Kafka internals (consumer lag), SQL at scale with index choice, LLD games (snake), auth design (WhatsApp QR + TOTP), search (inverted index). DSA is still a hard filter: Z16 was rejected on DSA despite design (Z12, Z16, Z17).
- **Blinkit SDE-2:** BookMyShow HLD with double-booking (B13), ZoomCar LLD (B14), BookMyShow LLD (B12). Resume ownership is scrutinised (B15).
- **Swiggy SDE-2:** LLD with working code. Cart, parking lot, train booking with concurrency, coupons, warehouse; a bar raiser who adds a rate limiter and a vending machine (S6–S11).
- **Zepto SDE-2:** LLD via BarRaiser with schema + APIs + patterns + "SQL or NoSQL per component". Quick-commerce orders/delivery is the signature question (P1–P3); IRCTC and Uber schema also appear.
- **Flipkart SDE-2:** Machine coding (90 min plus evaluation, concurrency questions), then a separate design round (parking lot, movie/airline booking, flash sale, racing leaderboard).
- **SDE-3** (Zepto): 2 LLD + 1–2 HLD rounds; DSA sometimes dropped (P13–P15).

---

## Part 5. (d) LLD / machine coding: question bank with expectations

### Formats by company

| Company | Format | What is judged (per reports) |
|---|---|---|
| **Swiggy** (SDE-1/ASDE and SDE-2) | 60-min LLD round (design + implement live in your IDE), sometimes through InterviewVector; older SDE-2 drives had a 2–3 hr machine coding round | Working code within the time (S1 was rejected for partial Splitwise in 45 min); data-structure choices cross-questioned (S2); dry run; concurrency for booking problems (S7) |
| **Flipkart** (SDE-2; some SDE-1 drives in 2025) | Group Google Meet with 50-60 candidates: 30-min problem brief, 90-min code (any language, in-memory, no DB/UI), zip upload, then a separate 30–45 min evaluation | **"Prioritize correct running code over neat OOP/SOLID"** (F4, F7); bonus features count; evaluation asks about concurrency, extensibility, "what if a DB is added" |
| **Zepto** (SDE-1/SDE-2) | 60-min LLD through BarRaiser/InterviewVector; mostly discussion, some code | Entities/classes, **DB schema**, **API design with payloads**, design patterns (Factory/Strategy/Observer), SQL vs NoSQL per component. Code is often not required (P12), but sometimes asked for one part (Factory for payments, P2) |
| **PhonePe** | CodeSignal, 1.5–3 hr solo (camera off), or a 24-hr take-home; then a 30-min code discussion | Modularity, data/service layer separation, extensible strategies; **concurrency and thread safety in the discussion** (Ph6, Ph9) |
| **Meesho** (SDE-1) | 90-min live MC on HackerRank (no own IDE), interviewers present | Core features working, then concurrency/thread-safety follow-ups (6701623, 6702200) |
| **Razorpay** | MC, runnable code | Patterns (CoR/Factory), in-memory queue semantics |
| **Zomato / Blinkit** | No formal machine coding round for backend SDE-1. LLD shows up inside mixed rounds as: implement a class (LRU/LFU, rate limiter, browser history, Ctrl+R, snake), model a schema (menu configs, banner caps), or write a working API (B5, Z8, Z15). Blinkit SDE-1 "LLD/System Design" rounds in 2026 were draw.io discussions with UML, not code (B1, B2) | Data-structure choice and complexity, clean code without AI, schema correctness |

### LLD question bank seen (2023–2026)

| Problem | Where | Typical expectations / twists |
|---|---|---|
| **Rate limiter class** | Zomato x4, District, Swiggy bar raiser | Implement token bucket + fixed window + sliding window (log and counter); explain the fixed-window boundary burst; leaky vs token trade-off; Redis-backed version; thread safety |
| **LRU / LFU cache class** | Blinkit SDE-1 x2, Zomato SDE-1 | O(1) get/put; HashMap + DLL; LFU frequency buckets; edge cases; Redis eviction policies as a follow-up |
| **Notification service (LLD + code)** | Swiggy SDE-1, Meesho HM, Blinkit | Channel strategy/observer, templates, retry, dedupe, priority; code the core classes |
| **Splitwise** | Swiggy SDE-1 and SDE-2 | Users, groups, equal/exact/percent splits, balance simplification; must run end-to-end in 45–60 min |
| **Parking lot / multi-company slot reservation** | Flipkart SDE-2, Swiggy SDE-2 x2 | Vehicle/Slot/Ticket/Payment; allocation strategy; fee strategy; extensibility; concurrent booking |
| **BookMyShow / movie / train (IRCTC) / airline / flight booking** | Blinkit x4, Flipkart x2, Swiggy, Zepto, PhonePe | Search; seat lock with a hold timeout; **concurrent seat booking** (optimistic vs pessimistic, DB unique constraint, Redis lock); waitlist; segments for trains; admin show management |
| **Food ordering (FoodKart) / restaurant selection** | Flipkart SDE-2 x2 | In-memory restaurants, menus, capacity; pluggable selection strategy (lowest price / highest rating); dispatch frees capacity; timestamp-ordered command processing |
| **Quick-commerce orders + inventory + delivery + payments** | Zepto SDE-2 x3, SDE-1 | Schema first; add-to-cart/place-order APIs with error handling; order prioritization at peak; payment Factory/Strategy; JWT auth; SQL vs NoSQL per component |
| **Cart with inventory, discounts, payment, notifications** | Swiggy SDE-2 | Validation against inventory; discount strategy; observer for notifications |
| **Order management with inventory blocking** | Meesho SDE-1 | Reserve at create, decrement at confirm, release on cancel/timeout; thread safety |
| **Coupon / discount engine** | Swiggy SDE-2, Blinkit SDE-1 | Rule types, expiry, per-user limits, usage counts, holding a coupon for a 5-minute payment window; caching |
| **Driver-to-order assignment** | Swiggy SDE-1 | VIP priority queues, ratings, matching rules; data-structure choice; code + dry run |
| **Vehicle rental (ZoomCar)** | Blinkit SDE-2, Zepto SDE-1 | Multiple stores, vehicle types, reservations with time ranges, pricing Strategy, availability search |
| **Chess / Snake & Ladder / Snake game / card deck / racing leaderboard** | Zepto, Nugget, Swiggy Instamart, Zomato SDE-2, Zomato Android, Flipkart | Piece hierarchy + move-validation Strategy; O(1) snake movement (deque + set); shuffle (Fisher–Yates); leaderboard with sorted sets |
| **Calendar / meeting scheduler (recurring)** | Zepto HM/SDE-3, Zomato UI | Schema for events, participants, RSVPs; recurrence rules vs materialized instances; conflict detection; notifications |
| **Restaurant menu with configurable options** | Zomato SDE-1 | Item -> option groups -> options (size/crust/toppings), price modifiers, constraints (min/max selections) |
| **Wallet / BNPL / billing / affiliate payouts** | PhonePe, Flipkart x3 | Balance invariants, transaction log, offer/discount strategies, credit limits under concurrency, payout after the return window |
| **Issue resolution / on-call / cron / task scheduler** | PhonePe x2, Blinkit, Swiggy, Flipkart | Assignment strategy, waitlist queues, priority queues, time-based triggers, thread safety |
| **Pub-sub queue / logger / circuit breaker / feature flags** | Razorpay x2, Blinkit x2 | Ordered per-consumer offsets, retries, persistence; Chain of Responsibility for loggers; breaker states (closed/open/half-open); flag targeting rules and percentage rollouts |
| **Others** | — | Library management (Swiggy Android), online bookstore (Zepto), clinic booking (Meesho), stock exchange order book (Meesho), hackathon platform, app version management, fitness class booking, ad engine (PhonePe), Google Drive permissions, SurveyMonkey, e-commerce app (Zepto) |

### Typical bar for an LLD round, synthesized from the reports
1. **Requirements (about 5–10 min).** State the functional requirements you will build and any bonus scope. Interviewers add requirements later (B6, Z15).
2. **Entities and relationships, then classes.** Show interfaces for the parts that vary (Strategy for pricing, selection or assignment; Factory for payment processors; Observer for notifications).
3. **For Zepto, Zomato and Blinkit:** DB schema (PK/FK, indexes) and REST APIs with payloads and status codes.
4. **For Swiggy, Flipkart, Meesho and PhonePe:** **runnable in-memory code** with a driver/demo. Correctness first, then patterns.
5. **Concurrency.** Be ready to say exactly where the lock goes (per seat/slot/SKU), optimistic vs pessimistic, synchronized/ReentrantLock/ConcurrentHashMap, and DB-level guards (unique constraint, `SELECT … FOR UPDATE`, conditional update).
6. **Extensibility.** "How would you add X?" should mean adding a class, not rewriting.

---

## Part 6. Gaps and caveats

- **Hyperpure:** no design questions found anywhere searched.
- **District:** 4 reports (Z11, Z13, Z15, Z31).
- **Nugget:** 2 reports (Z5, Z14).
- **Blinkit:** the best covered company in the group (17).
- **Medium author dates** are publication dates. Several 2026-dated posts describe 2025 interviews (Z14, B16).
- **Z29 and Z33** may be two descriptions from the same GFG URL; see their notes. Z34 is snippet-only.
- **Glassdoor/AmbitionBox** gave only snippets (Blinkit: "second round was LLD system design and was resume based"; a 2025 Blinkit SE-2 reviewer described an HLD round that was mostly resume discussion with one HLD question). Those are not counted.
- **Counts are approximate** and limited to what candidates chose to write; DSA content is under-recorded here on purpose.
