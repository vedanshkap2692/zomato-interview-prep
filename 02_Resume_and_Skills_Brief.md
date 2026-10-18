# Resume & Skills Brief: Zomato SDE-1

> **Reading time:** about 1.5–2 hours. **Candidate:** Vaideeka Agrawal, B.Tech CSE, IIT Bhilai (2022–26), CGPA 8.4.
> **What this file does:** for every line on the resume, it explains the idea in plain words and gives the answer to the questions an interviewer is likely to ask. It is meant to be enough on its own — you shouldn't need to look anything up.
> **Deeper material:** Myntra in `05`, projects in `03`, DSA code in `01b`, DBMS/OS/networks/system design in `01c`.

**How to read it.** Each topic is tagged by how likely it is to come up in a Zomato backend Round 1:
- **[Must]** — expect it. Practise saying the answer out loud.
- **[Likely]** — comes up if there's time or the interviewer digs. Know the idea and one example.
- **[Basic]** — unlikely, but it's on your resume. Know enough not to be caught blank.

**The golden rule:** anything on the resume is fair game. If you can't talk about a skill for two minutes, say *"I've used it at a basic level, in X, for Y"*. That is always better than bluffing.

---

## Part 1: Introducing yourself

### 1.1 The 60-second introduction (default)

> "Hi, I'm Vaideeka, a final-year Computer Science student at IIT Bhilai with an 8.4 CGPA.
> I've done two backend internships. At **Myntra**, on the Product Detail Page team, I worked on the Ratings & Reviews service in Go. I built an **aggregated ratings pipeline** that keeps ratings consistent across all variants of a product, for **5.7 million products**, using bulk APIs, caching, rate limiting and an error queue.
> Before that, at **Atlassian Marketplace**, I built an **event-driven app-publishing system** in Kotlin and Spring Boot on **AWS SQS with worker pools**.
> I also build full-stack projects. **TraceX** is a graph-based anti-money-laundering platform that placed top 10 of 27,000 teams at the Union Bank iDEA hackathon, and **SupplySenseAI** predicts shipment delays from live weather and disaster data.
> I'm excited about Zomato because real-time delivery at India's scale is a hard systems problem, and caching, queues and high-traffic services are exactly what I've been working on."

### 1.2 The two-minute version ("walk me through your resume")
1. **Education:** IIT Bhilai CSE, 8.4. Core courses: DSA, DBMS, OS, Cloud Computing, Information Security, ML.
2. **Myntra (Jul–Dec 2025):** problem (variants showed different ratings) → what you built (bulk fetch, cache, write to every variant, limiter, error queue, warmup) → impact (5.7M styles, 2 queries → 1). Also built a stage environment.
3. **Atlassian (May–Jul 2025):** app publishing had manual steps → event-driven pipeline: request → validation → SQS → worker pool → app created automatically.
4. **Earlier:** research internship at IIT Bombay (React Native app, ML for drone landing, Raspberry Pi data pipeline); RDSO (Flutter app).
5. **Projects:** TraceX, SupplySenseAI.
6. **Achievements:** Amazon ML Challenge top 0.5%, iDEA top 10, Hack-A-Sol winner, GSSoC contributor.
7. **Close:** "I like owning a system end to end, from the data model to how it behaves under load."

### 1.3 Delivery tips
- Say numbers slowly: **5.7 million**, **top 0.5%**, **top 10 of 27,000**.
- Don't list every skill; the interviewer has your resume. Steer toward what you know best (Myntra, caching, queues).
- End on a hook — *"the ratings pipeline was the most interesting problem I've worked on"* — so the first follow-up lands where you're prepared.
- **Say your strongest languages up front:** "C++ for DSA, Go for backend."

---

## Part 2: Experience

### 2.1 Myntra — Software Engineer Intern, Product Detail Page [Must]

Full detail is in `05_Myntra_Deep_Dive.md` (30-minute read). The essentials:

**Bullet 1 — "Scalable aggregated ratings pipeline using Go and MySQL … bulk APIs, cache warmup, rate limiting, error queues … 5.7 million products."**
- A product's colours and sizes are separate *styles*. Each showed its own rating, so customers saw 4.4★ on one colour, 4.3★ on another and "No Ratings" on a new one.
- You made every style in a group (*cluster*) show one combined rating. Related styles come from **Accio** (Myntra's source of truth for style relationships) in **batches of 20**; the set is cached in **Aerospike** for 7 days; the aggregate is recomputed and written to **every** style in the cluster.
- A **token-bucket rate limiter** protects Accio; rejected events go to a **Kafka error queue** to be retried.
- A **cache warmup** filled the cache for all 5,710,991 styles before launch. It took 5 runs; adding a 4 ms delay between events fixed the rate-limit failures.
- **The aggregate math:** sum the per-star counts across the cluster, then take a weighted average. Never average the averages.

**Bullet 2 — "Optimized queries and cache access patterns, reducing database connections by 50%."**
- Two SQL queries (ratings; review and image counts) became **one parameterised query**, with grouping moved into Go. One DB round trip instead of two per computation.
- While doing this you found a bug: the old query built `CASE` clauses from a Go map, and **Go randomises map order**, so styles were sometimes grouped wrongly.
- **If asked "50% of what?":** "Per aggregated-rating computation, two queries became one. It isn't a service-wide figure."

**Bullet 3 — "Production-parity stage environment … MySQL, Aerospike, Kafka, downstream services."**
- The service had only dev and production. You set up stage: endpoints and timeouts, MySQL schema with mock data, Aerospike provisioning, DB and event credentials stored in Vault, then deployed and verified.

### 2.2 Atlassian — Software Engineer Intern, Marketplace [Must]

> **Check before the interview:** the details below are a standard design reconstructed from the resume bullets. Replace anything that differs from what you actually built, and be ready to say "that part was owned by another team".

**Context.** Atlassian Marketplace is the store where outside vendors publish apps for Jira, Confluence and Bitbucket. Publishing used to need manual steps: a person checked each request and created the app. That was slow, error-prone and didn't scale.

**What you built:**

```
Vendor → Spring Boot API ──(validate; reject bad requests with 400)──┐
                │ valid                                             │
                ▼                                                   │
         put message on SQS  ── returns "202 Accepted" to vendor ───┘
                │
                ▼
      Worker pool (N instances × K workers)
        - skip if already processed (idempotency)
        - create the app via Marketplace APIs
        - on success, delete the message
        - on failure, leave it → it reappears → retried
        - after several failures → dead-letter queue (DLQ)
```

**The concepts behind each bullet, explained**

**Event-driven vs request-response.** In request-response, the caller waits while the server does all the work. In event-driven design, the API records the request as an *event* (a message on a queue) and replies at once; separate workers do the slow work later. Benefits: the API stays fast; a burst of requests simply waits in the queue; a failed step can be retried without the caller noticing; the API and the workers can be scaled separately. Costs: the result isn't immediate, there are more moving parts, and you need status tracking and duplicate protection.

**AWS SQS (Simple Queue Service).** A fully managed message queue: you put messages in, workers take them out.
- **Standard queue:** very high throughput, *at-least-once* delivery (a message can occasionally arrive twice), order not guaranteed.
- **FIFO queue:** strict order within a *message group*, duplicates removed within a 5-minute window, lower throughput.
- **Visibility timeout:** when a worker receives a message, SQS hides it for a set time (default 30 s). If the worker deletes it in time, it's done. If the worker crashes, the message becomes visible again and another worker gets it. This is how SQS gives fault tolerance with no extra code. Set it longer than your slowest processing time.
- **Dead-letter queue (DLQ):** after a message fails a set number of times (`maxReceiveCount`), SQS moves it to a separate queue so it can't block the pipeline. You alert on and inspect the DLQ.
- **Long polling:** a worker waits up to 20 s for messages instead of getting empty replies, which saves cost and CPU.
- **Limits worth knowing:** up to 10 messages per receive, 256 KB per message, retention up to 14 days.

**Worker pool.** A fixed number of workers (threads or coroutines) pulling from the queue. Why fixed? Unlimited workers would exhaust memory and overwhelm the downstream APIs and database. A bounded pool is a natural rate limit. Rule of thumb for I/O-heavy work: more workers than CPU cores, because workers mostly wait on network calls; tune with load tests. **Backpressure:** if all workers are busy, they stop polling and messages just wait in SQS — the queue is the buffer.

**Fault tolerance.** Worker crash → message not deleted → visible again → retried. A message that always fails → DLQ. Downstream outage → retry with **exponential backoff and jitter** (wait 1 s, 2 s, 4 s… plus a random bit, so all retries don't hit the recovering service at the same instant).

**Idempotency** (very likely follow-up). *Idempotent* means doing it twice has the same effect as doing it once. Because SQS delivers at least once, a worker might create the app, crash before deleting the message, and then another worker processes it again. Protection: give each request a unique ID; check "already processed?" before acting; add a database unique constraint (e.g. on vendor + app key + version) so a duplicate insert fails harmlessly; only allow forward status changes (`RECEIVED → IN_PROGRESS → PUBLISHED`).

**Horizontal scaling.** Workers keep no state of their own — everything is in the database or queue — so you can add more instances behind the same queue. SQS gives each message to one receiver at a time. Autoscale on queue depth (`ApproximateNumberOfMessagesVisible`).

**Validation.** *Syntactic* checks (required fields, formats) use Bean Validation annotations such as `@NotBlank` and `@Size` with `@Valid` on the controller. *Semantic* checks: does this vendor own the app key, is the version newer than the last one? Validating before queuing gives the caller a clear error straight away and stops workers wasting time on requests that will fail.

**Likely questions**
- *"What if a worker dies halfway?"* The message isn't deleted, so it reappears after the visibility timeout and another worker retries; the idempotency check prevents a duplicate app.
- *"Why SQS and not Kafka?"* SQS is a fully managed **work queue**: each message is handled by one worker, then deleted. Kafka suits many independent consumers reading the same stream, replay, or very high throughput with ordering. "I used both: SQS at Atlassian for jobs, Kafka at Myntra for events."
- *"Why not just a thread pool inside the API?"* In-memory work is lost on a crash and can't be scaled separately. A queue is durable, retries for free and decouples the two sides.
- *"10× traffic?"* The queue absorbs the burst; autoscale workers on queue depth; make sure downstream services can take the load (cap concurrency).
- *"What would you improve?"* **Outbox pattern:** writing the request to the DB and sending to SQS are two separate systems, so a crash between them loses the event. Write the request and an "outbox" row in the same DB transaction; a relay process then sends outbox rows to SQS. Also: a status webhook to tell vendors when publishing finishes.

### 2.3 TIH-IoT, IIT Bombay (Chanakya Fellowship) — Research Intern [Likely]

**Bullet 1 — React Native app for data visualisation; ML to assess soil suitability for drone (UAV) landing, 75% precision.**
- **React Native** writes mobile apps in JavaScript/React that render real native components on Android and iOS from one codebase.
- **Precision vs recall** (you will be asked this). For the "safe to land" class:
  - **Precision** = TP / (TP + FP): of the spots the model *called* safe, how many really were.
  - **Recall** = TP / (TP + FN): of the spots that *were* safe, how many the model found.
  - **F1** = 2·P·R / (P + R), the balance of the two.
- *"Why precision?"* Calling unsafe soil safe (a false positive) could crash the drone, so precision on "safe" is what matters most. Missing a safe spot only costs time.
- *"Is 75% good enough?"* It was a research prototype. In production you'd raise the decision threshold (fewer "safe" calls, but more trustworthy ones) and keep a human or a sensor check in the loop.

**Bullet 2 — Wireless SSH data-transfer pipeline from Raspberry Pi to mobile, 99% transmission accuracy.**
- **SSH** (Secure Shell) is an encrypted channel over TCP (port 22). It supports password or **key-based** login (a private key stays with the client; the server holds the public key). **SCP and SFTP** are file-transfer tools built on SSH.
- *"How did you measure 99%?"* Compare what was received with what was sent: record counts and checksums (a hash of the file on both ends; if the hashes match, the file is intact).
- *"What was the other 1%?"* Usually dropped Wi-Fi connections mid-transfer. Fixes: retry with resume, and verify checksums after every transfer.

### 2.4 RDSO, Indian Railways — Summer Intern [Basic]
- A **Flutter** Android app with dynamic forms, pop-up feedback and graphs driven by user input.
- **Flutter** is Google's cross-platform UI toolkit written in **Dart**. Everything on screen is a **widget**, arranged in a tree. A **StatelessWidget** never changes after it's built; a **StatefulWidget** holds state and rebuilds when `setState()` is called. Flutter draws its own UI with its own engine, whereas React Native uses the platform's native components.

---

## Part 3: Projects [Must]

The full code-verified deep dive is in `03_Projects_TraceX_SupplySenseAI.md`. Before the interview you must be able to: draw each architecture in under two minutes, explain why each technology was chosen, and explain one graph algorithm from each.

### 3.1 TraceX — graph-based anti-money-laundering (AML) investigation platform
- **What it does:** banks raise thousands of suspicious-transaction alerts. TraceX models accounts and transfers as a **graph**, detects laundering patterns (chains of quick hops, money going round in a cycle, many small deposits just under a reporting limit), scores accounts, ranks alerts, creates cases and gives investigators a workspace with the money-flow graph, timeline, similar cases and an AI assistant.
- **Context:** built for the Union Bank iDEA hackathon (top 10 of 27,000+), then rebuilt as a production-style system. Built with a teammate — claim with "I" only your own parts: **auth and roles, the detection port, alert generation and case workflow, similar cases, and the whole Next.js frontend.**
- **Stack and why:** FastAPI (Python, so ML and graph code run in the same process), SQLAlchemy + SQLite (zero setup; can switch to Postgres), NetworkX (graph algorithms out of the box, behind an interface so it could become Neo4j), XGBoost + Isolation Forest (labelled and unlabelled detection), LLMs (summaries and copilot), PyTest + CI.
- **Graph ideas to explain:** a cycle in the transfer graph = money returning to its source (round-tripping); strongly connected components = groups of accounts all moving money among themselves; BFS/DFS out N hops from an account = where the money went.

### 3.2 SupplySenseAI — supply-chain risk platform
- **What it does:** predicts which shipments will be delayed using live weather and disaster data, explains *why* (SHAP), shows how one disruption spreads through the supply network, and suggests alternative routes.
- **Stack:** FastAPI, Next.js/React/TypeScript, XGBoost, SHAP, NetworkX, Docker, Google Cloud Run.
- **Ideas to explain:** "cascade" = **BFS** from the disrupted node through the supply graph, multiplying the impact by 0.85 at each hop and stopping after 4 hops or when impact drops below 0.05. Route suggestions compare **three candidate routes** (direct road, via another port, air) on cost, time, carbon and risk, normalise each metric and pick the lowest weighted score. It is a weighted ranking, not a shortest-path search — say so precisely.
- **A critique to raise yourself:** the BFS pops from the front of a Python list, which is O(n) per pop; `collections.deque` makes it O(1).
- **Be honest about:** the ML metrics come from **synthetic labels** (there wasn't real delay data), so describe them as a proof of concept.

### 3.3 Questions that apply to both
- *"Why FastAPI over Flask or Django?"* Async support, automatic request validation with Pydantic, and auto-generated API docs; and Python keeps the ML code in-process.
- *"How would you scale it?"* Postgres instead of SQLite; a graph database or precomputed features instead of in-memory NetworkX; run model training as offline jobs; cache hot results; scale API pods horizontally.
- *"What if the LLM makes something up?"* Ground it in real case data, check every claim against the database, show sources, and keep the human investigator as the final decision-maker.

---

## Part 4: Skills, explained

Zomato's backend mainly uses **Go and Java/Kotlin**, with **MySQL, Redis, Kafka and Elasticsearch** on **AWS with Kubernetes**. Your resume overlaps heavily, so lean into that.

### 4.1 Programming languages

#### C++ [Must] — your DSA language
- **`vector`** is a dynamic array: O(1) access by index, amortised O(1) `push_back`. When it grows it may move to new memory, so existing pointers and iterators into it become invalid.
- **`unordered_map` vs `map`.** `unordered_map` is a hash table: average O(1) lookup, no ordering, O(n) worst case with many collisions. `map` is a balanced binary search tree (red-black tree): O(log n), but keys stay sorted, so you can find the next-larger key. Use `map` when you need order; otherwise `unordered_map`.
- **`set` / `unordered_set`:** the same distinction, without values.
- **`priority_queue`** is a **max-heap** by default (`top()` is the largest). For a min-heap: `priority_queue<int, vector<int>, greater<int>>`. Push and pop are O(log n).
- **`deque`** allows O(1) push and pop at both ends (used for sliding-window maximum). **`stack`** and **`queue`** are adaptors built on it.
- **`sort(v.begin(), v.end(), cmp)`** is O(n log n). The comparator must be a strict "less than" (return `a < b`, never `a <= b`).
- **`lower_bound`** returns the first element ≥ x; **`upper_bound`** the first element > x, both O(log n) on a sorted range.
- **Pass by reference** (`const vector<int>& v`) to avoid copying big containers — a common reason for "time limit exceeded".
- **Overflow:** `int` holds about ±2.1 × 10⁹. Use `long long` for sums and products of large numbers.

#### Go [Must] — your Myntra language, and Zomato uses it
- **Goroutines** are very lightweight threads managed by the Go runtime, not the OS. They start with a few KB of stack (OS threads use about 1 MB), and the runtime multiplexes thousands of them onto a few OS threads (*M:N scheduling*). Start one with `go f()`.
- **Channels** pass values between goroutines safely. An **unbuffered** channel makes the sender wait until a receiver takes the value (a hand-off). A **buffered** channel (`make(chan int, 10)`) lets the sender continue until the buffer is full. Closing a channel tells receivers no more values are coming. *"Don't communicate by sharing memory; share memory by communicating."*
- **`select`** waits on several channel operations and runs whichever is ready first — used for timeouts and cancellation.
- **`sync.WaitGroup`** waits for a group of goroutines to finish (`Add`, `Done`, `Wait`). **`sync.Mutex`** protects shared data so only one goroutine changes it at a time.
- **`context.Context`** carries deadlines and cancellation across calls: `ctx, cancel := context.WithTimeout(ctx, 2*time.Second)`. Every DB or HTTP call should take a context so a slow request can be abandoned.
- **Errors are values.** Functions return `(result, error)` and you check `if err != nil`. Wrap with context using `fmt.Errorf("fetching ratings: %w", err)`; check with `errors.Is` / `errors.As`.
- **Interfaces are satisfied implicitly:** a type that has the methods *is* the interface, with no `implements` keyword. That makes mocking in tests easy.
- **Slices vs arrays.** An array has a fixed size. A slice is a small header (pointer to an array, length, capacity). Two slices can share the same underlying array, so changing one can change the other. `append` may allocate a new array when capacity runs out.
- **Maps:** iteration order is **deliberately random** (your Myntra bug), and maps are **not safe for concurrent writes** — concurrent writes crash the program. Use a mutex or `sync.Map`.
- **`defer`** runs a call when the function returns, in reverse order — used to close files, unlock mutexes, or mark `wg.Done()`.

**The worker pool in Go** — worth writing from memory; it connects Myntra and Atlassian:

```go
func workerPool(jobs <-chan Job, results chan<- Result, n int) {
    var wg sync.WaitGroup
    for i := 0; i < n; i++ {        // start n workers
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := range jobs {     // each worker pulls until jobs is closed
                results <- process(j)
            }
        }()
    }
    wg.Wait()                         // all workers finished
    close(results)                    // tell the reader no more results
}
```

#### Java [Likely]
- **The four OOP pillars.** *Encapsulation:* hide data behind methods. *Abstraction:* expose what something does, not how. *Inheritance:* a class reuses another's behaviour. *Polymorphism:* one interface, many implementations — overloading is resolved at compile time, overriding at run time.
- **How `HashMap` works** (a classic). It's an array of *buckets*. To store a key, Java calls `hashCode()`, mixes the bits and picks a bucket index. Keys that land in the same bucket (*collisions*) are kept in a linked list; since Java 8, a bucket with more than 8 entries becomes a red-black tree, so the worst case is O(log n) instead of O(n). When the map is 75% full (the *load factor*), it doubles the array and redistributes entries. Average get/put is O(1).
- **`equals` and `hashCode` contract:** if two objects are equal, they **must** have the same hash code. Override one without the other and `HashMap` lookups silently fail.
- **`ConcurrentHashMap`** is thread-safe without locking the whole map (it locks or uses compare-and-swap per bucket). **`Hashtable`** locks everything and is outdated.
- **`ArrayList` vs `LinkedList`:** ArrayList has O(1) access by index and is cache-friendly; LinkedList has O(1) insert/delete at a known node but O(n) access. In practice ArrayList is almost always faster.
- **Checked vs unchecked exceptions.** Checked (e.g. `IOException`) must be caught or declared; unchecked (`RuntimeException`, e.g. `NullPointerException`) need not be.
- **`final`** (can't reassign, override or extend) vs **`finally`** (block that always runs after try/catch) vs **`finalize`** (old, deprecated cleanup hook).
- **JVM memory:** objects live on the **heap**, shared by all threads and cleaned up by the **garbage collector**; local variables and call frames live on each thread's **stack**.
- **`String` is immutable**, so it's safe to share and cache, and its hash can be stored. Use `StringBuilder` to build strings in a loop.

#### Kotlin [Likely] — your Atlassian language (listed under Web & App)
- **Null safety:** `String` can never be null; `String?` can. `?.` calls only if non-null; `?:` (Elvis) gives a fallback; `!!` forces it and throws if null.
- **`val`** is read-only, **`var`** is mutable.
- **Data classes** generate `equals`, `hashCode`, `toString` and `copy` automatically.
- **Coroutines** are lightweight, suspendable functions — similar in spirit to goroutines. `suspend fun` can pause without blocking a thread.
- **Extension functions** add methods to existing classes without inheriting from them.
- **Fully interoperable with Java,** which is why the resume lists both: the same Spring Boot service can mix Kotlin and Java code.

#### Python [Likely] — used in both projects
- Lists (ordered, mutable), tuples (immutable), dicts (hash maps, keep insertion order), sets.
- **Comprehensions:** `[x*x for x in nums if x > 0]`.
- **Generators** (`yield`) produce values one at a time, so a huge sequence never sits in memory.
- **Decorators** wrap a function to add behaviour (logging, auth checks) — FastAPI's `@app.get("/")` is one.
- **The GIL** (Global Interpreter Lock) lets only one thread run Python code at a time. Threads still help for I/O-bound work (waiting on network or disk), but CPU-heavy work needs `multiprocessing` or libraries written in C (NumPy).
- **`async`/`await`:** one thread juggles many waiting tasks — what FastAPI uses to handle many requests concurrently.

#### C [Basic]
Pointers hold memory addresses. `malloc` allocates on the heap and `free` releases it; forgetting `free` leaks memory, and using memory after `free` is undefined behaviour. The stack holds local variables and is freed automatically when a function returns.

#### JavaScript / TypeScript [Basic]
- JavaScript runs on one thread with an **event loop**: slow operations (network, timers) are handed off, and their callbacks run when ready, so the thread is never blocked.
- **Promises** represent a future value; **`async`/`await`** is cleaner syntax for them.
- `let` and `const` are block-scoped (`const` can't be reassigned); `var` is function-scoped and old.
- A **closure** is a function that remembers variables from where it was created.
- **TypeScript** adds static types to JavaScript; errors are caught at compile time and editors give better autocomplete.

### 4.2 Web and app development

#### SQL and databases [Must]
Zomato almost always asks database questions. `01c` has 14 practice queries; this is the concept core.

**Joins**
- **INNER JOIN:** only rows that match in both tables.
- **LEFT JOIN:** every row from the left table, with NULLs where the right side has no match. "Customers with no orders" = `LEFT JOIN orders … WHERE orders.id IS NULL`.
- **RIGHT JOIN:** the mirror image. **FULL OUTER JOIN:** all rows from both sides (MySQL doesn't have it; use a `UNION` of left and right joins).
- **SELF JOIN:** a table joined to itself, e.g. employee to manager.

**Grouping.** `GROUP BY` collapses rows into groups; `WHERE` filters rows **before** grouping; `HAVING` filters groups **after** (e.g. `HAVING COUNT(*) > 5`).

**Window functions** compute across related rows without collapsing them. `ROW_NUMBER() OVER (PARTITION BY city ORDER BY rating DESC)` numbers restaurants within each city.
- `ROW_NUMBER` gives 1, 2, 3, 4 (unique).
- `RANK` gives 1, 2, 2, 4 (ties share a rank; a gap follows).
- `DENSE_RANK` gives 1, 2, 2, 3 (no gap).
- "Top 3 per group" and "Nth highest salary" are classic uses.

**Indexes** — how lookups become fast.
- An index is a separate sorted structure, almost always a **B+ tree**: a short, wide tree whose leaves hold keys in order and are linked together, so both exact lookups and range scans are fast (O(log n)).
- Without an index, the database reads every row (a *full table scan*).
- **Composite index** on `(city, rating)`: works for queries filtering on `city`, or `city` and `rating`, but **not** `rating` alone — the **leftmost-prefix rule**, like a phone book sorted by surname then first name.
- **Covering index:** contains every column the query needs, so the table itself is never read.
- **Cost:** indexes slow down writes (each insert updates every index) and use space. Don't index low-variety columns such as a boolean.
- **`EXPLAIN`** shows whether a query uses an index or scans the table.

**Transactions and ACID** — a transaction is a group of operations that succeed or fail together.
- **Atomicity:** all or nothing (a payment debits *and* credits, or neither).
- **Consistency:** constraints still hold afterwards.
- **Isolation:** concurrent transactions don't see each other's half-done work.
- **Durability:** once committed, it survives a crash (it's written to a log on disk first).

**Isolation levels and the problems they prevent**
- *Dirty read:* reading another transaction's uncommitted data.
- *Non-repeatable read:* reading the same row twice and getting different values.
- *Phantom read:* re-running a query and getting new rows.
- **Read Uncommitted** allows all three. **Read Committed** prevents dirty reads. **Repeatable Read** (MySQL InnoDB's default) also prevents non-repeatable reads. **Serializable** prevents all three, at the cost of concurrency.

**Normalisation** — removing duplication so data can't contradict itself.
- **1NF:** one value per cell, no repeating groups.
- **2NF:** no column depends on only *part* of a composite key.
- **3NF:** no column depends on another non-key column (e.g. don't store `city_name` next to `city_id` in an orders table).
- **BCNF:** a stricter 3NF — every determinant is a key.
- **Denormalisation** deliberately duplicates data to avoid joins on hot read paths — which is what a cache does.

**SQL vs NoSQL.** SQL databases (MySQL, Postgres) give fixed schemas, joins and strong transactions. NoSQL (MongoDB documents; Cassandra/DynamoDB wide-column; Redis/Aerospike key-value) gives flexible schemas and easy horizontal scaling, usually with weaker guarantees. Rule of thumb: orders and payments → SQL; huge simple lookups, caches, feeds → NoSQL.

**Connection pooling.** Opening a DB connection is slow (network handshake, authentication). A pool keeps a set of open connections and lends them out. Too small and requests wait; too large and the database is overloaded. In Go, `database/sql` pools automatically (`SetMaxOpenConns`, `SetMaxIdleConns`).

#### REST APIs and HTTP [Must]
- **REST** treats everything as a *resource* with a URL (`/restaurants/42/orders`), uses HTTP methods as verbs and is **stateless** — each request carries everything needed (e.g. an auth token), so any server can handle it.
- **Methods:** GET (read), POST (create), PUT (replace), PATCH (partial update), DELETE.
- **Idempotency:** GET, PUT and DELETE are idempotent — repeating them doesn't change the result further. POST is not; repeating it creates duplicates. That's why payment APIs take an **idempotency key**.
- **Status codes:**
  - 200 OK, 201 Created, 202 Accepted (queued, like Atlassian), 204 No Content.
  - 400 Bad Request, 401 Unauthorized (not logged in), 403 Forbidden (logged in but not allowed), 404 Not Found, 409 Conflict, 429 Too Many Requests (rate limited).
  - 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout.
- **Pagination.** *Offset* (`?page=5&size=20`) is simple but slow on deep pages and skips or repeats items when data changes. *Cursor* (`?after=<last_id>`) is stable and fast — use it for feeds.
- **Versioning:** `/v1/orders`, so old clients keep working when the API changes.

#### Spring Boot [Likely] — Atlassian
- **Inversion of Control / Dependency Injection:** you don't create objects with `new`; Spring creates them (*beans*) and passes each one the others it needs. This makes code loosely coupled and easy to test with mocks. **Constructor injection** is preferred.
- **Annotations:** `@RestController` (handles HTTP), `@Service` (business logic), `@Repository` (database access), `@Component` (any bean), `@Autowired` (inject).
- **Layers:** Controller → Service → Repository. **DTOs** carry API data; **entities** map to database tables.
- **`@Transactional`** wraps a method in a DB transaction (commit on success, roll back on exception). Gotcha: calling it from another method in the *same* class skips the transaction, because Spring adds it through a proxy that the internal call bypasses.
- **Bean scopes:** singleton (one shared instance, the default) and prototype (new each time).

#### FastAPI [Likely] — both projects
A modern Python web framework. **Async** endpoints handle many concurrent requests; **Pydantic** models validate request bodies automatically (wrong types → 422 error); interactive **API docs** are generated at `/docs`; dependencies such as the DB session or current user are injected with `Depends()`.

#### React and Next.js [Likely] — TraceX frontend
- **React** builds UIs from **components** (functions that return UI). **Props** are inputs passed in by the parent; **state** is data the component owns and changes, and changing it re-renders the component.
- **Hooks:** `useState` holds state; `useEffect` runs side effects (fetching data) after render, with a dependency list controlling when it re-runs.
- **Virtual DOM:** React builds an in-memory copy of the UI, compares it with the previous one (*reconciliation*) and changes only what differs in the real page.
- **Next.js** adds file-based routing, **server-side rendering** (HTML built per request — fast first load, good SEO), **static generation** (HTML built at build time) and server-side API routes. In TraceX those server routes act as a **BFF** (backend-for-frontend): the browser talks only to Next.js, which keeps the login token in an `httpOnly` cookie that JavaScript can't read.

#### Basic-level web skills
- **HTML/CSS:** structure and styling. Know the box model (content, padding, border, margin) and Flexbox/Grid for layout.
- **PHP:** server-side scripting language. Fun fact: Zomato's original stack was PHP.
- **React Native and Flutter:** see 2.3 and 2.4.

### 4.3 AI / ML [Likely for project questions; otherwise Basic]

A Zomato SDE round won't test ML theory unless you steer there, but your projects and achievements use ML, so know these.

- **Supervised vs unsupervised.** Supervised learns from labelled examples ("this transaction was fraud"); unsupervised finds structure without labels (clusters, outliers).
- **Overfitting:** the model memorises training data and fails on new data. Prevent it with a held-out test set, cross-validation, regularisation, simpler models and more data.
- **Decision tree:** a series of yes/no questions on features. Easy to explain, but one tree overfits.
- **Random forest:** many trees on random samples of the data, averaged — reduces overfitting.
- **XGBoost** (gradient boosting): builds trees **one after another, each fixing the mistakes of those before**, with regularisation and built-in handling of missing values. Usually the strongest choice for tabular data — hence both projects.
- **Isolation Forest:** unsupervised anomaly detection. It builds random trees with random splits; an unusual point gets isolated in **very few splits** (it sits alone), a normal one needs many. Short average path = anomaly. Good for fraud, where labels are scarce.
- **SHAP:** explains a single prediction by fairly sharing it among the features (based on Shapley values from game theory). E.g. "delay risk is high: heavy rain +0.30, port congestion +0.20".
- **Metrics.** Accuracy is misleading on imbalanced data — if 1% of transactions are fraud, predicting "never fraud" scores 99%. Use precision, recall, F1 and **PR-AUC**. **ROC-AUC** measures how well the model ranks positives above negatives across all thresholds.
- **Class imbalance:** weight the rare class more (XGBoost's `scale_pos_weight`), resample (oversample rare cases, e.g. with SMOTE, or undersample common ones), and tune the decision threshold.
- **Deep learning:** neural networks with many layers. Weights are learned by **backpropagation** (computing how each weight affects the error) and **gradient descent** (nudging weights to reduce it).
- **NLP:** processing text — tokenisation, embeddings (words as vectors where similar meanings sit close together), classification, summarisation.
- **Computer vision:** processing images, classically with **CNNs** (convolutional filters that detect edges, then shapes, then objects).
- **Transformers:** the architecture behind modern LLMs. **Self-attention** lets every token look at every other token and decide which matter; each token produces a *query*, a *key* and a *value*, and attention weights come from query–key similarity. Positional encoding adds word order. Unlike older RNNs, all tokens are processed in parallel.
- **LLMs:** large transformers trained to predict the next token on huge amounts of text, then tuned to follow instructions.
- **Generative AI:** models that create new content (text, images, code).
- **RAG (retrieval-augmented generation):** first retrieve relevant documents (often by embedding similarity from a vector store), put them in the prompt, then generate. Answers are grounded in real data and less likely to be made up.
- **LangChain:** a Python library for chaining LLM calls, prompts, retrievers and tools.
- **Adversarial ML:** inputs crafted to fool a model — e.g. tiny pixel changes that flip an image classifier, or fraudsters shaping transactions to slip past detection. Defences: adversarial training and input checks.
- **Libraries.** scikit-learn: classic ML. PyTorch: deep learning with automatic gradients. Streamlit: quick Python web apps for demos.
- **Amazon ML Challenge 2024** (top 0.5%): the task was extracting entity values (weight, dimensions, voltage…) from product images. Prepare your approach and what improved your score most.

### 4.4 Tools and infrastructure

#### Kafka [Must] — Myntra, and Zomato uses it
- A **topic** is a named stream, split into **partitions**. Each partition is an append-only log; every message has an increasing **offset**.
- **Producers** choose the partition by key: same key → same partition → **order is guaranteed per key** (e.g. all events for one order stay in order). There is no ordering across partitions.
- **Consumer groups:** within a group, each partition is read by exactly one consumer, so adding consumers (up to the number of partitions) spreads the load. A different group reads the same data independently — this is how many services consume one stream.
- **Offsets:** consumers record how far they've read. Committing *after* processing gives **at-least-once** delivery (a crash means reprocessing, so make handlers idempotent).
- **Retention:** messages are kept for a set time (e.g. 7 days) whether read or not, so you can **replay**. That's the key difference from SQS, where messages are deleted after processing.
- **Durability:** each partition is copied to several brokers (*replication factor*); one is the *leader*, and followers that are up to date form the **ISR** (in-sync replicas).
- **Kafka vs SQS in one line:** Kafka is a replayable log for many consumers with per-key ordering; SQS is a simple managed job queue where each message is handled once and deleted.

#### Docker [Likely]
- An **image** is a read-only template (code + runtime + libraries), built from a **Dockerfile** in layers. A **container** is a running instance of an image.
- **Layer caching:** each Dockerfile step is cached, so copy the dependency file and install dependencies *before* copying source code; then code changes don't reinstall everything.
- **Container vs VM:** containers share the host's OS kernel, so they start in seconds and are lightweight. A VM runs a whole guest OS on a hypervisor — heavier, with stronger isolation.
- Run as a non-root user and keep images small (slim base images, multi-stage builds).

#### Kubernetes [Likely]
Kubernetes runs containers across many machines and keeps them in the state you declare.
- **Pod:** the smallest unit — one or more containers sharing a network address.
- **Deployment:** declares "run 3 replicas of this pod"; handles rolling updates and rollback, using a **ReplicaSet** underneath.
- **Service:** a stable address and load balancer in front of changing pods.
- **HPA (Horizontal Pod Autoscaler):** adds or removes pods based on CPU or custom metrics.
- **ConfigMap / Secret:** configuration and sensitive values injected into pods.
- **Probes:** *liveness* — "is it stuck? restart it"; *readiness* — "is it ready to receive traffic? if not, stop sending it requests".
- **Myntra link:** "per-pod rate limit" means each running replica had its own bucket.

#### Git [Likely]
- **merge** joins branches with a merge commit and keeps the history as it happened. **rebase** replays your commits on top of the other branch for a straight line. Never rebase commits others have already pulled.
- **cherry-pick** copies one commit onto another branch. **stash** shelves uncommitted changes. **Conflicts:** edit the marked sections, `git add`, then continue.
- `git reset` moves the branch pointer (can discard history); `git revert` makes a new commit that undoes an old one (safe on shared branches).

#### Linux and shell scripting [Basic]
- `grep` (search text), `awk` (pick columns), `sed` (find and replace), `ps` and `top` (processes), `kill -9` (force stop), `tail -f` (follow a log), `df`/`du` (disk usage), pipes `|` chain commands.
- **Permissions:** `chmod 755` = owner read/write/execute (7); group and others read/execute (5). Read = 4, write = 2, execute = 1.
- A shell script starts with `#!/bin/bash`; `$1` is the first argument, `$?` the last exit code.

#### GCP [Basic]
**Cloud Run** (used for SupplySenseAI) runs a container without you managing servers. It scales on request volume, down to zero when idle, and you pay per use.

#### Figma [Basic]
A collaborative UI design tool. You used it for mock-ups before building frontends.

### 4.5 System design [Likely] — it's on your resume, so expect at least one question

Tie each concept to your own work; that is far stronger than textbook answers. `01c` has full worked designs (rate limiter, notifications, food ordering).

- **Caching.** *Cache-aside* (what Myntra used): read the cache; on a miss, read the DB and fill the cache. *TTL* bounds staleness. *Eviction* when full: **LRU** (drop least recently used) or **LFU** (least frequently used). *Invalidation:* update or delete the cached value when the source changes — the hard part, as the Myntra mismatch bug showed. **Cache stampede:** a hot key expires and thousands of requests hit the DB at once; fix with warmup, letting only one request rebuild the value, or random jitter on TTLs.
- **Rate limiting.**
  - *Token bucket:* allows bursts, caps the average (Myntra).
  - *Leaky bucket:* processes at a constant rate, smoothing bursts.
  - *Fixed window:* "100 per minute" counter; simple, but allows double bursts at window edges.
  - *Sliding window:* fixes the edge problem.
  - Across many servers, use a shared counter in Redis (`INCR` with `EXPIRE`, or a Lua script so check-and-increment is atomic).
- **Message queues.** Decouple services, absorb spikes, enable retries and DLQs; at-least-once delivery means handlers must be idempotent (Atlassian).
- **Scaling databases.** In order: indexes and query fixes → caching → **read replicas** (copies that serve reads; may lag slightly behind) → **sharding** (split data across databases by a key such as `user_id` or city; cross-shard queries become hard).
- **CAP theorem.** When a network partition happens, a distributed system must choose **consistency** (refuse or wait rather than return stale data) or **availability** (answer, possibly with stale data). Payments choose consistency; ratings choose availability — your aggregated ratings are *eventually consistent*, which is a perfect example.
- **Load balancing.** Round robin, least connections, or hashing by key. **Consistent hashing** puts servers and keys on a ring; adding or removing a server only moves the keys next to it rather than reshuffling everything. Used by caches and sharded stores.
- **Observability.** *Logs* (what happened), *metrics* (how much and how fast — error rate, p99 latency, queue depth), *traces* (one request's path across services). Alert on the metrics users feel.

### 4.6 Coursework topics [Likely]

Each course links to something you built. Short versions here; `01c` goes deeper on OS and networks.

**Operating systems**
- **Process vs thread:** a process has its own memory space; threads inside it share memory, which makes them cheaper but needs locking.
- **Deadlock** needs four conditions together: mutual exclusion, hold-and-wait, no pre-emption, circular wait. Break any one — e.g. always acquire locks in the same order.
- **Mutex vs semaphore:** a mutex lets one holder in at a time; a semaphore lets up to N in (like a bounded worker pool).
- **Virtual memory and paging:** each process sees its own address space, split into pages that the OS maps to physical memory; a missing page triggers a page fault and is loaded from disk.
- **Your links:** worker pools (Atlassian), goroutines (Myntra).

**Computer networks**
- **TCP vs UDP:** TCP is reliable, ordered and connection-based (three-way handshake: SYN, SYN-ACK, ACK) — used by HTTP. UDP is fire-and-forget and faster — used for video calls and DNS.
- **What happens when you type a URL:** DNS lookup → TCP connection → TLS handshake → HTTP request → server response → browser renders.
- **HTTPS** = HTTP over TLS: the server proves its identity with a certificate, then both sides agree a shared key and encrypt traffic.

**Information security** — links to your SSH pipeline and TraceX auth
- **Hashing vs encryption:** hashing is one-way (store password hashes — with a salt, using a deliberately slow algorithm such as bcrypt); encryption is reversible with a key.
- **Symmetric vs asymmetric:** one shared key (fast, AES) vs a public/private key pair (RSA; used to exchange keys and sign).
- **JWT:** a signed token with user claims; the server verifies the signature instead of looking up a session, which keeps the API stateless (used in TraceX).
- **SQL injection:** user input treated as SQL. Prevent with parameterised queries — exactly what you did at Myntra. **XSS:** injected scripts in a page; prevent by escaping output, and use `httpOnly` cookies so scripts can't read tokens.

**Cloud computing:** AWS SQS (Atlassian), GCP Cloud Run (SupplySenseAI), Docker and Kubernetes. Know IaaS (virtual machines) vs PaaS (managed platforms) vs SaaS (finished software).

**Database systems:** section 4.2. **DSA:** graph traversal in both projects; see `01`/`01b`. **ML:** section 4.3.

### 4.7 Soft skills [Basic]
- **Agile:** work in short cycles (*sprints*, often two weeks) with planning, daily stand-ups, review and retrospective, delivering small working pieces rather than one big release. Say how your internship teams worked.
- **Communication, leadership, management:** back each with one story — the hackathon team for leadership (Hack-A-Sol, iDEA), and clear written design or status updates at your internships for communication. See `04` Part C for the story table.

---

## Part 5: Achievements [Likely]

| Achievement | Be ready to say |
|---|---|
| **Amazon ML Challenge 2024**, top 0.5% (rank 334 of 75,000+) | The task (extracting values such as weight and dimensions from product images), your approach and what improved the score most. |
| **iDEA Hackathon 2026** (Union Bank of India), top 10 of 27,000+ teams | This is TraceX. The problem statement was tracking fund flows within a bank for fraud detection. |
| **Hack-A-Sol 2024** (IIIT Raipur), AI/ML track winner, 1st runner-up overall | What you built in the time limit and your role in the team. |
| **GSSoC 2026** contributor | Name **one** specific contribution (repo, what it fixed). Vague open-source claims get probed. |

---

## Part 6: Lines that invite probing — prepare honest answers

1. **"5.7 million products"** = 5,710,991 live styles refreshed in the cache warmup.
2. **"50% fewer DB connections"** = two queries became one per aggregated-rating computation, not service-wide.
3. **Many languages listed.** "Which is your strongest?" → **C++ for DSA, Go for backend.** Say it at the start of coding rounds.
4. **Kafka, Kubernetes, System Design in Skills.** Expect "what's a consumer group?" or "what's a pod?" — revise 4.4.
5. **Long ML list** (Adversarial ML, CV, Transformers, GenAI) for an SDE role. Tie each to a project or course, or say "coursework-level exposure".
6. **"75% precision."** Research prototype; in production raise the threshold and keep a human in the loop.
7. **"99% transmission accuracy."** Measured with record counts and checksums.
8. **Overlapping dates.** Atlassian and Myntra meet in July 2025; TIH-IoT (Mar–Dec 2024) overlaps RDSO (Jun–Jul 2024). Have a one-line answer (e.g. the fellowship was part-time or remote).
9. **TraceX was a team project.** Say "I built" only for your parts; "my teammate built the AI layer, and here's how it works".
10. **SupplySenseAI metrics** come from synthetic labels — say so before they find it.
11. **"Why backend and not ML, given your resume?"** "My internships were backend and that's where I want to grow. ML is a tool I use. At Zomato, problems like dispatch, ETAs and search are backend problems that use ML."

---

## Part 7: Night-before checklist

- [ ] Say the 60-second intro out loud twice.
- [ ] Explain the Myntra design in 2 minutes, including the token bucket and the 50% answer.
- [ ] Explain the Atlassian flow: visibility timeout, DLQ, idempotency.
- [ ] Draw TraceX and SupplySenseAI from memory.
- [ ] Write the Go worker pool and an LRU cache from memory.
- [ ] Answer: HashMap internals, B+ tree index and leftmost prefix, ACID and isolation levels, Kafka consumer groups, Kafka vs SQS, TCP vs UDP.
- [ ] Have one STAR story each for: a hard problem, a bug, a failure, teamwork.
